- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# 参考サイト
* https://engineering.mercari.com/blog/entry/20220224-deeplink-for-mercari-shops/

# ディープリンクとは?
* ディープリンクとは、モバイルアプリの特定の機能に直接アクセスできるURL
    * 「ディープリンク」という用語はもともと、Webサイトのトップではなく特定のコンテンツのページに直接遷移するURLのことを指す意味
* ディープリンクを使用すると、Webページからアプリを起動したり、アプリから別のアプリを起動することができる。

# ディープリンク実現の仕組み
* iOS
    * iOSでディープリンクを実現するために最初に提供された機能はCustom URL Schemes
    * iOS9からUniversalLinksという機能が提供
* Android
    * version6からAppLinksというUniversalLinks同等の機能を提供する仕組みがある。
* Smart App Banner
    * アプリに対応するWebページの上部にアプリを起動するためのバナーを表示する仕組みで、Appleが提供。

# Custom URL Schemes
* アプリを起動するために指定する特殊なURLスキーム
* 例
    * instagram://
* アプリがインストールされていない場合、URLは無効となり何も起きない。
* 複数のアプリが同一のCustom URL Schemesを定義することができるという問題がある。
    * 複数存在する場合、OSやバージョンによって挙動が異なる。

# UniversalLinks
* UniversalLinksは通常のWebページのURLをディープリンクとして扱い、アプリを起動できるようになる仕組み
* Custom URL Schemesで起きていたディープリンクの重複の問題は解決
* PC で アクセスした場合やアプリがインストールされていない場合はそのURLのWebページに遷移し、アプリがインストールされている場合はアプリを起動して指定の画面に遷移。
* 実装
    * アプリ側でディープリンクとして扱いたいURLのドメインを指定
    * そのドメインの.well-known/以下に apple-app-site-associationというファイルを設置
    * ファイルの中にディープリンクとして扱うパスと起動するアプリの組み合わせを記述
    * apple-app-site-associationはアプリの初回起動時やアップデート時などにAppleのCDNを通して自動的にデバイスにダウンロードされる。
* しかし、下記のようなデメリットがあり通常はCustom URL Schemesと組み合わせて使用する。
    * アプリの各機能に対応するWebページが必要
        * UniversalLinksに対応していないページからUniversalLinksに対応したURLのリンクを設置しても、ページ遷移するだけでディープリンクとしては動作しない。
            * Webサイト内にアプリを起動したいリンクを設置することが難しい。
    * 確実にアプリを起動する機能ではない
        * UniveralLinksのリンクを長押ししてSafariで開く動作をすると、アプリでなくSafariでページ遷移し、以後そのリンクはディープリンクとして機能しないようデバイス内に保存されてしまう。
            * これを解除するには、開いた先のページ上部のバナーからアプリを起動するか、再度リンクを長押ししてアプリで起動するを選択する。
            * ユーザーにとっては便利な仕組みだが、アプリ側からはURLがディープリンクとして動作するのかブラウザで開くのか制御できない。
        * ブラウザによってはディープリンクが動作しない。
        * アプリの初回起動時にapple-app-site-associationファイルのダウンロードに失敗すると動作しない。
            * AppleのCDNを通すため、開発環境にアクセス制限などを設けていると失敗する。
            * 回避方法があるが、開発モードでのビルドかつデバッグモードになっている実機にインストールした時のみ有効。


# Custom URL Schemes と UniversalLinksの 設定
* 参考
    * https://rnfirebase.io/dynamic-links/usage
    * https://firebase.google.com/docs/dynamic-links/flutter/receive
    * https://dev.classmethod.jp/articles/ios-custom-url-scheme/
    * https://note.com/ymmtshny/n/n13504f26308c
    * https://qiita.com/yosshi4486/items/521210701fb5feaf3808
* UniversalLinksは、Associated Domainsを有効にする必要があるため、ADP加入が必要。
## apple-app-site-associationのホスティング
* UniversalLinks によって紐づけるドメインが提供するホスト環境へ apple-app-site-association を設置する必要がある。
    * ルートの.well-known配下へ「apple-app-site-association」というファイルを設置する
    * ファイル内容の例は以下。
    ```
    {"applinks":{"apps":[],"details":[{"appID":"チームID.xxx.xxx.example.com","paths":["NOT /_/*","/*"]}]}}
    ```
    * `ttps://対象のドメイン/apple-app-site-association` でアクセスできれば成功
