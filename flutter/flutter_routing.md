- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# NavigatorとRouter
* https://docs.flutter.dev/ui/navigation
    > Flutter provides a complete system for navigating between screens and handling deep links. Small applications without complex deep linking can use Navigator, while apps with specific deep linking and navigation requirements should also use the Router to correctly handle deep links on Android and iOS, and to stay in sync with the address bar when the app is running on the web.
* Flutterにおいては、NavigatorとRouterの２つの仕組みが提供されている。

# Navigator
* Navigator自体はStatefulWidgetである。
    * Navigator2.0と言われているものから通常のウィジェットのように状態を持つウィジェットとなった。
    * MaterialAppやCupetinoApp, WidgetsAppではデフォルトでNavigatorウィジェットで包含される。
* ナビゲーションスタックで遷移を管理する。
    * ナビゲーションスタック
        * ページが積まれる(push)ことで「戻る」アクションが可能になる。
        * スタックが空では無い状態では、NavigatorState.canPop()がtrueとなる。
        * AppBar.leadingではこのスタック状態を確認して戻るボタンの表示を制御している。
* スタック情報はNavigator.pagesとなる。
    ```
    class Navigator extends StatefulWidget {
        // ...
        final List<Page<dynamic>> pages;
        // ...
    }
    ```
    * MaterialAppで使われるMaterialPageはPageの派生クラス。
* Navigator.of(context) はBuildContextからNavigatorStateを取得する
* サンプルコード
    ```
    main() => runApp(MaterialApp(
            home: Navigator(
        pages: const [
            MaterialPage(child: Page("page C")),
            MaterialPage(child: Page("page B")),
            MaterialPage(child: Page("page A")),
            MaterialPage(child: Top())
        ],
        // pages利用の際は、onPopPageを設定する必要がある。（設定しない場合はエラーとなる。）（
        onPopPage: (route, result) => route.didPop(result),
        )));

    class Top extends StatelessWidget {
    const Top({super.key});

        @override
        Widget build(BuildContext context) {
            return Page("top page",
                child: Column(
                children: [
                    TextButton(
                    onPressed: () {
                        Navigator.of(context).push(MaterialPageRoute(
                            builder: (_) => const Page("pushed page")));
                    },
                    child: const Text("push page"),
                    ),
                    TextButton(
                        onPressed: () => Navigator.of(context).pop(),
                        child: const Text("pop page"))
                ],
                ));
        }
    }

    class Page extends StatelessWidget {
        const Page(this.pageName, {this.child, super.key});

        final String pageName;
        final Widget? child;

        @override
        Widget build(BuildContext context) {
            return Scaffold(
            appBar: AppBar(
                title: Text(pageName),
            ),
            body: child,
            );
        }
    }
    ```


# MaterialApp.routes(公式では推奨されていない)
* https://docs.flutter.dev/ui/navigation#using-named-routes
* https://docs.flutter.dev/cookbook/navigation/named-routes
* https://docs.flutter.dev/cookbook/navigation/navigate-with-arguments
* シンプルなルーティング情報の定義
* ディープリンクを処理可能。
* 引数はNavigator.pushNamedメソッドのargumentsで渡す。
* 動作は、常にNavigatorへのpushのみで、カスタマイズは不可。
    * シンプルな機能のため、公式ドキュメントでは"We don't recommend using named routes for most applications." と記載されている。
```
// 公式サンプル
MaterialApp(
    routes: {
        '/': (context) => HomeScreen(),
        '/details': (context) => DetailScreen(),
    },
)
onPressed: () {
    Navigator.pushNamed(
        context,
        '/details',
        arguments: 3, // Object?型
    );
},
```

# Router
* StatefulWidget派生クラス
* 以下のような処理が可能な、汎用的なカスタマイズ可能なルーティングの仕組みとなる。
    * historyの操作
    * Navigator を入れ子とする(Nested Navigator)
    * 宣言的なルーティングの定義
    * Webアプリの戻る/進むボタン、ロケーションバーとの連動
    * Deep linkの処理
* Routerウィジェットを直接利用するのではなく、MaterialApp.routerコンストラクタ等へRouterConfigオブジェクトを渡すことで利用する。            
    ```
    MaterialApp.router(
        // ...
        routerConfig: RouterConfig(//...),
    );
    ```
    * これによって内部的にRouterウィジェットが生成される。
    * RouterConfigにはデリゲーター(必須), ルーター情報のプロバイダ、パーサー、戻るボタンのディスパッチャとして抽象クラスを実装したオブジェクトを渡す必要がある。
    * サンプルコードとして下記のサイトが参考となった。
        * https://medium.com/flutter/learning-flutters-new-navigation-and-routing-system-7c9068155ade
