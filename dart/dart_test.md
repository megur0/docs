[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# 公式ドキュメント
* https://dart.dev/guides/testing

# Flutterのテスト
* [../flutter/flutter_test.md](../flutter/flutter_test.md)

# Matcher
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
test('', () async {
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

