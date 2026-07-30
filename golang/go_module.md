---
title: "モジュール・パッケージ - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > モジュール・パッケージ

## エクスポート
> In Go, a function whose name starts with a capital letter can be called by a function not in the same package. 
* Goでは、最初の文字が大文字で始まる名前は、外部のパッケージから参照できるエクスポート（公開）された名前( exported name )
  * 小文字は非公開
  
## モジュール、パッケージ
> Go code is grouped into packages, and packages are grouped into modules
* 名前空間的なもの？
* 1つのファイルに複数のパッケージを設定しない
* 1つのディレクトリに複数のパッケージを配置しない
* package main にある main 関数がプログラムのエントリーポイントとなる（処理の開始点）


## Goでの名前空間
* Go言語での名前空間はパッケージ単位であり、パッケージはディレクトリ単位。（すなわち名前空間はディレクトリ単位）
* なので、同じ命名が同じパッケージ内だとできないので注意。
* typeを定義して、それに対してレシーバーを定義して、そのtypeの変数をレシーバー経由でしかいじれないようにしたい場合は、パッケージを分ける必要がある。同じパッケージ内だとレシーバーを使わなくてもいくらでも変数をいじることができてしまうので。

## パッケージ、mainの仕様
* ディレクトリ名 と パッケージ名は一致させる必要はない。ひとつのディレクトリに複数のパッケージがあるとエラーになる。import はパッケージに対してではなく、ディレクトリに対して。
    * aaa/a.go -> package bbb // OK
    * aaa/b.go -> package ccc // エラー
    * main.go -> import "my-proj/aaa" // import "my-proj/bbb"だとエラー
* 異なるディレクトリで同一のパッケージ名を使ってもOK。mainパッケージもOK。ただし一緒にimportするとredeclaredとしてエラーになる。
    * aaa/a.go -> package aaa
    * bbb/a.go -> package aaa // OK
    * ccc/m.go -> package main // OK
    * main.go -> import "my-proj/aaa"、import "my-proj/bbb" //エラーになる。
* main関数はひとつのgo.modファイルの傘下に複数作成できる。ただし、packageはmainである必要がある。
    * go.mod
    * aaa.go -> package main、main関数
    * bbb/bbb.go -> package main、main関数

## import
* 名前をつけてimport
  * log "github.com/sirupsen/logrus" 


## gopls
* Goの公式の言語サーバー。
* VSCodeのGoの拡張でも規定の言語サーバーになっている。
* プロジェクトのトップに go.mod ファイルがないと上手く動作しない。



## モジュール


### モジュール名はURL
* なぜ?
  * https://github.com/golang/go/issues/30242
    > 独自の中央モジュール パス レジストラを維持する必要がなくなり、衝突の可能性が排除されます。
  * https://stackoverflow.com/questions/70724050/why-we-should-use-a-url-for-the-go-module-name
  * https://groups.google.com/g/golang-nuts/c/hLkhogyFLWI

### VSCodeのimportクリック時の動作
* デフォルトので動作として、importディレクティブをクリックすると、定義元へのジャンプに加えて、https://pkg.go.dev/(自分のモジュール名)/(パッケージ) へジャンプする。
* この使用は設定で変更可能
  * https://github.com/golang/vscode-go/issues/3125#issuecomment-1906415251

### アプリコードのモジュール名どうする -> アプリコードでもURLにしたほうが良さそう。
* アプリコードで非公開であり、モジュールを他から参照しない前提の場合。
* モジュール名をリポジトリのURLにせずとも利用可能ではある。
  * ほぼ問題なく利用できてはいる。
* ただ、例えば、ツールチェインの利用上困ったことがあった。
  * 「go list -m -u -retracted -versions all」では「go: malformed module path "xiii": missing dot in first path element」といったエラーが出た。
    * なお、「 -versions」オプションをつけなければエラーは出ない。
  * つまり、ツールチェインにおいて「モジュール名はURLの形式でしょう」という前提がある程度ありそう。
* したがって、アプリコードでもURLにしておいたほうが良さそうではある。

