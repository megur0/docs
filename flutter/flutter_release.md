[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# WIP: 随時更新
* このメモは執筆中のため随時更新 

# 注意
* このメモは対象環境はiOS/Xcodeとなる。

# 公式ドキュメント
* https://docs.flutter.dev/deployment/ios


# Flutterでビルドを生成してXcodeでアップロードする手順
* バージョンの設定
    * pubspec.ymlに設定を行う
    * 微修正等でTestFlightに新しい内容でアップロードする場合はビルド番号を上げる。
* Flutter側でビルドを行う
    * `flutter build ios (各種オプション)` 
* XCode側の作業
    * Xcodeでアーカイブを行い、アーカイブをApple Store ConnectへアップロードやAd hocでインストールする。 

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
* 初回はXcodeで実行して、その後にExportOptions.plistをXcodeで作成し、それをflutter build ipaの--export-options-plistで指定することで実行できる。
    * https://github.com/flutter/flutter/issues/106612#issuecomment-1271812839
* アーカイブファイルを作成
    * `flutter build ipa --export-options-plist=path/to/ExportOptions.plist`
    * 必要であれば例えば以下のオプションをつける。
         * `--dart-define-from-file=path/to/XXXXXX.json`
         * ` --obfuscate --split-debug-info=(出力先のパス。例: obfuscate/ios)`
* 以下のコマンドでApp Store Connectへアップロード
    * `xcrun altool --upload-app --type ios -f build/ios/ipa/*.ipa --apiKey your_api_key --apiIssuer your_issuer_id`



# (未読)codemagic-cli-toolsを利用したCI/CI
* https://docs.flutter.dev/deployment/ios#create-a-build-archive-with-codemagic-cli-tools
