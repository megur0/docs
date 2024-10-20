[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > Firebase Clashlytics


# 注意
* 筆者はMac環境で「XCode」「VSCode」にてiOSアプリをFlutterで開発しているため、このメモではAndroidについては触れない。
* このメモは(他のメモと比べて特に)筆者の推測や調査が混じっている点に注意。

# Firebase Clashlyticsについて
* [こちら](../firebase_clashlytics/firebase_clashlytics.md)を参照

# iOSのクラッシュレポートについて
* [こちら](../ios/ios_clash_report.md)を参照

# Flutterプロジェクトへの導入(iOS)
* 以下の手順に従う
    * firebase_crashlyticsプラグインを追加し、`flutterfire configure`を実行する。
    * ※ firebase_analyticsについては追加は任意。
    * https://firebase.google.com/docs/crashlytics/get-started?platform=flutter&hl=ja#add-sdk
* 上記の手順を実行するとflutterfire_cliは、ビルドフェーズに以下の名前の実行スクリプトを追加する。
    * `FlutterFire: "flutterfire upload-crashlytics-symbols"`
* (参考) flutterfire_cli
    * このアップロードスクリプトについてはflutterfire_cliのREADMEではあまり言及されていない。
        * https://github.com/invertase/flutterfire_cli
    * CHANGELOGの方には記載されている。
        * https://github.com/invertase/flutterfire_cli/blob/main/CHANGELOG.md#flutterfire_cli---v030-dev20
            > FIX: ensure Crashlytics Apple upload-symbol script works for every build type/flavor
        * https://github.com/invertase/flutterfire_cli/blob/main/CHANGELOG.md#flutterfire_cli---v030-dev19
            > FIX(apple): upload debug symbols for de-obfuscating Dart stack traces
            > FEAT: flutterfire reconfigure now updates android build.gradle files & Apple project.pbxproj for debug symbol script like flutterfire configure
        * https://github.com/invertase/flutterfire_cli/blob/main/CHANGELOG.md#flutterfire_cli---v030-dev17
            > FEAT: automatically add debug symbols script by detecting crashlytics dependency. 
        * https://github.com/invertase/flutterfire_cli/blob/main/CHANGELOG.md#flutterfire_cli---v030-dev15
            > FEAT: upload debug symbols script. 
## デバッグビルド時にdSYMを送信する
* デフォルトでは、デバッグビルドでdSYMを生成しない点に注意。
    * この場合、Clashlytics上でdSYMの欠損エラーが表示される。
        * ※ ただし、クラッシュが発生した際に何度かFlutterアプリをリスタートをしているとdSYMの欠損エラーが解消してClashlyticsでレポートを見ることができた。
            * (IMO)もしかするとデバッグモードでも「クラッシュが発生した時はdSYMを送る」といった処理が内部で行われているかもしれない
    * XCodeで Runner > Build Setting > Debug Information Format で確認すると、デバッグビルドにおいては「dwarf」となっている。
* デバッグビルドも出力したい場合は、こちらを`dwarf-with-dsym`へ変更する。
* 参考: https://github.com/flutter/flutter/issues/116493#issuecomment-1340259052
    * ※ このissue自体は別の内容。(Flutter Framwork自体のdSYMが不足してClashlyticsの詳細が確認できない、というissue)
## (注意) Clashlyticsのアップロードスクリプトが、ios/Runner.xcodeproj/project.pbxprojに含まれている場合
* Clashlyticsの方で、ビルドフェーズに以下の名前の実行スクリプトを追加されている場合は、削除する。(XCodeを開いて Runner > Build Phases でゴミ箱アイコンから削除する。 )
    * `[firebase_crashlytics] Crashlytics Upload Symbols`
    * ※ このスクリプトはdSYMファイルをClashlyticsへアップロードするもので、firebase_crashlyticsプラグインの方で追加されるもの。上記のFlutterfireのスクリプトでも同じコマンドを呼び出しており重複するため削除する。
    * https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?platform=flutter&hl=ja
## (参考)アップロードスクリプトの処理
* flutterfireが追加するビルドフェーズのスクリプトは下記。(flutterfire v1.0.0)
    ```sh
    #!/bin/bash
    PATH=${PATH}:$FLUTTER_ROOT/bin:$HOME/.pub-cache/bin

    flutterfire upload-crashlytics-symbols
    --upload-symbols-script-path=$PODS_ROOT/FirebaseCrashlytics/upload-symbols
    --platform=ios
    --apple-project-path=${SRCROOT}
    --env-platform-name=${PLATFORM_NAME}
    --env-configuration=${CONFIGURATION}
    --env-project-dir=${PROJECT_DIR}
    --env-built-products-dir=${BUILT_PRODUCTS_DIR}
    --env-dwarf-dsym-folder-path=${DWARF_DSYM_FOLDER_PATH}
    --env-dwarf-dsym-file-name=${DWARF_DSYM_FILE_NAME}
    --env-infoplist-path=${INFOPLIST_PATH}
    --default-config=default
    ```
* 上記はflutterコマンドが内部で`$PODS_ROOT/FFirebaseCrashlytics/upload-symbols`を実行する
* オプションで渡している各引数も適宜、上記コマンドへ渡されると考えられる。
* なお、DWARF_DSYM_FOLDER_PATHなどの環境変数は実行時に「flutter_tools/bin/xcode_backend.sh」によってエクスポートされる。
    * 環境変数は例えば下記のような値がエクスポートされる
    ```sh
    export DWARF_DSYM_FOLDER_PATH=/Users/path/to/project/build/ios/Release-iphoneos
    export DWARF_DSYM_FILE_NAME\=Runner.app.dSYM
    export INFOPLIST_PATH\=Runner.app/Info.plist
    export BUILT_PRODUCTS_DIR\=/Users/path/to/project/build/ios/Release-iphoneos
    export PROJECT_DIR\=/Users/path/to/project/ios
    ```
* 上記の処理は、XcodeのBuild Phaseの設定で、`"${PODS_ROOT}/FirebaseCrashlytics/upload-symbols" -gsp "${PROJECT_DIR}/Runner/GoogleService-Info.plist" -p ios "${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}" `を直接呼び出して、Input Filesに各種ファイルを設定することとほぼ同じと考えられる。
    * https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?platform=flutter&hl=ja#run-script-does-not-exist
    * https://github.com/firebase/flutterfire/issues/11547#issuecomment-1707347602
    * １点、違う点として、flutterfireの場合は「/build/ios/Release-iphoneos/Runner.app.dSYM」に加えて「/build/ios/Release-iphoneos/App.framework.dSYM」もClashlyticsへ送信した。
* 実行してみると以下の２つのdSYMがアップロードされた。
    * /path/to/project/build/ios/Release-iphoneos/App.framework.dSYM
    * /path/to/project/build/ios/Release-iphoneos/Runner.app.dSYM
* 以下はflutter build ios 〜や、flutter build ipa 〜を実行した際にXcodeのReportナビゲータ上に出力されたログの抜粋
    * warningとして、Flutterのバージョンを3.12.0+にアップグレードすることが表示されるが、実行時は3.24.3を使用していた為、おそらくobfuscateを利用している場合は必ず出力されるwarningと考えられる。
    ```sh
    Running upload-symbols in Build Phase mode
    warning: Flutter build with '--obfuscate' enabled. To view deobfuscated stack traces, upgrade to Flutter 3.12.0+ or upload dSYMs manually using the Firebase console web uploader at https://firebase.google.com/project/_/crashlytics
    Validating build environment for Crashlytics...
    Processing dSYMs...
    Successfully submitted symbols for architecture arm64 with UUID xxxxx in dSYM: /Users/path/to/project/build/ios/Release-iphoneos/Runner.app.dSYM
    Successfully submitted symbols for architecture arm64 with UUID xxxxx in dSYM: /Users/path/to/project/build/ios/Release-iphoneos/App.framework.dSYM
    Successfully uploaded Crashlytics build event and symbols
    Running upload-symbols in Build Phase mode
    warning: Flutter build with '--obfuscate' enabled. To view deobfuscated stack traces, upgrade to Flutter 3.12.0+ or upload dSYMs manually using the Firebase console web uploader at https://firebase.google.com/project/_/crashlytics
    Validating build environment for Crashlytics...
    Processing dSYMs...
    Successfully submitted symbols for architecture arm64 with UUID xxxxx in dSYM: /Users/path/to/project/build/ios/Release-iphoneos/Runner.app.dSYM
    Successfully submitted symbols for architecture arm64 with UUID xxxxx in dSYM: /Users/path/to/project/build/ios/Release-iphoneos/App.framework.dSYM
    Successfully uploaded Crashlytics build event and symbols
    ```

# Dartコードを難読化した際のシンボルファイルはアップロード不要
* dSYMは上記のスクリプトによってアップロードされるが、flutterのビルド時に難読化(`--obfuscate`)を行った場合でも、dSYMは難読化されない。
    * ※ もともとdSYMはiOSのリリースビルドではバイナリには含まれないため、dSYMが難読化される必要はない。
    * したがって`--split-debug-info`によって出力される.symbolファイルをClashlyticsへアップロードする必要はない。
* 参考
    * https://github.com/firebase/flutterfire/issues/8934#issuecomment-1521852271
    * https://github.com/firebase/flutterfire/issues/8934#issuecomment-1535457929
    * https://github.com/flutter/flutter/issues/124715
    * https://github.com/firebase/flutterfire/issues/10994
    * 参考: 上記のissueについて
        * `--obfuscate`を利用した際にClashlyticsのスタックトレースも難読化されてしまう、というIssueとなる。(8934, 10994のissue)
        * これは原因は`--obfuscate`を利用した際に、dSYMの方も難読化がされてしまったこととなる(124715のissue)  
        * これらのissueは現在、解消済み
            * https://firebase.google.com/support/release-notes/ios#crashlytics_8


# Flutter.framework.dSYM 
* flutterfire_cliによって追加されるアップロードスクリプトが実行されても、Flutter.framework.dSYMは含まれない。
* このdSYMは個別にアップロードする必要がある。(あるいはスクリプトへ追加する。)
## Flutter.framework.dSYMの場所
* https://github.com/flutter/engine/blob/main/docs/Crashes.md#ios
* 3.24以降
    * ~/flutter/bin/cache/artifacts/engine/ios-release/Flutter.xcframework の中に含まれる。
    * あるいは、アプリのアーカイブの中のdSYMsにFlutter.framework.dSYMが含まれるので、そちらをアップロードすれば良い。
        * ~/Library/Developer/Xcode/Archives
        * これは、XCode16以降で行われるApp Storeの検証で必要となったため、アーカイブに含まれるようにFlutterが対応したもの。
            * https://github.com/flutter/flutter/pull/153215#issue-2458940608
            > Xcode 16 以降、App Store の検証では、App Store にアップロードされたアプリが、埋め込まれている各フレームワークの dSYM デバッグ情報バンドルをバンドルすることが必要になりました。
            > dSYMバンドルは、エンジンパッチとしてios-release toolsアーカイブに同梱されているFlutter.xcframeworkにパッケージされています
            > これはFlutter.framework.dsymバンドルをtoolsキャッシュからflutter build ipaが生成するappアーカイブにコピーする。
* 3.24より前のバージョン
    * Google Cloud Storage からダウンロードする
## (参考)(IMO)3.24以降はアーカイブには含まれるようになったが、Clashlyticsに自動的にアップロードがされるわけではない?
* 下記のIssueは、ClashlyticsでFlutter.framework.dSYMが欠損していると報告されるというIssueだが、その原因は(Issueやその対応PRを読む限り)Xcode 16 以降、App Store の検証でFlutter.framework.dSYMが必要になった事、と考えられる。
    * https://github.com/flutter/flutter/issues/116493#issue-1475337907
        > Crashlyticsによってプロダクションクラッシュが報告されましたが、UUID 1F29F0F6-7F1B-3049-A2C2-B0101BD2AB95のdSYMが見つからないため、詳細を表示することができません。
        > 過去に何度かクラッシュが発生しましたが (同じビルドでも)、dSYM の欠落に関する問題は報告されていませんでした。そのため、dSYM のほとんどは問題なくビルド出力に含まれていると推測しています。
    * タイトルが途中で「アーカイブにFlutter.frameworkのdSYMが含まれていない」に変更されている。
        > changed the title ~~Flutter application missing framework dSYMs~~ Flutter application missing framework dSYMs, validation error "The archive did not include a dSYM for the Flutter.framework with the UUIDs" on Jul 10
* 最終的には、以下のように問題は解決した、とされている。
    * https://github.com/flutter/flutter/issues/116493#issuecomment-2282216179
        > これは #153215 (commit c375dd8) で master ブランチで修正されました。 Xcode Product > Archiveまたはflutter build ipa経由でApp Storeアーカイブを作成すると、Flutter.framework.dSYMバンドルが埋め込まれるようになりました。
* (IMO)
    * Flutter.framework.dSYMがアーカイブに含まれるようにはなったが、Flutter.framework.dSYMがClashlyticsにアップロードされるようになったわけではない。
        * 実際に3.24.3でビルドやアーカイブを行った際に、ログ上はFlutter.framework.dSYMがアップロードされていない。
        * flutterfireは"${PODS_ROOT}/FirebaseCrashlytics/upload-symbols"がアップロードする「Runner.app.dSYM」に加えて「App.framework.dSYM」もアップロードしているが、このdSYMとFlutter.framework.dSYMの関係性は分からない。
    * 上記のIssueの、Firebase Clashlyticsが「dSYMの欠損」として検知した事と、Flutter.framework.dSYMを(XCode16以降で)アーカイブに含める必要が発生した事、の因果関係はよく分からない。

# dSYM を Firebase Clashlyticsへ個別にアップロードする
* `flutterfire configure`によって追加されるスクリプトがアップロードするdSYMは、App.framework.dSYM と Runner.app.dSYM の２つのみとなる。
* したがって例えばFlutter.framework.dSYMや、その他利用しているプラグインといったoptionalなdSYMも、頻繁にスタックトレースに出現するものは個別にアップロードしておく必要がある。
* 以下は(筆者が調べた範囲での)アップロード方法となる。
* なお、ビルドスクリプトを書き換えてこれらのコマンドを自動的に実行しても良いだろう。
## (方法1)upload-symbolsコマンド
* 下記のようにdSYMファイルを直接、upload-symbolsコマンドへ渡す。
```sh
find ./ -name "*.dSYM" | xargs -I \{\} ios/Pods/FirebaseCrashlytics/upload-symbols -gsp ios/Runner/GoogleService-Info.plist -p ios \{\}
```
* https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?platform=ios&hl=ja#manually-upload-dsyms
## (方法2)upload-symbolsコマンド
* アーカイブファイルに含まれるdSYMsを渡す方法
    * Xcodeで生成されるアーカイブ(xxxx.xcarchive)には、dSYMsが含まれる。
* dSYMsの場所
    * Xcode > Window > Organizer > 対象のアーカイブを右クリック > show in finder で対象のアーカイブのパスを確認
        * あるいは ~/Library/Developer/Xcode/Archives を直接確認
```sh
ios/Pods/FirebaseCrashlytics/upload-symbols -gsp ios/Runner/GoogleService-Info.plist -p ios "/path/to/Library/Developer/Xcode/Archives/YYYY-mm-dd/Runner YYYY-mm-dd, hh.mm.xcarchive/dSYMs"
```
## (方法3)firebaseコマンド
* 筆者は、この方法はJavaのExceptionが発生して動作しなかった。
* 以下を実行する。
```sh
`firebase crashlytics:symbols:upload --app="(コンソール上で確認できるアプリのID)" obfuscate/ios/app.ios-arm64.symbols`
```
* https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?hl=ja&platform=flutter#android-split-debug-info
## (IMO)どのタイミングで直接アップロードするか？
* 利用しているFlutter Flameworkやパッケージのバージョン（ビルド）が変わらない限り、毎回アップロードする必要はない。
* 下記のようにバージョンが変わった際にアップロードすればよいだろう。
* Flutter Flameworkのバージョンを更新した際
    * `ios/Pods/FirebaseCrashlytics/upload-symbols -gsp ios/Runner/GoogleService-Info.plist -p ios ./build/ios/archive/Runner.xcarchive/dSYMs/Flutter.framework.dSYM`
* Pluginパッケージ等を更新した際
    * 単体
        * `ios/Pods/FirebaseCrashlytics/upload-symbols -gsp ios/Runner/GoogleService-Info.plist -p ios ./build/ios/archive/Runner.xcarchive/dSYMs/対象のパッケージ.dSYM`
    * 複数
        * 上記の方法1や2でまとめてアップロード
    


