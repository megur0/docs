- [TOP](./firestore_index.md)
- [このメモについて](../README.md)

# TTLポリシー（未読）
* https://firebase.google.com/docs/firestore/ttl?hl=ja
* 古いデータを自動で削除できる機能？

# データバンドル（未読）
* CDNにキャッシュしておくことでFirestoreへのアクセス回数を減らすことができる高度な機能
* https://firebase.google.com/docs/firestore/bundles?hl=ja
* https://firebase.google.com/docs/firestore/solutions/serve-bundles?hl=ja

# データのエクスポート（未読）
* https://firebase.google.com/docs/firestore/manage-data/export-import?hl=ja
* エクスポート / インポート オペレーションのコストは、費用制限の対象にはならない。
* オペレーションが完了するまで、エクスポート / インポート オペレーションで Google Cloud の予算アラートはトリガーされない。
* エクスポートとインポートのオペレーションは、コンソールの使用状況セクションに表示される使用量には反映されない。

# プロジェクト間でデータを移動（未読）
* https://firebase.google.com/docs/firestore/manage-data/move-data?hl=ja
* 開発環境を設定する場合やアプリを他のプロジェクトに永久的に移行する場合に役立つ。

# BigQueryで読み込む（未読）
* https://cloud.google.com/bigquery/docs/loading-data-cloud-firestore?hl=ja
* Cloud Storage バケットを経由させてエクスポート、インポートを行う。

# Firestore Lite
* Firestore Lite は、軽量なスタンドアロンの REST 専用 Firestore SDK
* 通常のウェブ SDK よりも小さなサイズで、単一ドキュメントの取得、クエリの実行、ドキュメントの更新をサポート。
* Firestore Lite では、レイテンシ補正、オフライン キャッシュ、クエリの再開、スナップショット リスナーの機能は省略

# 集約クエリの実現
* 方法1
    * Cloud Firestore はcount() 集計クエリをサポートしている。ただし現時点では、リアルタイム リスナーおよびオフライン クエリと、count() クエリを併用することはできない。
    * https://firebase.google.com/docs/firestore/query-data/aggregation-queries?hl=ja
* 方法2
    * 集約情報をFirestoreに保存: クライアントサイドでトランザクションを実行
    * クライアントサイドによるトランザクションなので、高度なセキュリティルールとか、オフライン時の失敗とかの考慮が必要。
* 方法3
    * 集約情報をFirestoreに保存: Cloud Functionsでトリガーして更新
    * 評価が追加されるたびに Cloud Functions の関数が呼び出されるため、コストが増加する可能性がある。

# 全文検索
* https://firebase.google.com/docs/firestore/solutions/search?hl=ja&provider=elastic
* Cloud Firestore では、ネイティブ インデックスの作成やドキュメント内のテキスト フィールドの検索をサポートしていない。
* サードパーティサービスを使って、高度なインデックス作成と検索の機能を使える。
* 例としてCloud Functionsで、notes/{noteId}が作成されるたびにElasticを使ってインデックスを作成。

# ユーザーのプレゼンス
* https://firebase.google.com/docs/firestore/solutions/presence?hl=ja
* オンラインでアクティブになっているユーザーやデバイスを検出
* 他の Firebase 製品を利用してプレゼンス システムを構築できる。

# 分散カウンタ
* https://firebase.google.com/docs/firestore/solutions/counters?hl=ja#swift_1
* シャードを使う。
* 要約(IMO)
    * サブコレクションとして複数のフィールドを作っておき、書き込みの際に対象をランダムに分散させ、読み込みの時はそれらを合計するというシンプルな解決方法。
    * これは前の状態に依存する更新だと難しい?
        * 前の状態に依存するような大量の更新はアプリで発生しないかもしれない。

# ユーザーとグループのアクセス保護
* https://firebase.google.com/docs/firestore/solutions/role-based-access?hl=ja
* ユーザーごとにロールを持たせてセキュリティルールで管理する具体例

# cloud functionsを使ったコレクション削除（未読）
* https://firebase.google.com/docs/firestore/solutions/delete-collections?hl=ja#web

# cloud functionsを使ったスケジュール処理の例（未読）
* https://firebase.google.com/docs/firestore/solutions/schedule-export?hl=ja#firebase-console
* 例えば、cloud functionsでNode.jsで、firebase-functionsのパッケージで、cloud schedulerと連携できる。
* このとき、Cloud Function は、プロジェクトのデフォルト サービス アカウントを使用する。
* cloud functionsで、GCS バケット等を使う場合は、デフォルト サービス アカウントに権限を設定する必要がある。

# 一つのコレクションに対する500 回/秒以上の書き込み。
* https://firebase.google.com/docs/firestore/solutions/shard-timestamp?hl=ja
* コレクション内に、timestampのような、単調に増加または減少するインデックス フィールドを含むドキュメントがある場合、Cloud Firestore は書き込みレートを 500 回/秒の書き込みに制限する。

# ジオクエリ
* https://firebase.google.com/docs/firestore/solutions/geoqueries?hl=ja
* Cloud Firestore では複合クエリごとに 1 つの range 句のみを使用できる。
* したがって、緯度と経度をそれぞれ別のフィールドとして保存して境界ボックスをクエリするだけでは、ジオクエリを実行することはできない。
* 緯度、経度からgeohashを保存する。geofire-commonというライブラリを使ってgeohashを計算する。
* geohashに対してクエリを実行して、検索を行う。

# データベースの作成を自動化する。(未読)
* https://firebase.google.com/docs/firestore/solutions/automate-database-create?hl=ja
* REST API、gcloud コマンドライン ツール、Terraform を使用して、Cloud Firestoreデータベースの生成を自動化
* Firebaseのコンソールで手動で操作する作業を外部から自動化する、といった話？

# インデックスの費用の削減
* https://firebase.google.com/docs/firestore/solutions/index-map-field?hl=ja
* 除外するば単一インデックスは減るが、除外の上限は200。
* 除外したいフィールドがかなり多い場合などは、特定のマップ型のフィールドに入れ子にして、そのフィールドを除外することで除外数は1になるので、節約できる。
    * IMO よほど多くないとこのソリューションは使わないかもしれない。

# REST APIの利用
* https://firebase.google.com/docs/firestore/use-rest-api?hl=ja
* 完全なクライアント ライブラリを実行できないモノのインターネット（IoT）デバイスなど
* データベース管理の自動化を行ったり、詳細なデータベース メタデータを取得したりする場合

# Firebase Realtime Database で Cloud Firestore を使用する。
* https://firebase.google.com/docs/firestore/firestore-for-rtdb?hl=ja

# サードパーティライブラリ
* FirebaseUI
* React Native Firebase
* https://firebase.google.com/docs/firestore/library-integrations?hl=ja