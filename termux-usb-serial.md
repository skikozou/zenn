# Termux環境でUSBデバイスをシリアルポートとして使用する

## 概要

Android（非root）環境のTermuxからUSBシリアルデバイスを操作する手順を説明する。

Termuxでは `/dev/tty*` デバイスへ直接アクセスできないため、`termux-usb`
コマンドで Android の USB permission を取得し、その file descriptor を `libusb`
に渡して使用する。

本手順では、USBシリアル通信を **PTY（pseudo terminal）**
にブリッジすることで、アプリケーション側から `/dev/pts/*`
を通常のシリアルポートとして扱えるようにする。

本手順は **iRobot Roomba 980**（VID `27a6:0002`、USB CDC ACM
デバイス）で検証している。

---

## 前提条件

- Android端末（非root）
- Termux および Termux:API（F-Droid版を推奨）
- USB OTGケーブル（またはアダプタ）
- ptyserial (https://github.com/MarkWllms/Termux-serial-tty)

## ptyserialを使用した理由

Termuxでは Android の権限制約により、USBシリアルデバイスを `/dev/tty*`
として直接扱うことができない。そのため、`termux-usb` を用いて USB デバイスの
file descriptor を取得し、`libusb` 経由で通信する必要がある。

既存ツールの中で、USBシリアル通信を **PTY（pseudo terminal）**
にブリッジし、アプリケーションから通常のシリアルポートとして扱えるようにするツールとして
`ptyserial` が存在する。この方法を利用すると、アプリケーション側の実装を変更せず
`/dev/pts/*` をシリアルポートとして利用できる。

そのため、まず `ptyserial` を用いて接続を試みたが、`ptyserial` を含む
Termux-serial-tty は FTDI / CH34x / PL2303 ドライバのみをサポートしており、CDC
ACM デバイスには対応していなかった。

本手順では、対象デバイス（iRobot Roomba 980）が CDC ACM
デバイスであるため、`ptyserial` に CDC ACM
ドライバを追加する形で拡張し、動作するようにしている。

---

## 1. パッケージのインストール

```bash
pkg install clang libusb pkg-config make git
```

---

## 2. ptyserialのビルド

USBの file descriptor を PTY（疑似端末）へブリッジするツールをビルドする。

```bash
mkdir -p ~/git && cd ~/git
git clone https://github.com/MarkWllms/Termux-serial-tty
cd Termux-serial-tty
```

### Makefileに `cdc_acm.o` を追加

`Makefile` の `OBJS` 定義内で `pl2303.o` の直後に次の行を追加する。

```makefile
cdc_acm.o                                                                   \
```

`ptyserial` ターゲットのリンク時に `-L./bin` を追加する。

```makefile
ptyserial: examples/ptyserial.o
	$(LD) $(LDFLAGS) -o $@ $^ -L./bin -lusbuart -lusb-1.0 $(shell pkg-config --libs libusb-1.0)
```

---

### CDCドライバの追加

`MarkWllms/Termux-serial-tty` は **FTDI / CH34x / PL2303**
のみをサポートしており、 CDC ACM デバイスには対応していない。

`src/cdc_acm.cpp` を追加することで CDC ACM デバイスに対応できる。

→ 後述の `src/cdc_acm.cpp` を参照。

---

### ptyserial.cppの修正

PTY作成後に slave 側を明示的に `open` しない場合、Bulk IN コールバック時に `EIO`
が発生する。

`examples/ptyserial.cpp` の

```cpp
fprintf(stderr, "PTY created: %s\n", ptyname);
```

の直後に次を追加する。

```cpp
// slaveを自分で開いてPTYを安定させる
int slave_fd = open(ptyname, O_RDWR | O_NOCTTY);
if (slave_fd < 0) {
    perror("open slave pty");
    kill(pid, SIGKILL);
    return EXIT_FAILURE;
}
```

`ctx.attach(...)` の `close(slave_fd)` は、ループ終了後（`kill(pid, SIGTERM)`
の直前）に移動する。

---

### ビルド

```bash
make
make ptyserial
```

`make ptyserial` が `-lusbuart` を見つけられない場合は手動でリンクする。

```bash
clang++ -c examples/ptyserial.cpp -o examples/ptyserial.o \
  -Iinclude $(pkg-config --cflags libusb-1.0) -Wall -fPIC -O0 -std=c++1y

clang++ -o ptyserial examples/ptyserial.o \
  -L./bin -lusbuart -lusb-1.0 $(pkg-config --libs libusb-1.0) -Wl,-rpath,./bin
```

---

## 3. デバイスのパーミッション取得

USBデバイスの一覧を確認する。

```bash
termux-usb -l
```

例

```
["/dev/bus/usb/001/003"]
```

対象デバイスに対して permission を要求する。

```bash
termux-usb -r /dev/bus/usb/001/003
```

Androidのダイアログで **許可** を選択する。

---

## 4. エンドポイントの確認（必要な場合）

デバイスの Bulk エンドポイントを調べるツール。

```c
// list_ep.c
#include <stdio.h>
#include <stdlib.h>
#include <libusb.h>

int main(int argc, char** argv) {
    if (argc < 2) return 1;
    int fd = atoi(argv[argc-1]);

    libusb_set_option(NULL, LIBUSB_OPTION_WEAK_AUTHORITY);
    libusb_set_option(NULL, LIBUSB_OPTION_NO_DEVICE_DISCOVERY);

    libusb_context *ctx = NULL;
    libusb_init(&ctx);

    libusb_device_handle *handle = NULL;
    if (libusb_wrap_sys_device(ctx, (intptr_t)fd, &handle) < 0) return 1;

    libusb_device *dev = libusb_get_device(handle);
    struct libusb_config_descriptor *cfg = NULL;
    if (libusb_get_config_descriptor(dev, 0, &cfg) < 0) return 1;

    for (int i = 0; i < cfg->bNumInterfaces; i++) {
        const struct libusb_interface *ifc = &cfg->interface[i];
        for (int j = 0; j < ifc->num_altsetting; j++) {
            const struct libusb_interface_descriptor *id = &ifc->altsetting[j];
            printf("ifc=%d class=0x%02x ep_count=%d\n", i, id->bInterfaceClass, id->bNumEndpoints);
            for (int k = 0; k < id->bNumEndpoints; k++) {
                printf("  ep=0x%02x attr=0x%02x\n",
                    id->endpoint[k].bEndpointAddress,
                    id->endpoint[k].bmAttributes);
            }
        }
    }
    libusb_free_config_descriptor(cfg);
    return 0;
}
```

コンパイル

```bash
clang list_ep.c -o list_ep $(pkg-config --cflags --libs libusb-1.0)
```

実行

```bash
termux-usb -e "./list_ep" /dev/bus/usb/001/003
```

Roomba 980 の場合の例。

```
ifc=0 class=0x02 ep_count=1
  ep=0x82 attr=0x03   # Interrupt IN（CDC Control）

ifc=1 class=0x0a ep_count=2
  ep=0x03 attr=0x02   # Bulk OUT（CDC Data）
  ep=0x81 attr=0x02   # Bulk IN（CDC Data）
```

---

## 5. ptyserialの起動

```bash
TTYDIR=~/git/Termux-serial-tty

termux-usb -e "env LD_LIBRARY_PATH=$TTYDIR/bin $TTYDIR/ptyserial 115200" \
  /dev/bus/usb/001/003
```

実行に成功すると次のような出力が表示される。

```
PTY created: /dev/pts/4
```

この PTY パスを通常のシリアルポートとして使用できる。

---

## 6. 起動スクリプト

上記手順を簡略化するため、以下のスクリプトで自動化できる。

```bash
#!/bin/bash
TTYDIR="/data/data/com.termux/files/home/git/Termux-serial-tty"
USB_DEV=$(termux-usb -l | tr -d '[]" \n' | tr ',' '\n' | head -1)

if [ -z "$USB_DEV" ]; then
    echo "USB device not found"
    exit 1
fi

echo "USB device: $USB_DEV"

termux-usb -e "env LD_LIBRARY_PATH=$TTYDIR/bin $TTYDIR/ptyserial 115200" "$USB_DEV" \
    2>/tmp/ptyserial.log &
PTYSERIAL_PID=$!

PTY=""
for i in $(seq 1 10); do
    PTY=$(grep "PTY created" /tmp/ptyserial.log | awk '{print $3}')
    [ -n "$PTY" ] && break
    sleep 0.5
done

if [ -z "$PTY" ]; then
    echo "PTY not found"
    kill $PTYSERIAL_PID
    exit 1
fi

echo "PTY: $PTY"

ROOMBA_PTY="$PTY" ~/g/roomba/roomba

kill $PTYSERIAL_PID
```

---

## トラブルシューティング

| 症状                    | 原因                                      | 対処                                               |
| ----------------------- | ----------------------------------------- | -------------------------------------------------- |
| `termux-usb -l` が空    | デバイス未認識 / Termux:API未インストール | F-Droid版Termux:APIをインストール                  |
| `error 6 (BUSY)`        | カーネルドライバがインターフェースを占有  | CDCドライバで `libusb_detach_kernel_driver` を呼ぶ |
| `I/O error 5 (EIO)`     | PTYのslave側が開かれていない              | `ptyserial.cpp` でslave_fdを開いたままにする       |
| `Status=6` がループする | slave_fdを早期にcloseしている             | ループ終了後にcloseする                            |
| `-lusbuart not found`   | `-L./bin` が不足                          | リンクコマンドに `-L./bin` を追加                  |
