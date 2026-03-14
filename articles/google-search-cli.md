---
title: "簡易的なGoogle検索ができるモノを作った"
emoji: "🔍"
type: "tech"
topics:
  - "go"
  - "golang"
  - "google"
published: true
published_at: "2025-01-26 23:34"
---

# リポジトリ
https://github.com/skikozou/ggrks
# 入れ方
1. Google Search JSON APIを用意
2. Custom Search Engineを用意
3. それぞれからAPI keyとIDをゲット
4. ggrksをインストール
5. 起動 (すると自動で設定画面になる)
6. Fun!
# インストールコマンド
```
go install "github.com/skikozou/ggrks@latest"
```
# 使い方
- Helpを見る
```
ggrks
```
- 検索
```
ggrks {words}
ggrks golang
ggrks golang install
```
- 検索した後に開く
```
ggrks -o {number} {words}
ggrks -o golang //この場合は1番目のページが開く
ggrks -o 2 golang
```
# configの設定方法
基本のファイル場所は

| OS                     | ホームディレクトリの場所              |
|------------------------|--------------------------------------|
| **Windows**            | `C:\Users\<ユーザー名>`             |
| **macOS**              | `/Users/<ユーザー名>`               |
| **Linux**              | `/home/<ユーザー名>`                |

ここに`config.ini`というファイルがあるので、それを編集すると設定できる
## configの構造

| 項目名 | 説明 | 型 |
|---------------------|------------------------|--------|
| **GoogleSearchAPI** | Google Search API のキー|`string`|
| **SearchEngineID**  | カスタム検索エンジンの ID|`string`|
| **MaxSearchResult** | 検索結果の最大数 (1~10)|`int`|

# 感想
こういう系のモノを作るのは二回目ですが、まあまあ良くできたのではないかと思っています。
僕自身、頻繫に物事が分からなくなって検索するので個人的に作りたいものでもありました。
名前についてはシンプルに「自分で調べよう」という意味です。
自分を過信せず、情報を集めて考えられるようにこの名前にしました。

# 最後に
何かおかしいところや、バグなどがありましたらプルリクを出して頂ければ随時確認します。