* アプリケーション開発で一般的なルーティング処理を目的とするのであれば、go_router等のパッケージを利用することで通常は問題ない。
    ```
    MaterialApp.router(
        // ...
        routerConfig: GoRouter(//...),
    );
    ```


# Deep link
* https://docs.flutter.dev/ui/navigation/deep-linking
* FlutterはRouter（あるいはroutesやonGenerateRouteパラメータ）によってDeep link（Custom URL Schemes と UniversalLink）をサポートしている。
* 以下のセットアップを行うことで有効化される。
    * なおFlutterの公式ドキュメントには以下の両方の設定について記載されている。
        * 各プラットフォームでディープリンクを有効化する設定
        * FlutterのRouterウィジェット等のハンドリングを有効化する設定(FlutterDeepLinkingEnabled や flutter_deeplinking_enabled の設定)
    * Router等を使わずに、別のパッケージ(uni_links等)でディープリンクをルーティング処理とは分けて処理するといった場合は前者の手順のみ実施する。
        * 後者の手順を実施すると、サードパーティツール側の動作を壊してしまうという記載がある。
        > Note: The FlutterDeepLinkingEnabled property opts into Flutter's default deeplink handler. If you are using the third-party plugins, such as uni_links, setting this property will break these plugins. Skip this step if you prefer to use third-party plugins.
* Web
    * 特に追加のセットアップは不要。
* iOS
    * https://docs.flutter.dev/cookbook/navigation/set-up-universal-links
    * Routerへハンドリングさせるにはplistで以下を設定する。
    ```
    <key>FlutterDeepLinkingEnabled</key>
    <true/>
    ```
* Android
    * https://docs.flutter.dev/cookbook/navigation/set-up-app-links
    * AndroidManifest.xmlにflutter_deeplinking_enabledを設定。
## (IME)(参考)uni_links を追加済の場合
* uni_linksを既にパッケージに追加している場合、FlutterDeepLinkingEnabledを動作させるにはuni_linksパッケージを削除する必要がある
* 筆者はuni_linksのプラグインをインストールした状態のまま、上記のiosの手順を実行してFlutterDeepLinkingEnabledをtrueにしたところ、以下の問題が発生した。
    * uni_links自体は動作した。一方、Flutter Framework側でRouterへハンドリングされない。
* おそらくプラグイン側のネイティブコードが上記の設定よりも優先されていると考えられる。
    * ドキュメントにはプラグイン側がbreakされると書かれているが、uni_linksの場合は逆となった。
* uni_linksを依存関係から削除して再度ビルドしたところ、Routerへハンドリングされるようになった。


# (参考)MaterialApp, CupertinoApp, WidgetsApp と Navigator
* _WidgetsAppStateでは下記のようにRouter または Navigatorを 内部で生成している。
    * ※ MaterialApp, CupertinoAppもWidgetsAppを利用している。
    * 例えば MaterialApp(home:Text("test")) といった指定方法の場合はRouterウィジェットは使われず内部でNavigatorウィジェットが生成される。
```
class _WidgetsAppState extends State<WidgetsApp> with WidgetsBindingObserver {
    // ...
    bool get _usesRouterWithDelegates => widget.routerDelegate != null;
    bool get _usesRouterWithConfig => widget.routerConfig != null;
    bool get _usesNavigator => widget.home != null
        || (widget.routes?.isNotEmpty ?? false)
        || widget.onGenerateRoute != null
        || widget.onUnknownRoute != null;
    // ...
    Widget build(BuildContext context) {
        Widget? routing;
        if (_usesRouterWithDelegates) {
            routing = Router<Object>(/* ... */, routerDelegate: widget.routerDelegate!,);
        } else if (_usesNavigator) {
            routing = FocusScope(
                //...
                child: Navigator(/* .... */),
            );
        } else if (_usesRouterWithConfig) {
            routing = Router<Object>.withConfig(
                //...
                config: widget.routerConfig!,
            );
        }
    }
}
```

