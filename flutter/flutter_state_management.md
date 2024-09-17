[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > 状態管理




# 公式ドキュメント
* https://docs.flutter.dev/data-and-backend/state-mgmt
* Flutterは宣言型のUI プログラミング
  * https://docs.flutter.dev/data-and-backend/state-mgmt/declarative


# (参考)WidgetとElementの概要図

<img src="./svg/state_management/abst.svg" width="100%"><br/><br/>  


* 詳細なFlutter framework内部の処理は下記を参照
  * [アーキテクチャ(コードリーディング)](./flutter_arch_code_reading.md)


# Ephemeral state（ローカルな状態） vs app state（共有状態）
* https://docs.flutter.dev/data-and-backend/state-mgmt/ephemeral-vs-app
* Ephemeral state（ローカルな状態）
  * 現在のページで扱っている情報や、アニメーションの進行状況 などのローカル、局所的に利用する状態
  * これらはStatefulWidgetを利用する事で達成できる。
    > In other words, there is no need to use state management techniques (ScopedModel, Redux, etc.) on this kind of state. All you need is a StatefulWidget.
* App state(共有状態）
  * ユーザー情報やログイン情報、その他複数の場所から共有したい状態
  * Flutterの機能で考えると、StatefulWidgetで扱えるものがEphemeral stateであり、それ以外がApp stateとなる。
  * アプリの複雑さや性質、チームのこれまでの経験などに応じてさまざまな選択肢がある。
    > For managing app state, you'll want to research your options. Your choice depends on the complexity and nature of your app, your team's previous experience, and many other aspects. Read on.
* 何をローカルとするか、共有するかの明確な正解はない。
  * 実際のところ、State and setState()があれば全て状態を管理することが可能ではある。
    > To be clear, you can use State and setState() to manage all of the state in your app. In fact, the Flutter team does this in many simple app samples (including the starter app that you get with every flutter create).  
  > When asked about React's setState versus Redux's store, the author of Redux, Dan Abramov, replied: "The rule of thumb is: Do whatever is less awkward."
    > https://github.com/reduxjs/redux/issues/1287#issuecomment-175351978

# StatelessWidget
* https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html
* ミュータブルな変数やオブジェクトを持たない
* フィールドはすべてfinalで変更することはできない。
* build()メソッドをオーバーライドしなければならない。

# StatefulWidget/State
* https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html
* https://api.flutter.dev/flutter/widgets/State/setState.html
* StatefulWidget
  * イミュータブルであり、フィールドはすべてfinalで変更することはできない。
  * createState()メソッドをオーバーライドする
* State
  * ミュータブルな変数やオブジェクトを扱うことができる。
  * StateからStatefulWidgetのフィールドを参照するときはState.widgetプロパティを参照する。
* Stateは以下のメソッドをオーバーライドできる(代表的なメソッドのみ抜粋)
  * void initState()
    * マウント時に呼ばれる
    * リビルドの際は呼ばれない。
  * Widget build(BuildContext context)
    * 必ずオーバーライドしなければならない。
    * BuildContextを扱うことが出来る。
      * BuildContextのruntimeTypeはElement派生クラスとなる。
      * これはElementがBuildContextを実装していて、開発者はBuildContextのAPIを通してElementのメソッドを実行しているからである。
    * マウント時に呼ばれる
    * 親がリビルドされた際、子が前回のビルドと異なるオブジェクトの場合は子のbuildが呼ばれる(詳細は後述)
    * setState()によってダーティとなった場合は次回のビルドフェーズで呼ばれる。
  * void didUpdateWidget(covariant T oldWidget)
    * 親がリビルドされた際に、子が前回のビルドと異なるオブジェクトでかつ、アップデート可能であればbuild()の前に呼ばれる(詳細は後述)
    * oldWidgetを引数として受け取る。
      * build()の場合は最新のウィジェットのパラメーターのみ扱える。
  * didChangeDependencies()
    * マウント時にinitState()の直後に呼ばれる。
    * 依存先に変更があった際のリビルドの際にbuild()の手前で呼ばれる。
    * 高価な処理で、リビルドの度ではなく条件を絞って実行したい場合に利用するため、オーバーライドする機会は少ない。
      * https://api.flutter.dev/flutter/widgets/State/didChangeDependencies.html
  * void dispose()
    * 参照されなくなったツリー上のノードは非アクティブ化された後に廃棄(unmount)される。
    * その際にState.dispose()が呼ばれる。
  * Widgetが前回のビルド時のオブジェクトと同一のものであれば再利用される。
  * Widgetに関しては、const値のオブジェクトではなく、かつ、共通のオブジェクトを使いまわししていない場合(例えばbuildメソッド内で直接生成したりビルダーから生成している等)は、リビルドの度に都度生成される。

# リビルド時のElementの再利用
* https://api.flutter.dev/flutter/widgets/Element/updateChild.html
* https://api.flutter.dev/flutter/widgets/GlobalKey-class.html
* Flutterは前回ビルドされたウィジェットツリーの各ノードと、今回のビルドしているウィジェットツリーの各ノードを比較して、Widgetのオブジェクトが異なる場合でも可能な限りElementのツリーやRenderObjectのツリーは再利用をしようとする。
* ウィジェットが前回のビルド時のウィジェットと同一オブジェクトの場合
  * 紐づくElementはそのまま再利用される。
* ウィジェットが前回のビルド時と異なるオブジェクトの場合
  * Element.update()が可能である場合・・・・(X)
    * 以下をすべて満たす場合はアップデート可能と判定される
      * 前回のウィジェットとWidget.runtimeTypeが同一である。
      * Widget.keyが同一である。(両方ともnull(デフォルト値)の場合も該当する。)
    * この場合は紐づくElementのElement.update()が実行される。このupdate()の具体的な処理内容はウィジェットの種類によって実装が異なるが、基本的にはStafulWidgetとStatelessWidgetについて覚えておけば良いだろう。
      * StafulWidget/Stateの場合
        * didUpdateWidget()とbuild()が実行される
      * StatelessWidgetの場合
        * build()が実行される
  * Element.update()が不可の場合
    * 上記を満たさない場合は、Elementは再利用できず、非アクティブ化(deactivate())された後に(フレーム処理の最後に）破棄（unmount()）される。
    * Widget.keyを使った再利用の確認
      * Widget.keyがGlobalKeyである場合は、そのkeyをもとにグローバルなマップからElementを取得する。
      * 今回のウィジェットと、取得したElement.widgetとで、再度(X)のElement.update()が可能かどうかを試みる。
    * Widget.keyによる再利用ができない場合
      * ウィジェットのcreateElement()メソッドからElementを新規に生成し、Element.mount()によってマウントを行う。
      * この時、子がStafulWidgetであればinitState()とbuild(), StatelessWidgetであればbuild()が実行される。

# GlobalKey
* https://api.flutter.dev/flutter/widgets/GlobalKey-class.html
* https://docs.flutter.dev/ui#keys
* GlobalKeyをウィジェットに設定することによって以下を実現することが出来る。
  * GlobalKey.currentContext を使って直接BuildContextを参照したり、currentStateでStateを参照することができる。
  * ウィジェット内部のElementやRenderObjectのツリー(およびState)を再利用
* Widget.keyは初回のビルド(マウント)を実行した時点で、グローバルなマップ(Map<GlobalKey, Element>)へ登録され、GlobalKey.currentContextはこの値を参照する。
* したがって、初回のマウント前に参照すると実行時エラーとなる点に注意。
* 複数のウィジェットに対して同一のGlobalKeyを利用するとアサートエラーとなる。


# (参考)親がリビルドされた際の、子のビルドについて具体例
* 以下のような親子関係 において、ParentWidgetのりビルド時にChildWidgetの各メソッドが呼ばれるかどうか、State内のデータは維持されるかを複数のパターンで実行。
  * ParentWidget
    * ┗ ChildWidget (またはChildWidget2)
* 結果の確認はdebugPrintをしているのみ。(特にexpect等で検証してはいない。)
* 簡便な方法として、flutter_testを利用して直接setStateを呼んでいる。
```dart
import 'package:flutter/widgets.dart';
import 'package:flutter_test/flutter_test.dart';

// デバッグの出力のためのtestWidgetsラッパー
void testWidgetsAndPrint(
  String description,
  WidgetTesterCallback callback,
) {
  testWidgets(description, (widgetTester) async {
    debugPrint(">>> start first build(mount)");
    await callback(widgetTester);
    debugPrint("<<< test end");
  });
}

// デバッグの出力のためのmixin
mixin DebugMixin<T extends StatefulWidget> on State<T> {
  int mutableData = 0;

  @override
  void initState() {
    super.initState();
    debugPrint("$runtimeType.initState@$hashCode");
  }

  @override
  void didUpdateWidget(_) {
    super.didUpdateWidget(_);
    debugPrint("$runtimeType.didUpdateWidget@$hashCode");
  }

  @override
  Widget build(_) {
    debugPrint("$runtimeType.build@$hashCode mutableData:${(++mutableData)}");
    return Container();
  }

  @override
  void activate() {
    super.activate();
    debugPrint("$runtimeType.activate@$hashCode");
  }

  @override
  void deactivate() {
    super.deactivate();
    debugPrint("$runtimeType.deactivate@$hashCode");
  }

  @override
  void dispose() {
    debugPrint("$runtimeType.dispose@$hashCode");
    super.dispose();
  }
}

// 指定のStatefulWidgetのsetStateを実行して次のフレームへ
Future<void> setStateAndPump(
    WidgetTester tester, Type type, String? print) async {
  (tester.element(find.byType(type).first) as StatefulElement)
      .state
      // ignore:invalid_use_of_protected_member
      .setState(() {});
  debugPrint(">>> next frame${print ?? ""}");
  await tester.pump();
}

main() {
  testWidgetsAndPrint(
      "同じオブジェクトを子として使う: 子のbuild(), didUpdateWidget()は実行されない。子のStateのデータが維持される。",
      (tester) async {
    // ignore:prefer_const_constructors
    final w = ChildWidget();
    await tester.pumpWidget(ParentWidget(
      childBuilder: () => w,
    ));

    await setStateAndPump(tester, ParentWidget, "(ParentのsetStateを実行)");
    await setStateAndPump(tester, ChildWidget, "(ChildのsetStateを実行)");
  });

  testWidgetsAndPrint(
      "同じオブジェクト(const)を子として使う: 子のbuild(), didUpdateWidget()は実行されない。子のStateのデータが維持される。",
      (tester) async {
    await tester.pumpWidget(ParentWidget(
      childBuilder: () => const ChildWidget(),
    ));

    await setStateAndPump(tester, ParentWidget, "(ParentのsetStateを実行)");
  });

  testWidgetsAndPrint(
      "都度新しいオブジェクト(同一のruntimeType)を子として使う: 子のbuild(), didUpdateWidget()が実行される。子のStateのデータが維持される",
      (tester) async {
    await tester.pumpWidget(ParentWidget(
      // ignore:prefer_const_constructors
      childBuilder: () => ChildWidget(),
    ));

    await setStateAndPump(tester, ParentWidget, "(ParentのsetStateを実行)");
  });

  testWidgetsAndPrint(
      "階層が変わる: 新しい子のinitState(), build()が実行される。前の子のdispose()が実行され子のStateのデータは維持されない。",
      (tester) async {
    int count = 0;
    await tester.pumpWidget(ParentWidget(
      childBuilder: () => (count++) % 2 == 0
          ? const ChildWidget()
          : const Column(
              children: [ChildWidget()],
            ),
    ));

    await setStateAndPump(tester, ParentWidget, "(ParentのsetStateを実行)");
  });

  testWidgetsAndPrint(
      "階層は変わるがkeyを使って再利用する: 子のbuild(), didUpdateWidget()が実行される。子のStateのデータが維持される",
      (tester) async {
    int count = 0;
    final key = GlobalKey();
    await tester.pumpWidget(ParentWidget(
      childBuilder: () => (count++) % 2 == 0
          ? ChildWidget(key: key)
          : Column(
              children: [ChildWidget(key: key)],
            ),
    ));

    await setStateAndPump(tester, ParentWidget, "(ParentのsetStateを実行)");
  });

  testWidgetsAndPrint(
      "異なるruntimeTypeの子を使う: 新しい子のinitState(), build()が実行される。前の子のdispose()が実行され子のStateのデータは維持されない。",
      (tester) async {
    int count = 0;
    await tester.pumpWidget(ParentWidget(
      childBuilder: () =>
          (count++) % 2 == 0 ? const ChildWidget() : const ChildWidget2(),
    ));

    await setStateAndPump(tester, ParentWidget, "(ParentのsetStateを実行)");
  });
}

class ParentWidget extends StatefulWidget {
  const ParentWidget({
    super.key,
    required this.childBuilder,
  });
  final Widget Function() childBuilder;
  @override
  State<ParentWidget> createState() => ParentWidgetState();
}

class ParentWidgetState extends State<ParentWidget> with DebugMixin {
  @override
  Widget build(BuildContext context) {
    super.build(context);
    return widget.childBuilder();
  }
}

class ChildWidget extends StatefulWidget {
  const ChildWidget({super.key});
  @override
  ChildWidgetState createState() => ChildWidgetState();
}

class ChildWidgetState extends State<ChildWidget> with DebugMixin {}

class ChildWidget2 extends StatefulWidget {
  const ChildWidget2({super.key});
  @override
  ChildWidgetState2 createState() => ChildWidgetState2();
}

class ChildWidgetState2 extends State<ChildWidget2> with DebugMixin {}

```

# State.setState()
* void setState(VoidCallback fn)
* 内部でmarkNeedsBuild()を実行している。（dirtyであるリストへ追加され、次回のフレームでリビルドされる。)
* 下記のようにBuildContextを非同期処理の後に参照するとFlutterのリンターによって警告が表示される。
  ```dart
  @override
  Widget build(BuildContext context) {
    return TextButton(
        child: const Text("button"),
        onPressed: () async {
          await Future.delayed(const Duration(milliseconds: 100));
          ScaffoldMessenger.of(context).showSnackBar(
              // Don't use 'BuildContext's across async gaps.
              const SnackBar(
            content: Text("s"),
          ));
        });
  }
  ```
  * これは1行目は非同期処理(別のイベントとして)を待つため、完了までにdisposeされてBuildContextの中身であるElementが破棄されてしまう可能性に対する警告となる。
  * この場合は`if (mounted) ScaffoldMessenger.〜〜` のように mountedが真の場合のみ実行するようにすれば良い。
  * mountedは下記のように定義されており、これはStatefulWidgetクラスのElementが生成されていればtrueとなる。
    ```dart
    bool get mounted => _element != null;
    ```
* (参考)Flutter framework内の処理は下記のようになっている
  ```dart
    @protected
    void setState(VoidCallback fn) {
      assert(() {
        if (_debugLifecycleState == _StateLifecycle.defunct) {
          // FlutterError
        }
        if (_debugLifecycleState == _StateLifecycle.created && !mounted) {
          // FlutterError
        }
        //...
      }());
      //...
      _element!.markNeedsBuild();
    }
  ```

# BuildContext
* https://api.flutter.dev/flutter/widgets/BuildContext-class.html
  > A handle to the location of a widget in the widget tree.  
* マウント状態, ウィジェット
  * BuildContext.mounted
  * BuildContext.widget
  * ※ ほとんどのケースはState.mountedやState.widgetを使うためBuildContextから参照する機会は少ないかもしれない
* InheritedWidget関連(詳細は次節に記載)
  * BuildContext.dependOnInheritedElement()
  * BuildContext.dependOnInheritedWidgetOfExactType()
  * BuildContext.getInheritedWidgetOfExactType()
  * BuildContext.getElementForInheritedWidgetOfExactType()
* ツリー構造の探索
  * アプリケーション開発で頻繁に利用するものではないが、BuildContextのAPIを使うとすべてのツリー構造にアクセスが可能である。
  * BuildContext.findRenderObject
  * BuildContext.findAncestorWidgetOfExactType()
    ```dart
    print((context as Element).findAncestorWidgetOfExactType<MaterialApp>());
    print((context).findAncestorWidgetOfExactType<CupertinoApp>());
    ```
  * BuildContext.visitChildElements
    * https://api.flutter.dev/flutter/widgets/BuildContext/visitChildElements.html
    * 子に対して走査を行いたい場合等に利用する。
    * この数に対してO(N)の処理となるため再起的に子孫を検索するといった目的の利用は避けたい。
  * etc
* State.context
  * build()では引数でBuildContextが与えられるが、それ以外でも参照することが可能である。
    ```dart
    @override
    void initState() {
      super.initState();
      debugPrint(Material.of(context).toString());
    }
    ```
  * ただ、build()メソッド以外でcontextを使って具体的な処理を書くことは通常はないだろう。
* サンプルコード
```dart
import 'package:flutter/widgets.dart';
import 'package:flutter_test/flutter_test.dart';

// デバッグの出力のためのtestWidgetsラッパー
void testWidgetsAndPrint(
  String description,
  WidgetTesterCallback callback,
) {
  testWidgets(description, (widgetTester) async {
    debugPrint(">>> start first build(mount)");
    await callback(widgetTester);
    debugPrint("<<< test end");
  });
}

// デバッグの出力のためのmixin
mixin DebugMixin<T extends StatefulWidget> on State<T> {
  @override
  void initState() {
    debugPrint("$runtimeType.initState@$hashCode");
    dumpBuildContext(context);
    super.initState();
  }

  @override
  void didUpdateWidget(_) {
    debugPrint("$runtimeType.didUpdateWidget@$hashCode");
    dumpBuildContext(context);
    super.didUpdateWidget(_);
  }

  @override
  Widget build(_) {
    debugPrint("$runtimeType.build@$hashCode");
    dumpBuildContext(context);
    return Container();
  }

  @override
  void dispose() {
    debugPrint("$runtimeType.dispose@$hashCode");
    dumpBuildContext(context);
    super.dispose();
  }
}

// 指定のStatefulWidgetのsetStateを実行して次のフレームへ
Future<void> setStateAndPump(
    WidgetTester tester, Type type, String? print) async {
  (tester.element(find.byType(type).first) as StatefulElement)
      .state
      // ignore:invalid_use_of_protected_member
      .setState(() {});
  debugPrint(">>> next frame${print ?? ""}");
  await tester.pump();
}

main() {
  testWidgetsAndPrint("", (tester) async {
    await tester.pumpWidget(NotificationListener<MyNotification>(
      child: const MyWidget(),
      onNotification: (notification) {
        debugPrint("Notification catched: $notification");
        return true;
      },
    ));

    BuildContext e = tester.element(find.byType(MyWidget).first);
    debugPrint(e.describeElement("describeElement").value.toString());
    debugPrint(e.describeWidget("describeWidget").value.toString());
    e.dispatchNotification(MyNotification());
    debugPrint(e.findRenderObject().toString());
    debugPrint(e
        .findAncestorWidgetOfExactType<NotificationListener<MyNotification>>()
        .toString());
  });
}

class MyNotification extends Notification {}

void dumpBuildContext(BuildContext context) {
  void debugWithIndent(String str) => debugPrint("   $str");

  debugWithIndent("mounted:${context.mounted.toString()}");
  if (!context.owner!.debugBuilding && context.mounted) {
    debugWithIndent("size:${context.size.toString()}");
  }
  if (context.mounted) {
    debugWithIndent("widget:${context.widget.toString()}");
  }
  debugWithIndent("debugDoingBuild:${context.debugDoingBuild.toString()}");
}

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});
  @override
  MyWidgetState createState() => MyWidgetState();
}

class MyWidgetState extends State<MyWidget> with DebugMixin {}

```


# (参考) StatefulWidget/Stateを一つのクラスで扱う
* widget側でStateのライフサイクルのメソッドを受け取るようにすれば可能。
* BuildContextやsetState()、ミュータブルの更新については、メソッドの引数や戻り値で渡される設計となる。
* 参考コード
  * https://stackoverflow.com/questions/53019294/what-is-the-usefulness-of-immutable-statefulwidget-and-state-in-flutter-but-ca/53019505#53019505
  * なお、本文の`It is however easily doable to reverse the lifecycles between State and StatefulWidget` の`reverse the lifecycles`の意味するところは "ライフサイクルを逆にする"というより"ライフサイクルメソッドをStatefulWidgetの方で指定する" というニュアンスで書いているものと考えられる。
* 関連
  * StatefulWidget syntax in Flutter requires 2 classes
    * https://github.com/dart-lang/language/issues/329


# 非同期処理
* Flutter(宣言的UI)を実装する上で、その特徴を強く意識するタイミングは非同期処理かもしれない(IMO)
* Flutterにおいて、非同期処理を行う上では以下のように非同期処理と表示制御を分けて考える必要がある。
    * 非同期処理
      * 非同期の処理が実行される
        * これは、明示的にコードでFutureを返す関数を呼ぶ場合や、ユーザーアクション(ボタンのタップ等)によるI/O処理が該当する。
      * 非同期処理が完了したタイミングの処理
        * 完了時にsetState()を行う。(例えば処理をawaitしてから呼ぶ、あるいはTextButtonのonTap内で呼ぶ、等)
          * setState()は次回のビルドの「スケジューリング」を行う関数となる
    * 表示制御(build())
      * 非同期処理が完了するまでの描画処理(例えばローディング表示など)
      * 非同期処理が完了した後の描画処理(取得結果の表示など)
* Flutterのフレーム処理は同期的
  * 具体的には、Flutterの各フレーム処理（ビルド〜レンダリング情報のエンジンへの送信）は同期的に行われる（イベントループ上、一つのイベント内で同期的に行われる）ため、その途中で非同期処理を実行したとしても、完了を待つことは無い。
  * 例えば、Stateの各メソッドをasyncにするとFlutter framwork内のassertチェックによってエラーが発生する。
    * ※ assertのためdebugモード以外では動作する。ただ、Flutter frameworkの設計意図に反した書き方のため避けた方が良いだろう。
* (IMO)
  * 上記のように同期的な表示制御と、非同期な処理（I/Oや外部からデータ取得 等)を疎結合にした仕組みが、宣言的UIの特徴と考えると分かりやすいかもしれない。
* 初心の際、筆者は下記のように全体を同期的な流れで考えていたのだが、1と4 に対して 2と3 は分けて考える必要がある。
    * 1.ローディング画面を表示する。
    * 2.非同期の処理を呼び出して非同期処理の結果を待つ。
    * 3.非同期の処理の結果に基づいて処理を行う
    * 4.画面を描画する。
* サンプルコード
```dart
import 'package:flutter/material.dart';

main() => runApp(const MaterialApp(
      home: MyWidget(),
    ));

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  List<int>? data;

  @override
  void initState() {
    super.initState();
    getDummyData();// 非同期処理の呼び出し(待たない。)
  }

  getDummyData() async {
    await Future.delayed(const Duration(milliseconds: 500), () {
      data = Iterable.generate(100, (i) => i).toList();// 非同期処理が完了したタイミングの処理
      setState(() {});
    });
  }

  @override
  Widget build(BuildContext context) {// 表示制御
    return data == null
        ? const Center(// 非同期処理が完了するまでの表示。
            child: CircularProgressIndicator(),
          )
        : ListView.builder(// 非同期処理が完了した後の表示
            itemCount: data!.length,
            itemBuilder: (_, i) => Text(data![i].toString()));
  }
}
```
* 以下のような記述に対してFlutterはassertエラーを出す。
  * これは、フレームワーク内の一連の initState() -> ... -> build() -> ... の流れは非同期処理を待つこと(awaitすること)なく行われるため、asyncをつけることはこの設計意図に反しているため、となる。
  ```dart
    @override
    void initState() async {// asyncをつけるとassertエラーとなる。
      super.initState();
      await getDummyData();
      setState((){});// なお、初回(マウント)は必ずbuild()が実行されるため、initState()内のsetState()は意味がない。
    }

    getDummyData() async {
      await Future.delayed(const Duration(milliseconds: 500), () {
        data = Iterable.generate(100, (i) => i).toList();
      });
    }
  ```
## FutureBuilder
* https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html
* FutureBuilderは 渡したFutureの状態に応じて内部でリビルドを行うStatefulWidget派生クラスである。
* 前節の例では非同期処理の結果を変数に格納したり、非同期処理が完了したタイミングでsetState()を実行していたが、例えば下記のようにFutureBuilderで記述する場合はそれらの記述は不要となる。
```dart
import 'package:flutter/material.dart';

main() => runApp(const MaterialApp(
      home: MyWidget(),
    ));

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  Future<List<int>> getDummyData() async {
    await Future.delayed(const Duration(milliseconds: 500), () {});
    return Iterable.generate(100, (i) => i).toList();
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      future: getDummyData(),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          return ListView.builder(
              itemCount: snapshot.data!.length,
              itemBuilder: (_, i) => Text(snapshot.data![i].toString()));
        } else if (snapshot.hasError) {
          return const Text("error");
        } else {
          return const Center(
            child: CircularProgressIndicator(),
          );
        }
      },
    );
  }
}
```
## StreamBuilder
* https://api.flutter.dev/flutter/widgets/StreamBuilder-class.html
* FutureBuilderと同様にStream値をそのまま渡す事ができるウィジェット


# InheritedWidget
* https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html
* ツリーの先祖にInheritedWidget派生オブジェクトが存在する際、子孫は以下のメソッドを通して監視、取得することができる。
  * BuildContext.dependOnInheritedElement()
  * BuildContext.dependOnInheritedWidgetOfExactType()
  * BuildContext.getInheritedWidgetOfExactType()
  * BuildContext.getElementForInheritedWidgetOfExactType()
* 通常、Flutterでは 上記のAPIをラップしてstaticなof(BuildContext)メソッドやmaybeOfメソッドを開発者へ提供する。
* (参考)
  * 内部の動作としてはInheritedElement.updated()の際に自身に依存するElementのdidChangeDependencies()を呼ぶ。
  * それによって依存先のElementのmarkNeedsBuild()が呼ばれてダーティとなる。
  * なお、StatefulElementの場合はState.didChangeDependencie()も呼ばれる。
## 具体的なユースケース
* 複数の子孫ウィジェットで共通で使うデータを持たせたい・監視させたい場合  
  * ChangeNotifier等はクラスの外側で宣言するかコンストラクタインジェクションをする必要があるが、InheritedWidgetの場合は子孫であればcontext経由で取得・監視が可能となる。
* 例として、_Backgroundウィジェットと_Textウィジェットで構成されるLogoウィジェットを考えてみる。_Backgroundと_Textだけが利用したい共通のイミュータブルなデータがあるとする。
* 構成としては以下の図のようなものが考えられる。
    * ポイントとしては下記。
      * _LogoDataの情報は_logoDataの子孫のウィジェットからのみ参照できる
        * ※ なお、BuildContext.visitChildren()を先祖から再起的に利用して_LogoDataを検索することで取得することも可能だが、通常は行うことはないだろう。
      * _Backgroundや_Textのリビルドは、_LogoDataが再生成されてsizeやcolorに変化があった時のみ。
        * それ以外でビルドされることはない。
    * ※ LogoContorllerに関しては主旨ではないが、ロゴのライブラリと利用側を完全に切り離したほうが実際のユースケースに近いと考えて用意した。


  <img src="./svg/state_management/inherit_widget_logo.svg" width="30%"><br/><br/>  

 

* ソースコードは下記のようになる。

```dart
// lib/logo.dart
import 'package:flutter/material.dart';

enum LogoAspect { backgroundColor, large }

class LogoController {
  late _LogoState _logo;

  Color get color => _logo._color;

  void attach(_LogoState logo) {
    _logo = logo;
  }

  void changeColor(Color color) {
    _logo._color = color;
    _logo._setState();
  }

  void changeSize() {
    _logo._large = !_logo._large;
    _logo._setState();
  }
}

class Logo extends StatefulWidget {
  const Logo({super.key, this.initialColor = Colors.blue, this.logoController});

  final Color initialColor;
  final LogoController? logoController;

  @override
  State<Logo> createState() => _LogoState();
}

class _LogoState extends State<Logo> {
  bool _large = false;
  late Color _color = widget.initialColor;

  void _setState() => setState(() {});

  @override
  void initState() {
    super.initState();
    if (widget.logoController != null) {
      widget.logoController!.attach(this);
    }
  }

  @override
  Widget build(BuildContext context) {
    return _LogoData(
      backgroundColor: _color,
      large: _large,
      child: const _Background(
        child: _Text(),
      ),
    );
  }
}

class _LogoData extends InheritedWidget {
  const _LogoData({
    this.backgroundColor,
    this.large,
    required super.child,
  });

  final Color? backgroundColor;
  final bool? large;

  static Color? backgroundColorOf(BuildContext context) {
    return context
        .dependOnInheritedWidgetOfExactType<_LogoData>()
        ?.backgroundColor;
  }

  static bool? sizeOf(BuildContext context) {
    return context
        .dependOnInheritedWidgetOfExactType<_LogoData>()
        ?.large;
  }

  @override
  bool updateShouldNotify(_LogoData oldWidget) {
    return backgroundColor != oldWidget.backgroundColor ||
        large != oldWidget.large;
  }
}

class _Background extends StatelessWidget {
  const _Background({required this.child});

  final Widget child;

  @override
  Widget build(BuildContext context) {
    final Color color =
        _LogoData.backgroundColorOf(context) ?? Colors.white;
    debugPrint("BackgroundWidget build");

    return AnimatedContainer(
      //padding: const EdgeInsets.all(12.0),
      color: color,
      duration: const Duration(seconds: 2),
      //curve: Curves.fastOutSlowIn,
      child: child,
    );
  }
}

class _Text extends StatelessWidget {
  const _Text();

  @override
  Widget build(BuildContext context) {
    debugPrint("LogoWidget build");
    final bool largeLogo = _LogoData.sizeOf(context) ?? true;

    return Icon(
      Icons.abc,
      size: largeLogo ? 200.0 : 100.0,
    );
  }
}
```
```dart
// lib/main.dart
import 'package:flutter/material.dart';
import './logo.dart';

void main() {
  runApp(MaterialApp(
    home: MyWidget(),
  ));
}

class MyWidget extends StatelessWidget {
  MyWidget({super.key});

  final LogoController controller = LogoController();

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: [
        Center(
          child: Logo(
            initialColor: Colors.blue,
            logoController: controller,
          ),
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            TextButton(
                onPressed: () => controller.changeColor(
                    controller.color == Colors.blue ? Colors.red : Colors.blue),
                child: const Text('Update background color')),
            TextButton(
                onPressed: () => controller.changeSize(),
                child: const Text('Resize Logo'))
          ],
        )
      ],
    );
  }
}
```

* 上記のコードは以下を参考にしている。
  * https://api.flutter.dev/flutter/widgets/InheritedModel-class.html

* その他
  * Flutter frameworkのようにパッケージやフレームワークとして機能提供する場合はInheritedWidgetを使う機会が多いだろう。
  * 以下のサンプルコードは自前でThemeクラス(のようなもの)をInheritedWidgetで定義したもの
  * https://gist.github.com/megur0/eb782ff7053508d20b24325fc2537cbf


# InheritedModel
* https://api.flutter.dev/flutter/widgets/InheritedModel-class.html
* 前節のInheritedWidgetでは、リビルドの際にすべての依存先をダーティにする。
* InheritedModelではダーティとする依存先を取捨選択することができる。
* 具体的には
  * 監視の際に InheritedModel.inheritFrom()でObject型の "aspect" を紐づけておき、
  * 依存先をダーティにする際の条件としてupdateShouldNotifyDependent()をオーバーライドして依存先ごとに保持している"aspect"に応じて条件設定を行うことができる。
* サンプルは公式のAPIドキュメントを参照

# (参考)InheritedNotifier
* https://api.flutter.dev/flutter/widgets/InheritedNotifier-class.html
* ChangeNotifierやValueNotifierなどのnotifierの通知を受け取ると、それをInheritedWidgetとしてツリーで通知する

# Listenable
* https://api.flutter.dev/flutter/foundation/Listenable-class.html
  > The listeners are typically used to notify clients that the object has been updated.
* Listenableは変更を知らせるために使う。

```dart
abstract class Listenable {
  const Listenable();
  //...
  void addListener(VoidCallback listener);
  void removeListener(VoidCallback listener);
}
```
* 利用側はaddListenerへコールバックを渡すことで、Listenableオブジェクトの変更をトリガーとして実行することが出来る。 
  * コールバックは同期的に呼ばれる。
* 代表的な派生クラスにはChangeNotifier, ValueNotifier がある。
* ChangeNotifierは mixin classであり extendsやwithすることができる。
  * (IMO)新たなnotifier系の機能を開発したい場合はextendsするが、通常はwithでChangeNotifierの機能を使えば十分だろう。
* BuildContext(Element)を経由して依存やデータ取得をビルドサイクルの中で行うInheritedWidget(ProxyWidget)とは異なり、Listenableはウィジェットから完全に独立した機能となる。
```dart
import 'package:flutter/material.dart';
main () {
  final notifier = MyNotifier();
  notifier.addListener(()=> print("notified: ${notifier.data}"));
  notifier.updateData(3);
  // notified:3
}

class MyNotifier with ChangeNotifier {
  int data = 0;
  
  void updateData(int data) {
    this.data = data;
    notifyListeners();
  }
}
```
## ListenableBuilder
* https://api.flutter.dev/flutter/widgets/ListenableBuilder-class.html
* Listenableの更新の度にリビルドされるウィジェット
```dart
main() {
  testWidgets("", (tester) async {
    final notifier = MyNotifier();

    dump() => debugPrint(
        (tester.element(find.byType(MyWidget).first) as StatelessElement)
            .toStringDeep());

    await tester.pumpWidget(MaterialApp(
      home: MyWidget(myNotifier: notifier),
    ));

    dump();
    notifier.updateData(3);
    await tester.pump();
    dump();
  });
}

class MyNotifier with ChangeNotifier {
  int data = 0;

  void updateData(int data) {
    this.data = data;
    notifyListeners();
  }
}

class MyWidget extends StatelessWidget {
  const MyWidget({super.key, required this.myNotifier});

  final MyNotifier myNotifier;

  @override
  Widget build(BuildContext context) {
    return ListenableBuilder(
      listenable: myNotifier,
      builder: (context, child) => Text(myNotifier.data.toString()),
    );
  }
}

/*

MyWidget
└ListenableBuilder(listenable: Instance of 'MyNotifier', state: _AnimatedState#36861)
 └Text("0", dependencies: [DefaultSelectionStyle, DefaultTextStyle, MediaQuery])
  └RichText(softWrap: wrapping at box width, maxLines: unlimited, text: "0", dependencies: [Directionality, _LocalizationsScope-[GlobalKey#dbb61]], renderObject: RenderParagraph#d5d13)

MyWidget
└ListenableBuilder(listenable: Instance of 'MyNotifier', state: _AnimatedState#36861)
 └Text("3", dependencies: [DefaultSelectionStyle, DefaultTextStyle, MediaQuery])
  └RichText(softWrap: wrapping at box width, maxLines: unlimited, text: "3", dependencies: [Directionality, _LocalizationsScope-[GlobalKey#dbb61]], renderObject: RenderParagraph#d5d13)

*/
```

# ValueNotifier
* https://api.flutter.dev/flutter/foundation/ValueNotifier-class.html
* 単一の値を保持するChangeNotifier。
* value が等価演算子 == によって評価された古い値と等しくないものに置き換えられるとリスナーに通知する。
```dart
void main() {
  test('valueNotifier test', () async {
    final notifier = ValueNotifier<int>(1);
    var isListenerCalled = false;
    notifier.addListener(() {
      isListenerCalled = true;
    });
    notifier.value = notifier.value;
    expect(isListenerCalled, false);// 値が変わらなければリスナーはコールされない。
    notifier.value += 1;
    expect(isListenerCalled, true);
  });
}
```


# (IMO) アプリ全体で利用するデータは、Listenable派生クラスとInheritedWidget派生クラスのどちらを利用するか?
* これについて、筆者はどちらでも好みのものを選択すれば良いと考える。
* Listenable派生クラスであればChangeNotifier等を継承させたクラスを外部に定義して、それを直接参照するか適宜インジェクションしたものを各画面でListenableBuilderなどで監視すれば良いだろう。
* InheritedWidget派生クラスであれば、runAppの直下に配置すればすべての画面から参照・監視が可能となる。
* また、これらは数多くのラッパーとなるサードパッケージが存在するため、それらを使っても良いし、自分でラッパークラスを作成しても良い。


