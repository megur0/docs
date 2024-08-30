[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > 環境構築・アップデート


# 注意
* 筆者はMac環境で「XCode」「VSCode」にてiOSアプリをFlutterで開発しているため、本メモは以下の内容については触れていない。
    * Android環境
    * Android Studio
    * Windows
* 内容もMac, VSCodeを前提としている、

# インストール
* 公式の手順に従う
    * https://docs.flutter.dev/get-started/install/macos
* MacのApple Silicon(M1/M2)の場合、Rosetta 2が必要となる
    * SDKに同梱されているバイナリの一部はx64のため。
* SDK内にdartのバイナリも含まれている。
    * したがってFlutterをインストールするとDartも利用可能となる。
* 必要に応じてXcodeやAndroid Studioのインストールを行う。
* CocoaPods
    * iOSアプリを開発する場合は基本的にCocoaPodsが必要となる。
    * `brew install cocoapods` によってインストールできる。
* インストール状態の確認
    * `flutter doctor`
    * 各種ツールチェインが正常にインストールされていることを確認
## (参考)Flutter SDK と dart-sdk
* 参考
    * https://qiita.com/kurun_pan/items/520d91b4f5da6f14345b
### Flutter SDK は Dartで書かれている
* Flutter SDKが提供する各種コマンドは基本的に Dart で書かれている。
    * Flutter SDKは、Dart VM (dart-sdk) 上で動作する。
    * Flutter SDKの本体
        * https://github.com/flutter/flutter 
* 初回のflutterコマンド実行時にDart-sdkがダウンロードされる。
    * Flutter SDK を git clone 後、何らかのFlutterコマンドを初めて実行した時に最初は単なるシェルスクリプトで動き始め、すぐに dart-sdk を${install先}/flutter/bin/cache/dart-sdkにダウンロードする。
    * dart-sdk は、Dartのソースコードをビルドしてその環境で実行するための環境
    * ダウンロードするデータは dart-sdk のビルド結果そのもの
* dart-sdkダウンロード後、Flutter SDK (flutter_tools) の dart コードをビルド
    * その結果は${install先}/flutter/bin/cache/flutter_tools.snapshotに置かれる。
    * 仮に Flutter SDK (flutter_tools) をローカルで修正し、それを動きに反映させるためには、${install先}/flutter/bin/cache/flutter_tools.snapshotを一度物理的に削除する必要がある
* ビルドされたDart SDKはDart VM (dart-sdk) 上で動作する。
    * VM経由のため、バイナリコードで実行する場合はよりは遅い？
### Flutter SDK と dart-sdk の 各種ダウンロード先
* packages
    * Flutterの各種ライブラリ (Framework) 本体
        * ${install先}/packages/flutter
    * Flutter SDKの各種コマンドを実現するスクリプト本体
        * ${install先}/packages/flutter_tools
    * インテグレーションテスト用のスクリプト
        * ${install先}/packages/flutter_driver
    * ユニットテスト用のスクリプト
        * ${install先}/packages/flutter_test
* bin
    * Flutter SDKのビルド結果
        * {install先}/bin/cache/flutter_tools.snapshot
    * Dart SDKのインストール先
        * ${install先}/bin/cache/dart-sdk  
    * Flutter Engine。
        * ${install先}/flutter/bin/cache/artifacts/engine
        * これは事前に各種ターゲットアーキテクチャ (CPU) 毎にクロスビルド（例えばiosとかandroid)された.soファイルとして用意されている。
* `which flutter`
    * 筆者の環境(Mac)では /Users/xxx/flutter/bin/flutter



# Channel
* https://docs.flutter.dev/release/archive
    > The Stable channel contains the most stable Flutter builds
* `flutter channel` で確認
* なお、これはflutterのブランチに対応しているため、`cd ~/flutter; git status` のように実行してもチャンネル(ブランチ)を確認できる。
    * 参考: stableブランチ
        * https://github.com/flutter/flutter/branches/all?query=stable
