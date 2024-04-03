- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# Extend a class
* https://dart.dev/language/extend
> Use extends to create a subclass, and super to refer to the superclass:

# Dartのさまざまなclass
## abstract class
* インスタンス化できない。
* 抽象メソッドを定義できる。
* 具象メソッドを定義することもできる。
* コンストラクタも定義できる。
* 継承も実装もできる。
## concrete class
* 抽象クラスとは反対にインスタンス化できるクラスはconcrete class(具象クラス)と呼ぶ。
* 抽象メソッドを定義できない。
## interface class
* Dart3から追加。
* 特徴
    * 継承できない
* 注意: "interface"を付与することで追加される制約は「継承ができない」のみであることに注意。
    * 抽象クラスとしたい場合はabstract interface classとする。
    * Java等の他言語の「インターフェース」とは異なり、具象メソッドもコンストラクタも定義可能である。
## base class
* Dart3から追加。
* 特徴
  * base or final or sealed のクラスのみ継承できる
  * implementsができない
## final class
* Dart3から追加
* 特徴
  * 継承できない
  * implementsができない
## sealed class
* library外ではextends/implementsができない。
* sealed classのサブクラスはlibrary外でextends/implementsができる。
* サブクラスの種類が決定的となるため、enumのようにswitchに渡すことが可能。
    * IMO enumでは出来ないextendsやconstではない値を含めたい時に使う?
    * enumのように.valuesメソッドで羅列することはできない。


# Implicit interfaces
* https://dart.dev/language/classes#implicit-interfaces
> Every class implicitly defines an interface containing all the instance members of the class and of any interfaces it implements. If you want to create a class A that supports class B's API without inheriting B's implementation, class A should implement the B interface.
* Dartではクラスの定義は、暗黙的なインターフェースの定義が含まれる。
```
class A {
  final String _v;
  A(this._v);
  String f() => _v;
}

class B implements A {
  @override
  String get _v => _v;
  
  @override
  String f() => '$_v $_v';
}

```

# implementsと、extendsの違い
* implementsは複数に対して指定することができるが、extendsは１つのみ指定可能。
* implementsはすべてのメソッドをオーバーライドする必要があるが、extendsは抽象メソッドのみオーバーライドすれば良い。
    * なお、サブクラス側がabstract classの場合はいずれの場合もオーバーライドは任意となる。
        ```
        class A {
            String f() => '';
        }
        abstract class B implements A {}
        ```
* implementsは元のクラスからコードを継承されず、型のみが継承される。
* implementsはextendsのようにsuperを使えない。


# (参考) extends, implements, withについて
> Extends is the typical OOP class inheritance. If class a extends class b all properties, variables, functions implemented in class b are also available in class a. Additionally you can override functions etc.

> Implements can be used if you want to create your own implementation of another class or interface. When class a implements class b. All functions defined in class b must be implemented.

> With is used to include Mixins. A mixin is a different type of structure, which can only be used with the keyword with.

> Mixins are a way of reusing a class’s code in multiple class hierarchies.
* https://stackoverflow.com/questions/70824365/dart-implements-extends-for-abstract-class
* https://stackoverflow.com/questions/55295782/extends-versus-implements-versus-with

## (参考) flutterの RenderObject
* extendsもwithもimplements も全てを活用している。
* https://github.com/flutter/flutter/blob/21797cbb034f48a384378efff0ee0b520e160072/packages/flutter/lib/src/rendering/object.dart#L1237
```
abstract class RenderObject extends AbstractNode with DiagnosticableTreeMixin implements HitTestTarget {
  // ...
}
```

# (IME) (参考) base class, final class, sealed class と テスタブル
* 関連
    * https://github.com/dart-lang/language/issues/3106#issuecomment-1593145359
    * https://github.com/dart-lang/language/issues/3422
* これらの機能がライブラリの中で使われてしまうと、アプリケーションコード側でスタブ化できない、という問題がある。
* アプリコード側でテスト可能とするには、ライブラリ側でモック用のクラスやミックスインを作成し@visibleForTesting を付与するといった方法がある。
    * ただ、いずれにしてもライブラリ次第となる。


# オーバーライド
* https://dart.dev/language/extend
> Subclasses can override instance methods (including operators), getters, and setters
* overrideはいくつか制約がある
    * 戻り値、引数はオーバーライドしたメソッドと同じ型(もしくはサブタイプ)である必要がある
    * オーバーライドされたメソッドがN個の位置パラメータを受け入れる場合、オーバーライドするメソッドもnN個の位置パラメータを受け入れる必要がある。
    * ジェネリックメソッドは非ジェネリック メソッドをオーバーライドできない。逆も同様。
* voidのメソッドはオーバーライド時に戻り値の設定が可能。
```
abstract class A {
  void a() {
    print('a called');
  }

  int b(int a);
  int c(int a);
}

class B extends A {
  @override
  int a() {
    // voidはオーバーライドによって型の設定が可能。
    return 3;
  }
  
  // void以外の戻り値は上書きできない。
  /*@override
  String b(int a) {
    return "aaa";
  }*/
  
  // 同じpositional parameterが必要
  /*
  @override
  String b() {
    return "aaa";
  }*/

  // optional parameterの追加は可能（optional positional parameter)
  @override
  int b(int a, [int a2 = 3]) {
    return a;
  }

  // optional parameterの追加は可能（optional named parameter)
  @override
  int c(int a, {int? a2}) {
    return a;
  }
  
  // 必須パラメータの追加はできない
  /*@override
  int c(int a, {required int a2}) {
    return a;
  }*/
}
```


# 具体例
```
void main() {
  // final a = A(); // エラー。抽象クラスはインスタンス化ができない。
  final b = B("test");
  b.a();
  b.a2();
}
abstract class A {
  void a();// 抽象メソッド。
  void a2() {
    print("a2 call");
  }
}
abstract class AX extends A {
  // 抽象クラスのため、a()の実装は不要である。
  void ax() {
    print("ax call");
  }
}
class B extends A {
  final String _a;
  B(this._a);

  // a()を実装する必要がある。
  @override
  void a() {
    print('a call: $_a');
  }
}
class C implements A {
  // 暗黙的に作成されるAのinterfaceを実装する。
  // aもa2もオーバーライドする必要がある。
  final String _a;
  C(this._a);

  @override
  void a() {
    print('a overrided call: $_a');
  }
  
  @override
  void a2() {
    print('a2 overrided call: $_a');
  }
}

class D implements AX {
   // aもa2もaxもオーバーライドする必要がある。
  final String _a;
  D(this._a);
  @override
  void a() {
    print('a call: $_a');
  }
  @override
  void a2() {
    print('a2 call: $_a');
  }
  @override
  void ax() {
    print("ax call");
  }
}
class E extends A implements AX {
  // Dとの違いとして、a2はAで実装されているものを継承されているため、Eの実装は不要。
  // aに関しては抽象メソッドのため実装する必要がある。
  // Flutter の Listenable, ValueListenableのサブクラスでこのテクニック?が使われていた。
  final String _a;
  E(this._a);
  @override
  void a() {
    print('a call: $_a');
  }
  @override
  void ax() {
    print("ax call");
  }
}
```

    





