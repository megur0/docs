- [TOP](./README.md)
- [このメモについて](../README.md)


# 公式・参考サイト
* https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html
* https://developer.apple.com/tutorials/swiftui/creating-and-combining-views
* https://gist.github.com/y-takagi/9f2cea659fb3f55b56aa04530bf0af39

# 実機やシュミレーターにインストールした際の構成
* iOSでインストールされたアプリケーションはサンドボックス(Sandbox)構造で保存される
    * サンドボックスとは、アプリケーションを保護された領域で動作させることによって、システム(iOS)が不正に操作されるのを防ぐセキュリティシステム
* アプリがファイルにアクセスできる場所はサンドボックス内のフォルダに限られる。
* 他のアプリからファイルを受け取ったり、他のアプリのファイルにアクセスする方法もあるが、Appleが用意した手順(機能)に従う必要がある。

# サンドボックス内の構成
* Bundle Container
    * アプリ本体そのもの
    * アプリの実行ファイルや画像などのリソースファイル
    * Assets.xcassetsに保存した画像やプロジェクトに直接保存したデータファイルなども保存
    * メインバンドルと呼ばれている。
* iCloud Container
    * iCloudとデータのやり取り
* Data Container
    * Documents/
        * ユーザーが生成したデータの保存先
        * デフォルトではユーザーは見ることはできないものの、基本的にユーザーが見る必要のないデータはここには入れないでLibraryフォルダの方を使うのが無難。
    * Documents/Inbox
        * 他のアプリからのデータを受け取るためのディレクトリ。読み取り専用。
    * Library
        * ユーザのデータファイルを保存する場所
    * Library/Caches
        * 高速にアクセスするためのデータを一時的にキャッシュして配置するディレクトリ
        * バックアップされない。
    * Library/Preference
        * UserDefaultsに保存したデータはここに保存される
    * Library/＜カスタムディレクトリ＞
        * ユーザーには見せないデータを配置することができる
    * tmp
        * アプリ利用中にメモリに持ち続ける必要のない一時的なデータを配置
        * バックアップされない。
* 参考
    * https://learn.microsoft.com/ja-jp/xamarin/ios/app-fundamentals/file-system#application-directories
    * https://www.techpit.jp/courses/80/curriculums/83/sections/627/parts/2182
    * https://qiita.com/inuha/items/171b7e1a858864c91c3a
    * https://gist.github.com/y-takagi/9f2cea659fb3f55b56aa04530bf0af39

# Data Managing Services
* iOSのSDKにはデフォルトでデータを管理する仕組みが用意されている。
* NSUserDefaults
    * Key/Value方式で任意のオブジェクトの保存・読み込みが可能。
    * plistとして保存される。
    * 大きすぎるデータを取り扱うと、メモリへのコピーに時間がかかってしまうのでアプリの動作が遅くなる。
    * Library/Preferencesフォルダに格納されている。
        * 参考
            * https://blog.flatt.tech/entry/ios_file_sharing
* Core Data
    * レコード形式でデータを保存
    * RDB(Relational DataBase)を構築することができる
    * DBファイルはDocuments/以下に作られる。
    * なお、Core Dataの代わりにRealmというモバイル向けデータベースを使うケースも増えてきている
* iCloud
    * 「key=value」形式、Core Data形式、ドキュメント形式、構造化データ形式(レコード単位のデータ)でデータを保存。
    * アプリが保存しているデータをサーバ上に配置し、ネットワークで繋がっている端末同士でデータを共有できる
* Keychain
    * 「key=value」形式
    * 他のアプリからは参照できない。
    * 保存時のアクセス設定によっては同一開発者のアプリ同士であれば参照・編集が可能。
    * 暗号化されたストレージ。パスワードや秘密鍵、証明書などを安全に保存する用途として使われる。
    * アプリを削除しても、デバイスにデータが残る。
    * アプリの再インストールを行うとまたアクセスできる。
* 参考
    * https://qiita.com/inuha/items/171b7e1a858864c91c3a


# アプリを削除すると消去されるデータ
* 「アプリを削除」
    * サンドボックス全体を除去
* 「アプリを取り除く」
    * そのサンドボックス上にあるアプリ本体が除去される。サンドボックスと作成されたデータ/書類は残る。
    * アプリを再びインストールすれば"取り除く"前の状態へ戻る。 
* なお、上記のいずれにおいてもキーチェーンに保存されたデータは残る。