[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# ドキュメント・ライブラリ
* ドキュメント
    * https://docs.flutter.dev
* 標準ライブラリリファレンス（package:flutter）
    * https://api.flutter.dev/index.html
* パッケージ
    * https://pub.dev/
    * Flutter favorite
        * https://pub.dev/packages?q=is%3Aflutter-favorite
* ウィジェットカタログ
    * https://docs.flutter.dev/development/ui/widgets



# About Flutter
* https://docs.flutter.dev/resources/faq#what-is-flutter
    > Flutter is Google’s portable UI toolkit for crafting beautiful, natively compiled applications for mobile, web, and desktop from a single codebase. Flutter works with existing code, is used by developers and organizations around the world, and is free and open source.
* https://dart.dev/overview
    > The Flutter framework is a popular, multi-platform UI toolkit that's powered by the Dart platform, and that provides tooling and UI libraries to build UI experiences that run on iOS, Android, macOS, Windows, Linux, and the web.
* 宣言的UIである
    * https://docs.flutter.dev/get-started/flutter-for/declarative


# About Dart
* [Dart](../dart/dart_introduction.md)


# (参考)React Nativeとの比較した各機能
* https://docs.flutter.dev/get-started/flutter-for/react-native-devs
* JSとの比較
    * null safeである
    * 値の初期値はnull
    * ifで使える値はboolean型のみ。
    * 非同期処理は似ている。
* ウィジェットはReact Nativeのコンポーネントに該当する。
    * Reactのpropsのようにwidgetに引数を渡す。
    * Flutterではスクリーン自体もウィジェットとなる。
* 状態を持つStatefulWidget(State)とStatelessWidgetの２つに分かれている
    * Flutterの場合は、ウィジェットが状態を持つか否かはどちらを継承するかによって決める。
    * ReactはどちらもReact.FC型の関数で、状態管理はuseStateに寄って行う。
* 状態の変更はState.setState() によってフレームワークに伝え、フレームワークによるリビルドによってStatefulWidgetとStatelessWidgetのbuildメソッドが呼ばれる。
* React Nativeのviewコンポーネントに相当するwidgetはContainer, Column, Row, and Center等。
* React NativeのFlatList、 SectionListに相当するウィジェットはListView
* キャンバス系はCustomPaint 
* ReactNativeの場合はviewコンポーネントでstyle propsによってスタイルでレイアウトを調整するが、Flutterの場合は各ウィジェットとコントロールウィジェットによって行う。
* アイコン系はIconクラスやCupertino (iOS-style) パッケージ
* テーマはThemeData
* ローカルストレージはSharedPreferences。
* React Nativeの場合は 主要なナビゲーターはStackNavigator, TabNavigator, DrawerNavigatorの３つだがFlutterの場合はRouteとNavigatorで構成される。
    * A Route
        * Abstraction for an app screen or page.
    * A Navigator
        * Widget that manages routes.
* タブに関しては TabController、TabBar、Tab、TabBarView
* ドローワーナビは Drawer。
    * AppBarはドローワーが利用可能な際にメニューを表示
* httpによるデータ取得はhttpパッケージを利用する。
* 入力はTextField, TextFormField.
    * TextEditingControllerを利用して管理する。
* プラットフォームの取得や判定は`Theme.of(context).platform`
* アニメーションはAnimation、AnimationController
