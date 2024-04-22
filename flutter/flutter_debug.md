- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# Devtools
* https://docs.flutter.dev/tools/devtools/overview
    > DevTools is a suite of performance and debugging tools for Dart and Flutter.
* 参考
    * https://marble-heroes.com/blog/cqldl6nt3xgw
 

# （こちらを推奨）VSCode上でDevtoolsを利用する
* Flutter extension for VS Codeを入れることでVSCode上でDevtoolsを使用可能となる。
* VSCode上で実行すると、自動的にDevtools上の出力を「DEBUG CONSOLE」へ出力する。
* VSCodeのインスペクタもVSCode上で開くことが可能。
* 「Run and Debug」パネルからDevtoolsのブレークポイントやログの設定ができる。
    * Devtoolsに組み込まれたUIと異なり、リスタートしてもブレークポイントがリセットされない。
        * IMO) DevtoolsのUIよりも高機能となっている。


# コマンドラインからDevtoolsを利用する
* flutter runから起動する
    * `flutter run`では内部的にDevtoolsを起動されていて、Dart DevToolsによるデバッグとパフォーマンス分析を有効になっている。
    * 「v」でブラウザから開く
    * (参考)Dartから起動する
        * Devtools自体を開く
            * `dart devtools`
            * 例）http://127.0.0.1:9100 で起動される。
            * ブラウザで開いて、画面のURL欄にデバッグするアプリのURLを入力
        * 現在のプロジェクト(pubspec.ymlがあるプロジェクト上で実行)をdevtoolsを有効にして実行する。
            * `dart run --observe`
        * https://dart.dev/tools/dart-run#debugging
        * https://dart.dev/tools/dart-devtools
* ブレークポイントを設定したい場合は、ソースコードに debugger() を設定して実行する。
    * その後のステップ実行などはDevtools上で行う。
* gdbでブレークポイントを操作する方法もある（未確認）
## コードとDevtoolsの連携
* https://api.dart.dev/stable/3.1.1/dart-developer/dart-developer-library.html
* dart:developerパッケージをimportする。
    * Dart ランタイムと対話することを目的とした特殊なライブラリ
    * Dart Web および Dart Native (VM) のプラットフォーム固有の実装が含まれている。
    * 開発モード（dart run）でのみ有効。
    * 実稼働モード（dart compile exe）では動作しない。
* デバッガ（Devtool）前提の機能
    * `dart run --observe`を実行して、Devtoolsを起動する。
* Dartコード内でブレークポイントを設定
    * https://api.dart.dev/stable/3.1.1/dart-developer/debugger.html
    * `debugger() `
    * Devtools上で、上記の「debugger()」で止めた箇所から再開ができる。
    * さらに別のブレークポイントもUIで指定できる。Exception発生時に停止させることも可能。
    * DevtoolはRestartするとブレークポイントなどの設定が全部クリアされてしまう
        * https://docs.flutter.dev/tools/devtools/debugger#known-issues
        * https://github.com/flutter/devtools/issues/1925
* Dartコード内でログを出力
    * `log()`
    * デバッガへログを送信する。
    * 標準出力には出力されないため注意。
* その他機能は下記を参照。
    * https://docs.flutter.dev/testing/code-debugging
* (参考)デバッガと Dart VMはどのように連携しているのか？
    * Flutter上のdart:developerの　サービスクラスによってwebサーバーが起動し、このサーバー経由でDart VM によって提供されるサービスへアクセスしていると考えられる。
    * https://api.dart.dev/stable/3.1.1/dart-developer/Service-class.html
* (参考)Dart:developerのlogと、debugPrint、printの比較
    * Dart:developer log
        * デバッガへ送信
        * Devtools上は表示されるがコマンドラインには表示されない。
    * debugPrint
        * 引数の型がString型
    * print
        * 引数の型はObject?型
        * 記述すると警告が表示される。


# (参考)VSCode vs コマンドライン+Devtools
* FlutterチームはVSCodeを推奨
    > If you write Flutter apps only with Dart code, you can debug your code using your IDE’s debugger. The Flutter team recommends VS Code.
    * https://docs.flutter.dev/testing/native-debugging


# VSCode
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
* 実行中のデバッグ
    * hot reload （cmd + s もしくは F5）
    * Restart (shift + cmd + F5)
    * inspectorの表示, 要素の選択
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
        // コード整形(Flutter/Dart独自では無いが参考として記載)
        "editor.formatOnSave": true,
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

