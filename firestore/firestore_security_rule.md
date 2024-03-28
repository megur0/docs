- [TOP](./README.md)
- [このメモについて](../README.md)


# セキュリティルール
* バージョン
    * 2019 年 5 月以降、Cloud Firestore セキュリティ ルールのバージョン 2 が使用可能になった。
    * セキュリティ ルールで rules_version = '2'; を最初の行にして、バージョン 2 にオプトインする必要がある。
* ルールのプレイグラウンドツール
    * https://firebase.google.com/docs/firestore/security/insecure-rules?hl=ja#simulator
    * コンソールのルールタブで手軽にテストできる。
    * ローカルでテストするには、Cloud Firestore エミュレータを使う。
* デプロイ
    * Cloud Firestore セキュリティ ルールの更新では、新しいクエリやリスナーに影響が及ぶまでに最大 1 分かかることがある。
    * 変更内容が完全に伝播され、アクティブなすべてのリスナーに影響が及ぶまでに最大 10 分かかる。
    * CLIを使ってデプロイすると、プロジェクトディレクトリで定義したルールで上書きされる。
        * https://firebase.google.com/docs/firestore/security/get-started?hl=ja#use_the_firebase_cli
    * IMO
        * Firebaseコンソール上でもデプロイできるが、コード上で管理した方が中長期の保守はしやすいかもしれない。

# セキュリティルールによって不正アクセスを防止する
* Cloud Firestore セキュリティ ルールを使用して、データベースに対する不正オペレーションを防止
* たとえば、ルールを使用することによって、悪意のあるユーザーがデータベース全体を繰り返しダウンロードする行為を防止

# データの保護
* モバイルおよびウェブ クライアント ライブラリ
    * Firebase Authentication と Cloud Firestore セキュリティルールを使用して、サーバーレスな認証、承認、データ検証を処理
    * App Check を使用して、ご自分のアプリだけが Cloud Firestore のデータにアクセスできるように保護
    * Cloud Storage セキュリティ ルールでアクセスできる、データベース ドキュメント内の Cloud Storage リソースへのアクセス条件を定義
* サーバー クライアント ライブラリ
    * Identity and Access Management（IAM）を使用して、データベースへのアクセスを管理
    * すべての Cloud Firestore セキュリティ ルールをバイパスし、代わりに Google アプリケーションのデフォルト認証情報を使用して認証を行う。

# ルールの構成
* 全体
```
service cloud.firestore {
  match /databases/{database}/documents {
    // ...
  }
}
```
* service cloud.firestore
    * サービスを指定
* match /databases/{database}/documents 宣言
    * プロジェクト内の Cloud Firestore データベースと一致するように指定
    * 現在、各プロジェクトには (default) という名前のデータベースが 1 つだけのため、データベースの指定は、ワイルドカード{database}のままで良い。
* ルールの構成
    * match ステートメント
        * コレクションではなくドキュメントを指す必要がある。
    * allow 式
        * read
            * より詳細に、getとlistでも指定できる。
            * get のルールは単一のドキュメントに対するリクエストに適用され、list のルールはコレクションに対するクエリとリクエストに適用される。
        * write
            * より詳細に、create、update、deleteでも指定できる。
* セキュリティルールでは、マッチしたアクセスのみ許可。したがって明示されていないアクセスはすべて拒否される。
    * ホワイトリスト形式。
* あるドキュメントが複数のmatchステートメントと一致する場合、いずれかのステートメント内の条件でtrueになればアクセスは許可される。
* match /cities/{city} のようにワイルドカードを使用して、指定されたパスのすべてのドキュメントを指すことができる。
* 一致しなければ適用されないので、例えば、match /cities/{city}はその下にある/cities/{city}/landmarks/{landmark}には適用されない。
* matchのネストは相対パスになる。 したがって以下は同等
    * `match /cities/{city}/landmarks/{landmark} { 〜`
    * `match /cities/{city} { match /landmarks/{landmark} {　〜`
* 再帰ワイルドカード構文
    * `match /{document=**} {〜`
        * データベース全体の任意のドキュメントに一致
    * `match /cities/{document=**} {〜`
        * /cities/SF/landmarks/coit_tower にも一致する。
        * document 変数の値は SF/landmarks/coit_tower になる。
    * `match /{path=**}/songs/{song} {〜`
        * songsコレクショングループ内のドキュメントに一致。
    * match ステートメントごとに使用できる再帰ワイルドカードは 1 つのみとなる。