### importでの自己参照時にエイリアスが使えないか？
* つまり、モジュール名はURL形式にしておき、アプリコード内のimport文ではエイリアスで参照するといった機能。
  * これがあると、import時に長いURLを記載する必要がない。
* まさに下記のような機能だが、Goの設計上不可能なようである。
  * https://www.reddit.com/r/golang/comments/13iul8r/how_can_i_have_an_alias_to_the_current_module_name/
  * https://stackoverflow.com/questions/61405061/module-name-alias-in-go-mod
  * プロポーザルも却下されている
    * https://github.com/golang/go/issues/25518#issuecomment-393345674


### ローカルのモジュールを利用する
* https://go.dev/wiki/Modules#can-i-work-entirely-outside-of-vcs-on-my-local-filesystem




## $GO_PATH、モジュール対応モード

### go env
* go env で確認可能
* go env -w で上書き可能。
* go env GOBIN でインストール場所がわかる。（何も設定されて無ければ ~/go/bin）


### $GO_PATH
* https://zenn.dev/spiegel/articles/20210223-go-module-aware-mode
* https://zenn.dev/tennashi/articles/3b87a8d924bc9c43573e
* デフォルトのGoワークスペースはユーザのホームディレクトリの直下
* GOPATH = $HOME/goとなっている。

  
### モジュール対応モード
* （今はこっちが標準）モジュール対応モード (module-aware mode)
  * 標準ライブラリを除く全てのパッケージをモジュールとして管理する。
  * コード管理とビルドは任意のディレクトリで可能で，モジュールはリポジトリのバージョンタグまたはリビジョン毎に管理される
* （古い。）GOPATH モード (GOPATH mode) 
  * バージョン 1.10 までのモード
  * 標準ライブラリを除く全てのパッケージのコード管理とビルドを環境変数 $GOPATH で指定されたディレクトリ下で行う。
    * 2018年末のGo 1.11が出るまではこれが不満となっていた。
  * パッケージの管理はリポジトリの最新リビジョンのみが対象


### $GOPATH/bin
* getやinstallなどのGo コマンドによりインストールされた実行ファイルを配置するためのディレクトリ
* ちなみに、$GOBINを指定すると別のディレクトリにもできる。例）GOBIN=$HOME/bin


### $GOPATH/pkg/mod
* ダウンロードされたパッケージ(module)が配置。
  * 問い合わせ結果のキャッシュも保持している
    * $GOPATH/pkg/mod/cache/download これかな？
* go get はこのディレクトリを参照するものの、あくまでキャッシュという扱い。
  * あればそれを使う、無ければダウンロード
* Go modules を有効にしている場合(module-aware modeモード)に利用。
  * 無効にすると　$GOPATH/srcが利用される
* $GOMODCACHE　（Go 1.15 から使える）を指定すると別のディレクトにもできる。
  * 例）GOMODCACHE=$HOME/.cache/go_mod


### $GOPATH/pkg/gosum
* Go 1.13 より Go module proxy と Go checksum database という仕組みが導入
  * リポジトリをホスティングしているサービスに依存しているという問題や悪意ある攻撃者により module をすりかえられるかもしれないという問題に対処するために用意
  * module proxy に問い合わせることで module の情報を取得し、checksum database に問い合わせることにより module の完全性を確認。
* $GOPATH/pkg/gosumは、checksum database へ問い合わせた結果のキャッシュが配置されるディレクトリ
  * module proxy へ問い合わせた結果のキャッシュは pkg/mod/cache 配下に保持される
* gosumのパスを変える方法は無い？


### $GOPATH/src
* モジュール対応モードを使う場合は不要。（なので使うことは無い気がする）
* GOPATH mode の場合は import された package の探索先として使われる。


### $GOCACHE
* デフォで ~/.cache/go-build になっている。
* ビルド時のキャッシュがここに入るようだ。
  * $GOPATH/pkg/modの方は ダウンロードしたモジュールや 問い合わせ結果のキャッシュで、こっちはビルド時のキャッシュって感じ？
* $GOCACHE を設定することで変えることができる。

### Go 1.13 から廃止
* $GOPATH/pkg/$GOOS_$GOARCH
  * Go コマンドによりインストールされた Binary-Only package（ package のコンパイルに使用したソースコードを含まずバイナリ形式で package を配布する仕組み） が配置




