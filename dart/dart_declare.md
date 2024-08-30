[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > 宣言


# 宣言
* Dartではトップレベルの変数が利用可能。
* 型推論
    * 型推論があるため型宣言は必須ではない
* 値は宣言時に初期化しなくても良い
    * 使用されるまでに初期化しないとコンパイルエラとなる。

# var, final, const
* constは、Dart独特の機能
  * const は それ以降で同じ引数で別の変数でオブジェクトを代入しても、両者はまったく同じオブジェクトが使われる。
  ```dart
  void main() {
    var a = const A(1);
    var b = const A(1);
    var c = const A(2);
    assert(identical(a, b));
    assert(!identical(a, c));// 引数が異なると異なるオブジェクトとなる。
  }
  class A {
    const A(this.a);
    final int a;
  }
  ```
  * IMO
    * Flutterでの開発においては、ウィジェットを書くことはリンターにconst化を促される事といっても過言ではないくらいに、コード中にconstを書くことになる。
* finalとconstの違い
  * constはコンパイル時点で定数の必要がある。
  * たとえばDateTime.now()のような値は実行時点で決定するためconstとして代入することは不可。
```dart
void main() {
  const test = "333";
  //test = "444"; // error
  print(test);
  
  final test2 = "444";
  //test2 = "444"; // error
  print(test2);
  
  var test3 = "555";
  test3 = "556";
  print(test3);
  
  //var test4 = const "666"; //error. 既にconstである値にconstをつけることはできない?
  
  var constantList = const [1,2];
  // constantList[1] = 1; // error
  // constantList.add(1); // error
  constantList = [1,2,3];
  print(constantList);// [1,2,3]
}
```
```dart
class A {
  A(this._test);
  int _test;
}
void main() {
  final aa = A(4);
  aa._test = 100;// finalで宣言した場合でもオブジェクトの中身の変更は可能。
  //const aaa = const A(5);// Aは定数constructorではないためcompile error.
}
```
```dart
void main() {
  const b = B(3);// 定数constructorのためconstとして作成できる。
  var bb = B(3);// constと書かなくてもconst扱いになる。
  var bbb = const B(3); // varで宣言しても右辺がconstならconst
  final bbbb = B(4);// finalで宣言してもconst扱いになっているはず。
}

class B {
  const B(this.test);
  final int test;
}
```
## constの省略
```dart
const pointAndLine = const {
  'point': const [const ImmutablePoint(0, 0)],
  'line': const [const ImmutablePoint(1, 10), const ImmutablePoint(-2, 11)],
};
```
* 上記は下記のように省略できる。
```dart
const pointAndLine = {
  'point': [ImmutablePoint(0, 0)],
  'line': [ImmutablePoint(1, 10), ImmutablePoint(-2, 11)],
};
```

# (参考)関数リテラルはconstではない
* 関数リテラル（例えば、(){}）は、定数ではなく都度、新しいインスタンスを作成する。
* したがって以下のようにconst値に代入するとエラーとなる。
```dart
void example1() {
  // const Function bar = () {}; // error.
  const Function bar = a; // ok.
}
void a() {} // トップレベルの関数はconstとなる
```
> This is because all functions in dart inherit Function class which doesn't have const constructor.
* https://stackoverflow.com/questions/75027076/why-cant-you-declare-constant-function-literals-in-dart
## 関連：Constant function literals
* https://github.com/dart-lang/language/issues/1048


# (参考)引数のデフォルト値のconstは省略ができない。
* 下記はエラーとなる。
  ```dart
  void f([List<int> a = []]) {} // エラー: The default value of an optional parameter must be constant.
  ```
* エラーは`void f([List<int> a = const []]) {}` とすると解消する。
* `[]`の時点でconstになると筆者は考えてしまったが、constをつけなければconstとはならない。
* なお下記は右辺がconstと推論されるためエラーにならない。
  ```dart
  const List<int> a = []; // エラーにならない
  ```

# late
* トップレベル変数やクラス変数の初期化処理は、その変数が初めて使われる際に実行される。（lazily initialized）
* 変数は使われるときまでに初期化されていれば良いが、それをコンパイラが上手く検出できないときがあり、その場合にlateを利用する
* lateをつけると初期化処理を使用直前にするため、使用する場合だけ初期化処理をさせたい場合などに使える。
* なお、イミュータブルなクラス（const コンストラクタ）の場合は lateのフィールドは使えない。lateが使えるフィールドはミュータブルなクラスのフィールドのみとなる。
```dart
Future<String> fetch(Future<(String, int)> Function() f) async {
    late (String, int) result;
    // int result; // lateを使わない場合は return文で未初期化としてエラーになる。
    try {
      result = await f();
    } catch (e) {
      // ...
    }
    // ...
    return result.$1;
}
```




