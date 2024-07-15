[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# Firestoreとは?
* 2019年2月1日に正式リリースされた、FirebaseとGCPからのモバイル、Web、サーバー開発に対応した柔軟でスケーラブルな NoSQL クラウド データベース
* FirestoreとRealtime Databaseの違いは?
    * https://firebase.google.com/docs/database/rtdb-vs-firestore?hl=ja
    * Firestoreを推奨されている。


# firestoreのネイティブモードと、Datastoreモード
* 基本的に新規開発であればネイティブモードを選択する。
* 参考
    * https://qiita.com/kento_gm/items/0ae03a6fee989a53ed2f
    * https://zenn.dev/google_cloud_jp/articles/a0a6b5f855fe90


# SDK
* HTTP 呼び出しまたは RPC 呼び出しを使用して Firebase API を直接呼び出すこともできるが、SDKを使うことを推奨。
* モバイル SDK とウェブ SDK
    * Firebase セキュリティ ルールと Firebase Auth を組み合わせると、クライアントは Firebase データベースに直接接続できる。（サーバーレス）
    * リアルタイムの更新、オフライン データの永続性をサポート
* サーバー クライアント ライブラリ
    * サーバー クライアント ライブラリはデータベースへの完全アクセス権を備えた特権的な Firebase 環境を作成
    * Firebase のセキュリティ ルールによってリクエストが評価されることはない。
    * アクセス保護をする場合は Identity and Access Management（IAM）を使用する。


# Firebase Local Emulator Suite 
* https://firebase.google.com/docs/firestore/quickstart?hl=ja#optional_prototype_and_test_with
* Firebaseのテストをローカルで行うことができる。
* Cloud Firestore、Realtime Database、Hosting、Cloud Functions for Firebase（セキュリティ ルール エンジンを含む）の高精度エミュレータのセット
* Cloud Firestore エミュレータは Local Emulator Suite の一部。

# Firestoreのロケーション
* https://firebase.google.com/docs/firestore/locations?hl=ja
* https://firebase.google.com/docs/firestore/best-practices?hl=ja#database_location
* レイテンシを低減して可用性を高めるため、データを必要とするユーザーとサービスに近いロケーションにする。
    * 広範囲に及ぶネットワーク ホップはエラーが発生しやすく、クエリのレイテンシを増加
* このロケーションはプロジェクトのデフォルトの Google Cloud Platform（GCP）リソース ロケーションになる。
* プロジェクトのデフォルトの GCP リソース ロケーションを一度設定すると、変更することはできない。
* デフォルトの GCP リソース ロケーションは、以前（プロジェクトの作成時か、ロケーション設定が必要である別のサービスの設定時）に設定されている可能性がある。
    * デフォルトの GCP リソース ロケーションは、ロケーション設定が必要なプロジェクト内の GCP サービスで使用される。
* 以下は、 デフォルトの GCP リソース ロケーションが必要。
    * Cloud Firestore
    * Cloud Storage
        * デフォルトの GCP リソース ロケーションは、デフォルトの Cloud Storage バケットにのみ適用される。
        * Blaze プランをご利用の場合は、それぞれ独自のロケーションを使用する複数のバケットを作成できる。
    * Google App Engine（GAE）アプリ
        * Cloud Scheduler（たとえばスケジュール設定された関数を実行する場合） を使用する場合にプロジェクトに App Engine アプリを配備する必要がある。
* ロケーションの種類
    * マルチリージョン ロケーション
        * 稼働率 99.999% 以上
        * データベースの可用性と耐久性を最大限にする場合は、マルチリージョン ロケーションを選択
        * ヨーロッパと米国しかない
    * リージョン ロケーション
        * 稼働率 99.99% 以上
* なお、リージョンロケーションのデータは、リージョン内の複数のゾーンに複製される。


# プロジェクトとFirestoreデータベース
* Cloud Firestore データベースは、プロジェクトごとに 1 つに制限されている。
* なお、FirestoreはApp Engine に依存しているので、プロジェクトでFirestoreを有効にすると、App Engineがそれに伴って有効になる。
* https://firebase.google.com/docs/firestore/solutions/automate-database-create?hl=ja

