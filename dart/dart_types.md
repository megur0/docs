[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > 型



# Dartsの型システム
* https://dart.dev/language/type-system
* Dartはタイプセーフ
* 静的型チェックと実行時チェックを組み合わせて使用する。
* 静的型チェックの利点の 1 つは、Dart の静的解析を使用して実行前に不具合を発見できること
## sound type system
* 健全な型システムとは、ある式が評価されたときに、その式の静的型と一致しない値が返されるような状態にならないことを意味する
* Java や C# の型システムと同様にDartも健全な型システムである。


# 型
* num
* int, double
  * numのサブタイプ
* bool
* String
* Map
* List
* Set
* dynamic
* Object

# Object?とdynamic
* https://dart.dev/effective-dart/design#avoid-using-dynamic-unless-you-want-to-disable-static-checking
* すべてのオブジェクトを許可することを単に表明したい場合はObject?やObjectを使う。
* dynamicは、すべてのオブジェクトを受け入れるだけでなく、すべての操作も許可する。
```dart
void f(Object o) {
  o.toString();
  o.aaaa();//コンパイルエラー
}

void f2(dynamic o) {
  o.toString();
  o.aaaa();//コンパイルエラーにならない
}
```

# String
* いろいろな書き方がある。
```dart
var s0 = "String";
var s1 = 'String '
    'concatenation'
    " works even over line breaks.";
print(s1) // String concatenation works even over line breaks.
var s2 = 'The + operator ' + 'works, as well.';
var s3 = '''
You can create
multi-line strings like this one.
''';
var s4 = """This is also a
multi-line string.""";
```
* 変数の展開
  * `'${s.toUpperCase()} is very handy!'`
  * `'Dart has $s, which is very handy.'`
## DartではStringはUTF-16でエンコードされる
> The characters of a string are encoded in UTF-16. Decoding UTF-16, which combines surrogate pairs, yields Unicode code points. Following a similar terminology to Go, Dart uses the name 'rune' for an integer representing a Unicode code point. Use the runes property to get the runes of a string.
* https://api.dart.dev/stable/2.18.7/dart-core/Runes-class.html
* runeプロパティから rune(Unicode コード ポイント) を取得できる。
```dart
const string = 'Dart';
final runes = string.runes.toList();
print(runes); // [68, 97, 114, 116]
```


# Iterable
* https://dart.dev/codelabs/iterables
* https://api.dart.dev/stable/3.3.4/dart-core/Iterable-class.html
* コレクション
  * 要素と呼ばれるオブジェクトのグループを表すオブジェクト
* Iterable 
  * コレクションの一種
  * DartではIterableは抽象クラスで、リスト、セットがこれを実装する
  * マップはIterableではないが プロパティのentriesやvalues などを利用することでIterableオブジェクトを利用できる。
## リスト
* `var names = <String>['Seth', 'Kathy', 'Lars'];`
* 末尾の要素に,をつけることができる。
```dart
var list = [
  'Car',
  'Boat',
  'Plane',
];
```
```dart
void main() {
  final l = [3,2,5,7,0];
  l.sort((int a, int b) => a - b);// 昇順
  print(l);
  l.sort((int a, int b) => b - a);// 降順
  print(l);
  l.sort((int a, int b) => a.compareTo(b));// 昇順
  print(l);
  l.sort((int a, int b) => b.compareTo(a));// 降順
  print(l);
}
```
## セット
* セットは値の重複不可。
* `var uniqueNames = <String>{'Seth', 'Kathy', 'Lars'};`
* `var halogens = {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};`
* Iterableクラスを実装しているため、リストと同様にコレクション操作(for文など)が可能。
* mapのリテラルと表記が似ている点に注意。
```dart
Set<String> names = {};//Creates a set
var names = {}; // Creates a map, not a set.
<String>{"aaaa"}.contains("aaaa"); // set
<String, String>{"aaaa": "aaaa"}.containsKey("aaaa"); // map
```
## マップ
```dart
var pages = <String, String>{
  'index.html': 'Homepage',
  'robots.txt': 'Hints for web robots',
  'humans.txt': 'We are people, not machines'
};
pages['spaces.txt'] = 'space';
print(pages['index.html']);
print(pages['notFound.html']);// null。キーがない場合はnullとなる。
print(pages[4]);// 型が異なってもエラーにならない。（警告は出る）
var empty = <String, String>{};// 空のマップ
```
## Pattern
* https://dart.dev/language/patterns#destructuring
```dart
var numList = [1, 2, 3];
var [a, b, c] = numList;
print(a + b + c);// 6
var (d, [e, f]) = ('str', [1, 2]);
for (var MapEntry(:key, value: c) in {'a': 23,'b': 100}.entries) {
  print('$key, $c');
}
//a, 23
//b, 100
```
## キーや値の取り出し、リスト<->マップの変換、マッピング、抽出、繰り返し
```dart
print({'a' : 'aaa', 'b' : 'bbb', 'c' : 'ccc',}.keys);// (a, b, c)
print({'a' : 'aaa', 'b' : 'bbb', 'c' : 'ccc',}.keys.toList());// [a, b, c]
print({'a' : 'aaa', 'b' : 'bbb', 'c' : 'ccc',}.values.toList());// [aaa, bbb, ccc]
print([1,2,3].where((e)=> e > 2 ).toList());// [3]
print(["a", "b", "c"].asMap());// {0: a, 1: b, 2: c}
print(["a", "b", "c"].asMap().entries);// (MapEntry(0: a), MapEntry(1: b), MapEntry(2: c))
print(["a", "b", "c"].asMap().entries.map((e)=>"${e.key}${e.value}").toList());// [0a, 1b, 2c]
["test", "test2"].map((item) => item.toUpperCase()).forEach(print);
```
* 参考 
  * Iteralbe.forEachを関数リテラルで使用することはEffective Dartでは避けるようにと記載されている。
  * 上記の例は関数リテラルではなく既存の関数(print)を渡しているので問題ない。
  * なお、マップはIterableではないため、マップのプロパティのForeachはその限りではない。
  * https://dart.dev/effective-dart/usage#avoid-using-iterable-foreach-with-a-function-literal
* 参考
  * mapに渡す関数は遅延評価される。
  ```dart
  ["test", "test2"].map(print);// printは実行されない。
  ["test", "test2"].map(print).toList(); // printはtoList()の際に初めて実行される
  print(["test", "test2"].map(print).runtimeType) // MappedListIterable<String, void>
  ```
  * https://api.dart.dev/stable/2.10.3/dart-core/Iterable/map.html



# Record型
* https://dart.dev/language/records
* https://dart.dev/language/pattern-types
```dart
void main() {
  print(getResponse());// ({result: success, data: }, 200)
  print(getResponse2());// (body: {result: success, data: }, responseCode: 200)
  print(getResponse().$1);
  print(getResponse().$2);
  print(getResponse2().body);
  print(getResponse2().responseCode);

  // pattern variable declaration
  final (b, r) = getResponse();
  final (:body, :responseCode) = getResponse2();
  // final (:body, :_) = getResponse2();//エラーとなる。値の無視はできない。
  final (body:b2, responseCode:r2) = getResponse2();
  final (body:b3, responseCode:_) = getResponse2();// 値の無視ができる
  print("$b, $r, $body, $responseCode, $b2, $r2");// {result: success, data: }, 200, {result: success, data: }, 200, {result: success, data: }, 200
}

(Map<String, dynamic> body, int responseCode) getResponse() => ({"result":"success", "data":""}, 200);
({Map<String, dynamic> body, int responseCode}) getResponse2() => (body:{"result":"success", "data":""}, responseCode:200);
```
* 分割代入(Record型のパターン割り当て)は、ローカルスコープの変数にしかできない。
  * ローカルスコープ以外の変数に分割代入で値を入れる場合、一度ローカルスコープの変数を経由する必要がある。
  * https://dart.dev/tools/diagnostic-messages#pattern_assignment_not_local_variable
  ```dart
  class C {
    var x = 0;

    void f((int, int) r) {
      // (x, _) = r; // NG

      // ローカル変数を経由する必要がある。
      var (a, _) = r;
      x = a;
    }
  }
  ```

# typedef
* https://dart.dev/language/typedefs
* パブリックAPIにおいてはプライベートのtypedefはできない
  * https://github.com/dart-lang/linter/issues/4658
