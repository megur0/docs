[TOP(About this memo))](../README.md) > [一覧](./README.md) >




# 注意:前提
* 筆者はiOSのみの開発のため、情報はiOSのみの情報となっている。

# flutterfire
* flutterfireはFlutterの公式のfirebaseのplugin全体を指す。
    * https://github.com/firebase/flutterfire
* flutterfire_cli
    * fluttefireのCLIツール
    * invertaseというソフトウェア企業が開発しているが、Firebaseの公式でもインストールはこのツールを使っているため、ほぼ公式のツールと考えて良い？
    * https://github.com/invertase/flutterfire_cli

# firebaseの構成ファイル
* Firebase 構成ファイルについて
    * https://firebase.google.com/docs/projects/learn-more?hl=ja#config-files-objects
* firebase_options.dart のGit管理
    * APIキー自体は秘匿情報ではないため非公開のリポジトリであればGit管理に含めることは基本的には問題ない。
    * ただしOSSのようなプロジェクトでは含めることは非推奨となっている。
    * https://github.com/firebase/flutterfire/discussions/7617

# flutterfire_cliのインストール、プロジェクトのconfigure
* https://firebase.google.com/docs/flutter/setup
* https://github.com/invertase/flutterfire_cli
* なお、firebaseのコンソールの「Flutterアプリ追加」を開くと手順が書いてある。以下は初めて実行するときにM1 Macでエラーになった内容も含めて記載している。

* (未インストールの場合)firebase をインストール
    * `npm install -g firebase-tools`
* flutterfire_cliをインストール
    * `dart pub global activate flutterfire_cli`
    * ~/.zshrc等のPATHに、`$HOME/.pub-cache/bin` を追加。
