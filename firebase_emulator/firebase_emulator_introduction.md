- [TOP](./README.md)
- [このメモについて](../README.md)

# Firebase Local Emulator Suite
* https://firebase.google.com/docs/emulator-suite?hl=ja
* Firebase Local Emulator Suite は、アプリのビルドとテストをローカルで行うことのできる、デベロッパー向けの高度なツールセット
    * 評価、プロトタイピング、開発、継続的インテグレーションのワークフローに適している。
* Emulator Suite UI
    * 使用すると、データベースの設計を何度も繰り返す、Cloud Functions の関数に関連するさまざまなデータフローを試す、セキュリティ ルールの変更を評価する、ログを参照してバックエンド サービスの動作を確認するといったことができる。最初からやり直す場合は、データベースをクリアして、新しい設計のアイデアから始めることもできる。
* 使用できる機能
    * Cloud Firestore
    * Realtime Database
    * Cloud Storage for Firebase
    * Authentication
    * Firebase Hosting
    * Cloud Functions（ベータ版）
    * Pub/Sub（ベータ版）
    * Firebase Extensions（ベータ版）
* 注意: エミュレータを本番環境として使ってはいけない。
    * エミュレータを Firebase サービスの「自己ホスト」バージョンとして使用してはいけない。
    * パフォーマンスやセキュリティではなく精度を重視して設計されているため、本番環境での使用には適していない。
* プロトタイプとテストのワークフロー
    * 単体テスト
        * Firebase Test SDK を使用
            * セキュリティ ルールの読み込み、テスト間のローカル データベースのフラッシュ、エミュレータとの同期インタラクションの管理を行うための便利なメソッドがある。
            * アプリのロジックに依存しないデータベース インタラクションの簡単なテストを作成するのに最適
    * 統合テスト
        * Emulator Suite 内の各プロダクト エミュレータは、本番環境の Firebase サービスと同じように、SDK 呼び出しと REST API 呼び出しに応答。
        * 独自のテストツールを使用して、Local Emulator Suite をバックエンドとして使用する自己完結型の統合テストを作成できる。
    * 手動テスト
        * 実行中のアプリケーションを Local Emulator Suite に接続して、Firebase アプリを手動でテスト
        * 本番環境データは危険にさられることなく、テスト プロジェクトの構成は不要
    * プロダクト評価
        * 安全なローカル環境で Firebase Extensions のインストールと管理を行うことで、請求額を最小限に抑えながらその機能を理解できる。


# Admin SDKでエミュレータを使う場合
* https://firebase.google.com/docs/emulator-suite/connect_auth?hl=ja#admin_sdks
* 環境変数に設定する必要がある。
    * export FIREBASE_AUTH_EMULATOR_HOST="localhost:9099"
* Cloud Functions エミュレータは Authentication エミュレータを自動的に認識するため、Cloud Functions エミュレータと Authentication エミュレータの統合をテストする場合はこの手順を省略できる。