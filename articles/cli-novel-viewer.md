---
title: "コンソールで小説を読んでみたい"
emoji: "📖"
type: "tech"
topics:
  - "go言語"
published: false
published_at: "2024-12-18 23:14"
---

# はじめに
この記事は思いついたアイデアを供養するために作っているプロジェクトがどんなもんか解説してあるものです。
所々読みにくいかもしれませんが、お許しください。

# とりあえずリポジトリ
https://github.com/skikozou/TerminalRPG/

# 経緯
2ヶ月ほど前に転スラの小説（原作）を読んでめちゃめちゃハマってたときに、二次創作というかifルートみたいなものを書きたいなぁと思ったんですよね。
それで、どうせならノベルゲームのような感じにしてみたいなーーーなんて思って作りました。

# できること
(2024年12月18日現在)
小説家になろう　から小説を取得、保存
自分で作ったプロジェクトを読み込める
ifルートが作れる

 # 使い方
## ファイル読み込み
 
 ```go
 package main

import (
        TerminalRPG "TerminalRPG/src"
        "fmt"
)

func main() {
        project, err := TerminalRPG.LoadData("novel.json")
        if err != nil {
                fmt.Println(err)
                return
        }
}
 ```

 ## 再生
 
```go
 package main

import (
        TerminalRPG "TerminalRPG/src"
        "fmt"
)

func main() {
        project, err := TerminalRPG.LoadData("novel.json") //ファイルをロード
        if err != nil {
                fmt.Println(err)
                return
        }

        TerminalRPG.Runner(project)
}
```