* チャネルにはstable, beta, dev, masterがある。
    * 参考
        * https://github.com/flutter/flutter/blob/master/docs/releases/Flutter-build-release-channels.md

# VSCode
* VSCodeを未インストールであればVSCodeをインストールする
* 拡張のFlutter extension for VS Codeを入れる
* 推奨されるeditorの設定を行う
    * コマンドパレットで「Dart: Use Recommended Settings」とするとグローバルな設定ファイルへ設定が行われる。
    * https://dartcode.org/docs/recommended-settings/
## 基本的な操作方法や設定
* https://docs.flutter.dev/tools/vs-code
* 新規プロジェクト
    * cmd + shift + p > Flutter: New Project
* 実行
    * エンドポイントのファイルを開いた状態でF5
    * シミュレータによる動作確認
        * 画面右下から選択することができる。
        * シミュレーターを立ち上げておく必要がある
    * launch.jsonで選択したコンフィグを選択して実行したい場合
        * エンドポイントを開いた状態で、左パネルの「Run and Panel」にて選択・実行ができる。
    * (参考)(IME)VSCodeは、Dart VM に接続している
        * Debug Consoleに「Connecting to VM Service at ws://127.0.0.1:57588/XXXXXXXXXXX=/ws」のように表示される。
        * 「ws://127.0.0.1:57588/XXXXXXXXXXX=/ws」は、VSCodeが内部的に起動（flutter run）した Dart VMのウェブソケットのURLとなる。
        * VSCode上のDevtoolsがウェブソケットによって上記URLへ接続し、各種API（ホットリロード等）を実行していると考えられる。
* レイアウトのデバッグ
    * https://docs.flutter.dev/tools/vs-code#debugging-visual-layout-issues
    * アイコンからスローアニメーションの切り替え、ベースラインの表示などを行える。
* Inspector
    * https://docs.flutter.dev/tools/devtools/inspector
* ブレークポイント、ステップ実行 等の操作
    * https://docs.flutter.dev/tools/vs-code#run-app-with-breakpoints
    * 「Run and Debug」パネルからDevtoolsのブレークポイントやログの設定ができる。
    * デバッグで行える操作やブレークポイントの設定等は、VSCodeが共通で提供するデバッグ機能に準拠
        * https://code.visualstudio.com/docs/editor/debugging#_debug-actions
* エディタ
    * hot reload
        * cmd + s もしくは F5
    * Restart
        * shift + cmd + F5
    * コードアシスト・クイックフィクス
        * エディタ上の電球を押下するか、cmd + .
    * デフォルトのスニペット
        * stless
        * stful
        * stanim
    * オーバーライドするメソッド
        * メソッド名を打ち込むとintelliSenseによって候補が出現する。
        * 選択すると@overrideやメソッドが挿入される。
* dart.flutterHotReloadOnSave
    * デフォルトでは手動保存時にホットリロードが実行されるされる。（"manual"）
    * こちらは"never"とすることでオフにすることが可能。
        * https://dartcode.org/docs/settings/#dartflutterhotreloadonsave
        * https://github.com/Dart-Code/Dart-Code/issues/3110#issuecomment-771561427
## (参考)設定ファイル
* .vscode/settings.jsonの例
    * "explicit"の箇所を "always" とするかは好みに依る
    ```
    {
        // フォーマット
        // dart formatが実行される
        "editor.formatOnSave": true,

        // 保存時に実行されるアクション
        "editor.codeActionsOnSave": {
            // dart fix --applyが保存時に自動適用される
            "source.fixAll": "explicit", 

            // organizeImportsを有効化するとDartの場合は不要なImportが削除される。一方、足りていないImportに関しては何も実行されない。
            "source.organizeImports": "explicit"
        },

        // 保存時にホットリロードをしない。
        "dart.flutterHotReloadOnSave": "never",

        // pubspec.yamlが更新された際にpub getを実行しない。
        "dart.runPubGetOnPubspecChanges": "never", 
    }
    ```
