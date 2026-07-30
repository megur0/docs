---
title: "DB - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > DB




## database/sql（標準パッケージ）

### database/sql
* https://www.wakuwakubank.com/posts/869-go-database-sql/
* 基本的には、ORMのパッケージは、database/sqlのラッパー的なものと考えてよいかと思う。
* database/sqlで、DBへの接続、prepared statementによるSQLの実行、取得結果をscanで変数へマッピング、あとはトランザクション処理など、すべて対応可能。

### 各データベースごとのドライバを使う必要がある。
* https://github.com/golang/go/wiki/SQLDrivers
* posgreSQLは　github.com/lib/pqとか。
* PostgreSQL のドライバは github.com/lib/pq より jackc/pgx　が最近は使われているようだ。
https://cloud.google.com/sql/docs/postgres/samples/cloud-sql-postgres-databasesql-connect-unix?hl=ja
https://future-architect.github.io/articles/20210916a/
https://zenn.dev/spiegel/books/a-study-in-postgresql/viewer/connect

### scanの注意点
* scanを使う時にnullableのカラムはdatabase.sql.Null****の型を使う。
  * 自分の場合はScanエラーにはならなかったけど後続につづくカラムが読み取れていないといった事象が発生した。
* ちなみにGormで取得している部分は普通に動作してて、これは内部で吸収されているのかな？
https://imagawa.hatenadiary.jp/entry/2019/08/13/090000


### database/sqlの rowsの扱い
* ここの注意事項が結構ある。
* https://golang.shop/post/go-databasesql-04-retrieving-ja/

### sql.Null〜型
* nullbleなカラムを表現するのに便利。
  * https://qiita.com/taizo/items/3e0b1ca583d8fe2a62a5

### database/sqlを使ってつくったやつ
* 一応作ってみたやつ -> https://github.com/megur0/xii_backend/tree/database-sql
* 面倒なところ
  * 普通のselect処理などが長いので取得処理をカラム名だけで実行するようにしたい -> scanするときにテーブルのカラム名とモデルのフィールドの対応関係が必要。これはモデルにタグをつけておくことでできそうだけど、それだとGORMと同じ
  

### 自作のORマッパー作成時の参考
* SQLの結果の任意のコラム数に応じてScanする。
    * https://ja.stackoverflow.com/questions/66039/go-と-database-sql-で構造体がない場合でも値を取得したい


### （WIP）モデルのIDは*uuid.UUIDを使ったほうが良い？
別にuuidとしてGoのプログラム中で型を保証する必要がないなら、stringで良いかと思う。
https://stackoverflow.com/questions/72023281/how-to-insert-null-value-to-uuid-instead-of-zeros
・・・でも、こういう感じでuuidを書いたほうが正しい気がしてきた。
https://gorm.io/ja_JP/docs/data_types.html




