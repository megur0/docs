- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)

# Apple IDの設定
* Xcodeプロジェクトの設定からApple IDを設定できる

# App ID、バンドルID
* Team IDとBundle IDの2つの文字からなるID。
    * {team_id}.{bundle_id}
* 同じIDは世界中の他の誰も使えない。
    * (参考)
        * Apple DeveloperのサイトでApp ID登録時に既に登録済のIDを登録しようとすると下記のエラーになる。
            * `An App ID with Identifier '(登録したいApp ID)' is not available. Please enter a different string.`
        * Xcodeで既に利用されているバンドルIDを利用しようとすると下記のようなエラーとなる。
            * `Failed to register bundle identifier. The app identifier "(利用したいID)" cannot be registered to your development team because it is not available. Change your bundle identifier to a unique string to try again.`
            * チームにApple IDを設定していない場合は、任意のバンドルIDが使うことが可能。
                * この場合は ID が グローバルな空間から切り離されている?

## 端末内のアプリのID 
* 同じ端末（iphoneやsimulator）には同じApp IDのアプリはインストールできない
    * IME 開発と本番別でインストールして使いたい場合などは、App IDを分ける必要がある。
* 参考
    * https://qiita.com/ko2ic/items/1926de0895e9f277871b

# 実機の確認 
* 実機確認するにはApple IDが必要。
* ADP未加入でも実機確認までは可能。
* push通知やuniversal linksといった機能を利用するにはADP加入が必要。
    * Provisioning ProfileやApp IDを手動でApple Developerサイト上で登録する必要があり、未加入の場合は登録ができない。

# Identifier（App ID, Bundle ID）の登録
* Apple Developerにて 「Identifier」からApp IDを登録ができる。
* Bundle IDは「Explicit」「Wildcard」から選択する。
* Wildcardによって1つ以上のアプリを表すことができる。
    * 例: チームIDが「ABCDE12345」のケース
        * ABCDE12345.com.aaaaaaa.*
        * ABCDE12345.*
            * この場合はチーム配下のすべてのappがマッチする。
    * 複数のアプリに一度に設定を反映させたい時などに便利?


# Certificate（証明書）
* Certificate（証明書）はアプリの開発元を電子的に確認・証明するためのファイル
* 種類
    * Development: ストアへリリースはできない
    * Distribution: デバッグができず、開発機へのインストールもできない。
* CSRファイル（Certificate Signing Requestの略で、「証明書の署名要求」という意味）というものが必要
* 開発するMacで生成したCertSigningRequestをADP(Apple ADP)に登録して、ADP側で発行された証明書(Certificates)をダウンロード
* 自機(Mac)で生成したCSRから取得したCertificatesを開発するMacのキーチェーンに登録することで開発者本人の本人性を証明している
    * 証明書と紐付いているものはMac実機とADPメンバーシップ（Team ID）のみという理解で良いか?
        * 個人のApple IDは紐付いていない?
* 開発用(Development)と配布用(Distribution)の2種類
    * 開発用：検証用の実機でビルドする際に開発元の本人性を証明する
    * 配布用：App Storeで配布 or Ad-Hocで組織内部に配布する際に開発元の本人性を証明する
* 参考
    * https://zenn.dev/mhackit/scraps/355fe56dc7b4c8
    * https://www.musen-connect.co.jp/blog/course/product/ios-ble-dev-provisioning-profile/
## Certificate（証明書）の作成と削除
* 証明書は以前より簡単に作成できるようになった
    * いくつかネット上のサイトを調べると以前は以下のような手順で行なっていたと考えられる
        * Keychain Access.app から Certificate Signing Request (CSR) を作成
        * Apple Developer サイトに行ってCSR登録、証明書(.cer)作成
        * 作成した証明書をダウンロードして Mac の Keychain にインストール
        * 参考
            * https://i-app-tec.com/ios/apply-application.html
    * 現在は、XcodeのPreference から作成できる。
        * Accounts > Apple IDsで対象のアカウントを選択（登録してなければ、「+」から追加）
        * 右下で、対象のチームを選択して「Manage Certificates」>「+」>「Apple Development」にて作成できる。
        * 参考
            * https://xyk.hatenablog.com/entry/2020/10/29/171732
* 仕様上、同じ証明書が２つ存在するとエラーとなる?
    * 既に証明書を作成済みの状態でdevelopの証明書をXcodeで作成すると、現在のプロジェクトのProvisioning profile上でエラーが発生した。なお削除するとエラーは消えた。
        * `Provisioning profile "XXXXXX" doesn't include signing certificate YYYY`
