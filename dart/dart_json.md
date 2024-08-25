[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# Dartにおけるjsonのエンコード・デコード
* https://dart.dev/guides/json
* https://dart.dev/libraries/dart-convert
* 標準ライブラリのdart:convertでは、Jsonテキストのマッピングに Map<String, dynamic> が使われる。
```dart
var scores = [
  {'score': 40},
  {'score': 80},
  {'score': 100, 'overtime': true, 'special_guest': null}
];
var jsonText = jsonEncode(scores);
assert(jsonText ==
    '[{"score":40},{"score":80},'
        '{"score":100,"overtime":true,'
        '"special_guest":null}]');
```
```dart
var jsonString = '''
  [
    {"score": 40},
    {"score": 80}
  ]
''';

var scores = jsonDecode(jsonString);
assert(scores is List);

var firstScore = scores[0];
assert(firstScore is Map);
assert(firstScore['score'] == 40);
```


# (参考)比較対象: Go言語
  * Goの構造体 <=> jsonのように、オブジェクト（クラス）をjsonに直接変換できる。
* Dartの場合、まずはMapオブジェクトとしてからjsonへ変換する必要がある。



# Flutter
* https://docs.flutter.dev/development/data-and-backend/json
* flutterでのjsonの一般的な取り扱いは2種類あり、dart:convertパッケージをつかった方法とコードジェネーションを使った方法がある。
```dart
import 'dart:convert';
Map<String, dynamic> userMap = jsonDecode(jsonString);
var user = User.fromJson(userMap);
```
* コードジェネレーションを使った方法は下記を参照
  * https://docs.flutter.dev/data-and-backend/serialization/json