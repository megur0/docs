- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# 注意
* 本メモはAndroidに関しては触れていない

# Flutterで環境を分ける
* flutter runのオプションの--dart-define-from-fileを利用する
* 例
    * `flutter run --dart-define-from-file=dart_defines/dev.json`
* バンドルID
    * 通常はXcode上でCFBundleIdentifierを修正(あるいはファイルの内容を修正)する必要があるが、--dart-define-fromで指定するファイル上で定義しておくと一括で変更することができる。
* Xcodeからの参照
    * ビルド時に自動生成されるios/Flutter/flutter_export_environment.shやios/Flutter/Generated.xcconfigによって参照可能となる。
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
## (補足)--flavor　や --dart-define を利用する方法
* これらを利用する方法もあるがおそらく --dart-define-from-fileを利用する方法が最も簡潔になると考えられる。


