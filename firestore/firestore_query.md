- [TOP](./README.md)
- [このメモについて](../README.md)



# 等価演算子（equality operators）
* ==、array-contains
* in、array-contains-any　はORクエリに変換される
    * in
        * (フィールド == 値) OR (フィールド == 値)...に変換される。
    * array-contains-any
        * (フィールド array-contains 値) OR (フィールド array-contains 値)...に変換される。
* 複数の等価演算子を複合（AND）クエリ）で連結する場合は、複合インデックスは、不要。
* 等価（=）または in 句に含まれるフィールドでクエリを並べ替えることはできない。
* array-contains
    * 1 つのクエリで使用できる array-contains 句は 1 つだけ
    * 分離（or グループ）ごとに 1 つの array-contains 句を使用できる。同じ分離の中で array-contains と array-contains-any を組み合わせることはできない。
* array-contains-any 
    * 同一フィールドで複数の array-contains 句を論理 OR で結合
    * array-contains-any の結果では重複が排除。（同一のドキュメントに対して、array-contains-anyで指定した値の複数にひっかかっても同一のドキュメントが重複して結果に入ることはない）

# 不等価演算子（inequality operators）、範囲比較
* !=, <, <=, >, >=
* not-inは　複合クエリ(AND)に変換される
    * (フィールド != 値) AND (フィールド != 値)...に変換される。
* 複合クエリ（AND)において、範囲（<、<=、>、>=）と不等値（!=、not-in）の比較は、すべて同一のフィールドに対するフィルタである必要がある。
    * たとえば 
        * 「開始時期 >= 2023-05-01 AND 終了時期 <= 2023-05-31」はできない。
        * 「開始時期 >= 2023-05-01 AND 開始時期 <= 2023-05-31」はできる
* not-in と不等値 != を組み合わせることはできない。
* not-in と in、array-contains-any、or を組み合わせることはできない。
* not-in は最大 10 個までの比較値をサポート
* 不等価（!=）と not-in が使用されるクエリでは、指定されたフィールドが存在しないドキュメントは除外される。
* 値が空の文字列（""）、null、NaN（数値ではない）でも、フィールドは存在するとみなされる。
* not-in 、!= クエリ句は、コレクション内の多数のドキュメントに一致する場合があるので注意。
* 範囲比較（<, <=, >, >=）は、暗黙的にorderByも適用される。
    * これは該当フィールドのorder-by句を加えたクエリと同等
* 範囲比較と order-byと最初のフィールドは、同じフィールドである必要がある。
    * 複数のorder-byの場合は、２つ目以降であればOK？
    * https://firebase.google.com/docs/firestore/query-data/order-limit-data?hl=ja#limitations
* 等価演算子と不等価演算子・範囲比較を複合（AND）クエリで連結する場合は、複合インデックスが必要。

# 論理ORクエリ
* or
    * プレビュー版
    * https://firebase.google.com/docs/firestore/query-data/queries?hl=ja#or_queries
    * (参考)Dart、GoのSDKでは、ORクエリを直接使う機能はサポートされていなかった。
* in, array-contains-any

# 分離句の通常の形式、制限
* https://firebase.google.com/docs/firestore/query-data/queries?hl=ja#limits_on_or_queries
* Cloud Firestore は、2 つのルール（平坦化、分配法）を適用して、クエリを分離句の通常の形式に変換
* Cloud Firestore では、クエリの分離句の通常の形式に基づいて、クエリを最大 30 の分離に制限
* 変換は乗算的であるため、複数の OR グループの AND を複数回実行した場合、上限に達する可能性が高くなるので注意。
    * 例えば、`and(where("a", "in", [1, 2, 3, 4, 5]), where("b", "in", [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]))`　は分離が50になり、エラーとなる。

# その他制限
* クエリ内のフィルタ、並べ替え順指定、親ドキュメント パス（サブコレクションの場合は 1、ルート コレクションの場合は 0）の数の合計は、100 を超えることはできない。

# データの並び替えと制限
* https://firebase.google.com/docs/firestore/query-data/order-limit-data?hl=ja
* デフォルトでは、クエリを満たすすべてのドキュメントが、ドキュメント ID の昇順で取得される。
* order-by
    * 降順、昇順
    * 複数指定
        * `db.collection("cities").orderBy("name").orderBy("state")〜`
        * この場合は複合インデックスが必要。
    * order-by フィールドが存在するドキュメントのみを返す。（order-byで指定したフィールドが存在しないドキュメントは除外される）
* limit

# クエリカーソル
* https://firebase.google.com/docs/firestore/query-data/query-cursors?hl=ja
    * 公式ドキュメントだと初見でカーソルには何を指定できるのか、という点が理解できなかったので以下を参考にした
    * 参考
        * https://mokajima.com/how-to-paginate-data-in-cloud-firestore/
* firestoreではモバイルやWEBのSDKにおいて、クエリでオフセットを指定できない。
* ページネーションのにはカーソルを使う。
* カーソルの設定
    * startAt(): クエリの開始点を指定（開始点を含む）
    * startAfter(): クエリの開始点を指定（開始点を含まない）
    * endAt(): クエリの終了点を指定（終了点を含む）
    * endBefore(): クエリの終了点を指定（終了点を含まない）
* カーソルの引数
    * 上記のメソッドには引数としてドキュメントのフィールドの値  or  ドキュメントスナップショットを渡すことができる。
    * ドキュメントのフィールドの値を指定
        * order-by句が複数ある場合はそれに合わせて複数指定することができる。
    * getでドキュメントスナップショットの配列が取得されるので、それを使う。
* admin SDK
    * offsetを使うことができる。
    * ただ、飛ばした分も課金対象に入ってしまう。
    * 公式ではクエリカーソルを使用することを推奨。
        * https://firebase.google.com/docs/firestore/best-practices?hl=ja#read_and_write_operations
* (参考)Firestoreにおいてページ指定で遷移するページングは難しい？
    * 全てのドキュメントにincrementするフィールドを入れる方法があるが、ドキュメントが削除されたり順番が変わると成立しない。
    * 参考
        * https://mokajima.com/how-to-paginate-data-in-cloud-firestore/

# コレクショングループ クエリ
* デフォルトで、クエリは、データベース内の単一コレクションから結果を取得
* 単一コレクションからではなく、コレクション グループからドキュメントを取得するには、コレクション グループ クエリを使用
* コレクション グループ クエリを使用する前に、コレクション グループ クエリをサポートするインデックスを作成する必要がある
* Web SDK とモバイル SDK の場合は、コレクション グループ クエリを許可するルールも作成する必要がある。
