[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > クラス（メンバー）

# クラス・オブジェクト
* https://dart.dev/language/classes
> Dart is an object-oriented language with classes and mixin-based inheritance. Every object is an instance of a class, and all classes except Null descend from Object. Mixin-based inheritance means that although every class (except for the top class, Object?) has exactly one superclass, a class body can be reused in multiple class hierarchies. Extension methods are a way to add functionality to a class without changing the class or creating a subclass. Class modifiers allow you to control how libraries can subtype a class.
* Dartはオブジェクト指向言語
* オブジェクトはクラスのインスタンスである
* オブジェクトを構成するメンバー
  * functions(methods）
  * data (インスタンス変数)
## (参考)(IMO)プロパティとインスタンス変数の違いは?
* 明確な定義はDartのドキュメントに無かったが、一般的にプロパティは「オブジェクトに付属するもの」であり、オブジェクト.xxxでアクセスするものを指す（はず）
  * オブジェクト.xxx()のようにメソッドやsetterを実行する場合は除く
* .xxxが内部的に変数なのかgetterなのかは関係ない。

# インスタンス変数（instance variables）
```dart
class A {
  A(this._test, this._test2, this._test3);
  int _test;
  //int get _test => _test + 1; //_testという名前は既に存在するためerror（変数と衝突している?それともデフォルトで作成されるsetterと衝突?）
  int get test => _test + 10;// 自分で定義したgetter
  int get test2 => _test + 11;// 自分で定義したgetter
  set test(int test){_test += 5;}// 自分で定義したsetter
  set test2(int test){_test += 6;}// 自分で定義したsetter
  
  final int _test2; // finalは初期化が必要
  final int? _test3; // finalはnullableでも初期化が必要
  //set test3(int test){_test2 = 5;}// finalなプロパティを変えることはできない。


  int? x;// nullとして初期化される。
  late int z;// lateはコンストラクタによる初期化は不要。
  late final int l;// finalをつけた場合も同様。

  int m = 0;// 0として初期化される。
}
void main() {
  var a = A(5,4,3);
  // var a = A(5,4); // エラー
  a._test = 1;// すべてのインスタンスは暗黙的にsetterが作成される。
  print(a._test);// すべてのインスタンスは暗黙的にgetterが作成される。
  print(a.test);
  print(a.test2);
  a.test = 100000;
  print(a._test);
  a.test = 1000000;
  print(a._test);
  print(a.z); // 初期化未済のため実行時エラー
}
```
* lateではないインスタンス変数の値はインスタンスの作成時に設定される
* lateではないインスタンス変数を宣言時に初期化している場合、その初期化はコンストラクターやそのInitializerが実行される前に実行される。
  > If you initialize a non-late instance variable where it’s declared, the value is set when the instance is created, which is before the constructor and its initializer list execute.
* 全てのインスタンスには暗黙のgetterが生成される。
* finalではない変数、initializerのないlate finalの変数は暗黙のsetterが作成される。
* このgetter/setterの仕様はJavaやC#とは異なるので注意。

# instance method
* instance methodの名前にはoperatorsを使うことができる。

# static variable
> Static variables aren’t initialized until they’re used.

# static method
* staticメソッドよりもトップレベルメソッドを使ったほうがよいとのこと。（コンパイル時に定数として使える為）
  ```
  Consider using top-level functions, instead of static methods, for common or widely used utilities and functionality.
  You can use static methods as compile-time constants. For example, you can pass a static method as a parameter to a constant constructor.
  ```
* IMO
  * 筆者は名前空間を区切るためにあえて使うことがある。

# classのスコープ
* classやenumの名前の先頭に "_"をつけることでプライベートとなりスコープがライブラリ内となる。
  * Dartではclassをネストすることはできないため最小のスコープがライブラリ内。
  * enumもclass内に入れることはできない。


# (IME) プライベートクラスの中のメンバーはプライベートとした方が良いか?
* いずれにせよ同じライブラリ内でのみアクセス可能となるため、意味は無い?


# (参考)Dartでのprivateフィールドの実現
* Dartでは同じライブラリ内であればPrivateの変数にもアクセスすることができる。
* Libraryの最小単位は１ファイルである。
```dart
void main() {
  var b2 = B2(4);
  print(b2._test);// 
}
class B2 {
  const B2(this._test);
  final int _test;
}
```
* したがって、あるクラスにおいてそれらのフィールドを完全にカプセル化することはできない。
  * フィールドをプライベートにしても、同じライブラリ内からはアクセスが可能。
  * デフォルトのsetterをオーバーライドして自由に設定できないようにしてみる? -> できない
    ```dart
    int _privateVariable = 10;
    set _privateVariable(int val) {// このような定義はできない。（既に定義されている、というエラーが生じる）
      throw "do not change this field";
    }
    ```
* カプセル化を実現する手段
  * 別のライブラリ（ファイル）に分ける。
  * クロージャを使う方法
    * 下記のようにクロージャを使うと内部構造を同じライブラリ内でも隠蔽することはできる。
      * 筆者はアプリケーションのコードでこういった書き方をしたことはない。
    * プライベートクラス
    ```dart
    class _Page {
      int _val_ = 1;// 同じライブラリ内の場合はアクセス可能。
      get val => _val;
      int inc() => ++_val;
      void reset() => _val = 1;
    }
    final _page_ = _Page();
    ```
    * クロージャ
    ```dart
    final _page_ = () {
        int val = 1;// 外のスコープからはアクセスできない。
        int inc() => ++val;
        void reset() => val = 1;
        return (val: val, inc: inc, reset: reset);
    }();
    ```
  * 諦める。


# (参考) Dartのsetter
* 呼び出し側からは、setterの内部処理はブラックボックスとなる。
    ```dart
    main() {
        final a = A(1);
        print(a.f);// 1
        a.f = 2;// 一般的にはfには2が代入されるように見える。
        print(a.f);// 3
    }
    class A {
        A(this._f);
        int _f = 1;
        int get f => _f;
        set f (int fa) {
            _f = fa + 1;
            // ...
        }
    }
    ```
* 関連
    * https://dart.dev/effective-dart/design#do-use-setters-for-operations-that-conceptually-change-properties

