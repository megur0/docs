- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# Cloud Functions for FirebaseはNode.jsのみ対応
* 他の言語で使う手段もある
    * 参考
        * https://qiita.com/norami_dream/items/b6bf6d5a472d0a1acccc
    * IMO
        * ドキュメントもNode.jsのため、Node.jsを利用して実装することが無難と考えている
    

# Cloud Functions による Cloud Firestore の拡張
* https://firebase.google.com/docs/firestore/extend-with-functions?hl=ja
* イベント
    * onCreate、onUpdate、onDelete、onWrite
    * 注意：更新でデータが変更されない場合（オペレーションなしの書き込み）、更新イベントや書き込みイベントは生成されない。
    * 特定のフィールドにイベントを追加することはできない。
    * onWrite は、onCreate、onUpdate または onDelete がトリガーされたときにトリガー
* 特定のドキュメントを指定
    * `functions.firestore.document('users/marie')〜`
* ワイルドカードを指定
    * `.document('users/{userId}').onWrite((change, context) => {〜`
        * users にあるドキュメントの任意のフィールドが変更されると、userId というワイルドカードと照合
        * users に含まれるドキュメントにサブコレクションがある場合、サブコレクションのいずれかに含まれるドキュメントのフィールドが変更されても、userId ワイルドカードはトリガーされない。
    * ワイルドカードは複数使える。
        * `.document('users/{userId}/{messageCollectionId}/{messageId}').onWrite((change, context) => {`
    * コレクションの指定はできない。ドキュメントを指定する必要がある。たとえば{messageCollectionId}がコレクションの場合は以下は無効になる。
        * `functions.firestore.document('users/{userId}/{messageCollectionId}')`
    * onCreate() 
        * ワイルドカードを使用すると、コレクションで新しいドキュメントが作成されるたびに関数をトリガーできる。
    * onDelete() 
        * ワイルドカードを使用して、ドキュメントが削除されたときに関数をトリガー
    * onWrite()
        * 発生するイベントの種類に関係なく、Cloud Firestore ドキュメントのすべての変更をリッスンするには、 onWrite()でワイルドカードを使用
* データの読み取り
    * `change.after.data()`
    * `change.before.data()`
* データの書き込み
    * 関数をトリガーしたドキュメントを変更することもできる。
    * 無限ループに注意。たとえば「あるフィールドが更新された場合に、また別のフィールドを更新する」といった条件を入れておく必要がある。
* トリガーイベント外のデータ
    * Cloud Functions はプロジェクトのサービス アカウントとして承認されているため、Firebase Admin SDK を使用して、読み取りと書き込みを実行できる。
        * したがって、関数をトリガーしたドキュメントではないドキュメントも取得・変更ができる
* 注意
    * 順序は保証されない。
    * イベントは必ず 1 回以上処理されるが、1 つのイベントで関数が複数回呼び出される場合がある。
        * IMO したがって冪等性の設計が必要
    * Cloud Functions 用 Cloud Firestore トリガーは、ネイティブ モードの Cloud Firestore でのみ使用可能。

# (参考)Google Cloud Platform 向けの Cloud Functions と Cloud Functions for Firebase
* https://firebase.google.com/docs/functions/functions-and-firebase?hl=ja
* モバイルアプリまたはモバイル ウェブアプリを構築するデベロッパーは、Cloud Functions for Firebase を使ったほうが便利
* Google Cloud Platform 向けの Cloud Functions
    * Cloud Functions は接続層として機能し、イベントをリッスンして応答することで、数行のコードを使用するだけで、デベロッパーがサーバーのプロビジョニングや管理を行うことなく、GCP サービスをより便利に、高度に組み合わせて使用できるようになる。
    * cloud functions自体は複数の言語に対応している。
        * https://cloud.google.com/functions/docs/create-deploy-gcloud?hl=ja#functions-deploy-command-go

