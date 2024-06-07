- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# デフォルトのフォント
```
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