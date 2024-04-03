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

* タイプセーフ
* nullセーフ
* 標準ライブラリの内容が豊富。
* DartのVMはJITコンパイラによってHot reloadを実現し、AOTコンパイラ（事前コンパイラ。一般的にコンパイラはこちらを指す事が多い）によって機械語（ARM or x64）を生成する。
* webの方はJavaScriptにコンパイルしている。
* 使用するプラットフォームやコンパイル方法に限らず、Dartはランタイム（GCや一部の動的型チェック、アイソレートの制御）を必要とする。
    * ネイティブ プラットフォームでは、Dart ランタイムは自己完結型の実行可能ファイル内に自動的に含まれ、dart runコマンドによって提供される Dart VM の一部
* flutter SDKのの中にはDartのライブラリが入っている
    * https://api.flutter.dev/index.html
