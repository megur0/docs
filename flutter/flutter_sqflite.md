- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# 基本的な利用方法
* https://docs.flutter.dev/cookbook/persistence/sqlite
* https://github.com/tekartik/sqflite/tree/master/sqflite/doc
* https://pub.dev/packages/sqflite


# データベースファイルの保存先について
* 保存先
    |OS|保存先|
    |-|-|
    |iOS|Library/| 
    |Android| data/data/ *1  | 
    * *1 このフォルダはJSSECでAndroidのDBファイル配置で推奨されるディレクトリとのこと
        * https://www.jssec.org/dl/android_securecoding.pdf

* 保存先の取得に利用するパッケージとAPI
    * sqfliteのgetDatabasesPathのみでは上記の要件を満たせないためpath_providerパッケージのAPIも合わせて利用する。  

    |パッケージ|API|iOS|Android|
    |-|-|-|-|
    |sqflite|getDatabasesPath|Documents/を返す。|data/data/を返す| 
    |path_provider|getLibraryDirectory|Library/を返す。|-| 

* データベースの削除(手動)
    * 端末やシミュレーター側でアプリを削除すると確実に消える。

* (参考)
    * https://zenn.dev/beeeyan/articles/b9f1b42de9cb67

* (参考)sqfliteのドキュメントには「getDatabasesPath」を使う方法で書かれている。
    * https://pub.dev/packages/sqflite#opening-a-database
    * ※ 「getDatabasesPath」のリファレンスの方ではpath_providerパッケージを使うことが推奨されている。
        * https://pub.dev/packages/sqflite#opening-a-database
        * https://pub.dev/documentation/sqflite/latest/sqflite/getDatabasesPath.html