## module-aware mode


### Go Modules
* https://go.dev/ref/mod
* https://qiita.com/eihigh/items/9fe52804610a8c4b7e41
* Go1.16（2021年2月頃）でかなりModlesの仕様が改善された感じ。

### $GO111MODULE
* Go modulesの有効・無効を制御
* デフォルトでon（Go1.16）

### go.mod
* Goモジュールのパスを書いておくファイル
* 例）require rsc.io/quote v1.5.2
  * 不明点：　v1.5.2ってtagの指定？？
* 解決されるバージョンの範囲
  * require モジュール v1.5.2のように指定すると、v1.5.2以上、v2.0.0未満となる。
  * https://poyo.hatenablog.jp/entry/2021/12/11/123810#require時のバージョンの指定

### (TODO) toolchain行
* https://go.dev/doc/toolchain
* go mod tidy -go=1.21を実行したら下記行も追加されたけどなんだろう？
```
toolchain go1.21.4
```

### go.sum
* go.modファイル内のモジュールの依存関係をハッシュ化したものを保存する
* go.modファイル同様にビルドする際に使用される。
* go.sumファイルがなくてもビルドすることはできる。
* go.modとの整合性を図り、一貫性と信頼性を担保する


### 共通オプション -mod
* https://go.dev/ref/mod
#### -mod=readonly （デフォルト）, -mod=vendor
* goコマンドにvendorディレクトリを無視し、go.modを更新する必要がある場合にエラーを報告するように指示。
* Goではプロジェクト直下にvendorフォルダがある場合は、そっちを優先的に読み込む。
  * go1.14以降では、vendor ディレクトリがある場合は -mod=vendor がデフォルトとなる。ない場合は -mod=readonly がデフォルト。
* https://qiita.com/haligon/items/3875dee8a4b1480199af
#### vendorディレクトリ
* vendorは go getでダウンロードするモジュールをGOPATH/pkg/modではなく、プロジェクトのルートディレクトリに置くことができる仕組み
* ./vendor ディレクトリは go mod vendor コマンドで生成することができる。
* 昔は同じモジュールに対して複数バージョンを使うには、venderディレクトリを使う必要だったようだ。（かなり不便だった）
* ただ、今は特別な用途（たとえば外部モジュール自体にログを仕込みたい、とか。）がなければ使う必要はなさそう。
    * あと、自分の場合たまにモジュールの中身を見ているときに誤って中身にタイピングしている時があって、そのあたりリカバリーできるようにvendorを使うって手はあるかも。  
* https://selfnote.work/20220218/programming/golang-how-to-debug/
* https://qiita.com/lamp7800/items/5454353ad0f001f10c37  
#### -mod=mod
* コマンド実行時に、go.modを更新したいって場合は、-mod=modというフラグを指定する。
* ※ Go 1.16から、go buildや go test、go runによって自動的にgo.mod の更新およびダウンロードはされることがなくなった。
  * go.modに存在しないものがある場合はこれらのコマンドでエラーになる。
* https://itchyny.hatenablog.com/entry/2022/04/22/120000



### ひとつのgo.modファイル下で複数のmain
* 複数のコマンドラインツールなど作成する際は、公式でのベストプラクティスがあるわけでないが、普通に複数のディレクトリに分けて複数のmainを作るのが一般的。
* https://qiita.com/y_shinoda/items/e4a4ffb25d11ccf74221
* バイナリサイズもそれぞれが依存するファイルのみ含まれる。


### go mod init 〜
* go.modファイルが生成される


### go mod graph
* 依存関係の確認
go mod graph | grep xii
    これはgo.modに書いてあるものが全部出る感じ。（direct, indirectの両方）
go mod graph | grep echo 


### go mod tidy
* *.goファイルをみてよしなに必要なパッケージをよしなにダウンロードしてgo.modに追記
  * GOPATH/go/mod/〜　にダウンロードしたものが入っている。
* go.sumが作成される。（チェックサムが書いてある）

