[TOP(About this memo))](../README.md) > Firebase Clashlytics


# 公式
* https://firebase.google.com/docs/crashlytics?hl=ja

# 導入
* https://firebase.google.com/docs/crashlytics/get-started

# Analyticsとの連携
* https://firebase.google.com/docs/crashlytics/start-using-analytics?hl=ja&platform=flutter
    > アプリ内でアナリティクスと Crashlytics を組み合わせて使用すると、問題を生成してクラッシュ データを細かくトラッキングするための機能を利用できます。たとえば、クラッシュが発生していないユーザーをトラッキングしたり、クラッシュの前の特定のイベントを追跡するパンくずリストや BigQuery を利用したりすることができ、アプリの主要な指標を可視化できます。

# Clashlyticsの画面でdSYMのバージョンが不明(unknown)となる
* この表示は期待通りの挙動であり、dSYMに紐づくバージョンのアプリがクラッシュすることで、バージョンが表示される。
* https://github.com/orgs/codemagic-ci-cd/discussions/2596
    > Apparently this is expected behavior. If a crash occurs after the file is uploaded the version number gets populated.

# コンソール>dSYMsに表示されるMissing(Optional)(見つからない(オプション))となるdSYMについて
* dSYMには"required"と"optional"がある。
* "required"のdSYMがアップロードされていない場合はFirebaseは問題を表示し、かつ、クラッシュが発生してもダッシュボードへ表示されない。
* 一方、"optional"の方は不足していてもダッシュボードには表示される。
    * ただし、スタックトレースの一部がシンボル化されない可能性がある。
* https://stackoverflow.com/questions/61640184/firebase-crashlytics-missing-optional-dsyms
## (参考)プロジェクト内のdSYMファイルの確認
* プロジェクトディレクトリのルートで `find ./ -name "*.dSYM"`
