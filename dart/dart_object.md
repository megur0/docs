- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# オブジェクト
> Dart is an object-oriented language with classes and mixin-based inheritance. 
  * https://dart.dev/language/classes
* Dartはオブジェクト指向言語
* オブジェクトはクラスのインスタンス
* 関数もリテラルもnullも全てオブジェクト
    * Dartは真のオブジェクト指向言語のため、関数もオブジェクトであり、Functionという型を持つ。
* null以外はObjectクラスを継承。
* オブジェクトは関数とデータを持つ（メソッドとvariable）
  * メンバへのアクセスは"." で行う
* 以下は同じ。
    * `var p1 = Point(2, 2);`
    * `var p1 = new Point(2, 2);`

# identical
* オブジェクトが同一かどうかを判定する
```
expect(identical(Object(), Object()), false);
```

# ==メソッド
* == はメソッドである。
  > 1. If x or y is null, return true if both are null, and false if only one is null.
  > 2. Return the result of invoking the == method on x with the argument y. (That’s right, operators such as == are methods that are invoked on their first operand. For details, see Operators.)
* クラス作成時にオーバーライドすることができる。
  * デフォルト は identical の処理となる。
  * インスタンスの同一性ではなく、フィールドの同一性で==を使いたい際にオーバーライドして独自の実装を行えば良い。


# Object.runtimeType
* ランタイムの型の取得


# 型テスト演算子
## as
* 型アサーション
* アサーションに失敗すると実行時エラーとなる。
## is
* is での比較と、== で比較は異なる。
* is の否定形は is! 


# サンプルコード
```
void main() {
  print(Person == Person); // true
  print(Person == Taro); // false

  final taro = Taro();
  print(taro.runtimeType == Taro);   // true

  final Person p1 = Taro();
  print(p1.runtimeType == Person);  // false
  print((p1 as Person).runtimeType == Person);  // false

  // オブジェクトではないためisでの比較はfalseとなる
  print(p1.runtimeType is Taro);   // false
  print(p1.runtimeType is Person);  // false

  print(taro is Taro);  // true。常に真のため警告が表示される
  print(taro is Object);  // true。常に真のため警告が表示される
  print(taro is Person);  // true。常に真のため警告が表示される
  print(10 is Object);     // true。常に真のため警告が表示される
  print(null is Object);   // false（Dart 2.12 より前は false）
}
class Person{}
class Taro extends Person{}
```
* 参考
  * https://qiita.com/kabochapo/items/925a2cee9199031272df


# print
* https://api.flutter.dev/flutter/dart-core/print.html
* Object? を引数にとりコンソールへ出力する。
* String以外のオブジェクトはオブジェクトのtoStringメソッドが呼ばれる。

# (参考)pretty print
* 任意のオブジェクトの構造を動的に取得してpretty printするにはリフレクションを使えば可能?
* リフレクションはFlutterでは使えない。
* Flutterではinspect関数が用意されている。
    * デバッガーが起動している状態でinspectで出力すると構造を見ることができる。
    * ただし、デバッガー経由と成るため、コンソール上で出力したいケースでは使えない?








