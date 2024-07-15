[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# Material 3
* https://m3.material.io/
* https://m3.material.io/develop/flutter
* https://docs.flutter.dev/ui/design/material
> Material Design is a design system built and supported by Google designers and developers. Material.io includes in-depth UX guidance and UI component implementations for Android, Flutter, and the Web.The latest version, Material 3, enables personal, adaptive, and expressive experiences – from dynamic color and enhanced accessibility, to foundations for large screen layouts and design tokens.
* Googleが提唱するユーザーが直感的に操作できるようにするためのデザイン。
* フラットデザインとは異なり、影があったり質感のある直感的なデザインとなる。
    * 奥行きを考える
    * 現実世界の物理法則に従う
    * アニメーションはユーザーを起点にする
* Floating Action Button(FAB)
    * フローティングアクションボタンはひとつのスクリーンでなるべく一つ置き、 その画面でもっとも一般的な動作を促すために使うことを推奨されている。


# Flutterのmaterialライブラリ
* Material library
    * マテリアルデザインに準拠したウィジェット
    * UI構築にあたってはWidgets library, Material libraryの両方利用可能である。また自身で作成することも可能。
* flutter/material.dart は flutter/widget.dartを継承している。
    * flutter/material.dartをimportすると、flutter/widget.dartの内容も利用可能

# マテリアルデザインの有効化
* Flutterでマテリアルデザインを利用する方法
    * Flutter 3.16以降では、マテリアル 3 がデフォルトで有効になっている。
    * pubspec.yaml
        * `uses-material-design: true`
    * ThemeData
        * `useMaterial3: true` 
    * アプリケーションコード
        * `import 'package:flutter/material.dart';`
    
    
    
    * colorSchemeSeedを指定することで、colorSchemeが自動的に設定される。
        * 設定されたcolorSchemeはマテリアルデザインのダイナミックカラーに基づいた色が設定される。


# 利用方法
* ColorScheme
    * https://api.flutter.dev/flutter/material/ColorScheme-class.html
        > A set of 30 colors based on the Material spec that can be used to configure the color properties of most components.
    * ほとんどのコンポーネントの色のプロパティを構成するために使用可能な30色のセット。
    * 主要なアクセントカラーは、primary, secondary, tertiary
        * primary
            * 目立つボタン、アクティブ状態など、UI 全体の主要なコンポーネントに使用
        * secondary
            * フィルター チップなどの UI 内のあまり目立たないコンポーネントに使用
        * teriary
            * 一次色と二次色のバランスをとったり、入力フィールドなどの要素への注目を高めるために使用
    * スキームの残りの色は、背景と表面に使用される中間色と、エラー、分割線、影に使用される特定の色で構成
* ThemeData
    * 生成時にcolorSchemeプロパティを指定することでカスタムColorSchemeに設定することができる。
    * ColorScheme.fromSeedコンストラクタを利用すると、シードカラーを基にしてMaterial3のカラーシステムに基づいたColorSchemeを生成。
        * これらの色はアクセシビリティのためのコントラスト要件を満たすように設計されている。
        * https://api.flutter.dev/flutter/material/ColorScheme/ColorScheme.fromSeed.html
    * なお、colorSchemeSeedプロパティからシードカラーを指定することも可能。(内部ではColorScheme.fromSeedが呼ばれている。)
        * ※ colorScheme: ColorScheme.fromSeed〜と、colorSchemeSeed:〜を同時に指定するとエラーになる。
    * (参考)シードカラーと生成されるColorSchemeは以下のサイトで確認すると便利
        * https://colorscheme.enoiu.com/
```
import 'package:flutter/material.dart';

const seedColor = Color.fromARGB(255, 164, 206, 255);

final theme = ThemeData(
  useMaterial3: true,
  colorScheme:
      ColorScheme.fromSeed(seedColor: seedColor, brightness: Brightness.light),
);

main() => runApp(MaterialApp(
    theme: theme,
    home: Scaffold(
      body: Column(
        children: [
          Row(
            children: [
              const Text("seed color:"),
              Container(
                decoration: const BoxDecoration(
                  color: seedColor,
                  borderRadius: BorderRadius.all(Radius.circular(10)),
                ),
                child: const SizedBox(
                  width: 10,
                  height: 10,
                ),
              )
            ],
          ),
          for (final (Color, Color, String, String) c in [
            (
              theme.colorScheme.primary,
              theme.colorScheme.onPrimary,
              "primary",
              "onPrimary",
            ),
            (
              theme.colorScheme.secondary,
              theme.colorScheme.onSecondary,
              "secondary",
              "onSecondary",
            ),
            (
              theme.colorScheme.tertiary,
              theme.colorScheme.onTertiary,
              "tertiary",
              "onTertiary",
            ),
            (
              theme.colorScheme.primaryContainer,
              theme.colorScheme.onPrimaryContainer,
              "primaryContainer",
              "onPrimaryContainer",
            ),
            (
              theme.colorScheme.secondaryContainer,
              theme.colorScheme.onSecondaryContainer,
              "secondaryContainer",
              "onSecondaryContainer",
            ),
            (
              theme.colorScheme.tertiaryContainer,
              theme.colorScheme.onTertiaryContainer,
              "tertiaryContainer",
              "onTertiaryContainer",
            ),
            (
              theme.colorScheme.background,
              theme.colorScheme.onBackground,
              "background",
              "onBackground",
            ),
            (
              theme.colorScheme.error,
              theme.colorScheme.onError,
              "error",
              "onError",
            ),
            (
              theme.colorScheme.errorContainer,
              theme.colorScheme.onErrorContainer,
              "errorContainer",
              "onErrorContainer",
            ),
            (
              theme.colorScheme.inversePrimary,
              theme.colorScheme.primary,
              "inversePrimary",
              "primary",
            ),
            (
              theme.colorScheme.inverseSurface,
              theme.colorScheme.onInverseSurface,
              "inversePrimary",
              "onInverseSurface",
            ),
          ])
            Container(
              decoration: BoxDecoration(
                color: c.$1,
                borderRadius: const BorderRadius.all(Radius.circular(10)),
              ),
              child: Text(
                "backgrond color: ${c.$3}, text color: ${c.$4}",
                style: TextStyle(color: c.$2),
              ),
            )
        ],
      ),
    )));

```


# Material系のウィジェットは先祖にMaterialウィジェットを必要とする
* したがって先祖にMaterialウィジェットを置くか、Materialウィジェットを生成する(かつ先祖にMaterialウィジェットを必要としない)ウィジェットが必要となる   
    * 例えば、Card, Dialog, Drawer, or Scaffoldウィジェットなど
```
testWidgets("", (widgetTester) async {
    await widgetTester.pumpWidget(MaterialApp(
      // home: ListTile(),// assert error
      home: Material(child: ListTile(),),
    ));
  });
```
* https://stackoverflow.com/questions/43947552/no-material-widget-found
* (参考) 実装
```
// lib/src/material/list_tile.dart
class ListTile extends StatelessWidget {
    // ...
    @override
    Widget build(BuildContext context) {
        assert(debugCheckHasMaterial(context)); // 先祖のMaterialウィジェットが無い場合にassert error
        // ...
    }
    // ...
}
```


# その他
## (参考)Scaffoldウィジェットを親とするかどうかでテキストの表示が異なるのはなぜか？
* 例えば下記で標準出力へ出力されるスタイルや画面に表示されるテキストはScaffoldで囲むかどうかによって異なる。
```
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
    ```
    contents = AnimatedDefaultTextStyle(
        style: widget.textStyle ?? Theme.of(context).textTheme.bodyMedium!,
        duration: widget.animationDuration,
        child: contents,
    );
    ```
