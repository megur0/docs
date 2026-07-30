---
title: "環境構築 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 環境構築



## オンラインで実行
* playground
  * https://go.dev/play/

## インストール
* インストーラーをDLして実行。
* https://golang.org/doc/install
```sh
% go version
go version go1.19.4 darwin/amd64
```

## アップデート
* 最新のインストーラーをDLして実行するだけでOK
* go version
* https://go.dev/dl/


## サンプル実行
* go version  
  go version go1.19.4 darwin/amd64
* mkdir go-test && cd go-test && go mod init example/go-test
  * 「go-test」の部分はちゃんと運用するならgithub.com/〜 とかのほうが良さそう？
  * go.modが作成される。
* touch hello.go
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```
* go run .
* 公開パッケージは下記から検索する。例えば以下。
  * https://pkg.go.dev/rsc.io/quote
  * ここに書いてある関数のリストがimportして呼び出しができる関数である。
* import "rsc.io/quote" を追加する
* go mod tidy　を実行すると *.goファイルを見てよしなにダウンロードとgo.modに追記してくれる。
  * 最新版を持ってくる？
  * ダウンロードしたものは ~/go/pkg/mod/rsc.io/とか  に入ってた。

## VScode
### インストール
* goで検索すると出てくるので追加。
* VScodeが　goplsやgo-outlineなどが入ってないよとメッセージをだすので「all install」を押す
```sh
Tools environment: GOPATH=/Users/user/go
Installing 7 tools at /Users/user/go/bin in module mode.
  gotests
  gomodifytags
  impl
  goplay
  dlv
  staticcheck
  gopls
  ・・・・
```
* githubからDL、インストールされる。 ls ~/go/bin/　で見ると増えていた。
```sh
% ls ~/go/bin/
dlv             gomodifytags    goplay          gopls           gotests         impl            staticcheck
```
### 便利な操作
* https://future-architect.github.io/articles/20200707/
* https://zenn.dev/optimisuke/articles/f82a5a919429ee54798e
* コード整形・自動インポート
  * コマンド + s
* 構造体補完
  * コマンド + .
* import
  * option + shift + o
* コマンドパレット系のメニューは右クリック> show all commandsで全部見れる。
* コマンドパレット：Go: Restart Language Server
  * コードを実装していて、何かしらうまく動かない (おかしなエラーが出る、補完が効かなくなる、etc) 場合
* コマンドパレット：Go: Generate Unit Tests For Function
  * カーソル下の関数のユニットテストを生成してくれる。
* それ以外は上記URL参照


## クロスコンパイル
### GOARCH, GOOS
* 設定することによって様々な環境に対してクロスコンパイルができる。
### CGO, スタティックリンク, ダイナミックリンク
* https://alpha3166.github.io/blog/20230430.html
* go buildで作った実行ファイルは、デフォルトではスタティックリンクになる。
* C言語で書いた既存のライブラリを呼び出すcgoという機能があって、これを使う場合はダイナミックリンク
* 標準ライブラリ「os/user」か「net」を使ったプログラムをビルドすると、
  * cgoを使ってダイナミックリンクで標準Cライブラリ(libc)の機能を呼び出す実行ファイルができあがる。
  * なお、Goのコンパイラはlibcに依存している。
* ダイナミックリンクがうまく動作しない例
  * Ubuntuなど多くのLinuxディストリビューション -> GNU Cライブラリ(glibc)を使っている
  * Alpine Linux -> 軽量化のためmuslを使っている。
  * Docker HubのAlpine公式イメージ(alpine:latest)にapk add goした環境でgo buildすると、
    * muslのso(たとえば/lib/ld-musl-x86_64.so.1)にダイナミックリンクされた実行ファイル　ができあがる。
  * この実行ファイルを、Ubuntuなど標準ではmuslが入っていない環境に持ち込んで起動すると、
    * muslのsoが見つけられず「No such file or directory」のエラーになる
### CGO_ENABLED
* CGO_ENABLED=0 の環境変数を付けてコンパイルすれば完全にlibc非依存になる。
* 「os/user」も「net」も、デフォルトでは標準Cライブラリを呼ぶ動きをするものの、
  * 実は同じ機能をピュアGo実装としても持っている。
  * ただ、標準Cライブラリを使ったほうが、ピュアGo実装より少し機能が豊富なので、デフォルトでは標準Cライブラリを使うようになっている。
### その他参考
https://qiita.com/Jxck_/items/02185f51162e92759ebe
https://www.getto.systems/entry/2020/06/11/013316#cgo_enabled
https://infraya.work/posts/alpine_with_go-singlebinary/
https://zenn.dev/tsuzu/articles/go-10-year-anniversary
https://www.kimullaa.com/posts/202011182358/




