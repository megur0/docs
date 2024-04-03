- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# nullable
* 型に?をつけると、nullableになる。初期化していない場合はnullが入る。
```
 int? a;
 print(a);
```

# null safety
* https://dart.dev/null-safety
* not null が保証されていない（or 保証されていない可能性があるとコンパイラが判断する）変数についてコンパイルエラーを出すこと。
* 参考
  * https://medium.com/flutter-jp/null-safety-fe6503a81d5c

# null assertion
* null safetyのデメリットとして、このチェックはコンパイラの推論によるものであり、実行時にnullになりえないケースでもコンパイルエラーとしてしまうものが多くあること。
* この場合、!(非nullであることのassert)を使う。
```
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
```
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
```
if (a != null && a.b != null) {
  // ....
}
// このように短く書ける場合もある
if (a?.b != null) {
  // ....
}
```

# 省略記法
```
 int? a;
 int b = a ?? 4;
 a ??= 3;
```
