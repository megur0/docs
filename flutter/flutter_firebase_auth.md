[TOP(About this memo))](../README.md) > [一覧](./README.md) >

# Admin SDKの変更はストリームで検知できない
* https://firebase.google.com/docs/auth/flutter/start?hl=ja
    > Firebase Admin SDK で User プロファイルを更新しても、idTokenChanges()、userChanges()、authStateChanges() は呼び出されません。最新の User プロファイルを取得するには、FirebaseAuth.instance.currentUser.reload() を使用して強制的に再読み込みする必要があります。
    Firebase Admin SDK や Firebase コンソールで User を無効にしたり削除しても、idTokenChanges()、userChanges()、authStateChanges() は呼び出されません。FirebaseAuth.instance.currentUser.reload() を使用して強制的に再読み込みする必要があります。これにより、アプリコードで検出して処理できる user-disabled または user-not-found 例外が発生します。
* なお、IDトークンの有効期限が切れた後はストリーム処理において再取得が行われるため、その際にはじめて検知することができる。
    * その際にエラーとなるのか、サインアウト状態となるのかは未検証

# メールアドレスのVerificationはストリームで検知できない
* メールアドレスのVerification(※)は、ストリームで検知することはできない。
    * ※ createUserWithEmailAndPasswordやswitchToPermanentFromAnonymousやverifyBeforeUpdateEmailによって送信されたメール内のリンクを押下して検証を完了させるもの
* currentUser().emailVerified が検証済みか否かの状態を表すが、この値を更新するにはreloadを行う、もしくはIDトークンの有効期限が切れて再取得が行われるまで待つ必要がある。
* なお、verifyBeforeUpdateEmailの場合はさらに以下の注意点がある。
    * 再度サインインをしない限り、currentUser()そのものが前のユーザーのままであること。
    * メールアドレスが変わった事によってユーザーの認証状態が無効となるため、reloadを行うと下記のエラーが発生する。(reloadを利用する場合は適切にハンドリングする必要がある)
         * エラー内容(firebase_auth 4.20.0の場合)
            >  [firebase_auth/user-token-expired] The user's credential is no longer valid. The user must sign in again.
    * メールアドレス変更後、特に再サインインをしないまま時間が経過すると、次回のIDトークン再取得時にサインアウト状態となる。（エラーは発生しない）
* 参考: [firebase_auth] userChanges() doesn't trigger with the verifyBeforeUpdateEmail() method
    * https://github.com/firebase/flutterfire/issues/11081
* 参考: updateEmailのdeprecate
    * https://pub.dev/packages/firebase_auth/changelog#4170
    * updateEmailは4.17.0にてDeprecatedとなっている。
    * このAPIのverifyBeforeUpdateEmailとの違いは以下となる。
        * 実行されるとemailVerified が falseとなる。
        * 変更後のメールアドレスへの検証メールは送信されない。(したがって、検証するためには別途sendEmailVerificationを送信する必要がある)
        * updateEmailは変更前のメールアドレスへのrevoke用メールを送信する。(verifyBeforeUpdateEmailは送信しない)
    * 24/7/30時点で、下記のドキュメントはupdateEmailが掲載されている。
        * https://firebase.google.com/docs/auth/flutter/manage-users?hl=ja#set_a_users_email_address

# アカウント生成時にエンドユーザーがメールアドレスを誤ってしまった場合
* 具体的には、createUserWithEmailAndPasswordやswitchToPermanentFromAnonymousの際に、ユーザーがタイポ等で(メールアドレスとしての形式は正しいが)誤ったメールアドレスにてアカウント作成してしまうケースとなる。
    * なお、これらのAPIはメールアドレスの形式チェックは行われる(不正な場合はコードとして"invalid-email"を返す)が、メールを送信できたかどうかはエラー情報として返されることはない。
* 筆者が確認した限り、このケースに対してユーザー操作のみで解決可能な直接的なAPIは無いと考えられる。
    * verifyBeforeUpdateEmailにて別のメールアドレスに対して、メールを再送信することは可能ではある。
    * ただし、verifyBeforeUpdateEmailはメールアドレスの変更用のAPIであり、(検証後に現在の認証状態が無効になる等)この目的での利用には適切ではないと考えられる。
    * また、前のメールアドレスが検証されていないにもかかわらず、メールの再送信が今後も可能ではあるという保証がない。
* (IMO)考えられる対応策
    * Admin SDKで対処する。
        * 例えば、エンドユーザーから問い合わせがあった場合にAdmin SDKを利用したバックオフィス側のシステムで処理する 等
    * メールアドレスを2度入力するといった方法で入力誤りを抑止する。
* 参考
    * https://stackoverflow.com/questions/68801390/how-to-allow-users-to-change-wrong-email-address-in-firebase-auth