# ルールの記述
* 認証
    * `〜　allow read, write: if request.auth != null;`
    * `match /users/{userId} { allow read, update, delete: if request.auth != null && request.auth.uid == userId;`
    * request.auth 変数には、データを要求しているクライアントの認証情報が含まれる。
        * Firebase Authentication によってrequest.auth 変数に値が設定される。
* データの検証
    * `allow read: if resource.data.visibility == 'public';`
    * resource 変数は要求されたドキュメントを参照
* 受信データのチェック
    * データを書き込む際は、request.resource 変数にはドキュメントの将来の状態が含まれる。
    * `allow update: if request.resource.data.population > 0 && request.resource.data.name == resource.data.name;`
* 他のドキュメントへのアクセス
    * get, exists
        * `exists(/databases/$(database)/documents/users/$(request.auth.uid))`
        * `get(/databases/$(database)/documents/users/$(request.auth.uid)).data.admin == true`
    * getAfter
        * トランザクションまたはバッチ書き込みが完了した後（トランザクションまたはバッチが commit される前）のドキュメントの状態にアクセスできる。
        * 具体的な用途については想像できていない。
    * このアクセスは課金対象になる。（ルールによってリクエストが拒否された場合でも課金される）
* カスタム関数
    * https://firebase.google.com/docs/firestore/security/rules-conditions?hl=ja#custom_functions
    * 一連の条件を関数としてラップし、ルールセット全体で再利用できる。
* クエリの制約の評価
    * https://firebase.google.com/docs/firestore/security/rules-query?hl=ja#evaluating_constraints_on_queries
    * `allow list: if request.query.limit <= 10;`
        * 10件以下に絞っていないクエリは不可。
* 更新時にフィールドを制限する
    * https://firebase.google.com/docs/firestore/security/rules-fields?hl=ja#restricting_fields_on_update
    * `request.resource.data.keys()`は更新後を参照するため、そのまま使うことはできない。変更しなかったフィールドも含めて完全なドキュメントを表すため。
    * したがって、`.diff(resource.data).affectedKeys()`を使う。
    * 禁止
        * `allow update: if (!request.resource.data.diff(resource.data).affectedKeys().hasAny(['average_score', 'rating_count']));`
    * 必須
        * `allow update: if (!request.resource.data.diff(resource.data).affectedKeys().hasAll(['name', 'location', 'city']));`
    * 限定
        * `allow update: if (request.resource.data.diff(resource.data).affectedKeys().hasOnly(['name', 'location', 'city', 'address', 'hours', 'cuisine']));`
* 特定のフィールドのみに限定してドキュメントを作成させる
    * https://firebase.google.com/docs/firestore/security/rules-fields?hl=ja#restricting_fields_on_document_creation
    * 禁止
        * `allow create: if (!request.resource.data.keys().hasAny(['average_score', 'rating_count']));`
    * 必須
        * `allow create: if request.resource.data.keys().hasAll(['name', 'location', 'city']);`
    * 限定
        * `allow create: if (request.resource.data.keys().hasOnly(['name', 'location', 'city', 'address', 'hours', 'cuisine']));`
    * IMO
        * 禁止を使うより、必須＋限定の方が、新しいフィールドがデフォルトで禁止されるので安全?
* フィールドタイプを制限する
    * https://firebase.google.com/docs/firestore/security/rules-fields?hl=ja#enforcing_field_types
    * スキーマレスなので、同一のフィールドでもドキュメントによって違うタイプを入れることができる。
    * セキュリティルールで制限することができる
    * 有効な型：
        * bool、bytes、float、int、list、latlng、number、path、map、string、timestamp
        * list データ型と map データ型は、汎用型または型引数をサポートしていない。
    * `request.resource.data.score is int`
    * 必須ではないフィールドをチェックしたい時は、フィールドが存在しないとrequest.resource.data.〜でエラーになってしまうのでgetを使う。（getで取得したものが存在しない場合は、引数で指定したデフォルト値を適用）
        * `request.resource.data.get('tags', []) is list;`