* launch.jsonの例
    * 複数の起動方法やオプションの管理は.vscode/launch.jsonで設定する。
    ```
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "dev",
                "request": "launch",
                "type": "dart",
                "flutterMode": "debug",
                "args": [
                    "--dart-define-from-file=dart_defines/dev.json"
                ]
            },
            {
                "name": "profile",
                "request": "launch",
                "type": "dart",
                "flutterMode": "profile",
                "args": [
                    "--dart-define-from-file=dart_defines/dev.json"
                ]
            },
            // ...
        ]
    }
    ```



# Flutterプロジェクトの作成
* 参考
    * https://docs.flutter.dev/get-started/codelab
* 以下のどちらかの方法で作成できる
    * コマンドから作成
        * `flutter create --org パッケージ名 プロジェクト名`
        * --org
            * 省略した場合は`com.example.プロジェクト名`となる。
            * このパッケージ名は後から変更するのは手作業が必要となるため注意。
    * VSCodeのコマンドパレットから（cmd + shift + P）作成
        * Flutter: New Project を実行
        * [Application] を選択し、プロジェクトを作成するフォルダを選択、プロジェクト名を入力して、作成。

# ビルド・動作確認
* 以下のいずれかの方法でビルド・動作確認ができる。
* VSCode上でビルド・実行
    * `flutter pub get`をしていない場合は実行しておく。
    * Run & Debugパネルから実行したい環境を選択してビルド・動作確認をする。
        * アプリケーションコードが書かれたファイルを開いた状態で「F5」を押下することで実行することも可能。
* コマンドからビルド・実行    
    * `flutter devices --machine` で 現在動作可能な端末を確認する
        * 必要に応じて、事前にiOSのシミュレータ等を立ち上げておく。
    * `flutter run -d "(上記コマンドで表示された端末)" --debug` にてビルド・実行



# プロジェクトフォルダの構成概要
* lib/
    * アプリケーションコード
    * 作成したDartコードはここに格納していく。
* test/
    * テストコード
* build/
    * Flutter buildをはじめて実行した際に作成される
    * アプリを各プラットフォームで実行するためのファイルが含まれる
    * プラットフォームごとにサブフォルダが存在する。
* pubspec.yaml
    * https://docs.flutter.dev/tools/pubspec
        > The pubspec file specifies dependencies that the project requires, such as particular packages (and their versions), fonts, or image files. It also specifies other requirements, such as dependencies on developer packages (like testing or mocking packages), or particular constraints on the version of the Flutter SDK.
    * https://dart.dev/tools/pub/versioning
        * Dartはセマンティックバージョニングに基づく
        * https://dart.dev/tools/pub/versioning#semantic-versions
    * 現在のバージョン、依存関係、同梱するアセットなど、アプリの基本情報を設定するYAML形式のファイル
    * 新しい Flutter プロジェクトを作成すると、基本的な pubspec が生成される。
    * environment
        * sdk: 対象のdart-sdkのバージョン。
        * flutter: Flutter (SDK) のバージョン
        * flutter create --template=app の場合は 初期状態でsdkのみが指定されている。
    * version
        * アプリのバージョン・ビルド番号の管理
            * (例) version: 1.0.0+1
            * この場合ビルド番号は"+1"の部分となる。
        * バージョン・ビルド番号の管理はpubspec.yamlにて行う。
            * pubspec.yaml のversionを元に、flutter build コマンド実行時に各OSのネイティブファイルも更新される。
            * バージョン・ビルド番号がネイティブ側でどの項目に対応するかはプラットフォームによって異なる。
            * 例えば「version: 1.0.0+1」の場合、ビルド時に下記のように上書きされる。
                * iOS
                    * ios/Flutter/Generated.xcconfig
                        ```
                        FLUTTER_BUILD_NAME=1.0.0
                        FLUTTER_BUILD_NUMBER=1
                        ```  
                    * ios/Flutter/flutter_export_environment.sh
                        ```
                        export "FLUTTER_BUILD_NAME=1.0.0"
                        export "FLUTTER_BUILD_NUMBER=1"
                        ```
                    * build/ios/archive/Runner.xcarchive/Info.plist や　build/ios/Debug-iphoneos/Runner.app.dSYM/Contents/Info.plist
                        ```
                        <key>CFBundleShortVersionString</key>
                        <string>1.0.0</string>
                        <key>CFBundleVersion</key>
                        <string>1</string>
                        ```
                    * iOSのCFBundleVersionは[メジャー].[マイナー].[パッチ]の形式だが、上記のように指定すると1.0.0と解釈される。
                        * https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleversion#discussion
                * Android
                    * android/app/build.gradle
                        ```
                        def flutterVersionCode = localProperties.getProperty('flutter.versionCode')
                        //...

                        def flutterVersionName = localProperties.getProperty('flutter.versionName')
                        //...
                        defaultConfig {
                            // ....
                            versionCode flutterVersionCode.toInteger()
                            versionName flutterVersionName
                        }
                        ```
        * ビルド時のオプションで上書きができる。
            * `flutter build ios --build-name=0.1.2 --build-number=5` 
    * パッケージのバージョン
        * some_package: 1.2.2
            * 1.2.2のみ許容
        * some_package: ">=1.2.2 <1.3.0"
        * some_package: ">=1.2.2"
        * some_package: "<1.3.0"
        * some_package: ^1.2.2
            * 指定されたバージョン以上、かつ メジャーバージョンが同じであれば許容
            * ">=1.2.2 <2.0.0" と同じ
        * some_package: any
            * バージョン指定なし
            * 空白も同じ意味と成る
