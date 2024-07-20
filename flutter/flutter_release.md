[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# WIP: 随時更新
* このメモは執筆中のため随時更新 

# 注意
* このメモは対象環境はiOS/Xcodeとなる。

# 公式ドキュメント
* https://docs.flutter.dev/deployment/ios


# Flutterでビルドを生成してXcodeでアップロードする手順
1. バージョンの設定
    * pubspec.ymlに設定を行う
    * 微修正等でTestFlightに新しい内容でアップロードする場合はビルド番号を上げる。
2. Flutter側でビルドを行う
    * `flutter build ios (各種オプション)` 
    * この作業はコマンドが変わらない場合は、2回目以降は省略可能
3. XCode側の作業
    * Xcodeでアーカイブを行い、アーカイブをApple Store ConnectへアップロードやAd hocでインストールする。 
## XCode側のアーカイブは最後のビルド構成を利用する
* https://github.com/flutter/flutter/issues/64626#issuecomment-681156022
* XCodeのアーカイブ処理はビルド処理も含まれており、そのビルド処理は最後のビルド構成を利用する。
    * 前回に`flutter build ios 〜` によって実行したビルド処理と同じビルド処理を行う。
* (IME)
    * 筆者の手元の環境でビルド処理をしていないにもかかわらずXcodeでアーカイブを行った際に、Dartコードの変更内容が反映されたため、疑問に思って調べたところこのような仕様となっていることが分かった。
    * したがって、`flutter build ios 〜` のコマンドが前回と変わらない場合は、XCodeのアーカイブのみで良いかもしれない。
    * ただし、pubspec.ymlのversionを変更しても、Xcodeのアーカイブのみでは反映されなかった。
    * 内部の処理がどのようになっているかは分からない。確実に変更を反映するにはflutter側でビルドも実施したほうが良いかもしれない。

# (参考)ビルド時の--debugや--profile
* ビルド時に--debugや--profileをつけても、Xcodeでアーカイブした時点で利用ができなくなる。
    * これはkDebugModeの状態によって確認できた。
    * TestFlightの場合でも、Ad hocの場合でもkDebugModeはfalseとなった。
* https://stackoverflow.com/questions/75911065/flutter-build-ignores-profile-flag

# (未検証)flutter build ipa を利用する方法
* 調べてみたところ、マニュアルで作成したProvisioning Profileの場合、初回のアーカイブファイル作成はflutter build ipa から実施する事はできないと考えられる。
    * 手元で実施してみたところ、
        * ビルドまでは完了するが、`error: exportArchive: No signing certificate "iOS Distribution" found` のエラーが発生
        * 試しにProvisioing Profileを手動でApple Storeで登録してみるが、`error: exportArchive: "Runner.app" requires a provisioning profile with the Associated Domains and Push Notifications features.` と表示され、何かが上手くいっていないことがわかる。
            * このProfileはAssociated DomainsやPush Notificationsは紐づいたApp IDに対して作成しているため、Flutter側の処理で問題が発生している可能性が高い。
    * この問題に対してのIssue
        * https://github.com/flutter/flutter/issues/106612
    * この点について特に公式ドキュメントには触れられていない。
    * アプリに対して通知などの機能を必要とする際はマニュアルでProvistioning Profileを作成する必要があるため、その場合は初回はXcodeで実行することなるだろう。
* (以降の手順は未検証)
* 初回はXcodeでアーカイブおよびApp Store Connectへのアップロード（あるいはipaファイルの作成）まで行う。
    * 完了後の画面で"Export..."を押下することでファイルとエクスポートできる。
    * エクスポートしたディレクトリ内にExportOptions.plistが含まれる。
* flutter build ipaの--export-options-plistへエクスポートされたExportOptions.plistを指定して実行ができる。
    * https://github.com/flutter/flutter/issues/106612#issuecomment-1271812839
* アーカイブ(ipa)ファイルをflutterのコマンドで作成
    * `flutter build ipa --export-options-plist=path/to/ExportOptions.plist`
    * 必要であれば例えば以下のオプションをつける。
         * `--dart-define-from-file=path/to/XXXXXX.json`
         * `--obfuscate --split-debug-info=(出力先のパス。例: obfuscate/ios)`
* 以下のコマンドでApp Store Connectへアップロード
    * `xcrun altool --upload-app --type ios -f build/ios/ipa/*.ipa --apiKey your_api_key --apiIssuer your_issuer_id`


# 難読化
## バイナリは逆コンパイル可能
* apkやipaファイルといったアーカイブファイル（圧縮ファイル）は、アプリ上で実行するためのバイナリを含んでいる
* Androidの場合は、apkファイルは公式のapktoolで逆コンパイルが可能である。
* 一方、iOSの場合ipaファイルは公式の方法は無いが、基本的に元のソースコードに近いコードを復元できると考えて差し支えないだろう。
    * 参考
        * https://atmarkit.itmedia.co.jp/ait/articles/1607/05/news008.html
        * https://forums.developer.apple.com/forums/thread/99653
        * ipaファイルに含まれるもの
            * https://www.micss.biz/2023/02/20/5857/
## 難読化をする
* https://docs.flutter.dev/deployment/obfuscate
* コード内の関数名とクラス名を隠し、各シンボルを別のシンボルに置き換えて、攻撃者が独自のアプリをリバース エンジニアリングすることを困難にする。
* `flutter build` に `--obfuscate --split-debug-info=ディレクトリ` を付与することで難読化する。
    * リリースモードでのみ機能する
    * --obfuscateによってクラス名と関数名をランダムな値に変換する。
        * このオプションは常に --split-debug-infoと合わせて利用する必要がある。
        * --split-debug-infoによって出力されるシンボルのマッピングファイルを利用して、スタックトレースの情報を人間が判読できる形に変換することができる
    * --split-debug-infoのオプション自体は、Dart プログラム シンボルをアプリケーション外の別のファイルに保存することで、アプリケーションのサイズを縮小する。
        * .symbolsファイルが出力される。
    * 詳細は`flutter build (ios/apk等) -h`
* なお、dSYMファイルでは難読化は解除されるため、これらを扱うツールへシンボルファイルをアップロードする必要はない。
    * [参照](./flutter_firebase_clashlytics.md)
* 参考
    * 現在はiOSも対応しているため、以下の公式ドキュメントは情報が古いようだ。
    * https://firebase.google.com/docs/crashlytics/get-started?platform=flutter&hl=ja#add-sdk
        > 現在、--split-debug-info のサポートは Android でのみ利用できます。  
        > Apple プラットフォームの場合、この機能のサポートは今後のリリースで利用可能になる予定ですが、Flutter SDK のマスター チャンネルを使用することで、今すぐアクセスすることができます。
        * https://github.com/firebase/firebase-tools/issues/5291#issuecomment-1338892219



# (未読)codemagic-cli-toolsを利用したCI/CI
* https://docs.flutter.dev/deployment/ios#create-a-build-archive-with-codemagic-cli-tools
