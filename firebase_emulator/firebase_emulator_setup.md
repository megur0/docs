[TOP(About this memo))](../README.md) > [一覧(Firebase Local Emulator Suite)](./README.md) > セットアップ


# インストールに必要な環境
* Node.js バージョン 16.0 以降
* Java JDK バージョン 11 以降

# インストール
* JDKをインストールしておく必要がある。
* firebase init
    * 質問にしたがって設定を行う。
    * エミュレータのダウンロード、firebase.jsonの構成が実行される。
* エミュレータインストール先はjarファイルが格納される。
    * ~/.cache/firebase/emulators
    * 参考
        * https://techblog.sgr-ksmt.dev/2019/11/06/151755/

# アップデート
* エミュレータがインストールされると、Firebase CLI のバージョンを更新するまでアップデートのチェックは行われない。
    * 追加の自動ダウンロードも行われない。
    * https://firebase.google.com/docs/emulator-suite/install_and_configure?hl=ja#install_the_local_emulator_suite
* したがってfirebase自体をアップデートした後は、emulatorも個別にアップデートする必要がある。
* 手動でアップデートする例
    * ~/.cache/firebase/emulators　を削除
    * 適当なプロジェクトフォルダを作ってfirebase initを実行。
    * 質問で emulatorを選択しておき、利用するEmulatorを選択後、「Would you like to download the emulators now? 」の質問でYesとする。
* Release Nots
    * CLIのリリースノートの中に記載される。
    * 例えば、firebase cli v13.16.0で、emulator v1.19.8がリリースされている。
    * https://firebase.google.com/support/release-notes/cli

# コマンド
* `emulators:start`
    * firebase.json で構成された Firebase プロダクトのエミュレータを起動
    * 未インストールの場合は、~/.cache/firebase/emulators/ にエミュレータがダウンロードされる。
* emulatorの起動コマンドオプション
    * `--only functions,firestore`
    * `--inspect-functions` で実行すると関数のブレークポイントのデバッグを有効になる。
        * 特別なシリアル化された実行モードに切り替わり、関数は単一のプロセスで順次（FIFO）実行される。
    * `emulators:exec scriptpath`	
        * firebase.json で構成された Firebase プロダクトのエミュレータを起動した後、scriptpath でスクリプトを実行する
        * スクリプトの実行が終了すると、エミュレータ プロセスは自動的に停止。
    * `--import` `emulators:export`
        * 起動時にインポート、停止時にエクスポートさせることでデータやユーザー情報を永続化させることができる
        ```
        firebase emulators:start --import=./(フォルダ名) --export-on-exit
        ```
    * https://firebase.google.com/docs/emulator-suite/install_and_configure?hl=ja#startup


# CIとの統合
* https://firebase.google.com/docs/emulator-suite/install_and_configure?hl=ja#integrate_with_your_ci_system
* コンテナ化された Emulator Suite イメージの実行

# Admin SDKでエミュレータを使う場合
* https://firebase.google.com/docs/emulator-suite/connect_auth?hl=ja#admin_sdks
* 環境変数に設定する必要がある。
    * 例
        * export FIREBASE_AUTH_EMULATOR_HOST="localhost:9099"
* Cloud Functions エミュレータは Authentication エミュレータを自動的に認識する
    * したがってCloud Functions エミュレータと Authentication エミュレータの統合をテストする場合はこの手順を省略できる。

# デモプロジェクト
* https://firebase.google.com/docs/emulator-suite/connect_auth?hl=ja#choose_a_firebase_project
* コンソールのプロジェクトの場合
    * サポートされているプロダクトの一部またはすべてに対してエミュレータを実行できる。
    * エミュレートしていないプロダクトに関しては、アプリとコードはライブリソース（データベース インスタンス、ストレージ バケット、関数など）とやり取りする
* デモプロジェクト
    * デモ Firebase プロジェクトを使用する場合、アプリとコードはエミュレータのみとやり取りする。エミュレータが実行されていないリソースとアプリがやり取りしようとすると、コードは失敗する
* デモプロジェクトを使うメリット
    * エミュレートされていない（本番環境の）リソースを誤って呼び出しても、データが変更されたり、使用量がカウントされたり、課金が発生したりする可能性がないため、安全性が高い
* 利用方法
    * `firebase emulators:start --project demo-xxxx`　のようにemulator起動時にdemo-をつける。
    * 実際のprojectがgcp上に存在しなくてもemulatorを起動することが可能?
    * Admin SDK側でinitializeAppをしている場合、分岐処理をしてプロジェクトIDにdemo-xxxを指定する必要がある。
    * 参考
        * https://zenn.dev/tkow/scraps/8160da23277fb8


# REST API
* https://firebase.google.com/docs/firestore/reference/rest?hl=ja
## Firestoreの情報取得
* https://firebase.google.com/docs/reference/rest/database?hl=ja
```
curl "http://localhost:8081/v1/projects/(プロジェクトID)/databases/(default)/documents/users
curl "http://localhost:8081/v1/projects/(プロジェクトID)/databases/(default)/documents/users/(UID)"
```
## サインイン
* https://firebase.google.com/docs/reference/rest/auth?hl=ja
```
curl -H "Content-Type: application/json" -d '{"email":"test@example.com", "password":"testtest"}' "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=aa" 
```
## ユーザーアカウントの全クリア
* https://firebase.google.com/docs/reference/rest/auth?hl=ja#section-auth-emulator-clearaccounts
```
# アカウント全削除
curl -H 'Authorization: Bearer owner' -X DELETE http://localhost:9099/emulator/v1/projects/(プロジェクトID)/accounts
```
## Firestoreのクリア
* https://firebase.google.com/docs/emulator-suite/connect_firestore?hl=ja#clear_your_database_between_tests
```
# Firestoreデータクリア
curl -v -X DELETE "http://localhost:8080/emulator/v1/projects/(プロジェクトID)/databases/(default)/documents"
```
## Cloud Storageのファイルの全削除
* おそらく、まとめて削除する手段は無く、ファイル１つ１つを取得して削除する方法となるだろう。
* 以下のように実行可能と思われるが、ファイルの削除の方でエラーとなってしまった。
    ```
    # バケット一覧表示
    curl http://localhost:9199/b | jq

    # バケットのファイル一覧表示
    curl -H 'Authorization: Bearer owner' http://localhost:9199/v0/b/(プロジェクトID).appspot.com/o/ | jq

    # ファイルの削除: 501 Not Implementedとなってしまった
    curl -v -X DELETE -H 'Authorization: Bearer owner' http://localhost:9199/v0/b/(プロジェクトID).appspot.com/(ファイルパス)
    ```
* 参考
    * https://stackoverflow.com/a/75649190/22090329