# 部分的な書き込みは許可されない。
* セキュリティ ルールは、クライアントにドキュメントの変更を許可するか、編集全体を拒否するかのどちらか
* ドキュメント内の一部のフィールドへの書き込みを許可する一方で、同じオペレーションにおいてその他の書き込みを拒否するようなセキュリティ ルールは作成不可。
* https://firebase.google.com/docs/firestore/security/rules-fields?hl=ja#partial_writes_are_never_allowed

# セキュリティルールはフィルタではない
* 例えば、`allow read: if resource.data.visibility == 'public';`　は　publicのものだけを抽出して返すわけではない。
* リクエスト内容として、publicなものだけを取得するクエリになっている必要がある。
* したがってリクエスト結果に1件でもpublicではないものが含まれているとリクエストは拒否される。
* 取得対象が複数件の場合、すべてのドキュメントの実際のフィールドの値ではなく、結果セットの可能性に対してクエリを評価。一方、ドキュメントIDを使用して単一のドキュメントを取得する場合、セキュリティ ルールと実際のドキュメント プロパティを使用してリクエストを評価。
* リクエストが成功するためには、セキュリティルールの条件がかならずtrueになる制約をクエリの条件に含める必要がある。（データの内容によって真偽が変わる場合は失敗する）
    * Cloud Firestore がセキュリティ ルールを適用するときには、データベース内のドキュメントの実際のプロパティに対してではなく、結果セットの「可能性」に対してクエリを評価する。したがってクエリの制約は、セキュリティルールの制約と一致させる必要がある。
    * https://firebase.google.com/docs/firestore/security/rules-query?hl=ja#queries_and_security_rules
* 例
    * `allow read, write: if request.auth != null && request.auth.uid == resource.data.author;`
    * NG
        * `db.collection("stories").get() `
        * たとえすべてのauthorが条件にマッチしたとしても、実際のドキュメントではなく含まれる可能性をチェックするため。
    * OK
        * `db.collection("stories").where("author", "==", user.uid).get()`

# 制限
* https://firebase.google.com/docs/firestore/security/rules-structure?hl=ja#security_rule_limits
* https://firebase.google.com/docs/firestore/quotas?hl=ja#security_rules
* リクエストあたりの exists()、get()、getAfter() 呼び出しの最大数
    * 単一ドキュメントに対するリクエストとクエリ リクエストの場合は 10
    * 複数のドキュメントに対する読み取り、トランザクション、一括書き込みの場合は 20。
        * 各オペレーションには、前述の上限（10）も適用
    * キャッシュが使用された場合はカウントされない
* リクエストあたり評価される式の最大数	1,000
* 関数引数の最大数	7
* 関数呼び出しの深さの最大数	20
* ルールセットの最大サイズ
    * テキストソースのサイズは 256 KB 
    * コンパイル済みルールセットのサイズは 250 KB
* その他制限はリンク参照。

# コレクショングループのクエリに関するセキュリティグループ(具体例)
* https://firebase.google.com/docs/firestore/security/rules-query?hl=ja#collection_group_queries_and_security_rules


# アクセスの安全性
* 開発中にオープンアクセスを許可するようにルールを設定している場合、デモプロジェクト ID を推測してデータにアクセスし、データを窃取、変更、削除することが可能であることは留意したほうが良い。
    * https://firebase.google.com/docs/firestore/security/insecure-rules?hl=ja#open_access
* クローズドアクセス
    * https://firebase.google.com/docs/firestore/security/insecure-rules?hl=ja#closed_access
    * Firebase Admin SDK と組み合わせ、サーバーのみのバックエンドとして Cloud Firestore を使用する場合、セキュリティルール上ではすべて拒否しておいて、クライアントからのアクセスはサーバー経由にする。

# セキュリティルールのテスト
* https://firebase.google.com/docs/firestore/security/test-rules-emulator?hl=ja

# カスタムクレーム
* Firebase Admin SDK では、ユーザー アカウントのカスタム属性の定義がサポートされている。
* これをつかうことでセキュリティルールでユーザーの役割ごとにアクセス制御などができる。
* カスタム クレームは、アクセス制御を提供するためだけに使用される。
* 追加のデータ（プロファイルやその他のカスタムデータなど）を格納するようには設計されていないので注意。
* https://firebase.google.com/docs/auth/admin/custom-claims?hl=ja

