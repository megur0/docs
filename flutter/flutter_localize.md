[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > ローカライズ


# 公式ドキュメント
* https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization

# Flutterのローカライゼーション/国際化
* MaterialAppまたはCupertinoAppウィジェットにはローカライゼーションの機能が備わっている。
    * これらのウィジェットのプロパティでLocalizationsDelegateを実装したオブジェクトを渡す。
* flutter_localizationsパッケージは、言語ファイルを設定する事でLocalizationsDelegateを実装したクラスを自動生成できる。
    * 手順は下記を参照
        * https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization#adding-your-own-localized-messages
        * 自動生成ファイルは「.dart_tool/flutter_gen/get_l10n」に生成される。
        * 自動生成されたファイルをアプリケーションコード側でimportすることで利用可能
            * (IME)自動生成されたファイルはVSCodeのエディタで入力してもサジェストに出てこない
        ```dart
        import 'package:flutter_gen/gen_l10n/app_localizations.dart';
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
        ```
        import 'package:flutter_gen/gen_l10n/app_localizations.dart';
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




    