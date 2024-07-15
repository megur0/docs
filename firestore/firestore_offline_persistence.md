[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# オフラインでのデータの永続性
* https://firebase.google.com/docs/firestore/manage-data/enable-offline?hl=ja
* Cloud Firestore は、オフライン データの永続性をサポート
* オフラインの永続性は Android、Apple、ウェブアプリのみでサポート
* Cloud Firestore データのコピーがキャッシュに保存
* キャッシュされたデータに対しては書き込み、読み取り、リッスン、クエリを行うことができる。
* デバイスがオンラインに戻ると、アプリがローカルで行った変更と Cloud Firestore バックエンドが同期。
* android, iosでは、デフォルトで有効になっている。
* ウェブの場合、オフラインの永続性はデフォルトで無効になっている。
* オフラインの永続性の有効・無効の設定が可能。
    * `db.settings = const Settings(persistenceEnabled: true);`
* その他はドキュメント参照。
