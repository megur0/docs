[TOP(About this memo))](../README.md) > [一覧(Dart)](./README.md) > 並行処理


# ドキュメント
* https://dart.dev/language/concurrency
* https://dart.dev/language/concurrency#how-isolates-work
* https://dart.dev/codelabs/async-await
## 参考
* https://medium.com/dartlang/dart-asynchronous-programming-isolates-and-event-loops-bffc3e296a6a
* https://www.cresc.co.jp/tech/java/Google_Dart2/language/isolates/isolates.html


# 並列処理とアイソレート
* すべての Dart コードはアイソレート内で実行される。
* 各アイソレートは1つのスレッドを持つ。
* アイソレート間ではメモリは共有されない。
* 並列処理を行いたい場合はアイソレートを生成する。
  * 複数のアイソレートを活用することで、マルチコアやマルチCPU環境を活用
* 大抵のアプリケーションは１つのアイソレート（メインアイソレート）のみで十分。
  * 以降に記載するイベントキューやマイクロタスクキューを使った非同期処理のみで十分。
* 以降のメモでは、並列処理は扱わない。いずれの処理も１つのアイソレート上で行っているものとする。

# イベントキュー
* イベント ループはDart での非同期および同時プログラミングを可能にするもの
* アプリケーションが実行されると、すべてのイベントがイベント キューと呼ばれるキューに追加される。
* イベントには、UI の再描画リクエストから、ユーザーのタップやキーストローク、ディスクからの I/O まで、あらゆるものが含まれる。
* イベント ループはキューに入れられた順序で一度に 1 つずつイベントを処理
* FutureやStreamオブジェクトを扱うことでイベントループの末尾にイベントが追加され、並行処理（非同期処理）が実現される。

# マイクロタスクキュー
* イベントループの先頭で、マイクロタスクキューにタスクがある場合、先に全て実行される
  * したがって、すでに実行が開始されたイベントを除くと、あらゆるイベントよりもマイクロタスクが優先されて実行される。

# 並行処理の順番を検証したサンプルコード
* IMO: 以下の処理の結果を予測できるのであれば、FutureやStreamがどのように動作しているかは大方理解できていると考えられる。
```dart
import 'dart:async';

void main() async{
  StreamController<void> sc = StreamController<void>();
  sc.add(null);
  sc.stream.listen((_) {
    print("1");
  });
  
  Future.delayed(const Duration(milliseconds:500), (){print("2");});
  Future((){print("3");});
  Future.microtask((){print("4");}).then((_){print("5");});
  print("6");
  Future.sync((){print("7");}).then((_){print("8");});
  test().then((_){print("9");});
  await Future.sync((){print("10");}).then((_){print("11");});
}

Future<void> test() async{
  print("12");
  Timer.run((){print("13");});
}

/*

6
7
12
10
1
4
5
8
9
11
3
13
2

*/
```


# Futureオブジェクト
* Futureオブジェクトを生成するとコールバックで渡した処理を、イベントキューの末尾にイベントとして追加する。
* Futureオブジェクト自体は以下の２つの状態遷移を辿る
  * https://dart.dev/codelabs/async-await
  * Uncompleted
    * Futureオブジェクト生成時に即時受け取る。
  * Completed
    * コールバック処理が完了した状態。このとき、値もしくはエラーを伴う。
  ```dart
  void main() {
    print(Future((){print("a");}));// イベントキューの末尾に追加。
    print("b"); // この処理はすでにイベントキューに入っているため、上記のコールバックよりも先に実行される。
    // Instance of '_Future<Null>'
    // b
    // a
    // ※ もし、他の処理でも非同期処理をしているのであれば、b と a の間に他の処理が入り込む可能性がある点に注意。
  }
  ```
## Future.microtask
* マイクロタスクキューへ追加する。
```dart
Future((){print("a");});
Future.microtask((){print("b");});
// b
// a
```
## Future.value(値)
* 値が非Futureであれば、Completedの状態としてFutureオブジェクトを返す。
  * `new Future.sync(() => 値)` と等価。
