[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# 公式ドキュメント
* Flutter公式
    * https://docs.flutter.dev/testing/overview
* Dart package:test
    * https://pub.dev/packages/test
* 参考
    * https://github.com/flutter/flutter/wiki/Style-guide-for-Flutter-repo#write-test-find-bug

# テストの分類
* https://docs.flutter.dev/testing/overview
* 公式のドキュメントに記載された内容ではないが、Flutterで提供される機能と実行時のruntimeTypeに基づいてテスト分類をすると下記のようになると考えられる。
* ユニットテスト
    * `flutter test` で実行される テストのうち、単一の関数、メソッド、またはクラスを対象とするもの。
* ウィジェットテスト
    * `flutter test` で実行される テストのうち、testWidgets()を利用するもの。
    * testWidgets()のコールバックで行う操作は、AutomatedTestWidgetsFlutterBindingオブジェクトのメソッドに基づいて行われる。
* 結合テスト
    * `flutter test integration_test/xxx` で実行するもの。
    * testWidgets()のコールバックで行う操作は、IntegrationTestWidgetsFlutterBindingオブジェクトのメソッドに基づいて行われる。
## ウィジェットテストと結合テストで検証可能なもの
||検証可能なもの|検証ができない・難しいもの|
|-|-|-|
|ウィジェットテスト|・ウィジェット<br/>・明示的なフレーム進行<br/>・FakeAsyncで進行させる非同期処理(モック化)|・HttpClientで実際のデータを取得する処理<br/>・現実の時間軸に沿ったフレーム進行<br/>・現実の時間軸に沿った非同期処理<br/>・ネイティブ側のAPIを呼び出す処理(プラグインの処理)|
|結合テスト|・HttpClientで実際のデータを取得する処理<br/>・現実の時間軸に沿ったフレーム進行<br/>・現実の時間軸に沿った非同期処理<br/>・ネイティブ側のAPIを呼び出す処理(プラグインの処理)|・ネイティブ側のUIを操作する処理<br/>・ネイティブ側のAPIの処理のエッジケース|

* 下記の点は、ウィジェットテストの段階では検証されず、結合テストの段階ではじめて検証できる点に注意したい
  * バックエンドやFirestoreのクエリが実際に成功するか否か、実行結果が想定通りか否か
## ネイティブ側のテスト
* (TODO)
* ネイティブ側のAPIの処理のエッジケース
    * 自身で書いたプラグイン等のエッジケースをテストしたい場合、多くの場合ネイティブの単体テスト(iOSの場合はXCTest)を書く必要がある
* ネイティブ側のUIの操作
    * 現状はFlutter側でネイティブのUIを操作する公式の方法は無い。
    * ネイティブ UI を操作を含めたエンドツーエンドのテストを行う場合はネイティブ側のテストコード(iOSの場合はXCUITest)を書く必要がある。
* 参考
    * https://github.com/flutter/flutter/wiki/Plugin-Tests
    * https://github.com/flutter/flutter/issues/86295
    * https://stackoverflow.com/questions/67908341/how-to-interact-with-native-ui-elements-from-flutter-integration-test
    

# パッケージ
## flutter_testパッケージ
* https://api.flutter.dev/flutter/flutter_test/flutter_test-library.html
* package:testの上に構築されたflutter用のテストライブラリ
    * (IME) "package:testの上に構築"とドキュメントに書かれているものの、実際のコード上は [dartのpackage:test](https://pub.dev/packages/test) の方ではなくpackage:test-apiというパッケージに依存しているようだった。
* 標準パッケージの為、デフォルトで利用可能
## integration_testプラグイン
* https://github.com/flutter/flutter/tree/main/packages/integration_test#integration_test
* https://docs.flutter.dev/testing/integration-tests
* https://docs.flutter.dev/cookbook/testing/integration/introduction
* 結合テスト用の公式のプラグイン
* デフォルトではインストールされていないためpubspec.ymlへ追加する必要がある
* プラグインの追加
    * `flutter pub add 'dev:flutter_test:{"sdk":"flutter"}' 'dev:integration_test:{"sdk":"flutter"}'`
    * pubsupec.yamlに下記が追加される。
        ```
        integration_test:
            sdk: flutter
        ```
* IntegrationTestWidgetsFlutterBinding.ensureInitialized()
    * https://api.flutter.dev/flutter/package-integration_test_integration_test/IntegrationTestWidgetsFlutterBinding/ensureInitialized.html
    * testWidgets()の前に記述すると、TestWidgetsFlutterBinding.instance に IntegrationTestWidgetsFlutterBinding が設定される。
        * 詳細は後述
    * integration_test/ フォルダ以外のテストファイルでこのメソッドを実行すると警告が表示される。
        ```
        If you're running the tests with `flutter drive`, please make sure your tests
        are in the `integration_test/` directory of your package and use
        `flutter test $path_to_test` to run it instead.
        ```

# flutter test
## flutter test
* https://docs.flutter.dev/cookbook/testing/unit/introduction
* ヘルプ
  *  `flutter test -h` で確認
* `flutter test`はデフォルトでデバッグモードのため`--debug`オプションは存在しない
* テストファイルのファイル名は xxx_test.dart である必要がある。
* test/に含まれるすべてのテストを実行
    * `flutter test`
    * test/ディレクトリが存在しない場合は`Test directory "test" not found.` と表示されエラーとなる。
* ファイルを指定して実行
    * `flutter test tests/xxx_test.dart`
* descriptionの文字列を部分検索(ファイル名は検索しない)
    * `flutter test --plain-name 'widget'`
    ```
        test("widget test", () async {}); // 実行される
        test("my widget", () async {}); // 実行される
        test("test", () async {}); // 実行されない

        group("widget", () {
            testWidgets(
            // 実行される
            'hello1',
            (_) async {},
            );
            testWidgets(
            // 実行される
            'hello2',
            (_) async {},
            );
        });

        for (int i = 1; i < 3; i++) {
            testWidgets(
            'widget $i',
            (WidgetTester tester) async {
                print(i);
            },
            );
        }
    ```
    * なお、flutter runの場合は--plain-nameのようなオプションは無い。
        * https://stackoverflow.com/questions/73542388/how-to-launch-specific-test-in-emulator
* --plain-nameと ファイル指定は同時には利用できない?
* --dart-define-from-fileで値を渡すこともできる
* グローバルオプションである -d でデバイスを指定しても無視される
* 並列化
    * https://pub.dev/packages/test#test-concurrency
    * Dartではtestの際にホストマシンのCPUコアの半分を利用するため、デフォルトで並列処理を行っている。
    * flutter testについては文書は見つからなかったものの、同様の動作となると考えられる。
    * --concurrency
        * 利用するCPUコアを明示的に指定する
    * 筆者の手元の環境(Macで8-core CPU)では'--concurrency=1'で実行すると体感でも明確に速度が低下した。
    * 並列処理時のメモリと 変数
      * Dartのマルチコアによる並列処理は別々アイソレート上で行われ、このアイソレートはそれぞれ独自のメモリを持つ。
          * https://dart.dev/language/concurrency#isolates
      * したがって、変数の状態などは共有されないと考えられる。
    * (IMO)並列処理の分割の単位は?
      * main処理内の一連の処理は同じメモリ空間で行う必要がある(と筆者は理解している)ため、おそらく並列処理の単位もmain関数(ファイル)単位と考えられる。
* テスト同士の依存について
  * testやtestWidgetsの外のスコープのイミュータブルを利用する場合はテスト同士の依存関係が発生する可能性があるため注意したい。
    * 例えば下記のテストは `flutter test` では成功するが --test-randomize-ordering-seed=シード値 によって順番をシャッフルすると失敗することが確認できる。
    ```
    int var1 = 0;
    void main() {
      for (int i = 0; i < 3; i++) {
        test(
          'MyTest$i',
          () {
            expect(i, var1);
            print(var1++);
          },
        );
      }
    }
    /*
      00:01 +0: MyTest0 
      0
      00:01 +1: MyTest1                                              
      1
      00:01 +2: MyTest2                                              
      2
    */
    ```
  * 外のスコープの変数を利用する必要がある場合はsetUp関数等で初期状態に戻し、テスト同士の依存は避けるようにすることが良いだろう。
* シーディング
    * TODO
    * https://pub.dev/packages/test#sharding-tests
    * https://github.com/flutter/flutter/pull/76382
    * テストをサブセットに分割して実行するためのオプション
    * テスト数が膨らんだ際にCI環境等実行速度を速くする場合などに有用となる

## flutter test integration_test
* integration_testプラグインを追加した状態で`flutter test`において、`integration_test`またはその配下のファイルを指定するとデフォルトのデバイスに対してビルドから実行される。
    * 明示的にデバイスを指定する場合は`-d (デバイスID)`にて指定する。
* `integration_test`以外の ディレクトリ名を指定すると、通常の`flutter test`で対象を指定した際の挙動となる。
* 全テストを実行
    * 例
        * `flutter test integration_test -d 'iPhone 15'`
    * chromeの場合
        * `flutter test integration_test -d 'chrome'`
            * `Web devices are not supported for integration tests yet.`と表示され失敗する。
        * ドライバーを利用する必要がある(未検証)
            * https://docs.flutter.dev/testing/integration-tests#running-in-a-browser
* 個別に実行
    * 例
        * `flutter test -d "対象のデバイスID" integration_test/xxx_test.dart`    
     * flutter runで実行する
        * `flutter run -d "対象のデバイスID" integration_test/xxx_test.dart`
        * 結合テストは実行のたびにビルドから実行するため、繰り返し実行する場合は`flutter run`で実行する方が早い。
        * (IME) `flutter run`で複数回リスタート・ホットリロードをしているとクラッシュすることがあるため、そういった場合は再度立ち上げる。

* IntegrationTestWidgetsFlutterBinding.ensureInitialized()
    * `flutter test integration_test` において自動的にこのメソッドがmain処理の前に暗黙的に実行される。
        * 2回目以降は無視されるため、明示的にmain内で呼んでいたとしても問題ない。
    * ※ `flutter run`の場合は暗黙的には呼ばれない。
## カバレッジ
* lcovをインストールしておく必要がある。
  * `brew install lcov`
* `flutter test --coverage`
* htmlファイルとして出力
    * `genhtml coverage/lcov.info -o coverage/html`
* カバレッジから行単位で除外
    * https://github.com/dart-lang/coverage/issues/162#issuecomment-456591358
    * 単一行
        * // coverage:ignore-line
    * 複数行
        * // coverage:ignore-start
        * // coverage:ignore-end
* カバレッジからファイル単位で除外
    * 例
        * `flutter test --coverage && lcov --remove coverage/lcov.info lib/main.dart -o coverage/lcov.info`
    * https://stackoverflow.com/questions/53649089/how-to-exclude-file-in-flutter-test-coverage
* すべてのファイルをカバレッジ対象に入れる
    * flutter testのカバレッジはテストファイルがimportしたファイルしか出力されない。
    * すべてのアプリケーションコードをカバレッジ対象とする間接的な手段として、mainのファイルをインポートするダミーテストファイルを用意しておくと良い。
    * https://github.com/flutter/flutter/issues/27997#issuecomment-1644224366
    ```
    // test/coverage_dummy_test.dart
    // ignore: unused_import
    import 'package:sample/main.dart';

    void main() {
        // Fake test in order to make each file reachable by the coverage
    }
    ```
    * なお、1行もカバレッジ対象のコードが含まれないファイルは出力されない。
      * 例えば、coverage:ignore-start 〜 endですべてのコードを囲んでいるファイルはカバレッジ対象であっても出力されない。

# setUpAll, setUp, tearDown, tearDownAll
```
int var1 = 0;
int var2 = 0;

void main() {
  setUpAll(() {
    print("setUpAll");
  });

  setUp(() {
    print("setUp");
    var1 = 0;
  });

  tearDownAll(() {
    print("tearDownAll");
  });

  tearDown(() {
    print("tearDown");
  });

  group("MyGroup", () {
    for (int i = 0; i < 3; i++) {
      testWidgets(
        'MyTest$i',
        (WidgetTester tester) async {
          print("var1:${var1++}, var2:${var2++}");
        },
      );
    }
  });
}


/*
flutter: setUpAll
00:01 +0: (setUpAll)                                   
setUpAll
00:01 +0: MyGroup MyTest0                                             
setUp
var1:0, var2:0
tearDown
00:01 +1: MyGroup MyTest1                   
setUp
var1:0, var2:1
tearDown
00:01 +2: MyGroup MyTest2                                     
setUp
var1:0, var2:2
tearDown
00:01 +3: (tearDownAll)                                             
tearDownAll
*/

```

# testWidgets()
* https://api.flutter.dev/flutter/flutter_test/testWidgets.html
  > Runs the callback inside the Flutter test environment.
* testWidgets()を実行時、渡したコールバックの実行開始時点で シングルトンのTestWidgetsFlutterBinding.instanceには必ずインスタンスが設定されている。
* ここで設定されるTestWidgetsFlutterBindingのruntimeTypeは実行条件によって異なる。

    ||実行条件(起動コマンド, ソースコード)|TestWidgetsFlutterBinding.instance のruntimeType|
    |-|-|-|
    |No.1|flutter test|AutomatedTestWidgetsFlutterBinding|
    |No.2|flutter run|LiveTestWidgetsFlutterBinding|
    |No.3|flutter test integrate_test|IntegrationTestWidgetsFlutterBinding|
    |No.4|flutter run integrate_test|LiveTestWidgetsFlutterBinding|
    |No.5|`IntegrationTestWidgetsFlutterBinding.ensureInitialized()`を実行|IntegrationTestWidgetsFlutterBinding|

    * No.1 ~ 4においてIntegrationTestWidgetsFlutterBinding.ensureInitialized()は実行していない前提とする
    * IntegrationTestWidgetsFlutterBindingはLiveTestWidgetsFlutterBindingの派生クラスである。
* サンプルコード
  ```
  import 'package:flutter/material.dart';
  import 'package:flutter_test/flutter_test.dart';

  main() {
      // 以下を実行した場合はすべてIntegrationTestWidgetsFlutterBindingとなる。
      // IntegrationTestWidgetsFlutterBinding.ensureInitialized();
      testWidgets("", (widgetTester) async {
        debugPrint(TestWidgetsFlutterBinding.instance.toString());
      });
  }

  // ・flutter testで実行
  // <AutomatedTestWidgetsFlutterBinding>
  // ・flutter runで実行
  // <LiveTestWidgetsFlutterBinding>
  // ・flutter test integration_test で実行(flutter test integration_test/xxx_test.dart でも同様)
  // <IntegrationTestWidgetsFlutterBinding>
  ```
## HttpClientのモック化
* AutomatedTestWidgetsFlutterBindingとLiveTestWidgetsFlutterBindingでは、HttpClientは自動的にモック化される。
* 処理自体でエラーは発生しないが、ステータスコードは400となりデータは空となる。
```
import 'dart:convert';
import 'dart:io';

import 'package:flutter/widgets.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('', (tester) async {
    final httpClient = HttpClient();
    final request = await httpClient.getUrl(Uri.parse("https://example.com"));
    final res = await request.close();
    final responseBody = await res.transform(utf8.decoder).join();
    debugPrint(responseBody);// ""
    debugPrint(res.statusCode.toString());// 400
  });
}
```
* NetworkImageウィジェット等の内部でHttpClientを利用している場合、400によってエラーとなってしまう場合があるため注意
```
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('', (tester) async {
    await tester.pumpWidget(const CircleAvatar(
      backgroundImage:
          NetworkImage("https://docs.flutter.dev/assets/images/dash/Dash.png"),
    ));
  });
}
/*

══╡ EXCEPTION CAUGHT BY IMAGE RESOURCE SERVICE ╞════════════════════════════════════════════════════
The following NetworkImageLoadException was thrown resolving an image codec:
HTTP request failed, statusCode: 400, https://docs.flutter.dev/assets/images/dash/Dash.png
...

*/

```
* IntegrationTestWidgetsFlutterBindingの場合は、HttpClientはモックされない。
  ```
  void main() {
      IntegrationTestWidgetsFlutterBinding.ensureInitialized();
      testWidgets(/* 上記の例と同じ処理 */);
  }
  // (integration_test/)
  // 00:01 +1: All tests passed! 
  ```
## フレームと非同期処理
* https://api.flutter.dev/flutter/flutter_test/TestWidgetsFlutterBinding/clock.html
    > The current time.In the automated test environment (flutter test), this is a fake clock that begins in January 2015 at the start of the test and advances each time pump is called with a non-zero duration.In the live testing environment (flutter run), this object shows the actual current wall-clock time.
* AutomatedTestWidgetsFlutterBinding
    * フレーム処理は自動で進行しない。
      * WidgetTester.pump()等のメソッドによって明示的に進行させる。
    * FakeAsyncによってcreateTimer等の処理がインジェクションされ、内部のclockオブジェクトの経過時間に基づいた処理と成る。
        * このcreateTimerは例えば、Timer.run()等の処理が利用しており内部のclockオブジェクトが進行しないとTimer.run()等も処理が進まない。
        * WidgetTester.pump()等のメソッドの引数でDurationを指定することで明示的に進めることができる。
            * なお、進行させない限りclockのタイムスタンプの値はDateTime.utc(2015).now().microsecondsSinceEpoch となる。
        * これによって非同期処理をモック化してテストを実行する際に、Future() や Future.delayed()を任意のタイミングで完了するように操作できる。
* LiveTestWidgetsFlutterBinding, IntegrationTestWidgetsFlutterBindingの場合 
    * 通常と同様にプラットフォームからのVsync信号に従ってフレーム処理が進行する。
    * FakeAsyncによるインジェクションはされない。
## 画面表示 
* AutomatedTestWidgetsFlutterBinding
    * UIは表示されない
    * https://github.com/flutter/flutter/wiki/Running-and-writing-tests#running-unit-tests
        > Unit tests run with flutter test run inside a headless flutter shell on your workstation, you won't see any UI. 
        * この文言の「Unit tests」が指すものはAutomatedTestWidgetsFlutterBindingの環境下で実行されるテストを指していると考えられる。
    * flutter testで`-d "iPhone 15"`のようにデバイスIDを指定しても無視される。
    * Dart VMと接続したい場合は --start-paused を利用する。
        > Start in a paused mode and wait for a debugger to connect. You must specify a single test file to run, explicitly. Instructions for connecting with a debugger are printed to the console once the test has started.
        * https://stackoverflow.com/questions/62491924/developer-log-does-not-print-log-in-the-test
* LiveTestWidgetsFlutterBinding, IntegrationTestWidgetsFlutterBindingの場合 
    * UIが表示される
    * 表示領域のサイズ
        * LiveTestWidgetsFlutterBinding
            * 表示領域は `Size(800.0, 600.0)` が適用される。
        * IntegrationTestWidgetsFlutterBinding
            * 物理的なサイズに合わせたサイズとなる
* TestWidgetsFlutterBinding.setSurfaceSize()
    * (未検証)
    * このメソッドで表示領域のSizeの調整が可能


# Finder
```
main() {
  testWidgets("", (widgetTester) async {
    await widgetTester.pumpWidget(MaterialApp(
      home: Column(
        children: [
          const Row(
            children: [Text("my text1")],
          ),
          const Row(
            children: [Text("my text2")],
          ),
          TextButton(onPressed: () {}, child: const Text("my text3")),
          ElevatedButton(onPressed: () {}, child: const Text("my text3")),
          const CircleAvatar(
            child: Icon(Icons.people),
          ),
          const CircleAvatar(
            backgroundColor: Colors.red,
            child: Icon(Icons.question_mark),
          ),
        ],
      ),
    ));
    expect(
        find.descendant(
            of: find.byType(MaterialApp), matching: find.byType(Text)),
        findsExactly(4));
    expect(find.ancestor(of: find.byType(Text), matching: find.byType(Row)),
        findsExactly(2));
    expect(find.ancestor(of: find.text("my text1"), matching: find.byType(Row)),
        findsOne);
    expect(find.textContaining("my text"), findsExactly(4));
    expect(find.widgetWithText(ElevatedButton, "my text3"), findsOne);
    expect(find.widgetWithIcon(CircleAvatar, Icons.people), findsOne);
    expect(find.byWidgetPredicate((widget) => widget is CircleAvatar && widget.backgroundColor == Colors.red), findsOne);
  });
}
```
```
ScrollController? getScrollController() {
  final finder = find.byType(ListView);
  if (finder.evaluate().isEmpty) return null;
  final listViewWidget = finder.evaluate().first.widget as ListView;
  return listViewWidget.controller;
}
```
* ジェネリクスを持つウィジェット
    * find.byTypeにおいて SomeWidget<dynamic>はSomeWidget<具体的な型>にはヒットしない為注意。
    * 例えば `find.byType(SomeWidget)` は SomeWidget\<dynamic\>にヒットするがSomeWidget\<int\>にはヒットしない
## byWidgetPredicateの活用: PageViewのScrollable
* 例えば下記のようにScrollableを含むウィジェットが子に存在するPageViewをスクロールしたいケースがあるとする。
* この場合、scrollUntilVisibleに指定するscrollableは、単に「PageViewの子孫のScrollable」と指定すると一意に定まらずエラーとなってしまう。
* この場合、筆者は以下のようにbyWidgetPredicateで属性を絞り込むことで PageView自身のScrollableを取得している。
```
PageView(children: [
  PageA(),// Scrollableを含む
  PageB()
])
```
```
final pageViewScrollable = find.descendant(
    of: find.byType(PageView),
    matching: find.byWidgetPredicate((widget) =>
        widget is Scrollable &&
        widget.axisDirection == AxisDirection.right));
```


# WidgetTester
* https://docs.flutter.dev/cookbook/testing/widget/introduction
* WidgetTesterはtestWidgets()のコールバック引数として渡され、WidgetTesterの各メソッドの内部では TestWidgetsFlutterBindingのシングルトンのメソッドを実行する。
* したがってこのシングルトンのruntimeTypeによって、WidgetTesterの各メソッドの振る舞いが変わる。
## WidgetTester.pump()
* https://api.flutter.dev/flutter/flutter_test/WidgetTester/pump.html
* 内部ではTestWidgetsFlutterBinding.pump()を実行する。
* AutomatedTestWidgetsFlutterBinding.pump()の場合
    * 処理
        1. Durationが指定されている場合はタイムスタンプを進める 
            * 内部ではFakeAsync.elapse(duration)が実行されている。
        2. フレームがスケジューリングされている場合(例えばsetState()の実行後など)は下記を実行され、ビルドからレンダリングまでの一連の処理が実行される。
            * マイクロタスクのフラッシュ
            * handleBeginFrame
            * マイクロタスクのフラッシュ
            * handleDrawFrame
            * マイクロタスクのフラッシュ
    * pump()を実行することでマイクロタスクがフラッシュされていること、Futureが実行されている事がわかる。
        ```
        void main() {
            testWidgets("", (widgetTester) async {
                Future.microtask(() => print("microtask1"));
                Future.microtask(() => print("microtask2"));
                Future(() => print("future"));
                Timer.run(() => print("timer run"));
                print("not yet future and microtask done");
                widgetTester.pump(Duration.zero);// durationを指定しないとFuture(※ 内部ではTimer.runを実行)は進まない。
            });

            /*
            not yet future and microtask done
            microtask1
            microtask2
            future
            timer run
            */
        }
        ```
* スタブ化した非同期処理の例
  ```
  import 'dart:convert';
  import 'dart:io';

  class OriginalClass {
    Future<int> getStubData() async {
      final request = await HttpClient().getUrl(Uri.parse("https://example.com"));
      final res = await request.close();
      final json = jsonDecode(await res.transform(utf8.decoder).join());
      return json["value"];
    }
  }
  ```
  ```
  import 'package:flutter/material.dart';
  import 'package:flutter_test/flutter_test.dart';
  import 'package:sample/original.dart

  void main() {
    testWidgets('', (tester) async {
      Stub().getStubData();
      debugPrint("get data started");
      await tester.pump(const Duration(milliseconds: 499));
      debugPrint("wait..");
      await tester.pump(const Duration(milliseconds: 1));
    });
  }

  class Stub extends OriginalClass {
    @override
    Future<int> getStubData() async {
      await Future.delayed(const Duration(milliseconds: 500));
      debugPrint("get data finished");
      return 1;
    }
  }
  /*

  get data started
  wait..
  get data finished

  */
  ```
* LiveTestWidgetsFlutterBinding.pump()の場合
    * サブクラスのIntegrationTestWidgetsFlutterBindingも同じ処理となる(同メソッドはオーバーライドされていない為)
    * FakeAsyncは使われず、フレームのスケジュールをするのみとなる。
    * 上記を`flutter run`で実行した際は、下記のようになる。(環境によっては"flutter: wait.."の方が先になるかもしれない?)
        ```
        flutter: get data started
        flutter: get data finished
        flutter: wait..
        ```
## WidgetTester.pumpWidget()
* https://api.flutter.dev/flutter/flutter_test/WidgetTester/pumpWidget.html
* 対象のウィジェットを子として、ツリーをビルドし、フレームをスケジュール、pump()を実行する。
## WidgetTester.pumpAndSettle();
* https://api.flutter.dev/flutter/flutter_test/WidgetTester/pumpAndSettle.html
* フレームがスケジューリングされている限り、pump()を繰り返し実行し続ける。
* 例えばアニメーションなどは終了するまでフレームをスケジューリングするため、アニメーションが終わるまでフレームを進行させたい場合等に利用する。
* pumpAndSettleではFakeAsyncのclockが進行する。
  * 引数のdurationにデフォルト値として100が設定されており、これがpump()のdurationへ渡される。
  * durationを0とするとアサートエラーとなる。
* タイムアウトがデフォルトで10分に設定されている。
    * したがって開始時点のタイムスタンプから10分進んだ時点でエラーとなる。
## WidgetTester.runAsync()
* https://api.flutter.dev/flutter/flutter_test/WidgetTester/runAsync.html
* 偽装されたTimerのみでは進行できない非同期メソッドを実行するために利用する。
* 例えば画像のキャッシュを行うprecacheImage()などはtestWidgets()内で直接実行してawaitしてもCompletedとはならず、処理が止まってしまう。
    * 参考
      * precacheImage()内部でImageProviderからImageStreamを生成しているが、偽装されたTimer環境下ではこのImageStreamからの受信が進まないため、と考えられる。
      * なお、pumpWidgets()で渡される画像ウィジェットはflutter test内でも処理が止まらずに非同期に読み込みが進行する。
      * 前者は処理が進まず、後者は処理が進む理由についてはコードリーディングできていない。(TODO)
## 拡張: 特定の要素が確認できるまでフレームを進める
* 例えば、結合テストでは外部のリソース等の応答時間が一定ではないため、特定の要素を確認するまでフレームを進める、といった処理を行いたいケースが多い。
* 以下の拡張は指定のFinderを確認できるまでpump()をし続ける。
```
// 参照元:
// https://github.com/flutter/flutter/issues/73355#issuecomment-805736745
// 元のコードはintegration_testではない`flutter test`で利用した場合にExceptionは出るがwhileが終了しなかったため、少し改変をしている。
extension WidgetTesterExtension on WidgetTester {
  Future<void> pumpUntilFound(
    Finder finder, {
    Duration? duration,// フェイクのクロックの際に使う場合は、durationを指定する
    Duration timeout = const Duration(seconds: 10),
  }) async {
    var found = false;
    var failed = false;
    final timer = Timer(
      timeout,
      () {
        failed = true;
      },
    );
    while (!found) {
      if (failed) {
        throw TimeoutException('Pump until has timed out $finder');
      }
      await pump(duration);
      found = any(finder);
    }
    timer.cancel();
  }
}
```

# UIの操作・他
## WidgetTester.tap, enter
```
main() {
  testWidgets("", (widgetTester) async {
    final textFieldController = TextEditingController();
    String message = "";
    await widgetTester.pumpWidget(MaterialApp(
      home: Material(
          child: Row(
        children: [
          Expanded(
              child: TextField(
            controller: textFieldController,
          )),
          IconButton(
            icon: const Icon(Icons.send),
            onPressed: () {
              message = textFieldController.text;
            },
          ),
        ],
      )),
    ));

    const inputText = "input text";
    await widgetTester.enterText(find.byType(TextField), inputText);
    await widgetTester.tap(find.widgetWithIcon(IconButton, Icons.send));
    expect(message, inputText);
  });
}
```
## WidgetTester.element
```
BuildContext getContext(WidgetTester widgetTester, [Type? type]) {
  return widgetTester.element(find.byType(type ?? Scaffold));
}
```
## アニメーションを含むウィジェットのテストは個別ウィジェットの実装に依存する。
* 例: go_routerの遷移
```
  testWidgets("", (widgetTester) async {
    final navigatorKey = GlobalKey<NavigatorState>();
    await widgetTester.pumpWidget(MaterialApp.router(
      routerConfig: GoRouter(navigatorKey: navigatorKey, routes: [
        GoRoute(
          path: "/",
          builder: (context, state) => Scaffold(
            appBar: AppBar(
              title: const Text("home title"),
            ),
          ),
        ),
        GoRoute(
          path: "/a",
          builder: (context, state) => Scaffold(
            appBar: AppBar(
              title: const Text("a title"),
            ),
          ),
        )
      ]),
    ));

    GoRouter.of(navigatorKey.currentState!.context).go("/a");
    // 単にフレームを進めても表示されない。
    // await widgetTester.pump();
    expect(find.text("a title"), findsNothing);
    // pumpAndSettleはdurationがデフォルトで指定されるためフェイクのクロックが進む
    await widgetTester.pumpAndSettle();
    expect(find.text("a title"), findsOne);
  });
```
* 例: SnackBar
    * 参考
        * https://stackoverflow.com/questions/61646891/how-to-write-a-simple-test-for-a-snack-bar-in-flutter
        * https://github.com/flutter/flutter/blob/master/packages/flutter/test/material/snack_bar_test.dart
    * 表示のアニメーションのスケジュール、開始、進行 -> 表示 -> 非表示のアニメーションのスケジュール、開始、進行 といった流れと成る。
    * 試行錯誤してみると、下記のコードが動作した。コメントは推測も混じっている。
    ```
    testWidgets('SnackBar control test', (WidgetTester tester) async {
        const String helloSnackBar = 'Hello SnackBar';
        GlobalKey key = GlobalKey();
        await tester.pumpWidget(MaterialApp(
        home: Scaffold(key: key),
        ));

        expect(find.text(helloSnackBar), findsNothing);
        ScaffoldMessenger.of(key.currentContext!).showSnackBar(const SnackBar(
            duration: Duration(seconds: 2), content: Text(helloSnackBar)));
        expect(find.text(helloSnackBar), findsNothing);
        await tester.pump(); // schedule, begin animation(show)
        expect(find.text(helloSnackBar), findsOneWidget);
        await tester
            .pump(const Duration(milliseconds: 260)); // progress animation(show)
        expect(find.text(helloSnackBar), findsOneWidget);
        await tester.pump(const Duration(
            milliseconds: 2000)); // display, schedule animation(dismiss)
        expect(find.text(helloSnackBar), findsOneWidget);
        await tester.pump(const Duration(
            milliseconds: 260)); // begin, progress animation(dismiss)
        expect(find.text(helloSnackBar), findsNothing);
    });
    ```
## WidgetTester.takeException()
```
testWidgets("", (tester) async {
    await tester.pumpWidget(MaterialApp(
        home: IconButton(
            onPressed: () => throw "error", icon: const Icon(Icons.abc))));

    await tester.tap(find.widgetWithIcon(IconButton, Icons.abc));
    expect(tester.takeException().toString(), contains("error"));
  });
```
## WidgetTester.pageBack()
* WidgetTester.pageBack()はlocaleが'en'の場合しか動作しない。
    * https://github.com/flutter/flutter/issues/51121
* 筆者は代替策として下記のように実装した。
```
Future<void> pageBack(WidgetTester tester) async {
    // 現在のlocaleに対応するページバックボタンのツールチップの文字列を取得する方法が分からなかったため、
    // flutter_localizationsの設定ファイル(app_en.arb)にハードコードしてそれを参照する。
    // (テストの際はロケーションはenがデフォルトとなる)
    Finder backButton =
        find.byTooltip(AppLocalizations.of(context)!.appbarBackTooltip);
    expect(backButton, findsOneWidget);
    await tester.tap(backButton);
    await tester.pumpAndSettle();
}
```
```
//lib/l10n/app_en.arb
{
  "@@locale":"en",
 
  "appbarBackTooltip": "Back",
  〜〜
}
```

# スクロール
* 末端へ移動
    * スクロールポジションを移動して、pumpをした時点で画面へ反映（レンダリング）される。
    ```
    Future<void> scrollJumpToEndAndPump(WidgetTester widgetTester) async {
        final finder = find.byType(ListView);
        final listViewWidget = finder.evaluate().first.widget as ListView;
        listViewWidget.controller!.jumpTo(listViewWidget.controller!.position.maxScrollExtent);

        await widgetTester.pump();
    }
    ```
    * maxScrollExtentがInfinityの場合はjumpできない。
        * maxScrollExtentが設定されている状態で呼ぶ必要がある。
        * なお、maxScrollExtentはリレイアウトの際に再計算されることで算出される。
* WidgetTester.drag
    * https://api.flutter.dev/flutter/flutter_test/WidgetController/drag.html
    * dragによってスクロールを移動させることができる。
    * スクロールポジションが変わるとリレイアウトされるのでmaxScrollExtentも再計算される。
* WidgetTester.dragUntilVisible, scrollUntilVisible
    * https://api.flutter.dev/flutter/flutter_test/WidgetController/scrollUntilVisible.html
    * scrollUntilVisibleの内部ではdragUntilVisibleが呼ばれている。
    * dragUntilVisibleメソッドの中では、find対象を発見するまでdrag/pumpのループする。
        * dragではdelta分のスクロールをドラッグし、pumpではduration分のクロックが進む。
        * dragの際にmaxScrollExtentを超える場合はmaxScrollExtentまで進む。
    * 発見したら、ループを抜けてScrollable.ensureVisibleが実行される。このensureVisibleは検証したところ「該当の要素の下端までjumpTo」する動作と考えられる。したがって、もし対象の要素がスクロールの下端にある場合は実行完了時点でスクロールの位置は最下端（maxScrollExtent）となる。
        * ただし、pumpはされないため画面にはこの分のスクロールの移動は反映されないと考えられる。
    * 参考
        * scrollUntilVisibleがListTileのonTapがnullかどうかで挙動が異なっていたためissueを作成した。
        * https://github.com/flutter/flutter/issues/143921
    
* refresyIndicatorのアクションを発生させる
    ```
    Future<void> refreshActionOnScrollable(WidgetTester tester,
        {FinderBase<Element>? scrollable,
        Offset delta = const Offset(0, 400),
        double speed = 100,
        Duration? pumpAndSettleDuration = const Duration(milliseconds: 1)}) async {
    await tester.fling(scrollable ?? find.byType(Scrollable), delta, speed);
        if (pumpAndSettleDuration != null) {
            await tester.pumpAndSettle(pumpAndSettleDuration);
        }
    }
    ```
    * 参考
        https://stackoverflow.com/questions/68357095/in-widget-tests-how-can-i-perform-a-pull-down-gesture
    * 上記はflingを使っているがdragを使ってもできそうではある。
        * flingとdragの違いは、コードを見たがあまりわからなかった。

# TestWidgetsFlutterBinding.delayed()
* https://api.flutter.dev/flutter/flutter_test/TestWidgetsFlutterBinding/delayed.html
* AutomatedTestWidgetsFlutterBindingの場合はフェイクのクロックを進める。
* LiveTestWidgetsFlutterBindingの場合はFuture.delayed()と同じ。
* Future.delayed()がペンディングのまま testWidgetsが終了すると assertエラーが発生する。
    ```
    void main() {
        testWidgets('', (tester) async {
            Future.delayed(const Duration(milliseconds: 500));
        });
    }
    /*
    Pending timers:
    Timer (duration: 0:00:00.500000, periodic: false), created:
    #0      new FakeTimer._ (package:fake_async/fake_async.dart:308:62)
    ...
    */
    ```
* これはflutter testの場合(AutomatedTestWidgetsFlutterBindingの場合)は必要であれば TestWidgetsFlutterBinding.delayed()によって下記のようにassertエラーを回避することができる。
  ```
  void main() {
      testWidgets('', (tester) async {
      Future.delayed(const Duration(milliseconds: 500));
      await tester.binding.delayed(const Duration(days: 1));
    });
  }
  ```

# ゴールデンテスト(matchesGoldenFile)
* https://api.flutter.dev/flutter/flutter_test/matchesGoldenFile.html
* https://www.youtube.com/watch?v=vka33yBz5e4
* (参考)
    * https://github.com/flutter/flutter/wiki/Writing-a-golden-file-test-for-package:flutter 
* コード例
    ```
    expect(find.byType(Scaffold), matchesGoldenFile('./golden/xxx.png'));
    ```
* --update-goldens
    * 実行時にこのオプションを指定すると、matchesGoldenFile()で指定した対象のファイルが存在しない場合は新規生成、存在する場合は上書きされる。
        * いずれの場合もmatchesGoldenFile()のexpectは成功する。
    * `flutter test --update-goldens`
        * このオプションをつけずに実行した場合
            * ファイルが存在しない場合： テスト失敗となる(したがって初回実行時はオプションを利用する必要がある)
            * ファイルが存在する場合: 画像を比較して異なる場合はテスト失敗、同一である場合はテスト成功となる。
        * matchesGoldenFile()の処理がテストコード中に存在しない場合はオプションをつけても何も生成されない。
* test/failureのディレクトリ
    * 失敗した箇所、既存のイメージ、今回のイメージのファイルが生成される。
## ゴールデンテストとImageProvider
* Flutter の 画像系ウィジェット(例えばAssetImage)で画像を読み込む際は、ImageProviderという抽象クラスを継承したオブジェクトを経由してイメージを非同期に読み込む。
* テストを実行した際、非同期のイメージの読み込みが完了していない場合は、ゴールデンテストのキャプチャに何も表示されない状態となってしまう。特にテストファイル内の冒頭のテストケースは読み込まれないことが多い。
* 対策
  * 直接、イメージの読み込みを待つことはできない。
  * precacheImageによって個別に各ウィジェットのImageProviderをキャッシュさせておく。
  * これによってイメージはキャッシュから取得されるため同期的にイメージが取得される。
  * なお、この処理は擬似的なクロックではなくリアルな非同期処理となるため、runAsync()内で実行して処理を進める必要がある。
    * 以下の参考にしたコードではrunAsync()でpumpWidget()も囲んでいるが、pumpWidget()は外に出しても問題ない。
    * https://github.com/flutter/flutter/issues/38997#issuecomment-524992589
* (参考)
  * なお、runAsync()でpumpWidgetを囲んでも解決しない
  * runAsync()は非同期処理自体を進めるAPIであり、各画像ウィジェットはもともと画像読み込みの非同期処理自体は進行するもののその完了をウィジェットが待たないため上記の問題が発生する。
    * https://github.com/flutter/flutter/issues/38997#issuecomment-524103154
    * https://github.com/flutter/flutter/issues/38997#issuecomment-524992589    
```
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

// flutter test --plain-name 'asset test'
void main() {
  group("asset test", () {
    for (int i = 1; i <= 3; i++) {
      testWidgets(i.toString(), (widgetTester) async {
        await widgetTester.pumpWidget(const MaterialApp(
            home: Scaffold(
          body: Center(
            child: CircleAvatar(
              radius: 100,
              backgroundImage: AssetImage('assets/Dash.png'),
            ),
          ),
        )));

        await cacheCircleAvatorImage(widgetTester);

        await expectLater(
            find.byType(Scaffold),
            matchesGoldenFile(
                "../golden/asset_test/${widgetTester.testDescription}.png"));
      });
    }
  });
}

Future<void> cacheCircleAvatorImage(WidgetTester widgetTester) async {
  // 確実にイメージが表示されるように、ImageProviderをキャッシュしておく。
  await widgetTester.runAsync(() async {
    // https://github.com/flutter/flutter/issues/38997#issuecomment-555687558
    for (var element in find.byType(CircleAvatar).evaluate()) {
      // print(element);
      final CircleAvatar widget = element.widget as CircleAvatar;
      final ImageProvider? image = widget.backgroundImage;
      if (image != null) {
        await precacheImage(image, element);
      }
      // 上記を実行するとrenderObject.debugNeedsPaint が trueになるためフレームを進める必要がある。
      await widgetTester.pump();
    }
  });
}
```
* また、画像が変更された場合に変更後の画像が画面に反映されるまでに複数フレームが必要であることが確認できた。下記のコードではゴールデンファイルの生成を実行する前にpumpを実行するかpumpAndSettleを実行するかによって結果のゴールデンファイルが異なる内容となる。
```

void main() {
  group("asset test", () {
    testWidgets("test", (tester) async {
      await tester.pumpWidget(const MaterialApp(
          home: Scaffold(
        body: MyWidget(),
      )));

      cat = true;
      (tester.element(find.byType(MyWidget)) as StatefulElement)
          .state
          // ignore:invalid_use_of_protected_member
          .setState(() {});

      // pumpとpumpAndSettlerのどちらを実行するかによって後続のゴールデンテストで生成されるファイルが異なる。
      //await tester.pump(); 
      await tester.pumpAndSettle();

      await cacheCircleAvatorImage(tester);

      // pumpの場合: ゴールデンテストのイメージはcatに変わっていない
      // pumpAndSettleの場合: ゴールデンテストのイメージはcatに変わっている
      await expectLater(find.byType(Scaffold),
          matchesGoldenFile("./golden/test/${tester.testDescription}1.png"));
    });
  });
}

bool cat = false;

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      backgroundImage: cat
          ? const AssetImage('assets/cat.png')
          : const AssetImage('assets/Dash.png'),
    );
  }
}
```

# スタブ・モック化
## ラッパーをスタブ化する
* 例えば下記のようにhttpClient.getUrl()のラッパー関数と、その関数をインジェクションするクラスがあるとする。
```
class Backend {
  Backend({
    required this.httpGet,
    required this.endpoint,
  });

  final String endpoint;
  final Future<(int statusCode, String responseBody)> Function(
    Uri url,
  ) httpGet;

  Future<(T, int)> request<T>(
      String path, T Function(Map<String, dynamic> json) fromJson) async {
    // ステータスコードやエラーのハンドリング等は省略
    late final int statusCode;
    late final String responseBody;
    late final Map<String, dynamic> json;
    (statusCode, responseBody) = await httpGet(Uri.parse("$endpoint/$path"));
    json = jsonDecode(responseBody);

    return (fromJson(json), statusCode);
  }
}
```
```
Future<(int statusCode, String body)> get(
    Uri url, Map<String, dynamic>? body) async {
  late String responseBody;
  late HttpClientResponse res;
  final httpClient = HttpClient();
  HttpClientRequest request;
  request = await httpClient.getUrl(url);
  request.headers.contentType = ContentType("application", "json");
  res = await request.close();
  responseBody = await res.transform(utf8.decoder).join();
  return (res.statusCode, responseBody);
}
```
* 上記のようにインジェクションされている場合は、例えばテストの際に下記のようにスタブ化させることができる。
```
test("", () async{
    final backend = Backend(
        httpGet: (_) async {
            return (200, '{"result":"success", "data":""}');
        },
        endpoint: "https://example.com");

    final result = await backend.request("/test", (json) {
        return json["result"] as String;
    });

    expect(result.$1, equals("success"));
});
```
* Mockitoを利用してモック化する方法もある
    * https://docs.flutter.dev/cookbook/testing/unit/mocking
## プラグインのテスト
* https://docs.flutter.dev/testing/plugins-in-tests
* 最も手軽な方法はプラグインのAPIの呼び出しをラップして、それをスタブ化する方法となる。 
* この方法でテストが実現できるのであれば、なるべくこの方法を採用する事が良いだろう。
* メリット
    * 利用するAPIのみ関心事とすれば実現ができる。
      * その他のAPIやクラスの仕様等の把握が必須ではない。
    * プラグインのAPIの仕様変更があっても、プロダクションコードの修正は必要になる可能性はあるが、テストのコードの修正は基本的に必要無い。
* 下記はFirebaseStorageをスタブ化する際の例となる。
* 下記のコードの場合、元のクラスをカバレッジ対象外としている。
    * この箇所についても動作確認をしておきたい場合はintegration_testで対応すればよいだろう。
* 例
  ```
  import 'dart:io';
  import 'package:firebase_storage/firebase_storage.dart';

  class FirebaseStorageClient {
    // coverage:ignore-start
    Future<String> uploadStorageFile(File file, String storagePath) async {
      final storageRef = FirebaseStorage.instance.ref().child(storagePath);
      await storageRef.putFile(file);

      final downloadUrl = await storageRef.getDownloadURL();
      return downloadUrl;
    }
    // coverage:ignore-end
  }
  ```
  ```
  import 'dart:io';
  import 'package:sample/storage.dart';

  class FirebaseStorageClientStub extends FirebaseStorageClient {
    FirebaseStorageClientStub();
    @override
    Future<String> uploadStorageFile(File file, String storagePath) async {
      return "dummy download url";
    }
  }
  ```
* プラグインのパブリックAPIをスタブ化する方法もある。
    * メリット
        * スタブ化の対象がプロダクションコード外のため、 プロダクションコード自体はカバレッジ対象にできる。
          * ※ ラッパーをスタブ化する方法は、基本的にラッパー自体はカバレッジ対象外となる。
        * プラグインのパブリックAPIを呼び出すコード自体が複雑で、スタブの出力を変えてロジックのテストが必要な場合はこの方法が良いだろう。
    * デメリット
        * プラグインのAPIに変更があるとテストの修正が必要となる。
        * プラグインが提供するクラスのコンストラクタがプライベートとなっている場合はextendsができないため、implementsする必要がある。
        * 利用するAPI以外にも関連するクラスのスタブ作成や、内部の処理を把握をする必要がある場合がある。
    * 参考
        * 下記のパッケージはcloud_firestoreパッケージのFirebaseFirestore等のクラスをスタブ化したものとなる。
            * https://github.com/megur0/firebase_stub
* プラットフォームチャネル
    * プラグインがプラットフォームチャネル(MethodChannel)を利用している際にプラットフォームチャネルをスタブ化する方法
    * 参考
        * ImagePickerのモック
        * https://stackoverflow.com/questions/76586920/mocking-imagepicker-in-flutter-integration-tests-not-working


# トラブルシューティング(IME)
* flutter_tester プロセスの残留
    * https://github.com/Dart-Code/Dart-Code/issues/4690
    * VSCodeからテストを実行するとflutter_testerのプロセスが終了せずに残ってしまう事象が確認された。
        ```
        /Users/xxx/flutter/bin/cache/artifacts/engine/darwin-x64/flutter_tester --vm-service-port=0 --start-paused --enable-checked-mode --verify-entry-points --enable-software-rendering --skia-deterministic-rendering --enable-dart-profiling --non-interactive --use-test-fonts --disable-asset-fonts --packages=/Users/path/to/project/.dart_tool/package_config.json --flutter-assets-dir=/Users/Users/path/to/project/build/unit_test_assets /var/folders/tv/xxxxxxxxxx/T/flutter_tools.xxxxxxxxxx/flutter_test_listener.xxxxxxxxxx/listener.dart.dill
        ```
    * 現状は下記コマンドでkillする事で対応している。また、テストの実行はなるべくVSCodeからではなくterminalから実行することで回避。
    * `ps aux | grep "flutter_tester --vm-service-port=0" | grep -v grep | awk '{print $2}' | xargs kill -9`
    * (参考)flutter_tester と flutter testは何が違うのか?
        * flutter testには--vm-service-portといったオプションは無い。
        * flutter_tester に関するドキュメントは見つからなかった
* ローカルネットワーク上の通信(テスト用のバックエンド等)が必要なテストにおいて`ローカルネットワーク上のデバイスの検索および接続を求めています`のダイアログによって結合テストがハングする。
    * 根本的な解決方法は見つからなかったため、筆者は結合テストの際は実機ではなくシミュレータを使っている。
    * ローカルネットワークのAPIではなく外部のエンドポイントを使う方法でも良いかもしれない。
    * https://github.com/flutter/flutter/issues/65217
* Bad state: Can't call test() once tests have begun running
    * Firebase SDKのinitializeAppやhttpのリクエスト処理を実行するとtestの開始に関するフラグが有効化され、testWidgetsの際にアサートエラーが発生してしまう。
        * `Bad state: Can't call test() once tests have begun running`
    * そのため、これらはtestWidgetsのコールバック内で実行する必要がある。
    * この事象はflutter runでの実行時のみ発生する。
        * flutter test や VSCode上の実行では発生しない。
    * スタックトレースなどを埋め込んで確認した所、Firebase.initializeAppをtestWidgetsの外で呼んだ場合、pub.dev/test_api-0.6.1/lib/src/backend/declarer.dart の build関数が、非同期に呼ばれていることがわかった。
        * これによって 内部のフラグのbool _built が trueとなり、後続のアサートでエラーとなってしまう。
    * https://github.com/flutter/flutter/issues/74706#issuecomment-955066033
    * https://github.com/firebase/flutterfire/issues/6833


# その他
## @visibleForTesting
* @visibleForTestingを付与したメンバーは外部のライブラリからはテスト以外で使用できないようにする。
```
// lib/some_library.dart
//...

@visibleForTesting
void innerLogic() {
    //...
}
```
```
// lib/another_library.dart
//...
// アナライザーによって警告が表示される
// The member 'innerLogic' can only be used within 'package:sample/main.dart' or a test.
innerLogic();

```
```
// test/some_library_test.dart
//...
innerLogic();//警告は発生しない
```
## ErrorWidget.builder, FlutterError.onError
* ErrorWidget.builderは testWidgets()のコールバック内で変更するとアサートエラーとなってしまうため、testWidgets()の外に書く。
    * `The value of ErrorWidget.builder was changed by the test`
* 一方、FlutterError.onErrorはtestWidgets()処理の中で上書きされているため、もしテストの為に上書きしたいのであればtestWidgetsのコールバック内で書く。
```
void main() {
  // testWidgetsの外に書いても上書きされてしまうため適用されない。
  // FlutterError.onError = (errorDetails) {};

  // testWidgetsの外に書く。
  ErrorWidget.builder = (_) => const Text("my error widget");

  testWidgets("setErrorWidget", (tester) async {
    FlutterError.onError = (errorDetails) {};
    await tester.pumpWidget(const MaterialApp(
      home: Scaffold(
        body: _MyApp(),
      ),
    ));
    expect(find.text("my error widget"), findsOne);
  });
}

class _MyApp extends StatelessWidget {
  const _MyApp();

  @override
  Widget build(BuildContext context) {
    throw Exception("error");
  }
}
```
## TestAsyncUtils
* https://api.flutter.dev/flutter/flutter_test/TestAsyncUtils-class.html
* awaitをせずにWidgetTester.pump()等を実行するとアサートエラーとなるが、これはWidgetTester.pumpが内部でTestAsyncUtils.guardを呼び出しているためである。
* 例えば、自前の処理でも下記のようにTestAsyncUtils.guardを呼び出すことでawait漏れを検出することができる。
```
import 'package:flutter_test/flutter_test.dart';

main() {
  testWidgets("", (widgetTester) async {
    f();
    f();
    // エラー:
    // Guarded function conflict.
    // You must use "await" with all Future-returning test APIs.
    // The guarded "f" function was called from
  });
}

Future<String> f() async {
  return await TestAsyncUtils.guard<String>(() async => "dummy text");
}
```

# (参考)flutter_testのクラスと処理
## `flutter test`における testWidgets()の処理
* 下記はflutter_testのtestWidgets()を呼んだ際、かつ`flutter test`で実行した際の処理の流れ、および関係するクラスとなる。

<img src="./svg/flutter_test/flutter_test_widgets.svg" width="100%"><br/><br/>  

* TestWidgetsFlutterBinding._instance
    * `flutter test`もしくはweb環境の場合は TestWidgetsFlutterBinding._instanceには AutomatedTestWidgetsFlutterBindingのインスタンスが設定される。

* TestWidgetsFlutterBinding.overrideHttpClient
    * trueとなっていることで、HttpClientの処理がモック化されている

* AutomatedTestWidgetsFlutterBinding.runTest()
    * FakeAsyncwを生成し、それを使って_clockに`_clock = fakeAsync.getClock(DateTime.utc(2015));`として2015年の日時を格納している。
        * AutomatedTestWidgetsFlutterBinding.pump()にてこのFakeAsyncはduration != nullの場合にelapse()を実行して進められている。また、_clockはSchedulerBinding.handleBeginFrame()にインジェクションされ現在のタイムスタンプとして設定される。
        * "2015"は特に2015である必然性は無く、elapse()によって相対的に経過する、という点にのみ意味がある(と考えられる)。
    * AutomatedTestWidgetsFlutterBinding.runTest() -> TestWidgetsFlutterBinding._runTest() -> _runBody()
        * _runBody()ではrunApp()とpump()を実行している。
        * runApp()ではダミーのウィジェットを渡してリセットのような操作をしているようだった。ただ、意図は掴めていない。

* AutomatedTestWidgetsFlutterBinding.pump()
    * この関数内ではhasScheduledFrame == trueの場合にhandleBeginFrame()やhandleDrawFrame()を実行する。
        * hasScheduledFrame == trueの場合は、scheduleFrame()がコール済の場合となる。
    * 本来はプラットフォーム側のvsync信号によってエンジン経由でこれらの関数が実行されるが`flutter test`の場合は、これが無いためpump()によって能動的に実行する。


## WidgetTester
* WidgetTester.pumpWidget(), pump(), pumpAndSettle()
    * `TestWidgetsFlutterBinding get binding => super.binding as TestWidgetsFlutterBinding;` にて TestWidgetsFlutterBindingまたはその派生オブジェクトの各メソッドを呼ぶ。
        * 実体としては、WidgetsBinding.attachRootWidget()およびwrapWithDefaultView()や SchedulerBinding.scheduleFrame()を呼んでいる。
        * また、TestWidgetsFlutterBindingの派生であるAutomatedTestWidgetsFlutterBindingや、LiveTestWidgetsFlutterBinding のpump()を呼ぶ。
## LiveTestWidgetsFlutterBinding, IntegrationTestWidgetsFlutterBinding
* 下記は LiveTestWidgetsFlutterBinding　と その派生クラス IntegrationTestWidgetsFlutterBindingのクラス図と処理の流れとなる

<img src="./svg/flutter_test/flutter_live_test_widgets.svg" width="85%"><br/><br/>  

* TestWidgetsFlutterBinding._instance
    * `flutter run`にて実行した場合は TestWidgetsFlutterBinding._instanceには LiveTestWidgetsFlutterBindingが設定される。

* LiveTestWidgetsFlutterBinding.pump()
    * FakeAsyncは使われない。
    * scheduleFrame()を呼ぶ。
        * 通常のアプリの動作と同様に後続でプラットフォーム側からのvsync信号によってhandleBeginFrame()やhandleDrawFrame()が呼ばれる。

* IntegrationTestWidgetsFlutterBinding.ensureInitialized()
    * `IntegrationTestWidgetsFlutterBinding.ensureInitialized()`を(testWidgets()の実行より前に)実行する場合はIntegrationTestWidgetsFlutterBindingが_instanceに設定される。

* IntegrationTestWidgetsFlutterBinding.overrideHttpClient
    * falseとなっているため、HttpClient処理はモック化されない。
## FlutterView
* WidgetsBinding.wrapWithDefaultView()を呼ぶため、ツリーにはView(RenderView)が存在し、View.view(dart:ui.FlutterView)にはPlatformDispatcher.implicitViewが設定されることになる。
* PlatformDispatcher.implicitViewはWidgetsBinding.wrapWithDefaultView()でnull assertionされているため、flutter testにおいても何らかの値が設定される必要がある。
* コードリーディングはできていないが、おそらく`flutter test`の場合は何らかのダミーのFlutterViewが設定されると推測している。
* なお、createViewConfigurationFor()は下記のようにオーバーライドされており、LiveTestWidgetsFlutterBindingの場合は _kDefaultTestViewportSizeが、IntegrationTestWidgetsFlutterBindingの場合はFlutterView.physicalSizeが利用されている事がわかる。
    * このメソッドはRendererBinding.addRenderView()内で呼ばれる。
    * なお、surfaceSizeの値がTestWidgetsFlutterBinding.setSurfaceSize()によってセットされている場合はそのSizeが利用される。
```
const Size _kDefaultTestViewportSize = Size(800.0, 600.0);
//...
class LiveTestWidgetsFlutterBinding extends TestWidgetsFlutterBinding {
    //...
    @override
    ViewConfiguration createViewConfigurationFor(RenderView renderView) {
        final FlutterView view = renderView.flutterView;
        if (view == platformDispatcher.implicitView) {
            return TestViewConfiguration.fromView(
            size: _surfaceSize ?? _kDefaultTestViewportSize,
            view: view,
            );
        }
        // ...
    }
    //...
```
```
class IntegrationTestWidgetsFlutterBinding extends LiveTestWidgetsFlutterBinding implements IntegrationTestResults {
    //...
    @override
    ViewConfiguration createViewConfigurationFor(RenderView renderView) {
        final FlutterView view = renderView.flutterView;
        final Size? surfaceSize = view == platformDispatcher.implicitView ? _surfaceSize : null;
        return TestViewConfiguration.fromView(
        size: surfaceSize ?? view.physicalSize / view.devicePixelRatio,
        view: view,
        );
    }
    //...
}
```
## flutter test integration_test の際に `IntegrationTestWidgetsFlutterBinding.ensureInitialized();` はどこから呼ばれるのか
* `flutter test integration_test` で実行する場合は暗黙的に `IntegrationTestWidgetsFlutterBinding.ensureInitialized();`が実行されている。
* この処理はintegration_code内のコード上では確認できなかった
* スタックフレームを出力してみたところ、下記のtest_apiというパッケージから、flutter_toolsというフォルダの_testMain を経由して実行されている。
```
#0      debugPrintStack (package:flutter/src/foundation/assertions.dart:1201:29)
#1      IntegrationTestWidgetsFlutterBinding.ensureInitialized (package:integration_test/integration_test.dart:159:5)
#2      _testMain (file:///var/folders/tv/xxxx/T/flutter_tools.xxx/flutter_test_listener.xxxxx/listener.dart:21:40)
#7      Declarer.declare (package:test_api/src/backend/declarer.dart:169:7)
#8      RemoteListener.start.<anonymous closure>.<anonymous closure> (package:test_api/src/backend/remote_listener.dart:120:24)
```
* 詳細は追っていないが 少なくともmain関数が実行される前時点で呼ばれているようだ。



