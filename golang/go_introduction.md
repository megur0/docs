---
title: "導入 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 導入



# 言語仕様
* https://go.dev/ref/spec
* https://go.dev/talks/2012/splash.article

# ドキュメント
* https://golang.org/doc/

# チュートリアル
* https://golang.org/doc/tutorial/

# go-tour
* https://go-tour-jp.appspot.com/

# effective go
* https://golang.org/doc/effective_go

# playground
* https://go.dev/play/
* shift + enterで実行

# パッケージ
* https://pkg.go.dev/
## パッケージを開発して公開する。
* https://golang.org/doc/modules/developing#design


# よくある質問
* https://go.dev/doc/faq#convert_slice_of_interface


# Goの特徴
* 型（typeで宣言したもの）を レシーバーとしたメソッドを定義することで暗黙的に実装したことになり、
* インターフェースを受け取る方は引数の型をそのインターフェースとして宣言する感じ。（そっちは明示的）
* ポインタを利用する。
    * C言語と同様にポインタが存在する。
        * a->what のような アロー演算子の文法は無い。 aがポインタなら自動的に *aになってるっぽい
    * 構造体は参照型ではないため、参照として渡すにはポインタを使う必要がある。
* null安全ではない
    * null(nil)許容型か否かの区別はない。
    * nilを参照するとランタイムエラー(panic)となる。
* 変数に値を入れない場合はデフォルト値が入る。
    * 例えばint型は0、stringは空文字、ポインタはnil
* 第一級関数、高階関数が使える。関数は全てクロージャある。
    * 高階関数を実現するために、
* 例外がない
    * エラーの場合はerror型を返す。
        * A->B->C　等の呼び出しで AでCのエラーを確認したい場合は、Bでエラーをリレーする必要がある。
* enumに該当する機能がない
* map関数やreduce関数は標準ライブラリに用意されていない
    * ジェネリクスを利用して定義することは可能
* オプショナル引数がない
    * https://raahii.github.io/posts/optional-parameters-in-go/
* クラス(class)がない
* 構造体(struct)がある
    * 構造体には埋め込み(embbed)が可能でメソッドの移譲や上書きが可能
    * サブタイプ化する機能はない。
        * embbedによってinterface型を満たす型とすることは可能
* 型にはメソッドをもたせることができる
    * (名前付きの)型はすべてメソッドセットを持つ。
* interface型がある
    * Goでは明示的なサブタイプ・スーパータイプを表現する機能はないが、代入可能性の判定や型アサーションでinteface型を利用して抽象化・具象化を行うことができる。
    * 型の抽象化はinterface型によって行われる。
        * 型がinterface型と同じメソッドを持つことで、そのinteface型として扱うことができる。
        * interface{ hello() string } はstringを返すhello()メソッドを持つ型であることを表す
        * interface{} は メソッドを持たないため、interface{}型(any型)は任意の値を格納できる
    * 型の具象化は inteface型から行う。
        * interface型には具象の情報が含まれる
        * 型アサーション(type assertion, type switch)によって具体的な値を得ることができる。
    * inteface型の中身を検査するための仕組みとしてreflectionパッケージが標準ライブラリとして提供されている。
*  nilや(*int)nilなどは通常は==が真となるが、interface型に格納すると型情報を持ち、一致しない。
* 三項演算子がない。
    * https://go.dev/doc/faq#Control_flow
* アロー関数がない。
* パッケージ ・名前空間はディレクトリ単位
    * したがって、ファイル名を変えてもimportディレクティブ等の記述への影響はない。
* メモリ安全
    * メモリへの割当や開放は自動化されている。
    * ガベージコレクタがある。
    * C/C++と異なり不正なメモリアクセスは起こらない
* 1.18からgenericsが導入された
* 言語の標準パッケージで提供されるAPIが一般的にフレームワークや外部パッケージで提供されるレベルのAPIも包含されている。
    * HTTPサーバー: net/http
    * データベースクライアント: database/sql
    * テスト: testing
* クロスコンパイラ・シングルバイナリ
    * 実行環境のランタイムが不要
    * スタティックリンクのため、単体のバイナリで動作する。
        * 正確にはlibcには依存するがオプションによって非依存とすることが可能