* 証明書の無効化・削除の方法
    * apple developerのサイトから「Revoke」を行う。
        * Preference > Account　からはこの操作はできない。（クリックしてもdelete メニューが非アクティブとなっている）
    * Revoke済みの証明書をローカル上でも削除する
        * アプリケーション>ユーティリティ>キーチェーンアクセスを開き、ログイン > 証明書からRevokeした証明書を削除する
        * 削除後はXcodeを再起動するとPreference > Accountからも消える。
    * 参考
        * https://qiita.com/warapuri/items/2a32cb2201ce75aa5f4b

# Provisioning Profile
* 作成したアプリに対してアプリ開発者が署名するために必要なデータ
* 以下が紐づいている
    * App ID（バンドルIDと紐づくもの）
    * 証明書
    * Team
    * 端末（DevelopmentやAd-Hocの際のみ。Ad-Hocではインストール可能な端末を100個まで指定可能？）
* Xcodeで「Provisioning Profile」の「i」を押すと内容を確認できる
* Provisioning Profileを開発するXcodeプロジェクトの中で登録(設定)し、登録されてる開発者本人が、許可されたアプリ(許可されたApp ID)の開発において許可された端末(Devices)を向き先としたビルドであれば、ビルドしたipaファイルの実機へのインストールが許可される仕組み。
* したがって実機で確認する場合は、Provisioning Profileがプロジェクトに署名されていないとエラーになる。
    * 参考
        * https://qiita.com/kenichiro-yamato/items/8ed35bfa5072fa8028c4
    * なお、シミュレーターで確認する場合は署名されていなくても確認が可能
* 参考
    * https://hirokuma.blog/?p=2783
    * https://tanihiro.hatenablog.com/entry/2016/07/26/194944
## Provisioning Profileの生成・設定
* Apple IDが必要となる。
* Provisioning Profileは、以下のいずれかの方法で取得したものを使う。
    * 自分でApple Developerサイトで作成したものをダウンロード（import）して使う 
    * 「Automatically manage signing」でローカル上で生成したものを使う
        * ADPに登録していない場合でもこの方法となる。

# Automatically manage signing
* Provisioning Profile, App ID, Certificatesを自動で作成・更新する機能。
* 自動的にcertificatesを自身のMac上で生成し、それをApple Developerに登録。
* 自動的にMac上のcertificatesを使ってProvisioning Profileを生成する。
* 期限は1週間になる。
* 対象のIDがApple Developerの「Identifier」に登録されていない場合は、自動で「Identifier」に登録し、それがProvisioning Profileに使われる。
    * このとき、Automatically manage signingの場合は、Bundle IDが *（ワイルドカード）となる。
* Automatically manage signingにおいても、通常のIDと同じように世界で一意なものである必要がある点に注意。
* Provisioning Profileの有効期限が切れればIDも無効となる。
* 参考
    * https://halzoblog.com/apple-developer-app-id/
    * https://zenn.dev/usamaru/articles/725b759d6a9561

# 実機確認
* 実機にインストールするためには、対象のApp ID、対象のデバイス（UDID）、チームに対する開発者の本人確認が必要で、それが含まれるProvisioning Profileが必要。
* ADPに登録していない場合でも実機確認までは可能
    * Automatically manage signing で作成されたProvisioning Profileを利用
    * 既に自動生成されたプロファイルに対して対象の実機を追加するには、Xcode上で実機を接続した状態で対象の実機を選択すると、プロファイルが自動アップデートされる。

# 実機のトラブルシュート
* 実機接続時に発生するエラーメッセージ
    * `codesignは、キーチェーンに含まれるキーへアクセスしようとしています。許可するにはキーチェーン”ログイン”のパスワードを入力してください。`
        * Macのログインパスワードを入力して「常に許可」を押す。
        * 実機が本人性を確認するためにMacのキーチェーン上の証明書を確認しようとしている
    * `Appを検証できません。信用を確認するにはインターネット接続が必要です"  "“iproxy”は、開発元を検証できないため開けません`
        * 証明書を検証するために、認証局（Apple）に接続が必要だが、アプリがインターネット接続を許可されていない
        * iOS側で一般＞VPNとデバイス管理でデベロッパAPPを承認
* 現在のXcodeと、実機のバージョンが合わない。
    * `This operation can fail if the version of the OS on the device is incompatible with the installed version of Xcode.`
    * MacのOS、Xcodeのバージョンを上げる必要がある。


# App Store申請(未整理)
* ADPへの登録が必要
* 申請したいアプリの配布用(Distribute)Provisioning Profileの作成
* App Store Connectでアプリの情報を登録
* 審査へ提出


# 参考
* 公式
    * https://developer.apple.com/jp/help/
* 参考サイト 
    * https://zenn.dev/mhackit/scraps/355fe56dc7b4c8

