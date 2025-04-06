[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > ローカライズ


# 公式ドキュメント
* https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization

# Flutterのローカライゼーション/国際化
* MaterialAppまたはCupertinoAppウィジェットにはローカライゼーションの機能が備わっている。
    * これらのウィジェットのプロパティでLocalizationsDelegateを実装したオブジェクトを渡す。
* flutter_localizationsパッケージは、言語ファイルを設定する事でLocalizationsDelegateを実装したクラスを自動生成できる。
    * 手順は下記を参照
        * https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization#adding-your-own-localized-messages
            * 注意: flutter_genのdeprecate
                * l10n.yamlの設定には`synthetic-package: false`が必要
                * マイグレーション対応の方を参照
                * 合わせて`output-dir:`にて出力先も設定する。

        * ~~自動生成ファイルは「.dart_tool/flutter_gen/get_l10n」に生成される。~~
            * `output-dir:`で指定したディレクトリへ出力
        * 自動生成されたファイルをアプリケーションコード側でimportすることで利用可能
            * (IME)自動生成されたファイルはVSCodeのエディタで入力してもサジェストに出てこない
        ```dart
        import 'package:(output-dirで指定した場所)/app_localizations.dart';
        import 'package:flutter_localizations/flutter_localizations.dart';

        // ...
            return const MaterialApp(
                title: 'Localizations Sample App',
                localizationsDelegates: [
                    AppLocalizations.delegate, // これが自動生成されたクラスとなる。
                    GlobalMaterialLocalizations.delegate, // 以下の3つはあらかじめflutter_localizationパッケージに定義されたdelegateクラス
                    GlobalWidgetsLocalizations.delegate,
                    GlobalCupertinoLocalizations.delegate,
                ],
                supportedLocales: [ // サポート対象のロケール
                    Locale('en'), // English
                    Locale('ja'), // Japanese
                ],
                home: MyHomePage(),
            );
        // ...
        ```
        ```dart
        import 'package:(output-dirで指定した場所)/app_localizations.dart';
        // ...
        appBar: AppBar(title: Text(AppLocalizations.of(context)!.helloWorld)),
        // ...
        ```
* intlパッケージを利用する事で数値や日付などのローカライゼーションが可能。

# サポート対象のロケール
* サポート対象として指定したロケールの言語が対応可能が言語となる。
* 実際にアプリを表示した際に表示される言語は、ユーザーが端末の設定にて選択している言語となる。

# 強制上書き
* https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization#overriding-the-locale
* Localizations.override()から生成したウィジェットでラップする事で子孫ウィジェットのLocaleを上書きすることが可能。


# マイグレーション対応(25/4/6 追記)
* ビルド時に以下のメッセージが表示される
```sh
Synthetic package output (package:flutter_gen) is deprecated:
https://flutter.dev/to/flutter-gen-deprecation. In a future release,
synthetic-package will default to `false` and will later be removed entirely.
```
* 下記のマイグレーション対応が必要
    * https://docs.flutter.dev/release/breaking-changes/flutter-generate-i10n-source#migration-guide
* 参考
    * https://blog.cutboss.work/2025/03/flutter-gen-deprecated.html
* 筆者はl10n.yamlを下記のように設定した。
    ```yml
    synthetic-package: false

    arb-dir: lib/i18n
    output-dir: lib/i18n/generated/
    template-arb-file: app_ja.arb
    output-localization-file: app_localizations.dart
    ```
    * 上記の設定をして、既存のlib/10nフォルダの名前を変更し、`flutter clean && flutter pub get`を実行する
    