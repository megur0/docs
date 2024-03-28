- [このメモについて](../README.md)


# Cloud Storageをはじめる
* デフォルトバケットの作成：
    * セキュリティルールのモードの選択
        * デフォルトではすべてのアクセスが拒否されるようになっているため、適宜変更する。
    * リージョンの選択
        * このリージョンは、firebaseプロジェクト全体へ適用される？
        
# Cloud Storageのセキュリティ
* https://firebase.google.com/docs/rules/basics?hl=ja#default_rules_locked_mode
## ２つのモード
* ロックモード
    * 認証されたユーザーのみが Storage バケットにアクセス
* テストモード
    * すべてのユーザーにアクセスを許可
## セキュリティルールによるアクセス制限の例
* https://firebase.google.com/docs/rules/basics?hl=ja#cloud-storage
* コンテンツ所有者のみ許可
```
〜
match /user/{userId}/{allPaths=**} {
    allow read, write: if request.auth != null && request.auth.uid == userId;
}
〜
```
* コンテンツ所有者のみ書き込み可能、所有者以外は認証されていれば許可
```
〜
match /users/{userId}/{allPaths=**} {
    allow read: if request.auth != null;
    allow write: if request.auth.uid == userId;
}
〜
```

# 一覧表示の禁止
* Firebase上でCloud Storageを有効化した場合はデフォルトで対象のバケットの一覧表示は禁止になっている。
* GCPコンソールのCloud Storageから確認すると `projectViewer:PROJECT_ID` のプリンシパルに対して「Storage レガシー バケット読み取り」の権限が付与されている。
    * (参考)
        * この`projectViewer:PROJECT_ID`は、IAM バケット ポリシーにのみ適用できる特別なプリンシパルと記載されている。
        * https://cloud.google.com/storage/docs/access-control/iam?hl=ja#convenience-values
* これはファイル自体は誰でもアクセス可能だが、ファイルの一覧表示は不可となる。
* `https://storage.googleapis.com/(バケット名)`へアクセスしてもdeniedとなる。