- [このメモ・独自表記について](../README.md)

# IDトークン と 更新トークン
* Firebase Authenticationの認証時にはIDトークンと更新トークン(refresh token)を受け取る。
* クライアントSDKによってバックグラウンドで 更新トークンを使ってFirebase ID トークンをリフレッシュし続けてセッションが維持される。
* 更新トークン
    * 有効期限が無く、管理はクライアントSDK内で行われ、基本的にアプリケーションコードで操作することはない。
* IDトークン
    * 有効期限は1時間
    * IDトークンはJWTであり、トークン検証の際はFirebaseとの通信が不要。
        * Googleの公開鍵を使って検証される。
        * 検証はSDKによって実行されている
            * 公開鍵は以下のサードパーティで実装する際のドキュメントを読むと、おそらくAdmin SDK内でもキャッシュとして保存していると考えられる。
            * https://firebase.google.com/docs/auth/admin/verify-id-tokens?hl=ja#verify_id_tokens_using_a_third-party_jwt_library
    * IDトークン自体の無効化はできない。
* 参考
    * https://medium.com/google-cloud-jp/firebase-auth-token-jp-d400a113a440、


# 更新トークンの無効化
* https://firebase.google.com/docs/auth/admin/manage-sessions?hl=ja
* 更新トークンの無効化は、ユーザの削除、無効化、大きな変更の検出（パスワードやメールアドレスの更新など）、Admin sdk側で無効化処理をした時 となる。
* 更新トークンが無効化された場合は、クライアントSDKが次回のIDトークン更新ができないため、そこでサインアウト扱いになる。
* したがって仮にadmin SDK側で更新トークンを無効化したとしても、1時間はクライアントSDK側はサインイン状態が維持されIDトークンが必要な操作も可能となる。
* 即時ユーザーのアクセスを遮断するにはバックエンド側の実装が必要。
    * 例
        * Admin SDK側でIDトークンの検証時に更新トークンが無効化されているかどうかの判定をする
        * 無効化されている場合はバックエンドから特定のレスポンスを返してフロントでサインアウトさせる
        * あるいは直接対象ユーザーをブラックリストへ追加してチェックする 等