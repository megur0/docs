- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# VSCodeのFlutter環境の構築
* [環境構築・アップデート](./flutter_install_update.md)を参照

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


# 様々なdebugフラグやメソッド
* https://docs.flutter.dev/testing/code-debugging
* 例えばエディタ上で"debug"と打つと様々なデバッグのグローバルなフラグやメソッドがレコメンドされる。
* これらのメソッドや、フラグをtrueにすることで様々な情報を確認する事ができる。
* 具体的にどの箇所で出力・実行されているかはFlutterのコードを読むと良い。
* スタックトレースを出力
    * debugPrintStack();
* 階層構造の出力
    * debugDumpApp()
        * runApp()によって構成されるウィジェットの階層を出力
        * 多くのFlutter標準のウィジェットはbuild()内で別のウィジェットやプライベートなウィジェットを生成する。
        * これらのオブジェクトを目視で確認するために便利なメソッド
    * debugDumpRenderTree()
        * RenderObjectのツリーを出力
    * debugDumpLayerTree()
        * Layerのツリーを出力。LayerはRenderObjectのツリーから生成された合成レイヤーである。
    * debugDumpSemanticsTree();
        * SemanticsNodeのツリーを出力
* フレームやレンダリングパイプラインのデバッグ
    * debugPrintBeginFrameBanner
        * フレームのスタート時にコンソールにバナーを出力
    * debugPrintEndFrameBanner = true;
        * フレームの終了時にコンソールにバナーを出力
    * debugPrintRebuildDirtyWidgets
        * Element.rebuild()のコールの度に対象のElementの情報が出力される
        * コールバックを渡す方法もある: debugOnRebuildDirtyWidget
    * debugPrintMarkNeedsLayoutStacks, debugPrintMarkNeedsPaintStacks
        * RenderObject.markNeedsLayout()やmarkNeedsPaint()の際にデバッグメッセージを出力
    * debugPrintBuildScope
        * BuildOwner.BuildScope()が呼び出された際に出力
    * debugPrintScheduleFrameStacks
        * フレームのスケジューリングが行われた時にデバッグメッセージを出力
* UI上にデバッグ表示を表示
    * debugPaintLayerBordersEnabled
        * 各レイヤーの境界を表示する
    * debugRepaintRainbowEnabled
        * 各ウィジェットの周囲に色付きの境界線を表示する
* ※ 上記以外にも多くのフラグやメソッドがある。


# パフォーマンス測定(未読)
* https://docs.flutter.dev/testing/code-debugging#trace-dart-code-performance
* TODO

# アニメーションのデバッグ(未読)
* https://docs.flutter.dev/testing/code-debugging#debug-animation-issues
* TODO