### go mod download 
* 引数にモジュールの指定がない場合は、go.modファイルに記載されたすべてのモジュールをダウンロード。
* tidyと違う点は、importがあるなしに関わらずgo.modに記載されていればすべてダウンロード。

### go mod edit -replace
* https://golang.org/doc/tutorial/call-module-code
* 呼び出し側（hello）で`go mod edit -replace example.com/greetings=../greetings`
  * go.modでreplaceディレクティブで  ../greetingｓに向けられる。
* 本当はgithub.comなどに公開されたものを使うんだけど、チュートリアルなので、ローカルのものを参照させる。



### go run
* `go run .`　や　`go run hello.go`
* `go help run` 参照
* 指定された名前の go プログラムから main パッケージをコンパイルして実行。（バイナリは作成されない）
* 既定でカレントディレクトリ配下のファイルを全て読み込むわけではない。 そのため main package が複数ファイル存在するときなどに、main関数のファイルだけを指定してもコンパイルエラーとなる。 その場合、明示的に複数ファイルを指定するか、* で複数ファイルを指定したり(go run *.go) . (カレントディレクトリ) を指定する。（go run .）
* 外部モジュールの実行
  * `go run github.com/aaa/bbb/v3@latest --version`
  * @バージョンを付けているとgo.modなどの依存関係が無視されるため依存関係に影響を与えずに実行できて便利。
  * つけないで実行した場合は、mainのコンテキスト内で実行される。つまりはgo.modに対象のモジュールが存在しない場合はエラーになる。（なお、-mod=modをつけると
  * -mod=mod 


### go build
* `go help build`
* https://golang.org/doc/tutorial/compile-install
* go runはバイナリを作らず実行してくれるショートカット的コマンドだが、go buildはバイナリ作成。
* 例）go buildして ./hello
* go helpには書いてなかったけど、依存モジュールが$GOPATH/pkg/modに存在しない場合、ダウンロードされていた。
* go buildが何をしているか分析: `go build -work -a -p 1 -x main.go`
  * https://qiita.com/Akatsuki_py/items/8041fba499d54d59e0dd
* -a
  * Golangは、以前にビルドされたパッケージをキャッシュします。 -aはgo buildにキャッシュを無視させるので、ビルドはすべてのステップを出力します


### go install
* ツールのグローバルインストール。
* go.modには影響を与えない。
* go build で実行ファイルを作成して、go installで binにインストールしてくれるのでコマンドツールなどを作成するのは便利そう。
* go install
  * $GOPATH/bin にバイナリがコピーされる。
  * インストール先は `go list -f '{{.Target}}'`　で確認できる
  * 作業ディレクトリをビルドして $GOPATH/bin に配置する機能
  * 実行したディレクトリが、aaa　だと  aaaってファイルがインストールされるはず。
  * 例）
    ```
    aaa
      └ main.go
    ```
  * Makefileでターゲットなどを設定することができる？
* go install <module>@<version>
  * この書式は、Go 1.16で新たに導入されたもの。
  * 例）go install golang.org/x/tools/gopls@latest
  * 指定したモジュール、versionを $GOPATH/bin にインストール
  * 基本的にversionをつけるほうが無難
    * https://qiita.com/eihigh/items/9fe52804610a8c4b7e41


### go get
* `go help get`参照
* go.mod編集とモジュールのダウンロードを行う。
* バージョンを指定しない場合は最新バージョンをダウンロードする
  * go.modも最新バージョンに更新される
* -u
  * マイナー・パッチリリースが利用可能な場合にそれらを利用するようにする。
* https://go.dev/doc/go-get-install-deprecation
* https://zenn.dev/spiegel/articles/20210223-go-module-aware-mode
* v1.18からは、モジュールのビルドは行わなくなった。
  * インストール機能はgo installと重複するため。
  * v1.16からgo installで モジュールを指定できるようになったため、go getのインストール機能と重複するため。
#### 依存するモジュールのアップデートをする
* 全部更新
  * `go get -u ./...` 
    * すべての依存モジュールのマイナーバージョン・パッチバージョンを最新にする。
    * Go言語はメジャーバージョン含めてアップデートするコマンドは無い。
    * メジャーバージョンをアップデートする場合はimportパス自体が変わるルールのため、別ライブラリ扱いになる
      * "github.com/user/lib/v2"
      * "github.com/user/lib"
