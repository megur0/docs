- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


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
