[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > フォント



# デフォルトのフォント
```dart
print(DefaultTextStyle.of(context).style);

/*
// 以下はiOS Simulator環境下での出力

TextStyle(debugLabel: (dense bodyMedium 2021).merge((blackCupertino bodyMedium).apply), inherit: false, color: Color(0xff1c1c14), family: CupertinoSystemText, size: 14.0, weight: 400, letterSpacing: 0.3, baseline: ideographic, height: 1.4x, leadingDistribution: even, decoration: Color(0xff1c1c14) TextDecoration.none)

*/
```
* 参考
    * https://stackoverflow.com/questions/52860153/what-is-the-default-font-family-of-a-flutter-app

# カスタムフォントの利用
* https://docs.flutter.dev/cookbook/design/fonts


# DefaultTextStyle
* https://api.flutter.dev/flutter/widgets/DefaultTextStyle-class.html
* 子孫のTextウィジェットにスタイルを適用する。
* DefaultTextStyle.mergeのコンストラクタを利用すると、部分的にオーバーライドが可能。
## Scaffoldウィジェットを親とするかどうかでテキストの表示が異なるのはなぜか？
* 例えば下記で標準出力へ出力されるスタイルや画面に表示されるテキストはScaffoldで囲むかどうかによって異なる。
```dart
main() => runApp(const MaterialApp(
        //theme: theme,
        home: Scaffold(
      body: MyApp(),
    )));

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    debugPrint(DefaultTextStyle.of(context).style.toString());
    return const Text("test");
  }
}
```
* Scaffold内部ではMaterial/_MaterialStateクラスを使っていて、_MaterialState.build()で Theme.of(context).textTheme.bodyMedium!をstyleに適用しているため。
```dart
contents = AnimatedDefaultTextStyle(
    style: widget.textStyle ?? Theme.of(context).textTheme.bodyMedium!,
    duration: widget.animationDuration,
    child: contents,
);
```
## Overlay
* Overlayは(作成しない限り)WidgetsApp、CupertinoApp、または MaterialAppのNavigatorによって作成されたものを利用することが一般的だが、
* ウィジェットツリー上、MaterialAppのchild以降で指定されるDefaultTextStyleは先祖であるOverlayには(当然ではあるのだが)適用されない。
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  // flutter test --plain-name 'overlay test'
  group("overlay test", () {
    testWidgets("test", (tester) async {
      await tester.pumpWidget(const MaterialApp(
        home: Scaffold(//このScaffoldによって適用されるDefaultTextStyleは、子孫に適用されるため、先祖であるOverlayには適用されない。
          body: MyWidget(),
        ),
      ));
      await tester.tap(find.byType(TextButton));
      await tester.pumpAndSettle();
    });
  });
}

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  Widget build(BuildContext context) {
    print(DefaultTextStyle.of(context).style);
    /*
    TextStyle(debugLabel: (englishLike bodyMedium 2021).merge((blackMountainView bodyMedium).apply), inherit: false, color: Color(0xff1d1b20), family: Roboto, size: 14.0, weight: 400, letterSpacing: 0.3, baseline: alphabetic, height: 1.4x, leadingDistribution: even, decoration: Color(0xff1d1b20) TextDecoration.none)
    */
    return Column(
      children: [
        // これはエラーとならない。(DefaultTextStyleのsizeが14)
        const Text(
          "a\na\na\na\na\na\na\na\na\na\na\na\na\na\na\na\na\n",
        ),
        TextButton(onPressed: createHelpOverlay, child: const Text("button"))
      ],
    );
  }

  OverlayEntry? overlayHelpEntry;

  void createHelpOverlay() {
    overlayHelpEntry = OverlayEntry(
      builder: (BuildContext context) {
        print(DefaultTextStyle.of(context).style);
        /*
        TextStyle(debugLabel: fallback style; consider putting your text in a Material, inherit: true, color: Color(0xd0ff0000), family: monospace, size: 48.0, weight: 900, decoration: double Color(0xffffff00) TextDecoration.underline)
        */
        return const Column(
          children: [
            // 前のTextと同じ行数だがオーバーフローしてエラーとなる。これはDefaultTextStyleのsizeが48となるため。
            Text(
              "a\na\na\na\na\na\na\na\na\na\na\na\na\na\na\na\na\n",
            ),
          ],
        );
      },
    );

    Overlay.of(context, debugRequiredFor: widget).insert(overlayHelpEntry!);
  }

  @override
  void dispose() {
    overlayHelpEntry?.remove();
    overlayHelpEntry?.dispose();
    super.dispose();
  }
}

```