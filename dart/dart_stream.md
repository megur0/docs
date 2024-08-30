[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > ストリーム



# Stream
> Asynchronous programming in Dart is characterized by the Future and Stream classes.

> A stream is a sequence of asynchronous events. It is like an asynchronous Iterable—where, instead of getting the next event when you ask for it, the stream tells you that there is an event when it is ready.

# Streamの種類
* https://dart.dev/tutorials/language/streams#two-kinds-of-streams
* https://dart.dev/articles/libraries/creating-streams
* Single subscription streams
  * 通常のストリーム。
  * 複数箇所からlistenはできない。
  * リスナーができるまでイベントの生成は開始されない。
  * サブスクが解除されるとイベントの送信が停止する。
  * 1 つのサブスクリプションで 2 回聞くことはできない。
* Broadcast streams
  * 複数のリスナーが同時に購読可能なストリーム。
  * リスナーの有無に関係なく準備ができるとイベントが起動される。  

# Streamの作成
* https://api.dart.dev/stable/2.18.5/dart-async/Stream-class.html 
* Streamオブジェクトを返す。
* async*をつける。
* returnの代わりにyieldを使い値を何度も返す。
```dart
Stream<int> countStream(int to) async* {
  int k = 0;
  while (k < n) yield k++;
}
```
* Stream.fromIterable()
  ```dart
  final stream = Stream.fromIterable([1, 2, 3]);
  final subscription = stream.listen((n) => print(n));
  ```
* Stream.periodic()
  ```dart
  final stream1 = Stream<int>.periodic(const Duration(seconds: 1), (i) {
    return i;
  }).take(5);
  ```
* Stream.value()
  ```dart
  Stream.value(1).listen(print);
  ```
* Stream.error()
  ```dart
  expect(Stream.error(Exception()), emitsError(isException));
  ```
## (参考)Iterableの作成
* https://dart.dev/guides/language/language-tour#generators
* Iterableオブジェクトを返す。
* sync*をつける。
* returnの代わりにyieldを使い値を何度も返す。
```dart
Iterable<int> naturalsTo(int n) sync* {
  int k = 0;
  while (k < n) yield k++;
}
```

# yield*
* yield
  * ジェネレーターから非同期または同期のいずれかで値を出力するために使用
* yield*
  * 呼び出しを別のジェネレーターに委任し、そのジェネレーターが値の生成を停止した後、独自の値の生成を再開
```dart
Iterable<int> r(int n) sync* {
  if (n > 0) {
    yield n;
    yield* r(n - 1);
  }
}
```
* 参考
  * https://stackoverflow.com/questions/57492517/difference-between-yield-and-yield-in-dart/57492636#57492636


# Streamから値を受け取る
## listen
* StreamSubscriptionオブジェクトが返却される。
* Streamのイベントは非同期に発生する。
* 下記の結果を確認するとマイクロタスクよりは遅く、Futureよりも早いタイミングで受け取る。
```dart
void main() {
  Future(() => print(5));
  countStream(1, 3).listen(print);
  countStream(6, 8).listen(print);
  Future.microtask(() => print(4));
}

Stream<int> countStream(int from, int to) async* {
  for (int i = from; i <= to; i++) {
    //if (i > 2) await Future((){});
    yield i;
  }
}

/* 
4
1
6
2
7
3
8
5
*/
```
## await for
* listenとは異なる点として、await for の場合は後続のステートメントには進まない。
  * awaitと同様に後続処理はawait forの完了後に即時実行される
* await forを使う関数はasyncである必要がある。
```dart
Future<void> main() async {
  await for (final value in Stream.fromIterable([1, 2, 3])) {
    print(value);
  }
  print("main end");
}
// 1
// 2
// 3
// main end
```

# Streamのエラー
* listenの際にonErrorでエラーを受け取ることができる
## Stream.error
* エラー後もイベントは継続する
```dart
void main() {
  countStream(6).listen(print, onError:print);
}
Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    if (i%3 == 0) {
      yield* Stream.error("error");
    } else {
      yield i;
    }
  }
}
// 1
// 2
// error
// 4
// 5
// error
```
## Exception
* throwするとイベントは停止する。
```dart
void main() {
  countStream(6).listen(print, onError:print);
}
Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    if (i%3 == 0) {
      throw Exception("error");
    } else {
      yield i;
    }
  }
}
// 1
// 2
// Exception: error
```
* (参考)Stream.periodicのコールバックで例外をthrowしても、その後のyieldは続いた
```dart
void main() {
  Stream<int>.periodic(const Duration(milliseconds: 1), (i) {
    if (i == 3) {
      throw "error";
    }
    return i;
  }).take(5).listen(print, onError: print);
}
// 0
// 1
// 2
// error
// 4
// 5
```
## await for
* try〜catchでハンドリング可能
  ```dart
  try {
    await for (final value in countStream(6)) {
      print(value);
    }
  } catch (e) {
    print(e);
  }
  // 1
  // 2
  // Exception: error
  ```
* await forの内部でtry〜catchで囲んでもエラーを受け取ることができない
  ```dart
  await for (final value in countStream(6)) {
    try {
      print(value);
    } catch (e) {
      print(e);
    }
  }
  // 1
  // 2
  // Uncaught Error: Exception: error
  ```
* handleErrorを使うことでもハンドリング可能。
  ```dart
  void main() async{
    await for (final value in countStream(6).handleError(print)) {
      print(value);
    }
  }
  // 1
  // 2
  // Exception: error
  ```

# Streamから他のStreamの値を使う
* yield* や await forを使う。
```dart
void main() {
  stream2().listen(print);
}
final stream1 = Stream.fromIterable([1, 2, 3]);

Stream<int> stream2() async* {
  yield 100;
  yield* stream1;
  await for (final n in stream1) {
    yield n;
  }

  // 下記のようには書けない。
  // stream1.listen((n){yield n;}); // エラー
}

// 100
// 1
// 2
// 3
// 1
// 2
// 3
```
## (参考) エラー後も引き続き値を受け取る
* 下記の場合は、エラーハンドリング後は値を受け取ることができない。
  ```dart
  void main() {
    stream2().listen(print, onError:print);
  }

  Stream<int> countStream(int to) async* {
    //...
  }

  Stream<int> stream2() async* {
    await for (final n in countStream(5)) {
      yield n;
    }
  }
  // 1
  // 2
  // error
  ```
* 下記のようにhandleErrorを利用して対応できるが、エラーハンドリングがstream2の関心事となってしまう
  ```dart
  void main() {
    stream2(print).listen(print);
  }

  Stream<int> countStream(int to) async* {
      //...
  }

  Stream<int> stream2(Function onError) async* {
    await for (final n in countStream(5).handleError(onError)) {
      yield n;
    }
  }
  ```
* 外部パッケージのStreamQueueを使う方法がある。
  * 参考
    * https://stackoverflow.com/questions/76863964/dart-is-there-any-way-to-handle-stream-error-without-stop-stream-when-we-await
* Streamのデータを一時的に保存しawait forを使わずに取り出せるオブジェクトを自前で作成する方法
  * 参考
    * https://stackoverflow.com/a/76891848/22090329
    


# Single/Broadcast
* シングルサブスクリプションストリーム
  ```dart
  main() {
    final sbsc = stream.listen(print);
    sbsc.pause();
    sbsc.resume(); 
    sbsc.cancel();// サブスクの破棄。
    //stream.listen((event) {});//エラーとなる。他のサブスクがキャンセルをしても、別のサブスクを作成することができない。
  }
  final stream = countStream(3);
  Stream<int> countStream(int to) async* {
    for (int i = 1; i <= to; i++) {
      yield i;
    }
  }
  ```
  * なお、Stream.fromIterable()の場合は_MultiStreamクラスを内部で生成していて、_MultiStreamクラスはlistenの度に新しいStreamControllerを生成して.streamを返している。
    * したがって何度でもlistenが可能である。
    ```dart
    main() {
      final stream = Stream.fromIterable([1,2,3]);
      stream.listen(print).cancel();
      stream.listen(print);// 
    }
    ```
* ブロードキャストストリーム
  ```dart
  main() async{
    stream.listen(print);
    stream.listen(print);
    
    await Future(()=>print(4)); 
   
    // Future実行後のlistenの時点で既に送信が完了しているため、受信は無い。 
    stream.listen(print);
  }
  final stream = countStream(3).asBroadcastStream();
  // 1
  // 1
  // 2
  // 2
  // 3
  // 3
  // 4
  ```
* ブロードキャストストリームは一度購読されるとキャンセルされて購読が0件となってもイベントが発生する
  ```dart
  main() {
  final sbsc = countStream(5).asBroadcastStream().listen((e)=>print("$e at listener"));
    sbsc.cancel();
  }

  Stream<int> countStream(int to) async* {
    for (int i = 1; i <= to; i++) {
      print("$i at generator");
      yield i;
    }
  }
  // 1 at generator
  // 2 at generator
  // 3 at generator
  // 4 at generator
  // 5 at generator
  ```


# (参考)Streamの破棄
## リスナー側
* 通常、リスナー側ではストリームの破棄は関心事ではなく購読をキャンセルするのみである。
* ストリームをどのように終わらせるかは、ジェネレータ側の実装による。
* https://github.com/dart-lang/sdk/issues/42480#issuecomment-649378170
## ジェネレータ側
* IMO(未検証)
  * シングルサブスクリプションストリームの場合、購読が終了、キャンセルされた場合は自動的にガーベッジコレクションされる?
  * ブロードキャストストリームの場合は全て送信が完了後はガーベッジコレクションされる?


# StreamController
* https://api.flutter.dev/flutter/dart-async/StreamController-class.html
* Streamを扱いやすくしたクラス。
* async* 〜 yieldを使ったStreamの作成は、Streamオブジェクトの宣言時にイベント自体を定義する必要がある。
  * もしStreamControllerを使わずにこれらを切り離すには仲介用のキューを自前で実装する必要がある?（未検証）
* StreamControllerを使うことで、手続的にイベント生成を行うことができる。
```dart
import 'dart:async';

main() async{
  sc1.stream.listen(print);
  sc1.add(3);
  sc1.add(4);
  sc1.sink.add(5);
}
final StreamController<int> sc1 = StreamController<int>();
// 3
// 4
// 5
```
## sink.addとaddの違い
* sink.addとaddの機能は同じ。
* sinkはStreamControllerオブジェクト自体は隠蔽して、addのみの操作をさせたいときに利用する。
  * StreamController自体を渡すとadd以外の操作も可能となってしまう。
* 参考
  * https://stackoverflow.com/questions/51395729/what-is-the-difference-between-streamcontroller-add-and-streamcontroller-sink


