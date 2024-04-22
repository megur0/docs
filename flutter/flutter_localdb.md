- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# ローカルファイルの保存
* https://docs.flutter.dev/cookbook/persistence/reading-writing-files
* Flutterアプリでは通常、path_providerプラグインとdart:ioライブラリを組み合わせて行う。
    * 詳細は上記URL参照

# データ保存のための各種パッケージ
* shared_preferences
    * https://pub.dev/packages/shared_preferences
    * https://docs.flutter.dev/cookbook/persistence/key-value
        > If you have a relatively small collection of key-values to save, you can use the shared_preferences plugin.
    * Flutter.devチームによるKey/Value方式でデータ保存をするためのパッケージ
    * iOSやmacOSのNSUserDefaults、AndroidのSharedPreferences等をラップするパッケージ
    * pub.devに下記のように記載されており、"critical data"には使わないようにと記載されている。
        > Data may be persisted to disk asynchronously, and there is no guarantee that writes will be persisted to disk after returning, so this plugin must not be used for storing critical data.
* sqflite
    * https://docs.flutter.dev/cookbook/persistence/sqlite
        > If you are writing an app that needs to persist and query large amounts of data on the local device, consider using a database instead of a local file or key-value store. In general, databases provide faster inserts, updates, and queries compared to other local persistence solutions.
    * 利用方法の全般は下記のドキュメントへ記載されている。
        * https://github.com/tekartik/sqflite/tree/master/sqflite/doc
        * https://pub.dev/packages/sqflite
    * ローカルで多くのデータを保持する、クエリを利用したい場合
* Hive
    * https://pub.dev/packages/hive
    * Key/Value方式のデータベースとして使用するためのパッケージ
    * shared_preferencesを高速化、高機能化したもの。
        * https://medium.com/flutter-community/using-hive-instead-of-sharedpreferences-for-storing-preferences-2d98c9db930f
* Isar
    * https://isar.dev/ja/
    * Hiveの後継として開発された。HiveよりIsarの利用が推奨されている。
        * https://isar.dev/faq.html#where-clauses
    * NoSQL
* Drift
    * https://drift.simonbinder.eu/
    * SQLiteをラップして抽象化したもの
      

