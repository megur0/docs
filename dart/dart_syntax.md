[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > 文法


# 前置、後置
```dart
void main() {
  int a = 0;
  final test = (int a) => print;
  test(a++);//0
  test(++a);//2
}
```

# comments
* Single-line comments
* Multi-line comments
    * `A multi-line comment begins with /* and ends with */`
* Documentation comments
    * `///` もしくは `/**`
    * []で囲んだクラスやメソッドなどにドキュメント上でリンクが貼られる。
    * `dart doc`でhtmlのドキュメント作成ができる

# 文（statement）
## 式（expression）
* Conditional expressions
    * `condition ? expr1 : expr2`
    * `expr1 ?? expr2`
* forやifのexpression
    * `[for (var i in [1,2,3]) '#$i', if(true)"#4"].forEach(print);`
## control statement
* if statement
    * The statement conditions must be expressions that evaluate to boolean values, nothing else. 
* for
    ```dart
    for(int i = 0; i<10; i++) 
      print('Hello World');
    }
    ```
    * for in
      * Iterableを実装するクラスのオブジェクトは inが使える。
      ```dart
      for(final value in [1,2,3]){
          print(value);
      }
      for(final value in {"a":"b"}.entries){
          print(value);
      }
      ```
* while,do-while
* break,continue
* where, forEach
* switch


# label
* ネストしたループのbreakやcontinueなど。
  * IME 使ったことがない。
* 参考
  * https://stackoverflow.com/questions/70300104/how-to-break-out-of-nested-loops-in-dart



# オペレーター
* https://dart.dev/language/operators
## Bitwise and shift operators
* https://dart.dev/language/operators#bitwise-and-shift-operators
* and
  * &
* or
  * |
* xor
  * ^
## assignment-operators
* https://dart.dev/language/operators#assignment-operators
* Null-aware operators
  * ??= 演算子
  * b ??= value;
## spread operator（スプレッド演算子）
```dart
var arrayInt = [1, 2, 3];
var arrayString = ['a', 'b', 'c'];
var addArray = [...arrayInt, ...arrayString];
print(addArray); // [1, 2, 3, a, b, c]
```
## カスケード（Cascade notation）
* https://dart.dev/language/operators#cascade-notation
```dart
List<String> test = ['a'];
test.add('b');
test.add('c');
// 上記は以下のように書くことができる
List<String> test = ['a']..add('b')..add('c');
```
## 算術演算子
* https://dart.dev/language/operators#arithmetic-operators
```dart
assert(2 + 3 == 5);
assert(2 - 3 == -1);
assert(2 * 3 == 6);
assert(5 / 2 == 2.5); // Result is a double
assert(5 ~/ 2 == 2); // Result is an int
assert(5 % 2 == 1); // Remainder
assert('5/2 = ${5 ~/ 2} r ${5 % 2}' == '5/2 = 2 r 1');
```

# pattern, switch, if-case(WIP)
* https://dart.dev/language/patterns
* https://dart.dev/language/branches
## Dart3のswitch
* breakがなくても、次のcase を実行することがなくなった。
* enumが網羅されていない場合はエラーを返す
* 一つのcaseの中に複数の項目を設定することができる
```dart
enum Test{
  a,
  b,
  c
}
void main() {
  Test t = Test.a;
  
  switch(t) {
      case Test.a:
      case Test.b:
      case Test.c:
        print("c");
  }
  
  switch(t) {
      case Test.a || Test.b || Test.c:
        print("c");
  }
}
// c
// c
```
## (参考) enum以外にswitch構文を使ってみる
* IMO
  * switchによる網羅性のメリットは無いため、この場合はswitchで書くメリットはあまり無さそうと考える。
  * 値の関係性がもう少し複雑でif文のネストが深くなる場合などはswitch文でフラットに書ける場合はありそうだが、好みの範疇かもしれない。
```dart
int compare(int a, int b) {
  return switch ((a, b)) {
    (int c1, int c2) when c1 == c2 => -1,
    (int c1, int c2) when c1 > c2 => 0,
    (int c1, int c2) when c1 < c2 => -1,
    (_, _) => throw "unexpected" // (_, _)を入れないとエラー
  };
}

int compare2(int a, int b) {
  switch ((a, b)) {
    case (int c1, int c2) when c1 == c2:
      return 0;
    case (int c1, int c2) when c1 > c2:
      return 1;
    case (int c1, int c2) when c1 < c2:
      return -1;
    default:
      throw "unexpected"; // defaultを入れないとエラー
  }
}
```