# (参考)NavigatorStateの実装
* 下記のような実装となっていた
    * _historyというIterable<_RouteEntry>のメンバーを保持
        * _RouteEntryは、Routeオブジェクト(実際には具象はMaterialRoute等)を持つ。
        * RouteオブジェクトはNavigatorStateを持っているため、これによってNavigatorがネスト可能になっていると考えられる。
        * RouteオブジェクトのList<OverlayEntry> get overlayEntriesが実際にウィジェットとしてビルドされるビルダーを保有する。
    * push, pop命令などによって_historyが操作される。
    * _historyが操作されるとNavigationNotificationがdispatchされる。（という処理が行われるコールバックをInitStateでaddListenerしている）
        * これは他のウィジェットがスタックの変更についてハンドリング（リビルド等）を行う為？
        * なお、自身のbuildメソッド内でもNotificationListener<NavigationNotification>を使っていて、少し処理を挟んで子孫へ伝播させている。
    * 各命令実行時には、_flushHistoryUpdatesが実行され、_historyの中の_RouteEntry.currentState（enum値）に応じて処理を行う。
    * _flushHistoryUpdatesの最後にOverlay(buildメソッドで作成するウィジェット。GlobalKey経由で参照)のメソッドを実行して、状態をdirtyにする。
```
class Navigator extends StatefulWidget {
    // ...
    final List<Page<dynamic>> pages;
    // ...
}
```
```
class NavigatorState extends State<Navigator> with TickerProviderStateMixin, RestorationMixin {
    //...
    _History _history = _History();
    //...
    bool get _usingPagesAPI => widget.pages != const <Page<dynamic>>[];// _usingPagesAPIはassertのみで利用。
    //...
    void initState() {
        //...
        _history.addListener(_handleHistoryChanged);
    }
    //...
    // restoreStateはState.initStateの直後に呼ばれる。
    @override
    void restoreState(RestorationBucket? oldBucket, bool initialRestore) {
        //...
        for (final Page<dynamic> page in widget.pages) {
            final _RouteEntry entry = _RouteEntry(
                page.createRoute(context),
                pageBased: true,
                initialState: _RouteLifecycle.add,
            );
            //...
            _history.add(entry);
            _history.addAll(_serializableHistory.restoreEntriesForPage(entry, this));
        }
    }

    //...
    void _handleHistoryChanged() {
        //...
            notification.dispatch(context); // NavigationNotificationをdispatch
    }
    //...
    Future<T?> push<T extends Object?>(Route<T> route) {
        _pushEntry(_RouteEntry(route, pageBased: false, initialState: _RouteLifecycle.push));
        return route.popped;
    }
    //...
    void _pushEntry(_RouteEntry entry) {
        //...
        _history.add(entry);
        _flushHistoryUpdates();
        //...
    }
    void _flushHistoryUpdates({bool rearrangeOverlay = true}) {
        //...
        int index = _history.length - 1;
        _RouteEntry? next;
        _RouteEntry? entry = _history[index];
        _RouteEntry? previous = index > 0 ? _history[index - 1] : null;
        while (index >= 0) {
            switch (entry!.currentState) {
                //... 
            }
        }
        // ...
        if (rearrangeOverlay) {
            // overlay は _overlayKey.currentStateでOverlayStateを取得。(これはbuildで生成しているOverlayオブジェクトのState)
            // _allRouteOverlayEntriesでは、entry.route.overlayEntriesは_historyから取得した
            // Iterable<OverlayEntry>であり、これがOverlay._entriesへ追加・setStateされている。
            overlay?.rearrange(_allRouteOverlayEntries);
        }
        // ...
    }
    Widget build(BuildContext context) {
         NotificationListener<NavigationNotification>(
            //...
            Overlay(
                key: _overlayKey,
                initialEntries: overlay == null ?  _allRouteOverlayEntries.toList(growable: false) : const <OverlayEntry>[],
            )
    }
}
```
```
enum _RouteLifecycle {
  staging,
  add,
  adding,
  push,
  pushReplace,
  pushing,
  replace,
  idle,
  pop,
  complete,
  remove,
  popping,
  removing,
  dispose,
  disposing,
  disposed,
}
```
```
class _History extends Iterable<_RouteEntry> with ChangeNotifier {
    //...
}
```
```
class _RouteEntry extends RouteTransitionRecord {
    // ...
    final Route<dynamic> route;
    // ...
    _RouteLifecycle currentState;
}
```
```
abstract class Route<T> extends _RoutePlaceholder {
    //...
    NavigatorState? get navigator => _navigator;
    NavigatorState? _navigator;
    //...
    List<OverlayEntry> get overlayEntries => const <OverlayEntry>[];
}
```
```
class OverlayEntry implements Listenable {
    //...
    final WidgetBuilder builder;
    //...
}
```
```
class OverlayState extends State<Overlay> with TickerProviderStateMixin {
  final List<OverlayEntry> _entries = <OverlayEntry>[];

  @override
  void initState() {
    //...
    insertAll(widget.initialEntries);
  }
  //...
  void rearrange(Iterable<OverlayEntry> newEntries, { OverlayEntry? below, OverlayEntry? above }) {
    // ...
    setState(() {
        //...
        _entries.addAll(newEntriesList);
    });
  }

  Widget build(BuildContext context) {
    //...
    // 最終的に子孫の _OverlayEntryWidgetState.build()内で　OverlayEntry.builder()が実行される。
  }
}
```