* プラグイン関連
    * パッケージにおいて、プラットフォーム固有の機能をFlutterアプリに提供する（例えばデバイスのカメラ機能を使う場合等）ものを特別に「プラグイン（Plugin）」と呼ぶ。
    * プラグインでは以下のファイルが自動生成、更新される
        * .flutter-plugins
        * .flutter-plugins-dependencies
        * 各プラットフォームごとのファイル（パッケージ管理ファイル、プラグイン本体）
* 各プラットフォームごとのビルドファイル
    * ios/, android/, windows/, macos/, linux/ 等
    * 開発しないosのフォルダを物理削除して良い。
    * 後で追加することも可能
        * `flutter create --platforms=windows,macos,linux`
* .gitignore
    * https://dart.dev/guides/libraries/private-files
    * ここに管理対象外のファイルがデフォルトで設定される
    * プラットフォームごとのビルドファイルやキャッシュファイルなどはGitの管理対象から外れる。
    * IME
        * 基本的にこのファイルに追記はすることはあっても、デフォルトの内容を変更することは無い。
* .dart_tool
    * https://dart.dev/guides/libraries/private-files
    * flutter pubやその他ツール等で使用されるファイルが格納されている。
    * FlutterのBuildキャッシュ(Flutter_Build)などもこのフォルダに格納されている。
* .idea
    * Android Studio(IDE)の設定情報などが記載されたファイルが格納されている。
        * VSCodeの場合は特に使われない(はず)

# 静的解析 の設定
* analysis_options.yamlに設定が記述される。
* flutter createで作成したプロジェクトはリンターとして、flutter_lintsパッケージがデフォルトで設定されている。
    
# Flutter本体のアップグレード
* バージョンを確認
    * `flutter --version`
    * `dart --version`
* Flutterのアップグレード
* `flutter upgrade`
    * flutterのSDK内にDartのバイナリも付属しているので、同時にDartのバージョンも上がる。
* リリースノート
    * https://docs.flutter.dev/release/release-notes
    * stableへのリリースが未済のbreaking change
        * https://docs.flutter.dev/release/breaking-changes#not-yet-released-to-stable
* プロジェクトのアップグレード
    * pubspec.yaml
        * environmentのsdk（これはdart-sdkのバージョン）を変更する。
        ```
        environment:
        sdk: ">=3.0.0 <4.0.0"
        ```
    * 新しいプロジェクトを作成したとき、上記のバージョンは現在インストールされているFlutterのバージョンに則した設定となる。
    * (参考)上記の設定で例えば「sdk: ">=2.17.0 <3.0.0"」として、3.0.5のdart-sdkで実行してもエラーは発生しない。
        * これは内部的にdart 2.12.0以上のときに、最大バージョンを3.0.0未満から4.0.0未満に置き換えるため。
        * 参考
            * https://zenn.dev/yumemi_inc/articles/20230518_dart_sdk_3