* 個別に更新
  * `go get モジュール名`
  * `go get モジュール名@バージョン`
  * `go get -u モジュール名`
  * 個別に更新すると、他のパッケージで依存関係でエラーに鳴ることが有る。その際は`go mod tidy`で解消する。
* 参考
  * https://daisuzu.hatenablog.com/entry/2021/11/15/142702


### go list
* 例
```sh
% pwd
プロジェクトフォルダ/model
% go list       
github.com/〜/model
% go list -m
github.com/〜
```
* -m パッケージではなくモジュールを表示
    * 
#### モジュールの最新バージョンを確認する
* https://zenn.dev/yoshii0110/articles/22456ac6761167
* `go list -m -u all`
    * -u 
        * 利用可能なアップグレードバージョンを表示
        * 現在のバージョンが撤回されている場合、モジュールの Retracted フィールドも設定
    * all
        * メインモジュール(go.mod内のmoduleで指定されるモジュール)以外に、依存するすべてのモジュールも表示
        * 最初にメイン モジュール、次にモジュール パスでソートされた依存関係を指定
* その他のオプション
    * `go help list`


### go clean 
* ビルドキャッシュや、テストのキャッシュ、ダウンロードしたモジュールの削除
> The -modcache flag causes clean to remove the entire module download cache, including unpacked source code of versioned dependencies
* https://kazuhira-r.hatenablog.com/entry/2021/03/14/003314



## マルチモジュール

### マルチモジュールについて
* マルチモジュール
    * 複数のモジュール（＝複数のgo.modファイル）
    * ポイントとして、「複数のリポジトリ」 「単一のリポジトリ（マルチモジュールのリポジトリ）」両方のケースを包含していること。（自分の解釈ではそう考えている。ネットの記事だと一部、マルチモジュール　＝　マルチモジュールのリポジトリ的な解釈で書かれているものもある。）
    * 厳密に考えると、外部モジュールを使っている時点で「マルチモジュール」だと個人的に思ってしまうけど、用語としては「複数のgo.modを自分達で開発する」的なニュアンスとして使われている。
* マルチモジュールのリポジトリ
    * 複数のモジュール（＝複数のgo.modファイル）が含まれるリポジトリ
* モノリシックモジュールのリポジトリ
    * 単一のモジュール（単一のgo.modファイル）が含まれるリポジトリ
* なお、golang固有の話ではなく一般的な「モノレポ」は「複数のプロジェクトを同じリポジトリで管理する」といったことを指す。

### goplsがプロジェクトのトップに go.mod ファイルがないと上手く動作しない。
* 以下のようにすることで一応、回避可能。トップではない複数のgo.modファイルに対応可能。（しかしこのワークアラウンドは実験的なもので将来的には削除される予定みたい。挙動としても本来のGoコマンドのビルドとは違う挙動のモジュールのバージョン選択がされる。https://poyo.hatenablog.jp/entry/2022/12/05/090000）
```
"gopls": {
        "build.experimentalWorkspaceModule": true
}
```
* https://text.baldanders.info/remark/2021/02/golang-with-vscode/


### 参照モジュールの置き換え
* マルチモジュールで開発する際に、参照先のモジュールを置き換えたいケースがある。
    * まだpushしていないけど、ローカルのモジュールで試したい。
    * あと、マルチモジュールじゃなくても、外部のモジュールをモック化したいときに使えそう。
* replace
    * go.modで下記のようにローカルのモジュールに置換できる。
    ```
    replace (
        github.com/ikawaha/kagome-dict/ipa => ../kagome-dict/ipa
    )
    ```
    * go runで実行すると、ちゃんと置き換えた方のモジュールが実行されている。
    * go buildについてもビルドした実行ファイルが置き換えた方のモジュールを使う。
    * ただ、欠点としてgo modを書き換えするため、リポジトリへpushの際に戻すのを忘れてしまったりする恐れがある。
