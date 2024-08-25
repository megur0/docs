[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# nullable
* 型に?をつけると、nullableになる。初期化していない場合はnullが入る。
```dart
 int? a;
 print(a);
```

# null safety
* https://dart.dev/null-safety
* not null が保証されていない（or 保証されていない可能性があるとコンパイラが判断する）変数についてコンパイルエラーを出すこと。

# null assertion
* null safetyのデメリットとして、このチェックはコンパイラの推論によるものであり、実行時にnullになりえないケースでもコンパイルエラーとしてしまうものが多くあること。
* この場合、!(非nullであることのassert)を使う。
```dart
class A{
  A(this.a);
  final Function()? a;
  
  void b() {
    if (a != null) {
      // a(); // 明らかにnullではないがコンパイルエラー
      a!();
    }
  }
}
```
* null assertionは、nullだった場合は実行時エラーが発生する。したがって確実にnullではない場合に使う。
```dart
void main() {
  String? test;
  print('$test');
  print('${test?.toUpperCase()}');
  print('${test!.toUpperCase()}');//ランタイムエラー
}
```

# Conditional　access
* Conditional subscript access
     * `fooList?[1]`の値が存在しない場合はnullを返す。
* Conditional member access
    * `foo?.bar`はfooがnullの場合はnullを返す。
* Conditional function ？（名称は不明）
  * `obj?.hoge()`は objがnullの場合はhogeを実行しない。
* Conditional　accessが役に立つシーン。
```dart
if (a != null && a.b != null) {
  // ....
}
// このように短く書ける場合もある
if (a?.b != null) {
  // ....
}
```

# 省略記法
```dart
 int? a;
 int b = a ?? 4;
 a ??= 3;
```

# (参考)null と 値自体が渡されていないことの区別
* optionalな引数や、名前付き引数において、引数がnullableの際に「nullとして渡されたのか？」「渡されていないのか？」を直接的に区別する事はできない。
* https://stackoverflow.com/questions/30830592/checking-if-optional-parameter-is-provided-in-dart/30838348#30838348
* https://www.reddit.com/r/dartlang/comments/u3o43q/can_i_differentiate_between_a_parameter_not_being/
* これは例えばインスタンス変数をコピーして別オブジェクトを作成するメソッドを実装などで考慮が必要となる。
  * https://stackoverflow.com/questions/68009392/dart-custom-copywith-method-with-nullable-properties
