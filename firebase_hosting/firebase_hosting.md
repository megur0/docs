[TOP(About this memo))](../README.md) > Firebase Hosting

# 既存のプロジェクトへhostingを追加する
* `firebase logout && firebase login`
* `firebase init hosting`
* デプロイ
    * `firebase deploy --only hosting `
    * アップロードされるファイル数が出力される。
    * Firebaseコンソールに表示されるファイル数はこのファイル数より多い。
        * メタファイルのようなものもアップロードしている?

# ファイルの削除
* ファイルを削除する専用のコマンドは無い
    * 削除したい場合、該当のファイルを削除してデプロイコマンドを打つ
* なお、履歴削除はコンソールで、該当の履歴の「削除」から行うことができる

# ファイルのエクスプローラ
* アップロードしたファイルのエクスプローラーのような機能は公式には提供されていない。

# Firebase hostingへのドメイン設定
* https://firebase.google.com/docs/hosting/custom-domain?hl=ja
* 参考
    * https://chaika.hatenablog.com/entry/2022/06/09/083000
* IME
    * ドメイン設定後、コンソール上で角煮すると保留中としばらく表示された。
    * 1〜2時間程経過後、接続されている状態に切り替わった。


