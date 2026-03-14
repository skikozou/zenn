---
title: "偽コードジェネレーターを作ってみた"
emoji: "🙃"
type: "tech"
topics:
  - "go"
published: true
published_at: "2024-08-23 01:42"
---

# はじめに
いつかtwitterのTLを見ているときにこんな画像が流れてきた。
![fakecodeAd](https://image.itmedia.co.jp/business/articles/2402/17/ak_uh_01.jpg)
ぱっと見、jsか何かのコードに見えるがよく文を読んでみるとローマ字でUrbanhacksについての説明とURLが書いてある。
初めてこれを見たときは「ふーん、こういう方法もあるのか」くらいにしか思わなかったが、最近これを自動で生成できるようになったら面白くね？...と考え、少し作ってみた。

# どんな感じ
### とりあえずコード
https://github.com/skikozou/FakeCodeGen
Goで書いてます
出力する言語の形式もGoにしてます

https://github.com/skikozou/FakeCodeGen/blob/main/main.go
### 読み取り、書き出し先のファイルパスの設定
---
```go
var (
	orignFile    string
	fakeCodeFile string
)
if len(os.Args) < 3 {
	fmt.Printf("orign file path\n>")
	var text string
	fmt.Scan(&text)
	orignFile = text
	fmt.Printf("fake code file path\n>")
	fmt.Scan(&text)
	fakeCodeFile = text
} else {
	orignFile = os.Args[1]
	fakeCodeFile = os.Args[2]
}
```
### 改行とスペースで分割し、配列に格納
---
```go
raw, _ := os.ReadFile(orignFile)
lines := strings.Split(string(raw), "\r\n")
rawcode := make([][]string, 0)
for _, l := range lines {
	if l != "" {
		line := strings.Split(l, " ")
		rawcode = append(rawcode, line)
	}
}
```
### 配列をいい感じに割り当てて構造体に入れる
---
```go
code := srcGen(rawcode)
```
### 構造体を文字列に変換
---
```go
source := code.Build()
```
### 保存したり出力したり
---
```go
fmt.Println("--------------------------------------------------------------------------------\n" + source + "--------------------------------------------------------------------------------")
f, _ := os.Create(fakeCodeFile)
f.Write([]byte(source))
f.Close()
```
# Generator
https://github.com/skikozou/FakeCodeGen/blob/main/genarator.go
基本的に一行の単語数をみて、それに合う形に割り当てるだけ
# Builder
https://github.com/skikozou/FakeCodeGen/blob/main/builder.go
割り当てられた単語をコードに組み込む
いい書き方が思いつかず、ほぼごり押し
rn, tab, spaceなどのよく使う文字の変数が意外と使いやすかった(かも？...)
# おしまい
正直改善点ありすぎてやばいのでいつか修正します()
いつかは形態素解析も入れていい感じに処理できるようにしたいね
汚い部分や、これおかしいだろと思うところがあったらこっそりgithubにコミットしといてください。