* IME
    * apple-app-site-associationの探索はApple CDNのキャッシュが効いている?
        * ユニバーサルリンクを押下した際に動作端末（シミュレーターや実機）はApple CDNを通してapple-app-site-associationを探しに行く。
        * この時に失敗するとおそらくキャッシュが働き、その後apple-app-site-associationをアップロードしてもすぐには成功せず、少しタイムラグが発生した。
## (参考)apple-app-site-associationのホスティングにFirebase Hostingを利用する場合
* 参考
    * https://docs.flutter.dev/cookbook/navigation/set-up-universal-links
* Firebase Hostingでは、apple-app-site-association をデフォルトでホスティングする。
* 任意のファイルをアップロードして「https://firebaseのプロジェクトID.web.app/」が有効になると自動的に生成される。
* ただし、プロジェクトの設定 > 全般 > Appleアプリ で 以下を登録しておく必要があり、登録していない場合は空の内容 `{"applinks":{"apps":[],"details":[]}}` が表示される。
    * Team ID
    * Bundle ID
    * Apple Store ID
        * こちらは開発中であれば適当なIDで良い。
* firebase hostingを「はじめる」のみでは「https://firebaseのプロジェクトID.web.app/」は有効にならないので注意。
    * なお、下記のFlutterの手順の場合はfirebaseが自動的に生成することは書かれておらず、手動でapple-app-site-associationをアップロードしている。
    * https://docs.flutter.dev/cookbook/navigation/set-up-universal-links
## Xcodeの設定
* 手動で設定する場合
    * 「Signing & Capabilities」>「＋ Capability」>「Associated Domains」を追加
        * Domainsを設定
            * 「applinks:xxxxx.xxx.xxx」のように入力
    * カスタムURLスキームを追加
        * [Info]タブから、URL タイプをプロジェクトに追加。[URL Schemes] フィールドに設定（自由なIDを設定可能。）
* ファイルで設定
    * Associated Domains
        * ios/Runner/Runner.entitlements　に「com.apple.developer.associated-domains」を設定する。
        ```
        <plist version="1.0">
        <dict>
            〜〜〜〜
            <key>com.apple.developer.associated-domains</key>
            <array>
                <string>applinks:xxxxx.xxx.xxx</string>
            </array>
        </dict>
        </plist>
        ```
    * カスタムURLスキーム
        * ios/Runner/Info.plistに設定する
        ```
        <plist version="1.0">
        <dict>
        〜〜〜〜
        <key>CFBundleURLTypes</key>
        <array>
            <dict>
                <key>CFBundleTypeRole</key>
                <string>Editor</string>
                <key>CFBundleURLName</key>
                <string>カスタムURL</string>
                <key>CFBundleURLSchemes</key>
                <array>
                    <string>カスタムURL</string>
                </array>
            </dict>
	    </array>
        </dict>
        </plist>
        ```
## Apple developerの設定
* identifierの設定のAssociated Domainsにチェックを入れる
* 現在使っているProvisioning Profileは無効になるので注意。
* 再度Profileを開き「save」することで変更を反映して有効化する
* ローカルのXcode上で再度ダウンロードする。


# 動作確認
* シミュレーター
    * UniversalLinksは実機でのみ動作する。
        * シミュレーターで押下しても、ブラウザが開いてしまう。
        * (?)ただ、WEBの記事ではapple-app-site-associationがアップロードされていればシミュレーターでも機能する、という記載が確認できる。
            * 現時点で、筆者の手元の環境では機能していない状況。
            * https://codewithandrea.com/articles/flutter-deep-links/#how-to-test-deep-links-on-ios
    * Custom URL Schemesは  "customURL://hoge/fuga/" というURLからアクセスすることでアクセス可能。
        * (例)
            * ブラウザのURLへ貼り付けて開く
            * リマイダー等に貼り付けてクリックして開く
            * コマンドから開く
                * xcrun simctl openurl booted "customURL://hoge/fuga/"
* 実機
    * UniversalLinks、Custom URL Schemesの両方を確認可能












