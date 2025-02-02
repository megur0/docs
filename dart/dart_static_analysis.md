[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > 静的解析

# 静的解析(static analysis)
* https://dart.dev/tools/analysis
* Flutterの静的解析(Static analysis)は以下で構成される。
    * アナライザー
        * https://dart.dev/tools/diagnostic-messages
        * プログラムが有効であることを確認
        * Dartの型システムによるチェックもここに含まれる。
            * https://dart.dev/language/type-system
                > One benefit of static type checking is the ability to find bugs at compile time using Dart's static analyzer.
    * リンター
        * https://dart.dev/tools/linter-rules
        * 有効なDart プログラムでのみ機能し、コーディングスタイルを強制する
        * 主なリンターパッケージ
            * Dart linter
                * https://pub.dev/packages/lints
            * Flutter_lints
                * Flutterでデフォルトで利用されるパッケージ
                * Dart linterのスーパーセットとなる。
                * https://pub.dev/packages/flutter_lints
* analyzerパッケージ
    * https://pub.dev/packages/analyzer
    * Dartの静的解析(アナライザー+リンターの実行)を実現するパッケージ。
    * Dartエコシステムにおいて静的解析を行う各ツールでは、このanalyzerのAPIを利用している
    * アプリケーション開発においては、直接このパッケージを利用することは無い。
* Dart Analysis Server
    * IDEやエディタをサポートするステートフルなサーバ
    * VSCodeやAndroid StudioはこのサーバーのAPIを実行することでstatic analysisの機能を提供している。
    * 内部的にはanalyzerパッケージが組み込まれている
* コマンドラインでの実行
    * https://dart.dev/tools/dart-analyze
    * `dart analyze` によって 静的解析(アナライザー+リンターの実行)が実行される。
        * コマンドの内部ではanalyzerパッケージのAPIが呼ばれていると考えられる。
    * IDEでDartのExtensionやプラグインを利用している場合は、このコマンドと同様の処理を実行している。
    * `dart fix`で自動修正されるものは、この結果のうちリンターの自動修正可能なもの(クイックフィックス)に対して行われる。
        * https://dart.dev/tools/linter-rules#quick-fixes
    * 参考
        * VSCodeの"editor.formatOnSave"では`dart fix --apply`が実行される
* (?)静的解析（`dart analyze`）は `dart compile` や `dart run` 実行時の 解析と同じ?
    * 基本的には同じ?
    * 関連
        * https://github.com/dart-lang/sdk/issues/51800
* analysis_options.yaml をカスタマイズすることで、リンターパッケージの指定、リンターやアナライザのカスタマイズが可能。
    ```yaml
    # 公式サンプルコード
    include: package:lints/recommended.yaml

    analyzer:
      exclude: [build/**]
      language:
        strict-casts: true
        strict-raw-types: true

    linter:
      rules:
        - cancel_subscriptions
    ```
    * 全体でカスタマイズせずに、ソースコード内に直接記述してファイルや行単位で例外扱いにすることも可能。
        * `// ignore: name_of_lint` 
        * `// ignore_for_file: name_of_lint`
    * コンストラクタ呼び出しの際のconst可能なものはconstを優先させる
        * `prefer_const_constructors: true` 
        * こちらは以前のFlutterではデフォルトでtrueとなっていたが、オプトインに変更されていたようだ。
        * (参考)関連issue
            * https://github.com/dart-lang/core/issues/833
            * https://github.com/dart-lang/core/issues/833#issuecomment-2556952070
            * IMO
                * https://github.com/dart-lang/core/issues/833#issuecomment-2556951886
                * 現状は「constをlinterで開発者に強制させる」ほどの「一般的なプロジェクトにおけるconstの測定可能なメリット」が十分にないためこの選択に至ったと考えられる。

# dart format
* https://dart.dev/tools/dart-format
* [Dart フォーマット ガイドライン](https://dart.dev/effective-dart/style#formatting)に従ったフォーマット
* `dart format`で実行
* dart formatで修正される内容は、静的解析(Static analysis)とは別のもの
* 参考
    * VSCodeの"editor.formatOnSave"では`dart format`が実行される
    