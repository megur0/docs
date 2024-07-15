[TOP(About this memo))](../README.md) > 


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

# アクティブプロジェクト、エイリアス
* https://firebase.google.com/docs/cli?hl=ja#project_aliases
* プロジェクトの一覧
    * `firebase projects:list`
    * アクティブなプロジェクトは「 (current) 」と表示される。
* アクティブプロジェクトの表示、エイリアスの表示
    * `firebase use`
    * デプロイターゲットを表示
        * `firebase target`
        * 「デプロイのターゲット」と「アクティブプロジェクト」 の違いは理解できていない。(実行すると基本的にアクティブプロジェクトが表示される)
* アクティブプロジェクトをクリア
    * `firebase use --clear`
* エイリアスを追加
    * `firebase use --add`
    * インタラクティブにエイリアスを追加し、アクティブプロジェクトとして設定する。
    * .firebasercが更新される
    * ※ `firebase use --alias` というコマンドもあったが利用方法がわからなかった。
* エイリアスを削除
    * `firebase use --unalias PROJECT_ALIAS`
    * .firebasercが更新される
* エイリアス or プロジェクトを アクティブプロジェクトとする。
    * `firebase use [エイリアス or プロジェクトID]`
* プロジェクトを都度指定する
    * `--project`で都度指定する。
    * アクティブプロジェクトよりも`--project`が優先される(はず)
* .firebaserc
    * 例
        ```
        {
            "projects": {
                "default": "プロジェクトID",
                "エイリアス名1": "プロジェクトID",
                "エイリアス名2": "プロジェクトID"
            }
        }
        ```
    * ファイル中に"default"として指定されたプロジェクトIDはアクティブプロジェクトとなる。
        * なお、ドキュメントではこの点に関しては触れられていない。
    * エイリアスを.firebasercへ記載しておくことで`firebase use [エイリアス]`で利用できる。
    * gitにコミットすることでチームでdefaultのプロジェクトとエイリアスを共有できる。
        * (IMO)それぞれの開発者が別のfirebaseプロジェクトを利用して開発をするような環境下ではgit管理対象外とした方が良いだろう。

# firebase init
* `firebase init`
* `firebase init [feature]`
    * [feature] は database,emulators,firestore,functions,hosting,storage など
* 対話形式で初期化を実施
* あるいは新しい機能を既存のFirebaseプロジェクトへ追加する
* 対話の中でデフォルトプロジェクトを選択する
* firebase.json と .firebaserc　が生成 or 更新される
    * 選択したプロジェクトは、.firebasercにdefaultとして設定される。

# プロジェクト新規作成
* `firebase projects:create [プロジェクトID]`

# 既存のGCPプロジェクトにfirebaseの設定をする
* `firebase projects:addfirebase [プロジェクトID]`
* firebase init コマンドとの使い分けは?

# トラブルシュート
* コマンド実行時に認証エラーが発生する時
    * 一度`firebase logout`して、`firebase login`すると 解決することが多い






