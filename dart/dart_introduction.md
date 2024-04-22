- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# ドキュメント・ライブラリ
* https://dart.dev/guides
* 言語ツアー
    * https://dart.dev/guides/language/language-tour
* ライブラリツアー
    * https://dart.dev/guides/libraries/library-tour
    * dart:core、dart:async、dart:math、dart:convert、dart:html、dart:io
* Effective Dart
    * https://dart.dev/guides/language/effective-dart
* 公式によるパッケージ
    * https://pub.dev/publishers/dart.dev/packages
    * コアライブラリを補完する公式によるパッケージ
    * http, path, logging, ffi, collection, async, test, js, intl, meta, mockito, os_detect など。


# About Dart
* https://dart.dev/overview#platform
* https://dart.dev/guides/language/effective-dart
* https://www.youtube.com/watch?v=5F-6n_2XWR8
> Dart is a client-optimized language for developing fast apps on any platform. Its goal is to offer the most productive programming language for multi-platform development, paired with a flexible execution runtime platform for app frameworks.  

> Dart provides the language and runtimes that power Flutter apps, but Dart also supports many core developer tasks like formatting, analyzing, and testing code.

> Dart was designed to be familiar, so it inherits many of the same statements and expressions as C, Java, JavaScript and other languages. But we created Dart because there is a lot of room to improve on what those languages offer. We added a bunch of features, from string interpolation to initializing formals, to help you express your intent more simply and easily.

> Be consistent. Be brief. 
* マルチプラットフォーム開発のための言語
    * ゴールは、アプリフレームワーク用の柔軟なランタイムを活用しマルチプラットフォーム開発の生産的なプログラミング言語を提供すること
* タイプセーフ
* nullセーフ
* 標準ライブラリの内容が豊富。
* Dart VM 
    * クライアントサイド / サーバーサイド
    * JITコンパイラ / AOTコンパイラ
        * JITコンパイラによってHot reloadを実現
        * 常に JIT コンパイルがされるわけではなく、AOT のパイプラインがある
    * Dart コードを実行する２種類の方法
        * ソースコードからコンパイルする方法
        * スナップショットから作成する方法
* webの方はJavaScriptにコンパイルしている。
* 使用するプラットフォームやコンパイル方法に限らず、Dartはランタイム（GCや一部の動的型チェック、アイソレートの制御）を必要とする。
    * ネイティブ プラットフォームでは、Dart ランタイムは自己完結型の実行可能ファイル内に自動的に含まれ、dart runコマンドによって提供される Dart VM の一部
* flutter SDKの中にはDartのライブラリが入っている
    * https://api.flutter.dev/index.html


# クラス、OOP志向
* DartやOOP, クラスを中心としたプログラミング
* Flutterのウィジェットはクラス
* Reactのコンポーネントは関数
    * https://ja.react.dev/learn/your-first-component#defining-a-component
    > React コンポーネントとは、マークアップを添えることができる JavaScript 関数



# Dartコードのコンパイル
* https://docs.flutter.dev/resources/faq#run-ios
* 参考
    * https://zenn.dev/tsuruo/articles/48909d22d49ffe
## リリースモード
* 機械語で実行（AOTコンパイル）
    * Dart コード（アプリケーションコードと SDK 両方）は機械語に AOT コンパイルされる。
* Dart VMも利用する
    * リリースモード（AOT コンパイル）においても Dart VM はランタイムに絞った軽量な形で利用されている
    * 理由は、ガベージコレクタや Dart ライブラリのネイティブメソッドの提供などで、これらはデプロイされたアプリケーション上にも存在。
    * コンパイルされた機械語は、precompiled runtime と呼ばれる軽量な（JIT パイプラインのサブセット）Dart VM 上のランタイム上で実行される
## デバッグモード/プロファイルモード
* 中間表現で実行（JITコンパイラ）
    * https://docs.flutter.dev/tools/hot-reload
    * ホットリロード
    * Dart VM上で動作
    * デバッグおよびプロファイルモードの時はJITコンパイラで、ホットリロードが提供される。
* JIT
    * Kernel ASTs （.dill）と呼ばれる中間表現（抽象構文木のバイナリ）に一度変換し、それを VM 上で実行
    * dart run コマンドは内部的には Dart VM の JIT コンパイラを使っている。