# (参考)Routerについて
* WidgetsApp.routerに渡すRouterConfigは４つの抽象で構成される。
    ```
    class RouterConfig<T> {
        const RouterConfig({
            this.routeInformationProvider,
            this.routeInformationParser,
            required this.routerDelegate,
            this.backButtonDispatcher,
        }) : assert((routeInformationProvider == null) == (routeInformationParser == null));

        /// The [RouteInformationProvider] that is used to configure the [Router].
        final RouteInformationProvider? routeInformationProvider;

        /// The [RouteInformationParser] that is used to configure the [Router].
        final RouteInformationParser<T>? routeInformationParser;

        /// The [RouterDelegate] that is used to configure the [Router].
        final RouterDelegate<T> routerDelegate;

        /// The [BackButtonDispatcher] that is used to configure the [Router].
        final BackButtonDispatcher? backButtonDispatcher;
    }
    ```
* RouterDelegate
    * 必須
    * Listenable派生クラス
    * https://api.flutter.dev/flutter/widgets/Router/routerDelegate.html
        > The router delegate for the router. This delegate consumes the configuration from routeInformationParser and builds a navigating widget for the Router. It is also the primary respondent for the backButtonDispatcher. The Router relies on RouterDelegate.popRoute to handle the back button.If the RouterDelegate.currentConfiguration returns a non-null object, this Router will opt for URL updates.
    ```
    abstract class RouterDelegate<T> extends Listenable {
        //...

        // OSによって新しいrouteがpushされたことをrouteInformationProviderからコンフィグ情報として受け取ってこのメソッドが呼ばれる。
        Future<void> setNewRoutePath(T configuration);

        // https://api.flutter.dev/flutter/widgets/RouterDelegate/build.html
        // 最終的にビルドされるウィジェット。
        // 通常はNavagatorオブジェクトを返す。
        // 基本的にはTのコンフィグ情報を元にして、Navagatorオブジェクトのpagesを構成する。
        Widget build(BuildContext context);
    }
    ```
* RouteInformationParser
    * https://api.flutter.dev/flutter/widgets/Router/routeInformationParser.html
        > The route information parser for the router.When the Router gets a new route information from the routeInformationProvider, the Router uses this delegate to parse the route information and produce a configuration. The configuration will be used by routerDelegate and eventually rebuilds the Router widget. Since this delegate is the primary consumer of the routeInformationProvider, it must not be null if routeInformationProvider is not null.
    ```
    abstract class RouteInformationParser<T> {
        //...

        // RouteInformation情報をRouterDelegateで処理できるコンフィグ情報（T）にパースする。
        // 例えば、RouteInformation.uriをパースして、その内容によってどのページへのルーティングかを判定する、といった処理が行われる。
        Future<T> parseRouteInformation(RouteInformation routeInformation) {
            //...
        }

        Future<T> parseRouteInformationWithDependencies(RouteInformation routeInformation, BuildContext context) {
            return parseRouteInformation(routeInformation);
        }

        // コンフィグ情報からRouteInformationを復元する。
        RouteInformation? restoreRouteInformation(T configuration) => null;
    }
    ```
    * routeInformationParserはListenableではないがaddCallbackというオブジェクトが変更された際にコールバックを呼ぶ役割の抽象メソッドが定義してある。
    
