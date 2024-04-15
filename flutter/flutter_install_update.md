- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


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

# VSCode
* VSCodeを未インストールであればVSCodeをインストールする
* 拡張のFlutter extension for VS Codeを入れる
* 推奨されるeditorの設定を行う
    * コマンドパレットで「Dart: Use Recommended Settings」とするとグローバルな設定ファイルへ設定が行われる。
    * https://dartcode.org/docs/recommended-settings/

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
    * 現在のバージョン、依存関係、同梱するアセットなど、アプリの基本情報を設定するYAML形式のファイル
    * 新しい Flutter プロジェクトを作成すると、基本的な pubspec が生成される。
    * environment
        * sdk: 対象のdart-sdkのバージョン。
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


# フォーマット
* https://docs.flutter.dev/tools/formatting
* `flutter format`
    * `dart format`のラッパーコマンド。
* IME
    * VSCodeを利用する場合、フォーマット実行は上記コマンドの処理をラップしているため、直接コマンドを使ってフォーマットすることはほとんどない。

# リンター
* flutter createで作成したプロジェクトはflutter_lintsパッケージがデフォルトで設定されている。
    * analysis_options.yamlにリンターの設定が記述される。   

# Flutter本体のアップグレード
* Channelを指定
* `flutter channel`
    * https://docs.flutter.dev/release/archive
        > The Stable channel contains the most stable Flutter builds
    * チャネルにはstable, beta, dev, masterがある。
    * 安定版であるstableが選択されていることを確認。
* バージョンを確認
    * `flutter --version`
    * `dart --version`
* Flutterのアップグレード
* `flutter upgrade`
    * flutterのSDK内にDartのバイナリも付属しているので、同時にDartのバージョンも上がる。
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


# パッケージをアップデート
* https://docs.flutter.dev/release/upgrade#upgrading-packages
* pubspec.yamlのバージョンを変更し、`flutter pub get`
* 古いパッケージの確認
    * `flutter pub outdated`
* 互換性のある最新バージョンに更新
    * `flutter pub upgrade`
* すべての依存関係の最新バージョンに更新
    * `flutter pub upgrade --major-versions`

# パッケージを削除
* パッケージを削除
    * `flutter pub remove パッケージ名` を実行して　`flutter pub get`
* ios/配下のpodfileなどは、ビルドまで実行しないと削除されない。
    

