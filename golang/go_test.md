---
title: "テスト - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > テスト



## 公式
* https://pkg.go.dev/testing

## Go言語のtesting.Tとtesting.Mとtesting.Bとtesting.Fの違いと使い方
https://sagantaf.hatenablog.com/entry/2023/07/21/235340
* https://pkg.go.dev/testing#pkg-overview
* testing.T
	* ユニットテスト
* testing.M
	* テスト前にセットアップやクリーンアップ
* testing.B
	* ベンチマーク
* testing.PB
	* 並列でベンチマーク
* testing.TB
	* TとBの両方が使える？？
* testing.F
	* Fuzzing（ランダムなデータ）を使ったテスト


## go test
* https://golang.org/doc/tutorial/add-a-test
* xxxx_test.goというファイル名にする。
* testingパッケージをインポート
* 関数をTest****にして、引数として*testing.Tを取る。
* Fatal、Error、Cleanup、Log、Run、Parallelとか。
* TestMainを書くことで共通の前後の処理を入れることもできる。
    * TestMainの中にRunコマンドを入れる必要がある。（Runによって各テスト関数が実行される）
* go test
    * -vをつけるとログの標準出力が出る。Logなどで出力をしても-vをつけないと標準出力は出ないので注意。
* https://future-architect.github.io/articles/20200601/
* https://www.twihike.dev/docs/golang-primer/testing


## 実行
* 「TestUserRepository」という名前の関数のみ実行（正規表現）
* -count=1でキャッシュを消している
```sh
docker compose exec api go test -v -count=1 -timeout 30s -run ^TestUserRepository$ ./app/repository
```
### すべてのパッケージで実行する方法はある？
* `go test ./...`



## 並列化
* go testの並列化は２つある。
* https://engineering.mercari.com/blog/entry/how_to_use_t_parallel/
### パッケージ単位での並列化(デフォルトで有効)
* 各パッケージのテストが並行で実行される。
	* 単体のパッケージのテストであれば並行実行はされない。
* -p 1を指定すると直列で実行を強制できる。
* 並列処理は、別々のプロセスで実行されるため、グローバル変数などはおそらく競合しないと思われる。（未検証）
	* ここの回答の１つによるとps auxで別のプロセスになっていることから、競合することはなさそう。(ただ、上記のように同じDBを使っていればDBで競合する。)
	* https://stackoverflow.com/questions/24375966/does-go-test-run-unit-tests-concurrently
* https://zenn.dev/media_engine/articles/testing-go-applications#db%E3%81%AE%E6%8E%83%E9%99%A4%E6%96%B9%E6%B3%95
### 個々のパッケージ・ファイル内の並列化
* WIP
* この機能に関してはデフォルトではオフになっている。
	* https://stackoverflow.com/questions/24375966/does-go-test-run-unit-tests-concurrently
* コードへt.Parallel()を指定することで並列化する(詳細は未確認)
* -parallel n で並列数を指定
	* デフォルトでは、GOMAXPROCSの値に設定



## ヘルパーメソッドでエラー行を出力しないようにする
* ヘルパーメソッドでtesting.T.Errorなどを呼ぶと、そのヘルパーメソッドの呼び出し箇所が出力される行数となってしまい特定に時間がかかってしまう。
* t.Helper()をヘルパーメソッド内で呼び出しておくことで、これを回避できる。
* https://pkg.go.dev/testing#T.Helper

## TestMain
* TestMainで書いた処理は、m.Run()の前の処理 -> m.Run()で各テスト関数を上から順に実行 -> m.Run()の後の処理 という順で実行される。
  * この関数は各Test***メソッドごとに実行されるのではなく、１度しか実行されない。
* なお、個別のテストを実行したとき（-runによって個別に指定したとき）もTestMainの処理は動作する。
* パッケージ内にひとつしか書くことができない。（他の関数と同様に再定義扱いになってしまう）
* os.Exit
	* 1.15から、TestMain実行後に外部のmain関数がos.Exitを実行するようになった。
		* os.Exitにはtesting.M.exitCodeが渡される。（これはm.Run実行時の結果がセットされる）
	* https://go.dev/doc/go1.15#testing
	* https://qiita.com/hgsgtk/items/40e63150affed01f6573


## Goにはsetupやteardonwに該当する機能がない。
* https://github.com/golang/go/issues/27927
* つまり、各Test***関数ごとに前後にセットアップとクリーンアップを実行する機能は提供されていない。



## テストカバレッジ
* https://qiita.com/takehanKosuke/items/4342ca544d205fb36eb0

## net/http/httptest
* NewRequest、NewRecorder
* NewServerでスタブサーバーも立てられる。（あんま使わない？）
* https://zenn.dev/media_engine/articles/testing-go-applications

## テスト時のDBのロールバック
 * ただ、このやり方だと外部APIで呼び出して更新するテストのロールバックは出来ないので、e2eテストではgo-migrationのdownとupを組合わせた。
* https://zenn.dev/media_engine/articles/testing-go-applications#db%E3%81%AE%E6%8E%83%E9%99%A4%E6%96%B9%E6%B3%95

## 日時をスタブする方法
* 主に以下の方法
	* 関数やインターフェースを引数として渡す
	* グローバルな関数を定義して、テストの際は置換する。(ただしスレッドセーフではない。テスト完了時に元に戻す必要がある)
* https://labs.yulrizka.com/en/stubbing-time-dot-now-in-golang/
* あるいはスタブ化すること自体を諦めて、テスト時に工夫して対応する方法の方が良いかもしれない。
	* jsonの比較時等に、日時を比較対象から外す
	* jsonの比較時等に、日時は個別に比較する

## カバレッジの除外方法
* https://stackoverflow.com/questions/50065448/how-to-ignore-generated-files-from-go-test-coverage
* （IME）自分の場合は、panicの行も除外したかった。
	* 標準機能ではこの機能は無かったため、htmlをパースして特定の印（コメント）があるものを除外する機能を自分でスクラッチで作成した。
	* なお、ignoreコメントの要望は公式でリジェクトされている
		* https://github.com/golang/go/issues/53271


## その他
### testifyパッケージ
* assert, require, suite
* testify/mockおよびmockery（mock自動生成用の外部パッケージ）
  * https://qiita.com/muroon/items/f8beec802c29e66d1918#arguments
* https://dev.classmethod.jp/articles/go-testify/
