[TOP(About this memo))](../README.md) > [一覧(Firestore)](./README.md) > データの操作



# データの追加・更新
* https://firebase.google.com/docs/firestore/manage-data/add-data?hl=ja
* set() を使用してドキュメントを作成する場合、作成するドキュメントの ID を指定する 
    ```
    db.collection("cities").doc("new-city-id").set({"name": "Chicago"});
    ```
* add()メソッドの場合は、IDが自動生成される。
    ```
    db.collection("cities").add(data);
    ```
* なおバックグラウンドでは .add(...) と .doc().set(...) は完全に同等
* update
    * 対象のフィールドのみ更新する
    * フィールドがネストされていると、setのmerge:trueと挙動が異なる点に注意。
        * 参考 
            * https://tomokazu-kozuma.com/difference-between-set-and-update-when-updating-cloud-firestore-data/
* タイムスタンプの取得
    ```
    FieldValue.serverTimestamp()
    ```
* 配列
    * https://firebase.google.com/docs/firestore/manage-data/add-data?hl=ja#update_elements_in_an_array
    ```
    FieldValue.arrayUnion(["greater_virginia"])
    FieldValue.arrayRemove(["east_coast"])
    ```
* increment
    ```
    FieldValue.increment(50)
    ```
    * 単純な足し算なら、incrementを使えばトランザクションを使わなくて済む。
* カスタム オブジェクト

# 削除
* コレクション全体またはサブコレクションを削除するには、コレクションまたはサブコレクション内のすべてのドキュメントを取得して削除
* 大きなコレクションがある場合は、メモリ不足エラーを避けるため、小さなバッチに分けてドキュメントを削除することをおすすめ
* コレクションを削除するには、数に制約のない個別の削除リクエストを用意する必要がある。コレクション全体を削除する必要がある場合は、信頼できるサーバー環境からのみ実行。
    * モバイル / ウェブ クライアントからコレクションを削除することも可能だが、その場合セキュリティとパフォーマンスに悪影響を与える。
* 効率的なデータ削除（未読）
    * https://firebase.google.com/docs/firestore/solutions/delete-collections?hl=ja

# データの取得
* get() を使用して単一のドキュメントの内容を取得
    * `db.collection("cities").doc("SF").get()`
    * `db.collection("cities").doc("SF")`の参照にドキュメントがない場合、結果の document は空になり、existsを呼び出すとfalseになる。
* デフォルトでは、get を呼び出すと、データベースから最新のドキュメント スナップショットを取得
* オフライン サポートがあるプラットフォームでは、ネットワークが利用できない場合やリクエストがタイムアウトした場合、クライアント ライブラリはオフライン キャッシュを使用する。
* get() を呼び出す際に source オプションを指定すると、デフォルトの動作を変更できる。
    * https://firebase.google.com/docs/firestore/query-data/get-data?hl=ja#source_options
* カスタム オブジェクトを使って取得結果をマッピングできる。
* クエリを指定することで、1 回のリクエストで複数のドキュメントをget()で取得することができる。
    * where()がなければ、コレクション内のすべてのドキュメントを取得する。
* 特定のフィールドのみを取得する方法はない。
    * https://firebase.google.com/docs/firestore/security/rules-fields?hl=ja#allowing_read_access_only_for_specific_fields
    * ドキュメントは全部取得するか、取得しないか、となる。
    * したがって、部分的にドキュメントを取得する方法はない。
    * 特定のフィールド抽出したい場合は、サブコレクションとして別のコレクションにする。