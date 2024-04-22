- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# mixin
* https://dart.dev/language/mixins
  > Mixins are a way of defining code that can be reused in multiple class hierarchies. They are intended to provide member implementations en masse.
    * "en masse": ひとまとめにして
* mixin利用側
  * classからwithによってmixinsを複数指定することができる。
  * mixinをimplementすることもできる
  * mixinをextendsすることはできない。(mixin classはextendsすることができる)
  * もしmixin側に親クラスのメソッドと同名のメソッドが存在した場合、親クラスのメソッドがオーバーライドされる。
  * mixin自体はインスタンス化できない。
* mixin側
  * 抽象メソッドを定義できる。
  * extendsを使うことは出来ない。
  * onを使うことで他のclassやmixinを継承できる。
    * onは複数指定ができる。
    * ただし、mixin利用側がそのclassやmixinを継承している必要がある。
  * 他のclassやmixinをimplementsすることも可能。
    * mixin側では抽象メソッドの実装は必須ではないが、実装されていない場合はmixinを使うクラスが最終的に実装する必要がある。
  * mixin が withを使うことはできない。
* 以下はサンプルコードを示す
  * 機能確認のため、実際にアプリケーションのコードでは使わないような書き方をしている。
```
class C1 extends A2 with M2 {
  // M1がonをしているA2を継承する必要がある。
  
  // M2がM1をonしているがM1.a()を実装していないため、こちらで実装する必要がある。
  void a(){}
}
mixin M1 {
  void a(); 
}
class A2 {
  void ab() {}
}
mixin M2 on A2 implements M1 {// mixinをimplementsすることもできる。
  // onで指定したクラス・mixinの任意のメソッドをオーバーライドできる。
  // 実装しなくてもエラーとはならないが、withをするclassの方で実装する必要がある。
  @override
  void ab() {}
}

mixin M3 on M1, M2 {// mixinに対してもonができる。
  void d();
}

class C2 with M4,M5 {}// M5がonをしているM4を継承している必要があるが、M4はmixinのためwithにする。
mixin M4 {}
mixin M5 on M4{}

// このあたりまで複雑になると筆者も良くわからない。
class C3 extends C1 with M3 {
  @override
  void ab() {}
  @override
  void a() {}
  @override
  void d() {}
}
```

```
class C1 {
  f() => print("C1");
}
class C2 extends C1 with M1 {
  // 同名のメソッドが存在する場合は、オーバライドされる。
  // この場合、C1のメソッドf()と同名のメソッドがM1に存在するため上書きされる。
  // C2.f()のsuper.f()はこのオーバーライドされたM1の方のf()になる。
  f() { 
    print("C2");
    super.f();
  }
}
mixin M1 {
  f() => print("M1");
}
void main() => C2().f();
// C2
// M1
```

# mixin class
* https://dart.dev/language/mixins#class-mixin-or-mixin-class
* 通常クラス、mixinの両方として利用が可能。
  * 通常クラス
  * mixin
* ただし、mixinができないこと、classができないこと、この両方がmixin classでもできない。
  * onが使えない。
  * extends, withができない。
* mixin classの使い所は？
  * https://dart.dev/language/mixins
  * 利用側がwithとextendsの両方で利用したい時など?
    * (参考)FlutterのChangeNotifierは mixin classとなっている。

# 複数のmixinが同名のメソッドをオーバーライドした場合
* withで先に書いたmixinのメソッドが、後に書いたmixinのメソッドにオーバーライドされる。
* 最終的にもっとも後ろのmixinのメソッドが採用される。
* superによってオーバーライドされたメソッドを呼ぶこともできる。
  * superが指すメソッドはmixinの定義のみでは定まらず、呼び出し側の定義にも依存する点に注意。
```
void main() => C2();
abstract class C1 {
  C1() {
    f();
  }
  void f() => print("C1");
}
class C2 extends C1 with M1, M2 {
  C2() {
    print("C2 constructor");
  }
}
mixin M1 on C1{
  @override
  void f() {
    print("M1");
    super.f();
  }
}
mixin M2 on C1{
  @override
  void f() {
    print("M2");
    super.f();
  }
}
/*
M2
M1
C1
C2 constructor
*/
```
## (参考)  Flutterの ensureInitialized()メソッド
* Flutterの初期化時に呼ぶ WidgetsFlutterBinding.ensureInitialized()メソッドでは、mixinによるオーバーライドを使って各初期化処理が実行されている。
* superで親呼ぶことで、各mixinで定義されているinitInstance　メソッドが順に呼ばれることで各シングルトンの初期化処理が行われている。
  ```
  class WidgetsFlutterBinding extends BindingBase with GestureBinding, SchedulerBinding, ServicesBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {
    // ...
    static WidgetsBinding ensureInitialized() {
      if (WidgetsBinding._instance == null) {
        // ここを起点として BindingBase() -> 各mixinのinitInstanceが呼ばれる。
        // withの最後のWidgetsBindingのメソッドから呼ばれるが、superでオーバーライド元を実行するため処理の中身はBindingBase.initInstance() -> GestureBinding.GestureBinding() -> ... という順に実行されていく。
        WidgetsFlutterBinding();
      }
      return WidgetsBinding.instance;
    }
  }
  ```
* IMO
  * 各**Bindingを統一的に記述できている一方、コードは一見すると追いかけづらい。

# 余談
* 筆者はmixinをどうしても「ミックスイン」とは呼べず、「ミクシン」と読んでしまう。