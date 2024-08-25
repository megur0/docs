[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# 共変と反変のサンプルコード
```dart
// ignore_for_file:unused_local_variable

class A {}
class B extends A {}
class C<T> {}

void fa(A a) {}
void fb(B a) {}

class X {
 A f1() => A();  
 A f2() => B(); // 共変: OK  
 // B f2() => A(); // 反変: NG
 B f3() => B();
 void f4(A a){}
}

class Y extends X {
  @override
  B f1() => B(); // 戻り値の共変: OK 
  
  //@override
  //A f3() => A(); // 戻り値の反変1: NG
  
  //@override
  //B f3() => A(); // 戻り値の反変2: NG
  
  @override
  //void f4(B b){} // 引数の共変1: NG
  void f4(covariant B a){}// 引数の共変2: OK
}


main() {
  fa(A());
  fa(B()); // 引数の共変: OK
  //fb(A()); // 引数の反変: NG
  Y().f4(B());// 引数の共変: OK
  //Y().f4(A());// 引数の反変: NG
  
  Iterable<Object> v1 = <A>[A(),A()];// ジェネリクスの型引数の共変: OK
  //Iterable<B> v2 = <A>[A(),A()];// ジェネリクスの型引数の反変: NG
}
```