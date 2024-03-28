- [このメモについて](../README.md)


# リファレンス
* https://firebase.google.com/docs/cli?hl=ja

# ヘルプ
* `firebase -h`
* `firebase use -h`
* `firebase login -h`

# update
* `npm i -g firebase-tools`

# ログイン・ログアウト
* `firebase login:list`
* `firebase login`
* `firebase logout`

# プロジェクトへfirebaseの設定をする
* `firebase init`
* `firebase init [feature]`
    * [feature] は database,emulators,firestore,functions,hosting,storage など
* 対話形式で初期化を実施
* あるいは新しい機能を既存のFirebaseプロジェクトへ追加する
* 'firebase.json' and '.firebaserc'　が生成 or 更新される

# プロジェクト新規作成
* `firebase projects:create [プロジェクトID]`

# 既存のGCPプロジェクトにfirebaseの設定をする
* `firebase projects:addfirebase [プロジェクトID]`
* firebase init コマンドとの使い分けは?

# current projectの設定
* `firebase use [プロジェクトID]`
* 設定すると --project オプション で都度指定する必要が無くなる

# トラブルシュート
* コマンド実行時に認証エラーが出る時
    * 一度`firebase logout`して、`firebase login`すると 解決することが多い