* RouteInformatioProvider
    * https://api.flutter.dev/flutter/widgets/Router/routeInformationProvider.html
        > The route information provider for the router. The value at the time of first build will be used as the initial route. The Router listens to this provider and rebuilds with new names when it notifies.This can be null if this router does not rely on the route information to build its content. In such case, the routeInformationParser must also be null.
    ```
    abstract class RouteInformationProvider extends ValueListenable<RouteInformation> {
        // 利用方法として、ValueListenable.valueにRouteInformationを持たせ、ChangeNotifierをwithしてnotifyListenerすることにより、リスナーであるRouterが
        // RouterInformationをパーサー、デリゲーターへハンドリングする、といった流れになると考えられる。

        // これはおそらく、他にInformation
        void routerReportsNewRouteInformation(RouteInformation routeInformation, {RouteInformationReportingType type = RouteInformationReportingType.none}) {}
    }
    ```
    * routeInformationProviderはValueListenable派生クラス
    * routeInformationProviderをセットした場合はrouteInformationParserも必須となる。
    * なお、PlatformRouteInformationProviderという具象クラスがあり、これは_WidgetsAppStateによってデフォルトとして利用されている。
        ```
        if (widget.routeInformationProvider == null && widget.routeInformationParser != null) {
            _defaultRouteInformationProvider ??= PlatformRouteInformationProvider(
            initialRouteInformation: RouteInformation(
                uri: Uri.parse(_initialRouteName),
            ),
            );
        } 
        ```
* backButtonDispatcherは戻るボタンのディスパッチを行う。
    * https://api.flutter.dev/flutter/widgets/Router/backButtonDispatcher.html

* (IMO) なぜこのような構成となっているのか?
    * 各プラットフォームのOSやFlutterエンジンを起点としたルーティング（ブラウザのロケーションバーからの入力やiOSアプリやAndroidのディープリンク）および通常のルーティング処理を、RouterInformation(メンバーとしてURIを保持)として統一的に扱うため?
        ```
        class RouteInformation {
            //...

            /// The uri location of the application.
            Uri get uri {
                if (_uri != null){
                    return _uri;
                }
                return Uri.parse(_location!);
            }
            final Uri? _uri;

            /// The state of the application in the [uri].
            final Object? state;
        }
        ```
    * 以下のように責務を分けるため。
        * Router/_RouterState
            * RouterInformationをOSやプロバイダーから受取り、パーサーへ渡して、結果をデリゲーターへ渡す等のオーケストレーション
        * RouteInformationProvider
            * 任意のタイミングでRouterInfomationを作成してRouterへハンドリングさせたいときに利用するプロバイダー
        * RouteInformationParser
            * RouterInfomationからルーティングのコンフィグへパースするパーサー
        * RouterDelegate
            * RouterInfomationを元にしてウィジェットをビルドするデリゲーター

## _RouterStateの実装
* _RouterStateはルーター情報のプロバイダーとパーサー、戻るボタンのディスパッチャとデリゲーターをオーケストレーションしている。

* routeInformationProviderのリスナーのコールバックとして routeInformationProviderから受け取ったRouteInformationをrouteInformationParserにパースさせ、戻ってきたコンフィグ情報でrouterDelegate.setNewRoutePath()によってルートパスとして設定して、setStateを行っている。
* backButtonDispatcherのコールバックとして、routerDelegate.popRoute()を実行してsetStateを行う。
* routerDelegateのリスナーのコールバックとして、setStateを行っている。
* buildメソッド内ではrouterDelegate.buildを呼び出している。

```
class _RouterState<T> extends State<Router<T>> with RestorationMixin {
    //...
    void initState() {
        super.initState();
        widget.routeInformationProvider?.addListener(_handleRouteInformationProviderNotification);
        widget.backButtonDispatcher?.addCallback(_handleBackButtonDispatcherNotification);
        widget.routerDelegate.addListener(_handleRouterDelegateNotification);
    }
    //...


    void _handleRouterDelegateNotification() {
        setState(() {/* routerDelegate wants to rebuild */});
        //...
    }

    //...
    void _handleRouteInformationProviderNotification() {
        // ...
        // 処理としてはwidget.routeInformationParser!.parseRouteInformationWithDependencies(widget.routeInformationProvider!.value, context) 
        //   ->  await widget.routerDelegate.setNewRoutePath(data); -> setStateされる。
        // dataはT型のルートのコンフィグ情報
    }
    //...

    Future<bool> _handleBackButtonDispatcherNotification() {
        //...
        return widget.routerDelegate
        .popRoute()
        .then<bool>(/* setStateしている */);
    }

    //...
    @override
    Widget build(BuildContext context) {
        return UnmanagedRestorationScope(
            bucket: bucket,
            child: _RouterScope(
                    //...
                    child: Builder(
                        builder: widget.routerDelegate.build,
                    ),
            ),
        );
    }
}
```