## トラブルシューティング
* Flutter本体をアップグレードした結果、利用するパッケージでエラーが発生
    * メジャーアップデートでは互換性が壊れる破壊的変更（Breaking change）を伴うことがある。
    * (例)(IME) qr_flutterパッケージで型エラーとなったケース
        * パッケージをアップデートする。基本的にpub.devを確認することで更新情報が分かる。
            * (例: Dart3.0に伴う変更) https://pub.dev/packages/qr_flutter/changelog
            * 4.0.1で Dart3.0にあわせた変更をしている。（BREAKING: Rename QrImage to QrImageView）
        * パッケージのアップデートを実行する
* `Your flutter checkout has local changes that would be erased by upgrading.`
    * 意図せずにFlutter frameworkのコードを更新してしまっている。
    * (IME)例えば、コードリーディングをしている際　に 誤って自動フォーマットしてしまった等
        * 問題なければ`flutter upgrade --force` で更新
        * 以下でどの箇所が変更されてしまったか確認できる。
            * `cd /Users/〜〜〜/flutter`
            * `git diff --stat` 

# パッケージのアップデート
* https://docs.flutter.dev/release/upgrade#upgrading-packages
## パッケージの状態を確認する
* 新しいパッケージの確認
* `flutter pub outdated`
## pubspec.ymlの指定の範囲内で更新する
* 下記コマンドで、互換性のある最新バージョンに更新する
* `flutter pub upgrade`
* `flutter pub upgrade パッケージ名`
* これによってpubspec.lockが更新され、同時にflutter pub getも実行される。
* セマンティックバージョニングに基づけば、APIの互換性は保たれる変更のみとなる。
## pubspec.ymlの指定を変更する
* 最新バージョンに更新
    * `flutter pub upgrade --major-versions`
    * `flutter pub upgrade --major-versions パッケージ名`
    * これによってpubspec.yml, pubspec.lockが更新され、同時にflutter pub getも実行される。
    * この変更はAPIの互換性が保たれないため、ソースコード自体の修正が必要になる可能性がある。
* 手動で更新する
    * pubspec.yamlのバージョン指定を変更
    * `flutter pub get`
## (IMO) pubspec.ymlの指定と、バージョン解決について
* pubspec.ymlは定期的にメンテナンスすることは勿論として、バージョン解決が狭くなりすぎないようにする必要がある。
* 例えば `some_packeage: 8.2.1` といった固定の指定は問題を引き起こすことが多い。
* 事由として下記がある
1. バージョン解決に失敗する可能性が上がる
    * 例えば、他のパッケージに`some_package: ^8.3.0`といった指定がある場合は競合してしまう。
2. 不具合が発生しやすい/解決しづらい
    * バージョン解決した場合でも、(筆者の経験上)必ずしも問題なく動作するとは限らない。
    * 各パッケージ側のpubspec.ymlで指定されているバージョンの範囲が、完全に隈なくテストされているとは限らない。
    * バージョンが固定化されてしまうことで、新しいバージョンを取得しづらくなり、よりニッチな組み合わせが発生しやすい。
    * その場合に問題が起きた場合の情報が少ない。（あるいは無い）




# パッケージを削除
* パッケージを削除
    * `flutter pub remove パッケージ名` を実行して　`flutter pub get`
* ios/配下のpodfileなどは、ビルドまで実行しないと削除されない。
    

# Flutterのダウングレード
* `flutter downgrade バージョン`
    * ただし現在のChannelに該当のバージョンがない場合はダウングレードできない。
* 直接gitのブランチを切り替えることでバージョンを変更できる
    ```
    cd ~/flutter/
    git tagで確認
    git checkout 対象のバージョン
    ```