[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > クラス(コンストラクタ)


# ドキュメント
* https://dart.dev/language/constructors
* 参考
  * コンストラクタに関しては初学の際にドキュメントのみでは理解が追いつかなかったのでネットの記事も参照しながら学習した。
  * https://qiita.com/otome0927@github/items/e18e713db1705171d68e
  * https://www.cresc.co.jp/tech/java/Google_Dart2/language/classes/classes.html


# (IMO) 所感
* 筆者がDartの初学の際、コンストラクタの学習コストが高かった。例えば下記のようなコードは多少の慣れが必要かもしれない。
```dart
class A {
  final double x;
  final double y;
  A.aaa({required this.x, required this.y});
}
class B extends A {
  final double z;
  B({required super.x, required super.y, required this.z}) : super.aaa();
}
```

# 無名コンストラクタ、名前付きコンストラクタ
```dart
class Book {
  String title = '';
  Book(String title) {// 無名コンストラクタ
    this.title = title;
  }
  Book.draft() {// 名前付きコンストラクタ
    title = "draft book";
  }
}
```

# コンストラクタのオーバーロードはできない
```dart
class Book {
  String title = '';
  Book(String title) {// 無名コンストラクタ
    title = title;
  }
  Book(String a, String b) {// 同じ名前で定義するとエラーになる。 
  }
}
```

# Automatic field initialization
* Automatic field initializationは、コンストラクタが走る前に値がセットされる。
```dart
class Book {
  String title = '';
  // コンストラクタの処理で設定する。
  Book(String title) {
    this.title = title;
  }
  // Automatic field initialization で 設定する。
  // この場合は、コンストラクタが走る前に値がセットされる点が特徴。
  Book.a(this.title);
  Book.an({required this.title});// 名前付き引数の場合
}
```

# 初期化が必要なフィールドはコンストラクタ実行前に設定する必要がある
```dart
class Book {
  String title;
  //Book(String title); // エラーとなる。non nullableなtitleが初期化されていないため。
  /*Book.byC(String title) { // この場合もエラーとなる。コンストラクタが走る前に初期化をする必要がある。
    this.title = title;
  }*/
  Book.byA(this.title);// Automatic field initialization（コンストラクタが走る前に値がセット）
  Book.byI(String t): title = t;//Initializer Listを使う方法。（こちらもコンストラクタ処理前にセットされる）
}
class Book2 {
  Book2(this.title, this.content);
  
  // エラー。finalは初期化する必要がある。
  //Book2.aaaa({this.content});
  
  // エラー。nullableでもfinalは初期化する必要がある。
  // Book2.bbbb(this.title);
  
  final String title;
  final String? content;
}
```

# Initializer List
* コンストラクタの宣言の後ろに「:処理内容」という形式で記述できる処理。 
* Initializer Listはコンストラクタの中身が実行される前に処理を行うことができる。
```dart
Point(double x, double y): this.x = x, this.y = y1 {
  // ここのコンストラクタ処理が実行される前にxとyには既に値が設定されている。
};
//コンストラクタの処理が不要の場合は下記のように書くことができる。
Point(double x, double y): this.x = x, this.y = y1;
```
## (参考) Flutterでよく見る記述
* Flutterのウィジェットでよく見られる下記の書き方も Initializer Listになる。
```dart
class MyWidget extends StatefulWidget {
  MyWidget({Key? key}) : super(key: key);
  // ...
}
class MyWidget2 extends StatefulWidget {
  const MyWidget2({required Key key, required this.id}) : super(key: key); 
  final String id;
  // ...
}
```
## Initilizerは式
* ステートメントをはエラーとなる。
```dart
class Book {
  String title;
  // Book(this.title): print(title); // エラー
}
```
## Initilizerは親クラスのフィールドの初期化ができない
```dart
class A {
  int a = 0;
  A() {
    print("a: $a");
  }
}
class B extends A{ 
  B() : a = 10{ // エラーとなる
    // a = 10; // コンストラクタ内は良い
  }
}
```

# デフォルトコンストラクタ
* https://dart.dev/language/constructors#default-constructors
* デフォルトコンストラクタは、引数なしの無名のコンストラクタとなる。
```dart
class Book {}
void main() => print(Book());
```
* コンストラクタを宣言しない場合、デフォルトコンストラクタが自動的に定義される。
  * IMO
    * 筆者は公式ドキュメントを読んだ際に、「デフォルトコンストラクタ」が自動定義されたコンストラクタ自体を指すのか、引数なしの無名のコンストラクタ を指すのか読み取れなかった。
    * このメモ上は前者として扱っている。
    > If you don't declare a constructor, a default constructor is provided for you. The default constructor has no arguments and invokes the no-argument constructor in the superclass.
      * "the no-argument constructor" と書いてある部分は "the no-argument and unnamed constructor"　の方が正確な気もする。



# コンストラクタの継承
* https://dart.dev/language/constructors#constructors-arent-inherited
* サブクラスはスーパークラスのコンストラクタを自動的に継承しない。
* したがって、親クラスでコンストラクタを定義していたとしても、子クラスが何もコンストラクタを定義していない場合は子クラスはデフォルトコンストラクタが自動的に定義される。
  * なお、親クラスで名前無しかつ引数なしコンストラクタが定義されていない場合はエラーとなる。（後述）

# サブクラスのコンストラクタ
* 要点は下記となる。
  * サブクラスは必ず親のコンストラクタを呼ぶ必要がある。
    * 明示的に呼ぶ もしくは 暗黙的に呼ばれる
  * 親のコンストラクタは継承されない。したがってオーバーライドもできない。
  * 親のコンストラクタを呼べるのはイニシャライザのみ。
    * 親のコンストラクタを呼ぶにはイニシャライザでsuperによって呼ぶ。
    * コンストラクタ内の処理ではsuperによってコンストラクタを呼ぶことはできない。
      * superによってコンストラクタ以外の親のメソッドを呼ぶことは可能
## 明示的にスーパークラスのコンストラクタを呼ばなかった場合
* この場合は、暗黙的にスーパークラスのコンストラクタが呼ばれる。
* デフォルトコンストラクタの場合
  * スーパークラスの引数なし、かつ無名のコンストラクタを呼ぶ。
* デフォルトコンストラクタ以外のコンストラクタの場合
  * スーパークラスの名前無しコンストラクタを呼ぶ。
```dart
class B extends A{
  B() {// エラー（親クラスに無名で引数なしコンストラクタが存在しないため。）
    print("b");
  }
  B.test() {//エラー（親クラスに無名で引数なしコンストラクタが存在しないため。）
    print("b test");
  }
}
class C extends A {
  // エラー。親クラスに無名で引数なしコンストラクタが存在しないため。
}
class A {
  // もしA.test()を定義していない場合はデフォルトコンストラクタが自動定義されるので、上記のエラーが消える。
  // 引数なしであっても名前付きコンストラクタしか定義されていない場合は、上記のエラーが発生する。（無名である必要がある）
  A.test() {
    print("a test");
  }
  /*
  //明示的に無名のコンストラクタを定義する場合も、上記のエラーが消える。
  A() {
    print("a test");
  }
  */
}
```
## 明示的にスーパークラスのコンストラクタを呼ぶ
* Initializer Listからsuperを使って呼ぶことができる。
```dart
class Person {
  String? firstName;
  Person.fromJson(Map data) {
    print('in Person');
  }
}
class Employee extends Person {
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
  // 上記のコンストラクタをコメントアウトすると、親クラスの無名かつ引数なしコンストラクタが存在しない為エラーになってしまう。

  // なお、上記は下記のように書くこともできる。
  /* Employee.fromJson(super.data) : super.fromJson() {
    print('in Employee');
  }*/
}
```
* (参考)他のオブジェクト指向の言語とは異なり下記のようにコンストラクタ内で親のコンストラクは呼べない点に注意。
```dart
class Person {
  String? firstName;
  Person.fromJson(Map data) {
    print('in Person');
  }
}
class Employee extends Person {
  Employee.fromJson(Map data) {
    super.fromJson(data);//エラー。
    // 「親クラスのfromJsonというメソッド」という評価がされるため、該当のメソッドが存在しないというエラーが発生する。
    // （かつ親の無名コンストラクタが存在しない、というエラーも発生する）
  }
}
```
```dart
class A {}
class B {
  B() {
    super();//エラー
  }
}
```

# Initializer Listと、親クラスのコンストラクタの実行順番
* Initializer Listはスーパークラスのコンストラクタの実行よりも先に実行される。実行順番は以下となる。
  * サブクラスのinitializer list
  * スーパークラスのinitializer list
  * スーパークラスのコンストラクタ
  * サブクラスのコンストラクタ
```dart
int c() {
  print("c");
  return 1;
}
int d() {
  print("d");
  return 2;
}
class A {
  int a = 0;
  A(): a = d() {
    print("a: $a");
  }
}
class B extends A{
  int b;
  B() : b = c(){
    a = 1;
    print("b: $b");
  }
}
void main() => print("main a: ${B().a}");
/*
c
d
a: 2
b: 1
main a: 1
*/
```

# Initializer List の superの省略記法
```dart
/*
// OK
class A {
  int a;
  A(this.a);
}
class B extends A{
  int b;
  B(super.a, this.b);
}
*/
/*
// OK
class A {
  int a;
  A({required this.a});
}
class B extends A{
  B({required super.a});
}
*/
// NG
class A {
  int a;
  A(this.a);
}
class B extends A{
  B({required super.a});
  // B({required a}): super(a); // この場合はOK
}
```
```dart
class Vector2d {
  final double x;
  final double y;
  Vector2d(this.x, this.y);
}
class Vector3d extends Vector2d {
  final double z;
  // Vector3d(final double x, final double y, this.z) : super(x, y);
  Vector3d(super.x, super.y, this.z);
}
```
```dart
class Vector2d {
  final double x;
  final double y;
  Vector2d.aaa(this.x, this.y);
}
class Vector3d extends Vector2d {
  final double z;
  // Vector3d(double x, double y, this.z) : super.aaa(x, y);//この呼び出しは、
  Vector3d(super.x, super.y, this.z) : super.aaa(); // このように書くこともできる。
  // Vector3d(super.x, super.y, this.z); //なお、この場合「無名コンストラクタが存在しない」というエラーになる
}
```
```dart
class Vector2d {
  final double x;
  final double y;
  Vector2d.aaa({required this.x, required this.y});
}
class Vector3d extends Vector2d {
  final double z;
  //Vector3d({required double x, required double y, required this.z}) : super.aaa(x:x, y:y);//この呼び出しは、
  Vector3d({ required super.x, required super.y, required this.z}) : super.aaa(); //このように書ける。
  // Vector3d(super.x, super.y, this.z); //この場合は「無名コンストラクタが存在しない」のエラーとなる。
}
```

# 定数コンストラクタ
* 定数コンストラクタのdataはすべてfinalで非lateである必要がある。
```dart
class ImmutablePoint {
  final num x, y;
  const ImmutablePoint(this.x, this.y);
}
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);
assert(identical(a, b)); // They are the same instance!
assert(!identical(a, const ImmutablePoint(1, 2)));
assert(!identical(a, ImmutablePoint(1, 1)));
```
## 定数コンストラクタで生成されるオブジェクトがconst値となるかどうかは、呼び出し側に依存する
* 定数コンストラクタで生成されるオブジェクトは必ずイミュータブルにはなる。
* 一方でオブジェクトがconst値となるかどうかは利用側に依存する。
  ```dart
  main(){
    print(f(3).hashCode); // 例: 1046420307
    print(const A(3).hashCode); // 例: 649446554
    print(const A(3).hashCode); // 例: 649446554
  }
  A f(final int a1) {
    return A(a1);
  }
  class A {
    const A(this.v);
    final int v;
  }
  ```


# ファクトリコンストラクタ
* factoryを先頭につけたコンストラクタを設定すると、自動でインスタンスが作成されず、コンストラクタ内でインスタンスを生成することができる。
```dart
class City {
  final String? name;
  final String? state;
  City({
    this.name,
    this.state,
  });

  factory City.fromJson(
    Map<String, dynamic> json,
  ) {
    return City(
      name: json['name'],
      state: json['state'],
    );
  }
}
```

# プライベートのコンストラクタを扱う
* _(アンダースコア)で始まる名前付きコンストラクタを使うことで、利用をライブラリ内に制限する。
```dart
class Foo {
    Foo._();
}
```

# リダイレクトコンストラクタ
* コンストラクタから別のコンストラクタを呼び出す。
```dart
class Dog {
  var name;
  Dog(): this.anonymous();
  Dog.anonymous() {
    this.name = 'Anonymous';
  }
  Dog.name(this.name);
}
```
