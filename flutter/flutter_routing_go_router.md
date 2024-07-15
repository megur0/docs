[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# go_router
* https://pub.dev/documentation/go_router/latest/index.html
* https://github.com/flutter/packages/tree/main/packages/go_router
* Routerに関するデリゲーター、プロパイダー、パーサー、戻るディスパッチャをラップするパッケージ
* GoRouterクラスとして宣言的にルーティング処理を記述するができる。
* GoRouterクラスはRouterConfigを実装しており、開発者はデリゲーター等の処理を意識せずにルーティング処理を記述できる。
* Flutter公式によるパッケージ
## (IMO) 名称について
* このパッケージおよび機能の集合体を指す名称として、公式には「go_router」「GoRouter」「Go Router」と少し揺れがある。
    * https://pub.dev/documentation/go_router/latest/index.html
* 筆者は「go_router」と表記している。
    * 「GoRouter」の場合はクラス名と重複するため
    * 「Go Router」の場合は筆者はGo言語を連想してしまうため
## アップデート頻度
* 現在のメジャーバージョンは14でかなりアップデート頻度の高いパッケージとなる。

# 主なプロパティ
```
GoRouter
  initialLocation: String? initialLocation
  redirect: FutureOr<String?> Function(BuildContext, GoRouterState)?
  refreshListenable: Listenable? 
  navigatorKey: GlobalKey<NavigatorState>?
  onException: void Function(BuildContext, GoRouterState, GoRouter)?
  debugLogDiagnostics: bool
  routes: [
    GoRoute
      path: String
      name: String
      builder: 
          Widget Function(
            BuildContext context,
            GoRouterState state,
        )
      routes: <RouteBase>[] 
    ...
    ShellRoute
      builder: Widget Function(
            BuildContext context,
            GoRouterState state,
            Widget child,
        ),
      branches: <RouteBase>[
        GoRoute
      ]
  ]
```
* GoRouter
    * initialLocation
        * 初期のルート
    * redirect
        * 各ルートがリビルドされる度に手前でコールされる。
    * refreshListenable
        * notifyListenerされる度にNavigatorがリビルドされる。
        * なお、GoRouter.refresh()によって命令的なリビルドを行うAPIも提供されている。
            * GoRouter.refresh()は内部ではGoRouteInformationProvider.notifyListener()を呼ぶ。
    * navigatorKey
        * 内部のNavigator.keyに設定される。
        * 参考
            * https://stackoverflow.com/questions/66139776/get-the-global-context-in-flutter
    * onException
        * https://pub.dev/documentation/go_router/latest/topics/Error%20handling-topic.html
        * どのルートにもマッチングしなかった際のエラー処理
        * 内部的には、GoRouteInformationParser内でエラー(GoException)がRouteMatchListに設定されていた際に呼ばれている。
    * debugLogDiagnostics
        * trueとすると内部でloggingパッケージによって主にGoRouterのメソッドを実行した際にログが出力される。
* GoRoute
    * path
        * 必須項目
        * サブルートは親ルートからの相対パスとなる。
    * name
        * 重複する名前でルートを定義すると実行時エラーとなる。
        * (IMO) enum等で定義して共通で利用するようにしておくと実行時エラーのリスクを軽減できるだろう
    * routes
    * GoRouteクラスをroutesでネストすることでサブルートを定義
    * 以下のどちらかを指定する必要がある。
    * builder
        * 各ルートで表示したいウィジェットを生成するコールバックを指定
        * ※ このウィジェットは内部処理において最終的にNavigator.pagesに渡す際にPage派生クラスでラップされる。
            * MaterialAppを利用しているのであればMaterialPage
            * CupertinoAppを利用しているのであればCupertinoPage
            * WidgetAppであればNoTransitionPage
    * pageBuilder
        * ウィジェットではなくPageを渡して表示動作をカスタマイズ
            * (例) MaterialPageを渡してfullscreenDialogを指定
        * CustomTransitionPageというカスタム用のクラスが用意されている。
            * https://pub.dev/documentation/go_router/latest/topics/Transition%20animations-topic.html
        
* ShellRoute/StatefulShellRoute
    * Navigatorをネストする
    * サンプルコード
        * https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/shell_route.dart
    * ShellRouteの場合は親ルートが変わるとナビゲーションスタックがリセットされる。
    * StatefulShellRouteを利用することで各画面のスタックの状態を維持する。
        * サンプルコード
            * https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/stateful_shell_route.dart

    
# GoRouter.go と GoRouter.push メソッド
|機能|メソッド|動作|
|-|-|-|
|go| GoRouter.go(...) または BuildContext.go(...)| 親ルートからサブルートへ移動したときのみスタックに積まれる。<br/>実行前のスタックは全てリセットされる。 |
|push|GoRouter.push(...) または BuildContext.go(...)|どのページへの移動でもスタックへ積まれる|
|push|GoRouter.pop(...) または BuildContext.pop(...)|スタックの先頭要素をpopする|

* BuildContextから呼ぶことができるのは、以下のようにextensionによって拡張が行われているため。
    ```
    extension GoRouterHelper on BuildContext {
        //...
        void go(String location, {Object? extra}) =>
            GoRouter.of(this).go(location, extra: extra);
        //...
    ```
* GoRouter.push, popについては、NavigatorState.push, popと等価?
    * 同じような処理になると考えられるが、実装やドキュメントを見ても"等価"かどうかは分からない。
    * https://pub.dev/documentation/go_router/latest/topics/Navigation-topic.html

* GoRouter.pushNamed, goNamed
    * GoRoute.nameを利用して遷移する。
    * タイポによるミスを軽減するために積極的に利用したほうが良いだろう。


# (参考) (例) StatefulShellRouteを使った ボトムナビゲーション(+ドローワー)を含む画面の構成の実現
* 画面の構成と対応するオブジェクトの構成のイメージは下記のようになる。
![](./svg/go_router/navigation.drawio.svg)

* Drawer
    * Drawerがボトムナビゲーションを含む全体を覆うために、親となるScaffoldへDrawerを配置する。
    * 個々の画面内のScaffoldにDrawerを設定すると、ドローワーを開いた際にそれぞれのStatefulShellRouteのスタックにつまれてしまう。
        * この場合、ボトムタブごとにDrawerの開閉状態が保持されてしまう。
* ボトムナビゲーション
    * ボトムナビゲーションのメニューボタンによって画面を切り替えても各ナビゲーションの状態は維持される。
        * これは、StatefulShellRouteは各ナビゲーションスタックの状態を維持するため。
    * Scaffold.bottomNavigationBar
        * https://api.flutter.dev/flutter/material/BottomNavigationBar-class.html
* サンプルコード
    ```
    final GlobalKey<NavigatorState> rootNavigatorKey = GlobalKey<NavigatorState>();

    final GlobalKey<ScaffoldState> rootScaffoldKey = GlobalKey<ScaffoldState>();

    void main() => runApp(MaterialApp.router(routerConfig: router));

    final router = GoRouter(
    navigatorKey: rootNavigatorKey,
    initialLocation: "/a",
    routes: <RouteBase>[
        StatefulShellRoute.indexedStack(
        builder: (BuildContext context, GoRouterState state,
            StatefulNavigationShell child) {
            return Scaffold(
            key: rootScaffoldKey,
            drawer: const Drawer(
                child: Align(
                alignment: Alignment.center,
                child: Text("drawer"),
                ),
            ),
            body: child,
            bottomNavigationBar: BottomNavigationBar(
                items: const <BottomNavigationBarItem>[
                BottomNavigationBarItem(
                    icon: Icon(Icons.home),
                    label: "A",
                ),
                BottomNavigationBarItem(
                    icon: Icon(Icons.business),
                    label: "B",
                ),
                BottomNavigationBarItem(
                    icon: Icon(Icons.notification_important_rounded),
                    label: "C",
                ),
                ],
                currentIndex: child.currentIndex,
                onTap: (int idx) {
                child.goBranch(
                    idx,
                    // 既にアクティブな箇所をタップした際は、initialLocationへ遷移させる。
                    // https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/stateful_shell_route.dart#L176
                    initialLocation: idx == child.currentIndex,
                );
                },
            ),
            );
        },
        branches: <StatefulShellBranch>[
            StatefulShellBranch(routes: [
            GoRoute(
                path: "/a",
                builder: (context, _) => getDemoScaf(
                "/a",
                ),
            ),
            ]),
            StatefulShellBranch(routes: [
            GoRoute(
                path: "/b",
                builder: (context, _) => getDemoScaf("/b",
                    onTextButtonPress: () => GoRouter.of(context).go("/b/bc"),
                    buttonText: "go /b/bc"),
                routes: [
                    GoRoute(
                    path: "bc",
                    builder: (context, _) =>
                        getDemoScaf("/b/bc", openDrawer: false),
                    ),
                ]),
            ]),
            StatefulShellBranch(routes: [
            GoRoute(
                path: "/c",
                builder: (context, _) => getDemoScaf(
                "/c",
                ),
            )
            ])
        ],
        )
    ],
    );

    Scaffold getDemoScaf(String title,
            {void Function()? onTextButtonPress,
            String? buttonText,
            bool openDrawer = true}) =>
        Scaffold(
        drawer: const Drawer(
            child: Align(
            alignment: Alignment.center,
            child: Text("drawer"),
            ),
        ),
        appBar: AppBar(
            title: Text(title),
            leading: openDrawer
                ? IconButton(
                    icon: const Icon(Icons.menu),
                    onPressed: () => rootScaffoldKey.currentState!.openDrawer(),
                )
                : null,
        ),
        body: onTextButtonPress == null
            ? null
            : TextButton(
                onPressed: onTextButtonPress,
                child: Text(buttonText ?? "button"),
                ),
        );

    ```

# (参考) (例) フルスクリーン かつ ナビゲーションスタックに積むページ
* 例えば、以下のようにプロフィール画面を任意の画面の上に表示したいケースを実現する方法の例を示す。
    ```
    任意の画面
        プロフィール画面(フルスクリーン)(「戻る」が可能)
            プロフィール編集画面(「戻る」が可能)
    ```
## 実現方法
* プロフィールとプロフィール名編集画面を親子関係にする。以下は例。
    * /profile
    * /profile/edit
* pushメソッドを利用
    * 任意の画面 -> (pushメソッド) -> /profile -> (pushメソッド) ->  /profile/edit
    * 任意の画面 <- (popメソッド)  <- /profile <- (popメソッド)  <-  /profile/edit
    * goメソッドの場合はスタックに積まれないため、「戻る」が実現できない。
    * また、下記のように２つ目をgoメソッドとする方法も不可。
        * 任意の画面 -> (pushメソッド) -> /profile -> (goメソッド) -> /profile/edit
        * この場合は２つ目の移動でスタックがリセットされてしまうので「任意の画面」にpopで戻ることができなくなる。
* (go_router 12以前の場合)parentNavigatorKeyの設定
    * Navagatorをネストしている場合(ShellRouteを利用している場合)はGoRoute.parentNavigatorKey(GlobalKey<NavigatorState>)によって対象のNavigatorを設定する必要がある。
        * pushの際に、指定したNavogatorStateのナビゲーションスタックに積まれるようになる。
        * 指定しない場合は、現在開いている画面に紐づくNavigatorのスタックに積まれる。
        * 13以降はGoRouterが定義されている箇所のNavigatorのスタックに積まれ、遷移元がどの画面でも同じ挙動となる。(Breaking change)
    * 注意(不具合?)
        * profileだけではなくeditにも、parentNavigatorKeyの設定が必要となる。
        * 設定しない場合は、/profile -> /profile/editへのpushを実行しても何もアクションが発生しない。
* サンプルコード
    ```
    final GlobalKey<NavigatorState> rootNavigatorKey = GlobalKey<NavigatorState>();

    void main() => runApp(MaterialApp.router(routerConfig: router));

    final router = GoRouter(
        navigatorKey: rootNavigatorKey,
        initialLocation: "/a",
        routes: <RouteBase>[
            GoRoute(
            // 12以前の場合は必要
            // parentNavigatorKey: rootNavigatorKey,
            path: "/profile",
            pageBuilder: (context, _) {
                return MaterialPage(
                fullscreenDialog: true,
                child: getDemoScaf(
                    "/profile",
                    onTextButtonPress: () => GoRouter.of(context).push("/profile/edit"),
                    buttonText: "push /profile/edit",
                ),
                );
            },
            routes: <RouteBase>[
                GoRoute(
                // 12以前の場合は必要
                // parentNavigatorKey: rootNavigatorKey,
                path: "edit",
                builder: (BuildContext context, GoRouterState state) {
                    return getDemoScaf(
                    "/profile/edit",
                    );
                },
                ),
            ],
            ),
            ShellRoute(
            builder: (BuildContext context, GoRouterState state, Widget child) {
                // ScaffoldWithNavBarは下記を参照
                // https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/shell_route.dart#L118
                return ScaffoldWithNavBar(child: child);
            },
            routes: <RouteBase>[
                GoRoute(
                path: "/a",
                builder: (context, _) => getDemoScaf("/a",
                    onTextButtonPress: () => GoRouter.of(context).push("/profile"),
                    buttonText: "push /profile"),
                ),
                GoRoute(
                    path: "/b",
                    builder: (context, _) => getDemoScaf("/b",
                        onTextButtonPress: () => GoRouter.of(context).go("/b/bc"),
                        buttonText: "go /b/bc"),
                    routes: [
                    GoRoute(
                        path: "bc",
                        builder: (context, _) => getDemoScaf("/b/bc",
                            onTextButtonPress: () =>
                                GoRouter.of(context).push("/profile"),
                            buttonText: "push /profile"),
                    ),
                    ]),
                GoRoute(
                path: "/c",
                builder: (context, _) => getDemoScaf("/c",
                    onTextButtonPress: () => GoRouter.of(context).push("/profile"),
                    buttonText: "push /profile"),
                )
            ],
            )
        ],
    );

    Scaffold getDemoScaf(
        String title, {
        void Function()? onTextButtonPress,
        String? buttonText,
    }) =>
        Scaffold(
        appBar: AppBar(
            title: Text(title),
        ),
        body: onTextButtonPress == null
            ? null
            : TextButton(
                onPressed: onTextButtonPress,
                child: Text(buttonText ?? "button"),
                ),
        );
    ```


# エラーハンドリング
* https://github.com/flutter/packages/blob/main/packages/go_router/example/lib/exception_handling.dart
* https://pub.dev/documentation/go_router/latest/topics/Error%20handling-topic.html

|機能|原因|ハンドリングの例|
|-|-|-|
| GoError/AssertionError|GoRouterが正しく使用されていない(ソースコードの修正が必要)| main関数でキャッチしてスタックトレース|
| GoException|ルートが存在しない場合| GoRouter.onExceptionでハンドリングしてエラー画面を表示|
```
void main() {
  testWidgets('', (WidgetTester tester) async {
    // Build our app and trigger a frame.
    await tester.pumpWidget(MaterialApp.router(
      routerConfig: GoRouter(
        routes: [
          GoRoute(
            path: '/',
            builder: (context, state) => const Text("home"),
          ),
          GoRoute(
            path: '/404',
            builder: (context, state) => const Text("not found"),
          )
        ],
        onException:
            (BuildContext context, GoRouterState state, GoRouter router) {
          router.go('/404', extra: state.uri.toString());
        },
      ),
    ));

    await tester.pump();
    expect(find.text("home"), findsOne);
    tester.element(find.text("home")).go("/not/exist/route");
    await tester.pumpAndSettle();
    expect(find.text("not found"), findsOne);
    expect(find.text("home"), findsNothing);
  });
}
```

# 実行されるビルダー
* 基本的には、ある操作をした後に残ったスタック(Navigator.pages)内のPageに紐づくルートのビルダーが実行されると考えられる。
* GoRouter.go()の場合は親ページもスタックに追加されるが、pushの場合は親ページは追加されない。
* 複数のルートのビルダーが実行される際、各ビルダーの実行順番のルールや法則はわからなかった。
* (参考コード)
    ```
    import 'package:flutter/material.dart';
    import 'package:flutter_test/flutter_test.dart';
    import 'package:go_router/go_router.dart';

    final GlobalKey<NavigatorState> rootNavigatorKey = GlobalKey<NavigatorState>();

    // flutter test --plain-name 'GoRouter'
    void main() {
    getGoRouter(String path, [List<RouteBase> children = const []]) => GoRoute(
        path: path,
        routes: children,
        builder: (_, __) {
            debugPrint(path);
            return Scaffold(
            appBar: AppBar(
                title: Text(path),
            ),
            );
        });

    testWidgets("GoRouter", (tester) async {
        await tester.pumpWidget(MaterialApp.router(
        routerConfig: GoRouter(
            navigatorKey: rootNavigatorKey,
            debugLogDiagnostics: true,
            routes: [
            getGoRouter("/"),
            getGoRouter("/a", [getGoRouter("b")]),
            getGoRouter("/c", [
                getGoRouter("d", [getGoRouter("e")])
            ]),
            ],
            redirect: (BuildContext context, GoRouterState state) {
            debugPrint("redirect");
            return null;
            },
        ),
        ));

        debugPrint("");
        for (final t in [
        (
            "go",
            "/a/b",
            (GoRouter r, String path) => r.go(path),
        ),
        (
            "pop",
            "",
            (GoRouter r, String path) => r.pop(),
        ),
        (
            "push",
            "/c/d/e",
            (GoRouter r, String path) => r.push(path),
        ),
        (
            "pop",
            "",
            (GoRouter r, String path) => r.pop(),
        ),
        (
            "go",
            "/a",
            (GoRouter r, String path) => r.go(path),
        ),
        ]) {
        debugPrint("${t.$1}${t.$2 != "" ? " ${t.$2}" : ""}: ");
        t.$3(GoRouter.of(rootNavigatorKey.currentContext!), t.$2);
        await tester.pumpAndSettle();
        debugPrint(Navigator.of(rootNavigatorKey.currentContext!)
            .widget
            .pages
            .toString());
        debugPrint("");
        }
    });
    }

    /*

    redirect
    /

    go /a/b: 
    redirect
    /a
    b
    [MaterialPage<void>("/a", [<'/a'>], {}), MaterialPage<void>("b", [<'/a/b'>], {})]

    pop: 
    /a
    [MaterialPage<void>("/a", [<'/a'>], {})]

    push /c/d/e: 
    redirect
    e
    /a
    [MaterialPage<void>("/a", [<'/a'>], {}), MaterialPage<void>("e", [<'gnyrwj[qh_jmsptntylphgqjhd]teli\'>], {})]

    pop: 
    /a
    [MaterialPage<void>("/a", [<'/a'>], {})]

    go /a: 
    redirect
    /a
    [MaterialPage<void>("/a", [<'/a'>], {})]

    */
    ```

# ディープリンク
* Flutter frameworkはディープリンクをサポートしており、FlutterDeepLinkingEnabledを有効にすることでgo_routerへディープリンクがハンドリングされる。
* go_routerのルート定義は、ディープリンクの場合でも通常のルーティング処理と同様となる。
* https://pub.dev/documentation/go_router/latest/topics/Deep%20linking-topic.html
## ディープリンクの遷移はスタックがリセットされる
* ディープリンクによる遷移ではスタックがリセットされて対象のルートへ遷移する
    * GoRouter.go()と同様の処理となる。
* ディープリンクをGoRouter.push()でハンドリングするには?
    * 方法1
        * WidgetsBindingObserver.didPushRouteInformation()を利用したStatefulWidgetでラップすることでこの動作はカスタマイズ可能だが、分かりづらいかもしれない。
        * https://github.com/flutter/flutter/issues/138632#issuecomment-1985629802
    * 方法2
        * ディープリンクの処理はFlutter Framework(go_router)ではなく、別パッケージで行う。
* 関連Issue
    * https://github.com/flutter/flutter/issues/138632


# NavigatorObserver 
* ShellRouteでは NavigatorObserver が発火しない。
* https://github.com/flutter/flutter/issues/112196


# optionURLReflectsImperativeAPIs
* go_routerの8.0.0では命令的なpushがデフォルトではURLを変更しなくなった。
    * https://pub.dev/packages/go_router/changelog#800
    * https://github.com/flutter/flutter/issues/129893#issuecomment-1617762284
    * https://github.com/flutter/flutter/issues/131083
* 具体的にはGoRouterState.uriが変わらないため、uriを使った処理をしている場合は注意が必要となる。
    * 例えばredirect処理においてはUri.pathによって分岐処理を行う事がある。
        ```
        GoRouter(
            redirect: (context, state) {
                final path = state.uri.path;
                //...
            }
        )
        ```
    * こういった場合、push命令を使った遷移を利用している箇所は意図通りに動作するか注意が必要である。
    * 参考
        * https://codewithandrea.com/articles/flutter-navigation-gorouter-go-vs-push/
* 反映させるにはGoRouter.optionURLReflectsImperativeAPIs = trueとする必要がある。


# go_router_builder
* https://pub.dev/packages/go_router_builder
* ルートやパラメータを定義して静的にチェックできる。
* コードジェネレータ(build_runner)に依存
* 以下は公式のサンプルコードから抜粋
    * 利用側
        ```
        FamilyRoute(f.id).go(context)
        ```
    * GoRouter
        * routesに`$appRoutes`を設定する。
        ```
        final _router = GoRouter(routes: $appRoutes, /* ...  */);
        ```
    * 各ルートの定義
        ```
        class FamilyRoute extends GoRouteData {
        const FamilyRoute(this.fid);

        final String fid;

        @override
        Widget build(BuildContext context, GoRouterState state) =>
            FamilyScreen(family: familyById(fid));
        }
        ```
        ```
        @TypedGoRoute<HomeRoute>(
        path: '/',
        routes: <TypedGoRoute<GoRouteData>>[
            TypedGoRoute<FamilyRoute>(
            path: 'family/:fid',
            routes: <TypedGoRoute<GoRouteData>>[
                TypedGoRoute<PersonRoute>(
                path: 'person/:pid',
                routes: <TypedGoRoute<GoRouteData>>[
                    TypedGoRoute<PersonDetailsRoute>(path: 'details/:details'),
                ],
                ),
            ],
            ),
            TypedGoRoute<FamilyCountRoute>(path: 'family-count/:count'),
        ],
        )
        ```
    * `flutter pub run build_runner build`によって上記からGoRouteの定義が生成される。


# (参考)ディープリンクでCustom URL Schemesを渡した際のエラー
* ※ 12.1.3にて確認
* 例えば「myApp://details」というカスタムURLで遷移しようとすると、下記のようなエラーが発生する。
    ```
    The following StateError was thrown while dispatching notifications for GoRouteInformationProvider:
    Bad state: Origin is only applicable schemes http and https: myapp://details
    When the exception was thrown, this was the stack:
    #0      _Uri.origin (dart:core/uri.dart:2829:7)
    #1      GoRouteInformationParser.parseRouteInformationWithDependencies (package:go_router/src/parser.dart:83:47)
    ...
    ```
    * エラーとしては、GoRouteInformationParserが内部でUriライブラリのget originを呼び出す際にhttp形式ではない場合に例外となる。
        ```
        String get origin {
            // ...
            if (scheme != "http" && scheme != "https") {
                throw StateError(
                "Origin is only applicable schemes http and https: $this");
            }
            //...
             return "$scheme://$host:$port";
        }
        ```
* 一方、「myApp://hoge/details」とした場合は動作する。
    * この場合、パスは"/details"となり"hoge"はドメイン部分として扱われる。
    * ただ、このURLもhttpではないため、Uri.originではエラーとなる。
        ```
        void main() {
            final uri = Uri.parse("http://test.com/aaaa");
            print("scheme: ${uri.scheme}, origin: ${uri.origin}, host: ${uri.host}");
            
            final uri2 = Uri.parse("myApp://hoge/details");
            print("scheme: ${uri2.scheme}, host: ${uri2.host}");// 
            print("origin: ${uri2.origin}"); // error

            // scheme: http, origin: http://test.com, host: test.com
            // scheme: myapp, host: hoge
            // エラー
        }
        ```
    * したがってこの動作は一貫性が無い事象のようにも考えられる。
* 関連するissueを確認したが仕様なのか否かがわからなかった。
    * https://github.com/flutter/flutter/issues/100624


# (参考)その他Issue
* 同じフレームで更新メソッドを使用すると、pop メソッドがルートをポップしない
    * https://github.com/flutter/flutter/issues/142394


# (参考) GoRouterのクラスの構成
![](./svg/go_router/go_router_class.svg)
## GoRouter.redirect()内でGoRouter.of(context)はエラーとなる
* 下記はエラーとなる。
```
void main() {
  testWidgets("GoRouter", (tester) async {
    await tester.pumpWidget(MaterialApp.router(
      routerConfig: GoRouter(
        navigatorKey: rootNavigatorKey,
        redirect: (context, state) {
          GoRouter.of(context);
          return null;
        },
        routes: [],
      ),
    ));
    await tester.pump();
  });
}
/*
...
No GoRouter found in context
...
When the exception was thrown, this was the stack:
#2      GoRouter.of (package:go_router/src/router.dart:507:12)
#3      main.<anonymous closure>.<anonymous closure> (file:///〜/flutter_application_12/test/widget_test.dart:14:20)
#4      RouteConfiguration.redirect.processRedirect (package:go_router/src/configuration.dart:417:80)
#5      RouteConfiguration.redirect (package:go_router/src/configuration.dart:429:14)
#6      GoRouteInformationParser._redirect (package:go_router/src/parser.dart:169:10)
#7      GoRouteInformationParser.parseRouteInformationWithDependencies (package:go_router/src/parser.dart:104:32)
#8      _RouterState._processRouteInformation (package:flutter/src/widgets/router.dart:741:8) 
...
*/
```
* これはredirectがGoRouteInformationParserから実行される際はツリー上にはInheritedGoRouterが存在しないためである。
    * InheritedGoRouterはGoRouter.ofメソッドを実現するために内部で生成されている。
* それに対してGoRoute.builder()はGoRouterDelegate.build()にて実行される。このbuild内ではInheritGoRouterオブジェクトが生成される。
    * したがってGoRoute.builder内ではof()によってGoRouterを取得することが出来る。




