[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# ドキュメント
* ウィジェットの基礎（ベーシックなウィジェット、ウィジェットのビルドの仕組み）
* https://docs.flutter.dev/development/ui/widgets-intro

# ウィジェットにおける責務
* Flutterでは可能な限り継承ではなく、Compositionを使っている。
* たとえば"padding"というものをTextウィジェットに持たせるのではなく、Paddingというウィジェットを使い責務を分けている。
* IMO
    * 内部実装では継承はかなり多い。
    * ただ、責務が異なる機能はCompositionの使用、同じ責務のクラスの拡張は継承という使い分けがされている。

# ウィジェットは@immutable
* Widgetクラスは@immutableになっている
    ```
    @immutable
    abstract class Widget extends DiagnosticableTree {
        //...
    }
    ```
* したがってconstコンストラクタではない場合はアナライザによって警告が表示される。
    * https://dart.dev/tools/diagnostic-messages#must_be_immutable

    ```
    class A extends StatelessWidget {
    A({super.key, this.a = true});
    bool a;

    @override
    Widget build(BuildContext c) {
        return const Text("a");
    }
    }
    // This class (or a class that this class inherits from) is marked as '@immutable', but one or more of its instance fields aren't final: A.adartmust_be_immutable
    ```


# ステートレス、ステートフルなウィジェット
* ステートレスなウィジェット
    * Icon, IconButton, Text 等
    * Stateless widgets は StatelessWidget のサブクラスである。
* ステートフルなウィジェット
    * Checkbox, Radio, Slider, InkWell, Form, TextField 等
    * Stateful widgets は StatefulWidget のサブクラスである。

# Scaffold
* Scaffoldはよく使われるhelpfulなwidget
* 以下を提供する
    * a default color
    * has API for 
        * adding drawers, 
        * snack bars, 
        * bottom sheets.
* 主なプロパティ
    * appBar, drawer, body

# レイアウトの基本的なウィジェット
* AppBar
* Container
* Align
* Center
* SizedBox
* Column, Row
* Stack
* Expanded
* Flexible 
* Spacer
* Divider
* Padding
* Border
* ListTile
* TabBar, TabBarView
* Card
* ConstrainedBox
* UnconstrainedBox
* OverflowBox
* LimitedBox
* FittedBox
* Table, DataTable

# テキスト関連
* Text
* RichText
* TextField
* TextFormField

# カラー
* Colors.blue
* Color.fromRGBO(0, 255, 0, 1.0)
* Color(0xFF00FF00)

# アイコン
* Icon
* CircleAvatar
* FlutterLogo
* アイコンは下記から探すことができる。
* https://fonts.google.com/icons
* 例 `Icon(Icons.notifications)`

# ボタン関連
* TextButton
    * テキストボタン
* ElevatedButton
    * https://api.flutter.dev/flutter/material/ElevatedButton-class.html
    * フラットなレイアウトに立体感を加えるために使用。既にelevatedなコンテンツ上では使用しないほうが良い。
    > Avoid using elevated buttons on already-elevated content such as dialogs or cards.
* OutlinedButton
    * アウトラインボタン
* 以下は廃止となっている為注意
    * https://docs.flutter.dev/release/breaking-changes/buttons
    * FlatButton, RaisedButton, OutlineButton(ButtonTheme)

# 画像関連
* Image.asset
    * アプリ内(アセット)の画像を表示
* Image.network
    * ネットから画像を表示
    * 内部ではNetworkImage(ImageProviderの派生クラス)を利用している。
* Image.file
    * ローカルファイル形式で画像を表示
    * pubspec.yamlに設定が必要
* Image.memory
    * Uint8List形式で画像を表示
* ImageProvider
    * https://api.flutter.dev/flutter/painting/ImageProvider-class.html
    * Flutterの画像関連のウィジェットでは内部でImageProviderを利用して画像を読み込みしている。
    * 派生する具象クラスとして、FileImage, ResizeImage, AssetImageやMemoryImage等がある。
    ```
    const assetImage = AssetImage('assets/Dash.png');
    final 1pixeldot = MemoryImage(base64Decode("R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw=="));

    testWidgets(i.toString(), (widgetTester) async {
        await widgetTester.pumpWidget(MaterialApp(
            home: Scaffold(
            body: Center(
            child: CircleAvatar(
                radius: 100,
                //backgroundImage: 1pixeldot,
                backgroundImage: assetImage,
            ),
            ),
        )));
    });
    ```

# 非同期関連
* FutureBuilder
* StreamBuilder

# リスナー、ジェスチャー関連
* ListenableBuilder
* NotificationListener
* GestureDetector

# スクロール関連
* https://api.flutter.dev/flutter/widgets/ListView-class.html
* ListView, ListView.Builder
* SingleChildScrollView
* GridView
* ListTile
    * ListViewやCardでよく使われる。
    > Organizes up to 3 lines of text, and optional leading and trailing icons, into a row.
    > Less configurable than Row, but easier to use
* SwitchListTile
* RefreshIndicator
## page by page
* PageView
    * https://api.flutter.dev/flutter/widgets/PageView-class.html

