[TOP(About this memo))](../README.md) > Firebase Authentication

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



# メールアドレス列挙保護の有効化
* https://cloud.google.com/identity-platform/docs/admin/email-enumeration-protection?hl=ja
* Firebase Authenticationの 設定 > ユーザーアクション にて有効／無効の設定ができる
* なお、以下のように新しいプロジェクトはデフォルトで有効となっている。
    > 2023 年 9 月 15 日以降にプロジェクトを作成した場合、メール列挙保護はデフォルトで有効になっています。
* 上記を有効にした場合、サインイン時のエラーメッセージでパスワードやメールアドレスが間違っていた場合に、それらを特定できるエラーコードではなく、"invalid-credential"という形で返ってくる。
    * コードをプログラム中で参照している場合は、有効化に伴ってプログラムの修正が必要な可能性があるため注意。
* なおFirebaseに関係なく、仕組み上、メールアドレス・パスワードによるサインアップやパスワードリセットに関しては列挙保護を回避することはできない。
    * https://stackoverflow.com/questions/75455503/how-to-protect-against-email-enumeration-on-sign-up
   

# パスワードのセキュリティポリシー
* Firebase Authenticationにはパスワードのセキュリティポリシーを設定する機能が無い。
* Firebase Authenticationはパスワードの条件が6文字以上という条件のみのため、そのまま利用した場合はセキュリティ上の課題が残る
    * https://firebase.google.com/docs/auth/admin/manage-users?hl=ja
* 対策として考えられるもの
    * 呼び出し側でバリデーションを行う
        * パスワードをFirebase の APIへ渡す前にアプリケーション側でバリデーションを行う方法となる。
        * ただしこの方法の場合では、パスワードの再設定メールの送信機能についてはカバーできないため、個別に対応する必要がある。
            * パスワードの再設定メールの送信機能については、アクションURLから独自のサーバー処理へ渡してパスワードの設定処理を行う事で対応可能ではある。
            * しかし、最大のデメリットは独自のサーバーおよびサーバー側の処理の実装が必要となること
    * (未検証)Firebase Authentication on Identity Platform へアップグレードする
        * Firebase Authentication とは 料金体系が異なる点に注意
            * https://firebase.google.com/docs/auth?hl=ja#identity-platform
        * 現時点(24/7/30)で、管理コンソールからパスワードポリシーを設定する方法は無く、SDKを利用した設定方法のみとなる。
            * https://cloud.google.com/identity-platform/docs/password-policy?hl=ja
* 参考
    * https://stackoverflow.com/questions/49183858/is-there-a-way-to-set-a-password-strength-for-firebase



