---
title: "コマンド - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 標準ツール（cmd/go）




# cmd/go ドキュメント
https://pkg.go.dev/cmd/go#section-documentation


# go build
## バイナリ
* docker scanした際に、バイナリのはずなのにgolangの使っているモジュールまで解析できているようだったので不思議だった。
* 調べたところ、goのバイナリには結構な情報があるらしく、使っているモジュール情報も含まれているようだ。
* https://knqyf263.hatenablog.com/entry/2021/02/12/162928
## go buildの際のコンパイル対象
* 前提
    * コンパイルしたファイルの依存先（import先）もすべてコンパイルされ、ひとつのバイナリになる。
    * 依存していないファイルはコンパイルされないので、その分はバイナリサイズが増えない。
        main.go // aaaを参照している。
        aaa/a.go // main.goコンパイル時にコンパイル対象に含まれる。
        bbb/b.go //mainから参照されていない。main.goコンパイル時にコンパイル対象に含まれない。
        go.mod // ここに a.goやb.goが使っている外部モジュールが含まれているとして、main.goコンパイル時にはa.goが使っている外部モジュールのみコンパイルされる。
* go build 
    * 直下の*.goファイルがコンパイルされる。
* go build ./xxx
    * xxxディレクトリ直下の*.goファイルがコンパイルされる
* go build main.go
    * main.goがコンパイルされる。注意点として、同ディレクトリ内の他のgoファイルはコンパイルされないので注意。
* go build main.go server.go
    * main.goとserver.goがコンパイルされる。
## 何もしないmain.goファイルをbuildすると、バイナリサイズは1.2MBくらいだった。
```sh
-rwxr-xr-x  1 user  staff  1177168 Mar 24 08:15 aaa
```
## ビルドタグ
```go
// +build foo
go build -tags foo
```
  * https://qiita.com/ueokande/items/fac0d1219dbbc8f44db7 
## ldflags
* sやwを指定することで、シンボルテーブルやデバッグに関する情報を取り除き、ビルドサイズを小さくできる。
* -ldflags="-s -w"
    * https://qiita.com/ssc-ynakamura/items/da37856f7f217d708a07
```sh
-s
	Omit the symbol table and debug information.
-w
	Omit the DWARF symbol table.
```



# go fmt（gofmt）
* gofmt main.go
  * -l 修正を行ったファイルを表示
  * -w 上書きする（つけないとファイルが更新されない）
* gofmt -l -w  は go fmt と同じ挙動
* go fmt ./...
  * 配下のすべてのファイルをフォーマット
* https://dev.classmethod.jp/articles/go-setup-and-sample/
* https://qiita.com/suin/items/9f9bdaa0cb9cb80cf752


# go generate
* go generate ./... or 対象のディレクトリでgo generate
* https://qiita.com/yaegashi/items/d1fd9f7d0c75b2bb7446
    * ちょっと内容が古いかも。（例えばgo runの外部モジュール実行は最新版だと、go:generate go run golang.org/x/tools/cmd/stringer@latest -type=Status ってバージョン指定をしないと動かない）
* https://qiita.com/smith-30/items/e8a98781638e4a6450d7
    * ちょっと内容古い。go get　〜だと、v1.18からはそもそもコマンドはインストールされない。
    * あと「//go:generate stringer -type=Pill」って書くとローカルのインストール状況に依存するので外部モジュールとして書いたほうが良い


