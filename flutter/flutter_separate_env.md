[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > 環境を入れ替え可能とする


# 注意
* 本メモはAndroidに関しては触れていない

# Flutterで環境を分ける
* flutter runのオプションの--dart-define-from-fileを利用する
* 例
    * `flutter run --dart-define-from-file=dart_defines/dev.json`
* 設定
    * jsonファイル
        * キーバリューで変数名と値を定義
    * ネイティブ側(iOS)への展開するRun scriptの設定
        * ※ Flutterが3.19以降の場合に必要。
            * (参考) 当初はflutter_toolsにおいて--dart-define-from-fileオプションで指定した変数はデコードされてネイティブへ展開されていたが、これは設計の意図に沿っていない実装だったという事由で削除され3.19にて反映された。
                * https://github.com/flutter/flutter/pull/136865
                * https://github.com/flutter/flutter/issues/138793#issuecomment-1832732494
                * 代替機能に関するissue
                    * https://github.com/flutter/flutter/issues/139289
        * 設定内容は下記が参考になる
            * https://github.com/flutter/flutter/issues/139289#issuecomment-1954103555
* Xcodeからの参照
    * 例えば以下のようにios側の設定ファイルで参照する事ができる。
        * dart_defines/dev.json
            ```
            {
                "appName": "my app",
                "applinks": "example.my.app",
                "bundleId": "com.example.my.app",
                "developmentTeam": "XXXXXXXXXX",
                "provisioningProfileSpecifier": "test app profile",
            }
            ```
        * ios/Runner/Info.plist
            ```
            〜〜
                <key>CFBundleIdentifier</key>
	            <string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
                <key>CFBundleDisplayName</key>
                <string>$(appName)</string>
            〜〜
            ```
        * ios/Runner/Runner.entitlements
            ```
            〜〜
                <key>com.apple.developer.associated-domains</key>
                <array>
                    <string>applinks:$(applinks)</string>
                </array>
            〜〜
            ```
        * ios/Runner.xcodeproj/project.pbxproj
            * Debug, Profile, Releaseの３つのbuildSettingsに設定
            ```
            /* ... */
            buildSettings = {
                /* ... */
                "DEVELOPMENT_TEAM[sdk=iphoneos*]" = "$(developmentTeam)";
                /* ... */
                PRODUCT_BUNDLE_IDENTIFIER = "$(bundleId)";
                /* ... */
                "PROVISIONING_PROFILE_SPECIFIER[sdk=iphoneos*]" = "$(provisioningProfileSpecifier)";
                /* ... */
            };
            /* ... */
            ```
* Dartコードで利用する
    * Stringやint, bool などのstaticメソッドとしてfromEnvironment()を利用する事ができる。
    ```
    final appName = const String.fromEnvironment('appName');
    ```

# --flavorや--dart-defineを利用する方法
* これらを利用する方法は割愛する
* https://docs.flutter.dev/deployment/flavors
* https://dart.dev/guides/environment-declarations