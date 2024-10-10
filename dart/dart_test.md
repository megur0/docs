[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > テスト



# 公式ドキュメント
* https://dart.dev/guides/testing

# Flutterのテスト
* [../flutter/flutter_test.md](../flutter/flutter_test.md)

# expect
* https://api.flutter.dev/flutter/package-matcher_expect/expect.html
* dynamic型として渡した値actualが、 同じくdynamic型のmatcherにマッチすることをassertする。
* matcherにはMatcher派生型を指定する。
  * Matcher派生型以外を指定した場合、equalsでラップされる
* assertに失敗した場合はTestFailureを返す。
* Future型を渡すと、Futureが値を返すまでテストが完了しない。
  * 完了を待ってからテストを続けたい場合は、代わりに expectLater を利用する

# Matcher
* https://api.flutter.dev/flutter/package-matcher_matcher/Matcher-class.html
* expectで指定するmatcherの基本クラス。
* DartおよびFlutterには様々なMatcherが用意されている。以下は一例。
* prints
```dart
test('', () {
  expect(() {
    print('my test');
    }, prints('my test\n'));
});
```
* equal
```dart
test('', () {
  expect(true, true);
  expect(true, equals(true));//上記と同じ
  expect(true, isTrue);
  expect(false, isNot(isTrue));
  expect(false, isFalse);
  expect([1], isNotEmpty);
  expect([], isEmpty);
  expect(1, isNonZero);
  expect(0, isZero);
  expect(null, isNull);
  expect(1, equals(1));
  expect(1, lessThanOrEqualTo(2));
  expect([1, 2, 3], orderedEquals([1, 2, 3]));
  expect([1, 2, 3], unorderedEquals([1, 3, 2]));
});
```
* エラー
```dart
test('', () async {
  expect(() {
    throw Exception();
  }, throwsA(isException));
  expect(() {
    throw Exception();
  }, throwsException);// Throws(isException)でも良い
  expect(() {
    assert(false);
  }, throwsA(isA<AssertionError>()));// flutter_testの場合はthrowsAssertionErrorが用意されている
});
```
* Future
```dart
test('', () async {
  expect(Future.value(0), completes);
  expect(Future.value(0), completion(1));
});
```
* Stream
```dart
test('', () {
  expect(Stream.fromIterable([0, 1, 2, 3]), emitsInOrder([0, 1, 2, 3]));
  expect(Stream.fromIterable([3]), emits(3));
  expect(Stream.fromIterable([0, 1, 2, 3]), neverEmits(4));
  expect(
      Stream.fromIterable([0, 1, 2, 3]), neverEmits((value) => value == 4));
});
```
* expectAsync0~6
```dart
test('', () async {
  var callback = expectAsync0(() {});// 渡したコールバック処理が呼び出されるまではテストを終了しない。0は引数0のコールバックを対象としていることを表す(コールバックは戻り値から実行可能)
  Timer(const Duration(milliseconds: 100), () {
    print("timer done");// expectAsync0を利用しない場合、この処理が実行される前にテストが終了する。
    callback();
  });
});
```
* predicate
```dart
  test('', () async {
    var isTwoDigits = predicate((e) => e is int && e > 9 && e < 100, 'is two digits');
    expect(15, isTwoDigits);// ok
    //expect(150, isTwoDigits);// ng
  });
```

