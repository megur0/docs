[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# fake_async
* https://pub.dev/packages/fake_async
* 明示的に経過時間を制御することで実際の経過時間を待たずに動作確認等を行うことができる。
```dart
// README.md記載のコード
void main() {
  test("Future.timeout() throws an error once the timeout is up", () {
     fakeAsync((async) {
      expect(Completer().future.timeout(const Duration(seconds: 5)),
          throwsA(isA<TimeoutException>()));

      async.elapse(const Duration(seconds: 5));
    });
  });
}
```
* 経過時間と紐づけたClockオブジェクトを取得することもできる。このClockは経過時間と紐づいているため、elpase()を実行すると経過時間が反映される。
```dart
void main() {
  final fakeAsync = FakeAsync();
  final clock = fakeAsync.getClock(DateTime.utc(2023));
  print(clock.now());// 2023-01-01 00:00:00.000Z
  fakeAsync.elapse(const Duration(days: 3));
  print(clock.now());// 2023-01-04 00:00:00.000Z
}
```
* なお、fakeAsyncのコールバック内ではClockパッケージのclockを通して経過時間が反映された日時を取得できる。
  * FackeAsyncオブジェクトの中では_clockを持っており、これはコールバックの文脈においてclockへインジェクションされるため。
```dart
void main() {
  fakeAsync((async) {
    print(clock.now());
    async.elapse(const Duration(days: 10));
    print(clock.now());
  }, initialTime: DateTime.utc(2001));
}

/*

2001-01-01 00:00:00.000Z
2001-01-11 00:00:00.000Z

*/
```
* マイクロタスクやTimerのフラッシュ
```dart
void main() {
  final fakeAsync = FakeAsync();
  fakeAsync.run((self) {
    Future(() => print("future"));
    Future.microtask(() => print("microtask1"));
    Future.microtask(() => print("microtask2"));
  });
  print("no microtask done yet.");
  fakeAsync.flushMicrotasks();
  print("no future done yet.");
  fakeAsync.flushTimers();

  /*

no microtask done yet.
microtask1
microtask2
no future done yet.
future

  */
}
```
## (参考)FakeAsyncの実装
* FakeAsyncは内部的にClockオブジェクトを保持している。(デフォルト値はclock.now()のため現在日時となる)
```dart
class FakeAsync {
  late final Clock _clock;
  Duration get elapsed => _elapsed;
  var _elapsed = Duration.zero;
  //...
  final _microtasks = Queue<_Microtask>();
  final _timers = <FakeTimer>{};
  //...
  FakeAsync({DateTime? initialTime, this.includeTimerStackTrace = true}) {
    var nonNullInitialTime = initialTime ?? clock.now();
    _clock = Clock(() => nonNullInitialTime.add(elapsed));
  }
  //...
}
```
* FakeAsync.run()ではClockパッケージのwithClockを利用して上記の_clockをインジェクションしている。またZoneを利用してFakeTimerを返すcreateTimer()などをインジェクションしている。
```dart
T fakeAsync<T>(T Function(FakeAsync async) callback, {DateTime? initialTime}) =>
    FakeAsync(initialTime: initialTime).run(callback);
```
```dart
class FakeAsync {
  //...
  T run<T>(T Function(FakeAsync self) callback) =>
      runZoned(() => withClock(_clock, () => callback(this)),
          zoneSpecification: ZoneSpecification(
              createTimer: (_, __, ___, duration, callback) =>
                  _createTimer(duration, callback, false),
              createPeriodicTimer: (_, __, ___, duration, callback) =>
                  _createTimer(duration, callback, true),
              scheduleMicrotask: (_, __, ___, microtask) =>
                  _microtasks.add(microtask)));
  //...
```
* createTimerはTimerクラスが利用している。
```dart
// flutter/bin/cache/pkg/sky_engine/lib/async/timer.dart
abstract interface class Timer {
    //...
    factory Timer(Duration duration, void Function() callback) {
        if (Zone.current == Zone.root) {
        // No need to bind the callback. We know that the root's timer will
        // be invoked in the root zone.
        return Zone.current.createTimer(duration, callback);
        }
        return Zone.current
            .createTimer(duration, Zone.current.bindCallbackGuarded(callback));
    }
    //...
}
```
* Timerクラスは例えばTimer.runやFuture.timeout()の実装、Future.delayed()が利用する
```dart
abstract interface class Timer {
  // ...
  static void run(void Function() callback) {
    new Timer(Duration.zero, callback);
  }
  // ...
}
```
```dart
// flutter/bin/cache/pkg/sky_engine/lib/async/future_impl.dart
class _Future<T> implements Future<T> {
  //...
  Future<T> timeout(Duration timeLimit, {FutureOr<T> onTimeout()?}) {
   //...
   timer = new Timer(timeLimit, () {/* ... */});
   //...
  }
  //...
}
```
```dart
// flutter/bin/cache/pkg/sky_engine/lib/async/future.dart
abstract interface class Future<T> {
  //...
  factory Future.delayed(Duration duration, [FutureOr<T> computation()?]) {
      // ...
      _Future<T> result = new _Future<T>();
      new Timer(duration, () {
          //...
          result._complete(computation());
          //...
      });
      return result;
      }
  //...
}
```


# clock
* https://pub.dev/packages/clock
* トップレベルのgetterであるclockはZoneによってインジェクションされる。withClock()を利用することでZoneへ指定のClockをもたせた文脈でコールバックを実行することができる。
```dart
// clock-1.1.1/lib/src/default.dart
Clock get clock => Zone.current[_clockKey] as Clock? ?? const Clock();

T withClock<T>(
  Clock clock,
  T Function() callback, {/* ... */}) {
  //...
  return runZoned(callback,
      zoneValues: {_clockKey: clock, _isFinalKey: isFinal});
}
```
* デフォルトではclock.now()にはDateTime.now()が利用される。
```dart
// clock-1.1.1/lib/src/clock.dart
class Clock {
  final DateTime Function() _time;
  const Clock([DateTime Function() currentTime = systemTime])// DateTime systemTime() => DateTime.now();
      : _time = currentTime;
  Clock.fixed(DateTime time) : _time = (() => time);
  DateTime now() => _time();
  //...
}
```
* withClockの例
```dart
void main() {
  withClock(Clock.fixed(DateTime.utc(2023)), () => print(clock.now()));
}
/*

2023-01-01 00:00:00.000Z

*/
```
* fakeAsyncは内部でwithClockを利用してFakeAsync._clockをclockへインジェクションしている。
```dart
import 'package:clock/clock.dart';
import 'package:fake_async/fake_async.dart';

void main() {
  fakeAsync((async) {
    print(clock.now());
    async.elapse(const Duration(days: 10));
    print(clock.now());
  }, initialTime: DateTime.utc(2001));// initialTimeを指定するとFakeAsync._clockの初期値として設定される。
}

/*

2001-01-01 00:00:00.000Z
2001-01-11 00:00:00.000Z

*/
```





