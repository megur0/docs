[TOP(About this memo))](../README.md) > 


# Cloud Storageをはじめる
* デフォルトバケットの作成：
    * セキュリティルールのモードの選択
        * デフォルトではすべてのアクセスが拒否されるようになっているため、適宜変更する。
    * リージョンの選択
        * このリージョンは、firebaseプロジェクト全体へ適用される？

# アップロード処理の実装
* ※ リンクはFlutterの場合
* アップロード
    * https://firebase.google.com/docs/storage/flutter/upload-files?hl=ja#error_handling
    * アップロード処理は完了を待つこと、あるいは進捗状況を随時ストリームで受け取ることができる。
* エラー
    * https://firebase.google.com/docs/storage/flutter/handle-errors?hl=ja
    * セキュリティルールに違反した際の失敗は、この例外には含まれない点に注意。（IME: Flutter SDKでは個別にストリーム内でハンドリングする必要があった）
* バケット
    * https://firebase.google.com/docs/storage/flutter/start?hl=ja#use_multiple_storage_buckets
    * デフォルトで使用されるバケットはデフォルトバケット  
    * カスタム バケットを使用する場合は明示的にAPIで指定する。


# Cloud Storageのセキュリティ
* https://firebase.google.com/docs/storage/security?hl=ja
* https://firebase.google.com/docs/rules/basics?hl=ja#default_rules_locked_mode
## ２つのモード
* ロックモード
    * 認証されたユーザーのみが Storage バケットにアクセス
* テストモード
    * すべてのユーザーにアクセスを許可
## セキュリティルールによるアクセス制限の例
* https://firebase.google.com/docs/rules/basics?hl=ja#cloud-storage
* 書き込みは'image/png'、5MB以下、/users/(uid)/profile.pngのパスのみ許可。読み込みは認証されている場合。
```
〜
match /users/{userId}/profile.png {
    allow read: if request.auth != null;
    allow write: if request.resource.contentType == 'image/png' && request.resource.size <= 5 * 1024 * 1024 && request.auth != null && request.auth.uid == userId;
}
〜
```

# 一覧表示の禁止
* Firebase上でCloud Storageを有効化した場合はデフォルトで対象のバケットの一覧表示は禁止になっている。
* GCPコンソールのCloud Storageから確認すると `projectViewer:PROJECT_ID` のプリンシパルに対して「Storage レガシー バケット読み取り」の権限が付与されている。
    * (参考)
        * この`projectViewer:PROJECT_ID`は、IAM バケット ポリシーにのみ適用できる特別なプリンシパルと記載されている。
        * https://cloud.google.com/storage/docs/access-control/iam?hl=ja#convenience-values
* こちらはファイル自体は誰でもアクセス可能だが、ファイルの一覧表示は不可となる。
* `https://storage.googleapis.com/(バケット名)`へアクセスしてもdeniedとなる。


# Admin SDK
* https://firebase.google.com/docs/storage/admin/start?hl=ja
* デフォルトバケットのAPIは、初期化の際の設定したバケットを使用する。
    * 自動でデフォルトバケットを取得するAPIではない点に注意。設定していない場合は実行時エラーとなる。
* ダウンロードURLの取得
    * おそらく、SDKとしてはNode.jsのみ対応?
    * https://firebase.google.com/docs/storage/admin/start?hl=ja#shareable_urls