* 値がもしFutureなら、completeとなるまで待ってからその値でCompletedの状態のFutureオブジェクトを返す。

# then
* https://api.flutter.dev/flutter/dart-async/Future/then.html
> Register callbacks to be called when this future completes.
> When this future completes with a value, the [onValue] callback will be called with that value. If this future is already completed, the callback will not be called immediately, but will be scheduled in a later microtask.
* Futureのイベントが終了した直後にコールバックを実行する。
  * すでにFutureがCompletedの場合は、コールバックは即時実行はされず、マイクロタスクとして登録される。
* thenの戻り値はFutureオブジェクトとなる。
```dart
void main() {
 print(Future(() => 1).then((a) => a).then((a)=> print(a))); 
 print(2);
}
// Instance of '_Future<void>'
// 2
// 1
```

# async/await
* awaitはthenと同様の処理を行うが、thenを見やすくしたものである。
* awaitを書くには、その関数がasyncである必要がある。（main関数でも必要）
* awaitがついているexpressionの値は通常はFutureオブジェクトである。
  * そうでない場合は、Futute.valueでラップされる。また、警告が表示される。
* 対象の関数は、awaitを付与したFutureのイベントが完了するまでは動作が止まる。
  * ドキュメントの記載は見つからなかったが、下記の結果から、thenと同様にFutureがすでにcompletedであればマイクロタスクへ後続の処理を登録すると思われる。
    ```dart
    import 'dart:async';
    void main() async{
      Future((){print(1);});
      Future.microtask((){print(2);});
      await test();
      print(4);
    }
    Future<void> test() async => print(3);
    // 3
    // 2
    // 4
    // 1
    ```
* asyncをつけた関数は、voidやdynamic以外の戻り値の場合は、戻り値をFutureのサブタイプにする必要がある。
  ```dart
  void main() async {
    print(await f1());
    print(await f2());
  }

  Future<int> f1() async {
    return Future(()=>1);
  }

  Future<int> f2() => Future.value(2);
  // 1
  // 2
  ```

# (参考)Futureではない値のFutureのawait
* Futureではない値のawaitは、Future.valueでラップされるため即時completedとなる。
* イベントループの末尾に登録されるため、通常の処理よりは後に処理されるが、Uncompletedから始まるFutureオブジェクトのawaitよりも早く処理される。
* なお、asyncをつけたのみの関数内の処理は通常の処理と同様の実行順となる。
```dart
import 'dart:async';

void main() async{ 
  f1();
  f2();
  f3();
  f4();
  f5();
  print(0);
}

Future<void> f1() async => print(await f1c());
Future<int> f1c() async => Future(() => 1); // 最も遅い
Future<void> f2() async => print(await f2c());
Future<int> f2c() async => 2;
Future<void> f3() async => print(await 3);
Future<void> f4() async => print(await Future.value(4));
Future<void> f5() async => print(5);// 単なるasync関数は通常の処理と変わらない
// 5
// 0 
// 2
// 3
// 4
// 1
```

# 複数のawait
```dart
  await Future.wait(
    [
      Future.delayed(const Duration(milliseconds: 100), () => print(1)),
      Future.delayed(const Duration(milliseconds: 1), ()=> print(2)),
    ],
  );
  print(3);
  // 2
  // 1
  // 3
```


# Timer.run
* equivalent to new Timer(Duration.zero, callback).
* 参考
  * Flutterのブートストラップ処理の随所で使われている。
  * Timer.run((){print("aaa")}) と Future((){print("aaa")})) は 結果は等価
    * Future()の中で内部でTime.run()が呼ばれている。
    ```dart
    factory Future(FutureOr<T> computation()) {
      _Future<T> result = new _Future<T>();
      Timer.run(() {
        try {
          result._complete(computation());
        } catch (e, s) {
          _completeWithErrorCallback(result, e, s);
        }
      });
      return result;
    }
    ```

# Future.sync
* Future.value(値) と Future.sync(()=>値) は等価
```dart
Future.sync((){print("1");}).then((_){print("2");});
print("3");
// 1
// 3
// 2
```

# FakeAsync
* [参照](./dart_fake_async.md)