* Workspace mode  
    * replaceの欠点を解消したもの。
    * Workspace mode 導入の経緯は、この記事がわかりやすい。
        * https://poyo.hatenablog.jp/entry/2022/12/05/090000
    * go.workによって依存関係を書き換えることでgo.modを変更せずに済む。
    * モノリシックモジュールなリポジトリであるかどうかに関わらず、go.work は一時的な外部依存の上書きに使用する際に使う。
    * go.workはgit管理対象外にするべきである。
    * プロジェクトのルートにgo.modが無くてgoplsを動作させたい場合は、エディタの機能を使う。（VSCodeならMulti-root Workspace）
* その他参考
    * https://zenn.dev/hohner/articles/fd9d682871a12b
    * https://zenn.dev/ikawaha/articles/20220701-a053ec54b77435
    * https://re-engines.com/2022/05/09/golang-workspaces/
    * https://future-architect.github.io/articles/20220216a/
    * https://goodbyegangster.hatenablog.com/entry/2022/10/11/081836
    * https://zenn.dev/kimuson13/articles/go-workspace-mode-impressions
    * https://zenn.dev/ikawaha/articles/20220701-a053ec54b77435


### （IME）マルチモジュールのリポジトリについて
* https://poyo.hatenablog.jp/entry/2022/12/05/090000
* 上記の記事も踏まえて個人的な所感
    * マルチモジュールのリポジトリは出来れば採用しない方が良さそう
    * マルチモジュールのリポジトリを使わざるを得ない場合、
        * VSCodeのMulti-root Workspaceを使う。
            * これによって各サブディレクトリをそれぞれルートとして開くことが出来る。
        * それぞれのサブディレクトリをそれぞれ別のウィンドウで開く。
        * いずれにしても、サブディレクトリの親にmakefile等がある構成だとそっちも開かないといけないのでちょっと面倒。
* そもそもモノリシックモジュールなリポジトリしか使って無くても、複数のリポジトリをひとつのVSCodeの画面上で作業する際は同様にgoplsの問題が起きる。（go.modがルートではないところにあるので。） なのでその場合もVSCodeのWorkspaceを使う。


### (IME) リポジトリを分けたけど、戻した話
* オンライン処理、バッチ処理はdockerのイメージが別で、オンラインのバイナリサイズをなるべく小さくするためにバイナリを分けたいと考えていた。
* そのため、マルチモジュール にするか複数のリポジトリに分けるか検討して、マルチモジュールだとgopls関連で扱いが面倒そうだったため、リポジトリを分けることを選択。
* ただ、グローバル変数とか関数は密結合とした。つまり責務の分解が目的でリポジトリを分けたのではなく、バイナリサイズを小さくすることが目的ということに成る。
* ところが、いざ検証してみると、ひとつのgo.modファイルでオンライン処理やバッチ処理で別々のmainが書けることがわかった。また、それぞれコンパイルしてみる、importで依存関係にあるパッケージしかバイナリサイズには追加されないことも分かった。（Go言語すばらしい!）
* 上記にあたって結果的に、マルチモジュールについて、外部のモジュールをモックする手段、プライベートリポジトリからダウンロードする手段（ローカル、docker）、あとはモジュールがどこにどのように保存されているか、 importの指定やmainとかディレクトリの仕様、ビルドの仕様等を学ぶことができた。




## その他


### 外部パッケージ
* 外部パッケージのリポジトリ名にはハイフンを使って良いのか？
  * https://chatgpt.com/share/67dce908-db90-8000-ab11-24289f4c92b6
  * 結論
    * 使っても良いと思う。
    * 実際、github.com/google/go-cmpとかは使っている。
    * ただし、パッケージ名はハイフンを使えないため、リポジトリ直下には/cmpのディレクトリを作成してパッケージを内部に作成しておく。
      * importする側は`import "github.com/google/go-cmp/cmp"`のようになる。
