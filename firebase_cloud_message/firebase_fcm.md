- [このメモ・独自表記について](../README.md)

# 概要
## Firebase Cloud Message
* https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ja
* FCM SDK
    * 次のサービスを介して各端末へプッシュ通知を送信
    * Android: Android Transport Layer (ATL)
    * Apple Push Notification service (APNs)
    * Web Pushプロトコル
* Admin SDK 
* Notifications Composer 
    * テスト、マーケティング等で利用
    * コンソールから送信可能
## FCM登録トークン(デバイストークン)
* https://firebase.google.com/docs/cloud-messaging/fcm-architecture?hl=ja
* メッセージを送信する先のデバイス・アプリを特定するためのトークン。
* 一つのデバイスで複数のトークンが存在(新規発行するたびに新しいものが生成)するが、あるトークンが指すデバイスとアプリの組み合わせは一意となる。
* 参考
    * https://fintan-contents.github.io/mobile-app-crib-notes/react-native/santoku/application-architecture/push-notification/overview/


# メッセージの種類と構成
* https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ja
* 通知メッセージ（表示メッセージ）
    * FCM SDKによって自動的に処理される。
    * アプリがバックグラウンドで実行されているときに、FCM SDK で通知の表示を自動的に処理する場合は、通知メッセージを使用。
    * カスタムの Key-Value ペアのデータ ペイロードを任意に指定できる。
        * データメッセージとの使い分けはどのようになるのか?
    * データペイロードがあると、アプリの実行状態（フォアグラウンド、バックグラウンド）に応じて処理が変わる。
        * バックグラウンドにある場合
            * アプリは、通知トレイで通知ペイロードを受け取り、ユーザーが通知をタップしたときにのみデータ ペイロードを処理
        * フォアグラウンドにある場合
            * アプリは、どちらのペイロードも取得できる状態でメッセージ オブジェクトを受け取る。
* データメッセージ
    * クライアント アプリがデータ メッセージの処理を行う。
* メッセージ
    * 共通のフィールドセット。
    * プラットフォーム固有のフィールド
        * 特定のプラットフォームだけに値を送信する場合は、共通フィールドを使用せずにプラットフォーム固有のフィールドを使用
    * 折りたたみ可能かどうか
        * 通知メッセージを除き、デフォルトではすべてのメッセージが折りたたみできない。
        * collapseKey（Android の場合）、apns-collapse-id（Apple の場合）
   * 優先度
       * 優先度: 標準、優先度: 高がある。
        * Apple デバイスにデータメッセージを送信する場合は、優先度を 5（標準の優先度）に設定する必要がある。（高優先度にすると拒否されてエラーになる）
    * 有効期間の設定
        * FCM では通常、メッセージが送信されるとすぐに配信。
        * しかし、デバイスが電源オフまたはオフラインあるいは利用できない状態になっている場合はすぐに配信はできない。
        * アプリによる過剰なリソースの消費やバッテリー寿命への悪影響を防ぐために FCM が意図的にメッセージの配信を遅らせることがある。
        * その場合、FCM はメッセージを保存しておき、配信に適した状態になり次第メッセージを配信する。
        * time_to_live の値を 0 にすると、直ちに配信できないメッセージは破棄される
   *  スロットル処理
        * ロットル処理は、ユーザーの電池への影響を抑えるために厳重に行われる。


# APIエンドポイント
* https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages/send?hl=ja
* https://firebase.google.com/docs/reference/fcm/rest

# サーバー側からメッセージを送信する
* FCM サーバーとの対話は、FCM HTTP v1 APIにて行う。
* Firebase Admin SDK（Node、Java、Python、C#、G ） はこのプロトコルに基づいている。
* https://firebase.google.com/docs/cloud-messaging/server?hl=ja

# FCMの制限
* https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ja#throttling-and-scaling
* https://firebase.google.com/docs/cloud-messaging/auth-server?hl=ja#authorize-an-xmpp-connection
* Android の場合、1 台のデバイスに送信できるメッセージは、1 分あたり最大 240 件、1 時間あたり最大 5,000 件
* iOS の場合、レートが APNs の制限を超えるとエラー
* 「XMPP サーバーのスロットル処理」の「FCM は、プロジェクトあたり 2, 500 件の同時接続を許可」と書いてあるが、こちらは旧来の方法のXMPPを使っている場合にXMPPサーバーに接続する際の同時接続の話という理解で良いか?

# FCM登録トークンの管理方法
* https://firebase.google.com/docs/cloud-messaging/manage-tokens?hl=ja
* IMO 要約
    * 基本的に登録トークンが有効なものか判断する完全な方法はない。
    * トークンへメッセージ送信時にエラーが発生する際は明らかに無効のため削除する。
    * 古くなったことをモニタリングして、適宜更新する。推奨は2ヶ月。
    * 初回起動時、アプリをアンインストール/再インストール、アプリのデータを消去、別のデバイスで復元した場合も更新する。


# (IME)登録トークンの挙動
* 以下は筆者がアプリを動作して確認できた挙動となる
* iOS Simulator
    * Simulatrorでもデバイストークンは発行できる。
    * ただし、対象のデバイストークンに対して送信を行う際に失敗する。
    * エラーの際はAdmin SDKから「Request contains an invalid argument」というエラーが返ってきた
* アプリをアンインストールした際の挙動
    * アンインストール前に発行したデバイストークンに対してFCMを送信しても、送信に失敗する。
    * 再インストールしてFCMを送信して場合も同様に送信に失敗する。
    * したがって、「現在インストールされているかどうか」ではなく、一度アプリをアンインストールすると、紐づくデバイストークンは無効になると考えられる。
    * 送信失敗時はAdmin SDKからから「Requested entity was not found」というエラーが返ってきた
* 発行されるデバイストークン
    * (当然かもしれないが)FCMのSDKで取得できるデバイストークンは、現在Firebase authで認証中のユーザーとは無関係にデバイス単位で発行される。
        * したがって、例えば複数のアカウントで同一の端末を利用した際は、同一のデバイストークンがそれぞれのアカウントで利用されることになる。
    * アプリを再インストールすると、発行されるデバイストークンは前とは別のトークンとなる。
        * 前述しているように前のトークンに対しての送信は失敗するため、アンインストールした時点でデバイストークンは無効となると考えられる。



# メッセージ配信の分析（未読）
* https://firebase.google.com/docs/cloud-messaging/understand-delivery?hl=ja&platform=ios
* Firebase コンソールのメッセージ配信レポート
* Firebase Cloud Messaging Data API が提供する集計済みの Android SDK 配信指標
* Google BigQuery への包括的なデータ エクスポート