* firebaseプロジェクトをflutterへconfigureする。(以下はiOSのみの場合)
    * `firebase login`
    * `flutterfire configure --project=プロジェクトID --platforms=ios --ios-bundle-id=バンドルID`
    * ios/androidを選択する。
    * (IME) 以下のエラー(RubyでXcodeプロジェクトを編集するためのツール)が発生する場合は、`sudo gem install xcodeproj`を実行
        ```
        〜〜
        kernel_require.rb:54:in `require': cannot load such file -- xcodeproj (LoadError)　`require': cannot load such file -- xcodeproj 
        〜
        ```
        * https://github.com/invertase/flutterfire_cli/issues/127
     
* 上記によって下記のファイルが生成される。
    * ios/Runner/GoogleService-Info.plist
    * ios/firebase_app_id_file.json
    * lib/firebase_options.dart
* 上記によって下記のファイルが更新される
    * ios/Runner.xcodeproj/project.pbxproj
        * GoogleService-Info.plistに関する記述が追加
    * ios/Podfile.lock
        * (IME) 以下のパッケージについて依存するバージョンの記述が追加されていた
        * Firebase 
        * firebase_auth
        * firebase_core
        * FirebaseAuth
        * FirebaseCore
        * FirebaseCoreInternal
        * GoogleUtilities
        * GTMSessionFetcher
        * PromisesObjC
* アプリケーションコードでは下記のように読み込みを行う。
    ```dart
    await Firebase.initializeApp(
        options: DefaultFirebaseOptions.currentPlatform,// 上記で作成した「firebase_options.dart」を参照
    );
    ```
* 上記のoptionsをつけていない状態でコード中でFirebaseの機能を実行しようとするとエラーが発生する。（Firebaseプロジェクトに紐付いていないため。）
    ```
    flutter: [core/no-app] No Firebase App '[DEFAULT]' has been created - call Firebase.initializeApp()
    ```
## Firebaseコンソールに追加されるアプリ
* 上記のconfigureを行うことで、Firebaseプロジェクト上にiOSアプリが存在しない場合は追加される。
* IME
    * コンソール上、デフォルトで非表示になっていたが、Firbaseコンソールのトップの「2個のアプリ」から表示をONにすることができた。
    * アプリのニックネーム、バンドルIDは  適当な名前が設定されている。
    * iOSのApp Store ID、チームIDは空。

# (参考)(IME)Firebaseプロジェクトを入れ替え可能とする。
* 複数のFirebaseのテストプロジェクトを扱う場合は、ローカル環境のFirebaseプロジェクトをスイッチしたい時がある。
* ネット上で検索すると方法1が多く見られた。
* 筆者は方法2を使用した。
## 方法1
* シェルなどを使ってファイルを生成・制御する方法
* Xcode
    * XcodeのBuild PhasesでシェルによってGoogleService-Info.plist、lib/firebase_options.dart、ios/firebase_app_id_file.json の生成を行う。
    * GoogleService-Info.plist
        * 名前を固定にする必要があるため、環境ごとに予めファイル名を分けておいて、ビルド時にシェルでGoogleService-Info.plistファイルを作成する。
    * ios/firebase_app_id_file.json
        * pod installの際にファイル名で参照される。GoogleService-Info.plistと同様にビルド時に生成される。
        * ビルド時に環境別のファイルへ置き換わるよう、シェルでリネームする。
        * 参考
            * https://note.com/rect_angle/n/n7370f15c6a14?magazine_key=m869781f0a74f
* lib/firebase_options.dart
    * Flutter側のコードでdart-defineの値に応じて制御する
## 方法2
* Firebaseプロジェクトを入れ替える際は、都度、FlutterFire CLIで`flutterfire configure 〜`を実行する。
* 以下のファイルはFirebaseプロジェクト依存のため、.gitignoreへ追加する。
    * ios/Runner/GoogleService-Info.plist
    * ios/firebase_app_id_file.json
    * lib/firebase_options.dart
* 注意点
    * 既にファイルが存在する場合、lib/firebase_options.dart、ios/firebase_app_id_file.jsonは上書きするかどうかCLIツールから確認があるが、ios/Runner/GoogleService-Info.plistに関しては特に表示されず、更新が何も行われなかった。
         * したがって、`flutterfire configure 〜`を実行する前に上記のファイルをすべて削除するしておくと良い。（シェルファイル等で削除してから実行するようにすると良いかもしれない）




# トラブルシュート
* Flutterプラグインが依存するFirebaseのSDKのバージョンと、Podsに入っているライブラリのバージョンが不一致
    * ビルドの際に "CocoaPods could not find compatible versions" のエラーが発生する 
    * 以下の２つのバージョンがマッチしていない
        * ios/Podsに入っているFirebaseのライブラリ(iOS用のライブラリ)のバージョン
        * Flutter用のプラグインが依存するFirebaseのライブラリのバージョン
            * これはPodfile.lock内で`firebase_core: 〜〜 :path: ".symlinks/plugins/firebase_core/ios"`の記述が指すフォルダ内にある`firebase_sdk_version.rb`が示すバージョン
    * この場合、対象となる依存先のライブラリのバージョンをアップデート等すると解決する。
    * 例えば、下記のケースは `pod update Firebase/CoreOnly` を実行する事で Firebase/CoreOnlyをアップデートする。
        ```
        Analyzing dependencies
        firebase_auth: Using Firebase SDK version '10.7.0' defined in 'firebase_core'
        firebase_core: Using Firebase SDK version '10.7.0' defined in 'firebase_core'
        firebase_messaging: Using Firebase SDK version '10.7.0' defined in 'firebase_core'
        [!] CocoaPods could not find compatible versions for pod "Firebase/CoreOnly":
        In snapshot (Podfile.lock):
            Firebase/CoreOnly (= 10.3.0)
        In Podfile:
            firebase_core (from `.symlinks/plugins/firebase_core/ios`) was resolved to 2.10.0, which depends on
            Firebase/CoreOnly (= 10.7.0)
        You have either:
        * changed the constraints of dependency `Firebase/CoreOnly` inside your development pod `firebase_core`.
        You should run `pod update Firebase/CoreOnly` to apply changes you've made.
        ```