#### 不明点
* 例えば下記のようにコードをコピペしたとき、コピペ先にはserverやemptycheckはimportされていないため、エラーとなる。
```go
func f(w http.ResponseWriter, r *http.Request) {
	server.Bind(r, &struct{}{})
	emptycheck.EmptyCheck(&struct{}{})
}
```
* このときVSCode上で「"source.fixAll"」や「source.organizeImports」によってコードを自動修正しても、serverの方は成功するのだが、emptycheckの方は下記のように名前付きimportをしてしまい、結果的にエラーとなってしまう。（両者にどのような違いがあるのか？）
```go
emptycheck "github.com/megur0/empty-check" // エラーになる
"github.com/megur0/simple-server/server" // こっちは成功する
```
* 補足として、serverもemptycheckも下記のように手作業で書いてimportができる。
```go
"github.com/megur0/empty-check/emptycheck"
"github.com/megur0/simple-server/server"
```
* chatgptにも聞いたが、分からなかった。
  * https://chatgpt.com/share/67dcef85-4e98-8003-bae8-6fb98bf048e3


### internalパッケージ
* internalというディレクトリに入れることで外部からそのファイル使うことを許容しない。
* https://zenn.dev/ikawaha/articles/20220701-a053ec54b77435


### (IME) プライベートリポジトリを外部モジュールとして利用したい。
* proxy、チェックサムの問題
    * 下記のgoコマンドのデフォルト動作は、公開されているソースコードでうまく機能する。
        * デフォルトでgoproxy.ioのパブリックGoモジュールミラーからモジュールをダウンロード
        * デフォルトでは、ソースに関係なく、sum.golang.orgにあるパブリックGoチェックサムデータベースに対してダウンロードされたモジュールを検証する。
    * 環境変数GOPRIVATEに値を設定しておくことで、プロキシやチェックサムデータベースを使用しないように制御する。
        * https://note.com/artefactnote/n/n2d43681517d9
        * これが設定されると以下も自動的に設定される？（go envで確認したらいつの間にか設定されていた）
            GONOPROXY="github.com/megur0"
            GONOSUMDB="github.com/megur0"
* プライベートリポジトリへのアクセス
    * goコマンドでリポジトリにアクセスする際、httpでアクセスが行われる。この際にプライベートリポジトリにアクセスするときは認証する必要があるはず。
    * ただ、上記のGOPRIVATEを設定後VSCode上で go get〜でプライベートのモジュールを取得したらブラウザで認証画面が開いて認証処理が行われて（たしかVSCodeへの連携だったはず。）、それ以降は特に認証必要なくプライベートリポジトリからgo get やtidyができるようになった。
    * そして、VSCodeじゃなくてterminalから実施しても動作する。この認証情報（トークン）はどこで付与されている？VSCodeが中継してるのであればterminalで実行したときはうまくいかないはず？そして~/go/pkg/mod/cache内を削除したり、go clean -modcacheしても上手くいく。なぜなのか？
    * ネットだとGOPRIVATE以外にGithubのPersonalアクセストークンやらGOPROXYやらsshの設定の情報も色々出てくるのだが、結果的にGOPRIVATEの設定だけでうまく動作したのが何故なのか分からない
        * この記事は自分と一緒
            * https://note.com/artefactnote/n/n2d43681517d9
        * 他の記事はGOPRIVATE以外の設定も必要な感じで書いている。
            * https://gist.github.com/StevenACoffman/866b06ed943394fbacb60a45db5982f2
            * https://zenn.dev/shootacean/articles/go-get-from-github-private-repository
            * https://pet2cattle.com/2022/09/go-get-private-repository
            * https://kawaken.dev/posts/20220426_goprivate/
#### Dockerfile内でプライベートリポジトリにアクセスする。
* Dockerfile内だと、GOPRIVATEをつけても駄目だった。
* gitのinstead ofによってgithubのurlをPAT（personal access token）を含むものに置換する方法を使った。
    * ただ、DockerfileへARGとして入れると、イメージのhistoryに残ってしまうため、ARGは使わない方法にした。
        * https://note.com/artefactnote/n/ncd33d920b0f1
    * https://zenn.dev/momota/articles/f953352940c22a
    * https://zenn.dev/jerome/scraps/473b4548467b16
    * instead of について
        * https://kakakakakku.hatenablog.com/entry/2020/03/27/093150
* また、なぜかDockerfileの場合だとGOPRIVATEを指定しなくても問題なかった（？）
