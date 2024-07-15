[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# Generics
* https://dart.dev/language/generics
> Why use generics? Generics are often required for type safety, but they have more benefits than just allowing your code to run:
```
// サブタイプによって制限
class Foo<T extends Object> {
  // Any type provided to Foo for T must be non-nullable.
}
// メソッドもgenericsが使える。
T first<T>(List<T> ts) {
  // Do some initial work or error checking, then...
  T tmp = ts[0];
  // Do some additional checking or processing...
  return tmp;
}
```

# サンプル
```
import 'dart:convert';
import 'dart:io';

typedef Response<T> = ({int code, T data});

Future<Response<T>> request<T>(Uri url, T Function(Map<String, dynamic> json) converter) async{
  late final String responseBody;
  late final Map<String, dynamic> json;
  late final T data;
  late HttpClientResponse res;

  try {
    final httpClient = HttpClient();
    final request = await httpClient.getUrl(url);
    //...
    res = await request.close();
    responseBody = await res.transform(utf8.decoder).join();
    json = jsonDecode(responseBody);
    data = converter(json);
  } catch (e) {
    //...
  }

  return (code:res.statusCode, data:data);
}
```

# Genericsの型に対してnewはできない
* 例えば下記のような処理は不可
  ```
  import 'package:flutter/material.dart';

  mixin CreateElement<T extends State> on StatefulWidget {
    @override
    T createState() => new T(); // エラー
  }

  class SW extends StatefulWidget with CreateElement<S> {
    const SW({super.key});
  }

  class S extends State<SW> {
    @override
    Widget build(_) => Text("");
  }
  ```
* 関連
  * https://github.com/dart-lang/sdk/issues/30074


# 派生クラスの親のジェネリクスの型
* 派生クラスは親のジェネリクスの型を明示的に指定しない場合はdynamicとなる
* mixinがonで指定したクラスのジェネリクスの型は明示的に指定しなくてもwithの際に決定される。
```
import 'package:flutter_test/flutter_test.dart';

void main() {
  test("", () {
    expect(Parent(Param()) is Parent<Param>, true);
    expect(Child() is Parent<Param>, false); //Parent<dynamic>となる
    expect(Child() is M<Param>, false); //M<dynamic>となる
    expect(Child2(Param()) is Parent<Param>, true);
    expect(Child2(Param()) is M<Param>, true);
    expect(Child3() is Parent<Param>, true);
    expect(Child3() is M<Param>, true);
  });
}

class Child extends Parent with M {
  Child() : super(Param());
}

class Child2<T> extends Parent<T> with M {
  Child2(super.m);
}

class Child3 extends Parent<Param> with M {
  Child3() : super(Param());
}

class Parent<T> {
  const Parent(this.m);
  final T m;
}

class Param {}

mixin M<T> on Parent<T> {}

```