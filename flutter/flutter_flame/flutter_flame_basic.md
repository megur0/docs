[TOP(About this memo))](../../README.md) >  [Flutter](../README.md) > [一覧(Flame)](./README.md) > GameWidget/Game/Component


# GameWidget
* https://docs.flame-engine.org/latest/flame/game_widget.html
* StatefulWidget派生クラス
* Game派生クラスを引数として持つ。
* GameWidgetはウィジェットツリー上のどの階層にも配置ができる。
* GameWidgetは可能な限り大きくなろうとする。
* GameWidgetはCanvas内のコンテンツをクリップしないため、範囲外にも描画を行う。
    * これを許容しない場合はFlutter標準のClipRectウィジェットで囲めば良い。
* ウィジェット内のbuildで、GameWidgetを生成するとそのウィジェットがリビルドされる度にGameWidgetもリビルドされる。
    * これを回避するためにはGameWidget.controlledを利用するか、ウィジェットの外で生成したGameWidgetを利用する
    * https://docs.flame-engine.org/latest/flame/game.html#flamegame
* (IME)GameWidgetの先祖のGestureDetectorは反応しない。(なぜそのようになるのか実装を少し見たが分からなかった)
  ```dart
      GestureDetector(
        onTap: () {
          print("GestureDetector called");//こちらの処理は反応しない
        },
        child: GameWidget<_MyGame>(
          //...
        ),
      );
  ```

# Game
* https://docs.flame-engine.org/latest/flame/game.html
* Game
    * abstract mixin class
    * Gameのサブクラスを作成する事も可能だが、通常はFlameGameを利用する。
        * 多くのAPIがFlameGameを前提としている
    * backgroundColor
        * デフォルトは黒
        * オーバーライドして背景色を変更できる
    * size, canvasSize
        * viewportのサイズを返す
* FlameGame
    * Gameでもあり、コンポーネントでもあるクラスとなる。
    * Gameの派生クラス(with)
    * ComponentTreeRootの派生クラス(extentds)
        * ComponentTreeRootはComponentのサブクラス
    * Worldと、CameraComponentを引数として持つ(指定がなければ生成される)。
        * この２つがデフォルトのWorld, CameraComponentとなる。
        * コンストラクタ内でこれらをaddしている。
        
# Component
* https://docs.flame-engine.org/latest/flame/components.html
    > コンポーネントは、Flutter のウィジェットや Unity の GameObject に非常に似ています。  
    > ゲーム内のあらゆるエンティティは、特にそのエンティティに何らかの視覚的外観がある場合や、時間の経過とともに変化する場合は、コンポーネントとして表すことができます。  
    > たとえば、プレイヤー、敵、飛んでくる弾丸、空の雲、建物、岩など、すべてをコンポーネントとして表すことができます。  
    > 一部のコンポーネントは、エフェクト、動作、データ ストア、グループなど、より抽象的なエンティティを表すこともできます。  
* コンポーネントはツリーを形成する
* FlameGame はコンポーネントツリーのルートとなる。
* ロードとマウント
    * https://pub.dev/documentation/flame/latest/components/Component/add.html
    * ロードはコンポーネントに必要な非同期処理を含む初期化処理を指す。(非同期処理が必要なければ含まなくても良い。)
    * ロードは任意のタイミング(addを呼び出した際)に開始される。
    * マウントはツリーへの追加のことを指す。(親のchildrenへ追加されること。)
    * マウントはTickのタイミングでのみ行われる
        * ロードが完了、および親のマウントが完了している時点で、次回のTickの際にマウントされる。
    * 親がマウントされていない場合は、ロードはすぐにはじまるが、マウントは親がマウントされるまでは遅延される。
    * 上記の仕組みによって、以下が実現される
        * コーディング: 親のロード内(onLoad)で子をaddをする構成
        * 実行時: ロードが完了した要素から成るツリー構造が形成される
* 引数にchildren, priority
* parent
    * 親コンポーネント
* children
    * 子コンポーネントのセット(ComponentSet)
* priority
* FutureOr<void> add
    * https://pub.dev/documentation/flame/latest/components/Component/add.html
    * addやremoveによる追加や削除は、即時行われるのではなくスケジューリングされる(キューに追加される)
        * キューはComponentTreeRoot._queue が実体となる。FlameGameがComponentTreeRootのサブクラスとなる。
        * Component._addChildから、FlameGame.enqueueAddによって追加される。
        * FlameGame.updateTree の中でキューが処理される。
        * (IMO)これはFlutterフレームワークのState.setStateのように、APIの実行タイミングと、コンポーネントツリーの構築およびレンダリング、を疎結合にするため設計と考えられる。
    * **同じ親に対する子は、それぞれの読み込みの早さに関係なく、スケジューリングした順番と全く同じ順番で親に追加されることが保証される**
    * ロードと、親のマウントが完了していたときに完了となるFutureを返す。このFutureは自身がマウントされていることを保証するものではない。
    * 詳細はComponent._addChildの実装を参照
* Future<void> addAll
* void remove
    * 子から削除する（即時ではなくキューに入れる。）
* removeFromParent
    * 自身を親から削除する
    * _parent?.remove(this)
* removeは子孫もremoveする
    * 下記は検証コード。親をremoveすると子孫のonRemoveが呼び出しされることが確認できる。
    ```dart
    import 'package:flame/components.dart';
    import 'package:flame/events.dart';
    import 'package:flame/game.dart';
    import 'package:flutter/material.dart';

    void main() {
      runApp(
        SafeArea(child: GameWidget(game: Example())),
      );
    }

    class Example extends FlameGame with TapCallbacks {
      Example() {
        debugMode = true;
      }

      late Parent c;

      @override
      Future<void> onLoad() async {
        add(c = Parent(size: size));
      }

      @override
      void onTapUp(TapUpEvent event) {
        remove(c);
      }
    }

    class Parent extends PositionComponent {
      Parent({super.size});
      late Child ember;

      @override
      Future<void> onLoad() async {
        add(ember = Child(position: Vector2(size.x / 2, size.y / 2)));
      }

      @override
      void onRemove() {
        print("onRemove Parent");
      }
    }

    class Child extends TextComponent {
      Child({super.position, super.key})
          : super(
              size: Vector2.all(100),
              anchor: Anchor.center,
            );

      @override
      Future<void> onLoad() async {
        text = "test";
        add(GrandChild());
      }

      @override
      void onRemove() {
        print("onRemove Child");
      }
    }

    class GrandChild extends PositionComponent {
      @override
      void onRemove() {
        print("onRemove GrandChild");
      }
    }    
    ```
* key
    * https://docs.flame-engine.org/latest/flame/components.html#component-keys
    * ComponentKey型
    * ComponentTreeRoot.findByKeyによって検索可能となる(FlameGameがComponentTreeRootをextendsしている)
* findGame
    * 最も近い先祖のFlameGameを取得する
* ライフサイクルメソッド(onLoad, onGameResize, onMount, onRemove)
    * FutureOr<void> onLoad
        * ロードに使う
            * コンポーネントを初期化する主な場所　
            * 「非同期コンストラクタ」の役割を持つ
            * マウントと描画の前に完了させておきたい非同期処理は、awaitしておく。
                * たとえばスプライト(Spriteクラス)の読み込みや、バックエンドAPIでデータ取得 等
        * このメソッドは1回のみ呼ばれる。
        * この時点で、ゲーム キャンバスのサイズが既に決定している
        ```dart
        // ~/.pub-cache/hosted/pub.dev/flame-1.18.0/lib/src/components/core/component.dart
        FutureOr<void> _addChild(Component child) {
            //...
            if (!child.isLoaded && !child.isLoading && (game?.hasLayout ?? false)) {// 1回のみ呼ばれる
                return child._startLoading();
            }
        }

        FutureOr<void> _startLoading() {
            assert(_state == _initial);
            assert(_parent != null);
            assert(_parent!.findGame() != null);
            assert(_parent!.findGame()!.hasLayout);
            _setLoadingBit();
            final onLoadFuture = onLoad();//ここで呼ばれている
            if (onLoadFuture is Future) {
                return onLoadFuture.then((dynamic _) => _finishLoading());
            } else {
                _finishLoading();
            }
        }
        ```
        * コンストラクタとonLoadのどちらに処理を書くか？
            * 非同期処理をawaitをしない場合や、サイズを必要としない場合はコンストラクタ上でadd処理をしても動作する
            * したがってそういった場合は分かりやすい方に書けばよいだろう
            ```dart
            class Example extends FlameGame with DragCallbacks, ScaleDetector {
              Example() {
                // 動作する
                add(TextComponent(text: "test", position: Vector2(30, 30)));
                
                // sizeが決定していないため実行時エラーとなる
                //add(TextComponent(position: Vector2(size.x/2, size.y/2)));
              }
            }
            ```
            

    * void onGameResize
        * 以下のタイミングで呼ばれる
            1. 最上位の Canvas のサイズが変更されるたびに呼び出される
            2. マウントの際に、onMountの前に呼ばれる
        ```dart
        // ~/.pub-cache/hosted/pub.dev/flame-1.18.0/lib/src/components/core/component.dart
        @mustCallSuper
        void onGameResize(Vector2 size) => handleResize(size);// onGameResizeでは、子のonGameResizeも呼び出す
        //...
        @mustCallSuper
        @internal
        void handleResize(Vector2 size) {
            _children?.forEach((child) {
                if (child.isLoading || child.isLoaded) {
                    child.onGameResize(size);// ここで呼ばれる
                }
            });
        }
        //...
        void _mount() {
            assert(_parent != null && _parent!.isMounted);
            assert(isLoaded && !isLoading);
            _setMountingBit();
            onGameResize(_parent!.findGame()!.canvasSize);//ここで呼ばれる
            //...
            onMount();
            //...
             _parent!.children.add(this);
            //...
            _clearMountingBit();
            //...
        }
        ```
    * void onMount
        * マウント中に呼ばれる
    * void onRemove
        * ツリー上にから削除（アンマウント）されたときに呼ばれる
* ライフサイクルメソッド(update, updateTree, render, renderTree)
    * void updateTree(double dt)
        * ゲームのTickごとにFlameGame.updateTreeを開始地点として呼び出される
        * updateを呼び、childrenのupdateTreeを呼ぶ
    * void update(double dt)
        * 前回のupdateからの経過時間(microseconds)を受け取る
    * void renderTree
        * ゲームのTickごとに、RenderGameWidget.markNeedsPaint()が呼ばれることでRenderGameWidget.paintが実行されて、そこからFlameGame.renderTreeを開始地点として呼び出される
        * renderを呼び、childrenのrenderTreeを呼ぶ
    * void render(Canvas canvas)
        * すべてのコンポーネントのアップデートが完了後に呼ばれる
        * FlutterのCustomPainter.paintのようにCanvasを受取り、Canvasに対して描画処理を行う
* デバッグ
    * debugModeをtrueとすることでコンポーネントの座標やHitBoxの境界を表示することができる
    * https://docs.flame-engine.org/latest/flame/other/debug.html
* クエリ
    * https://docs.flame-engine.org/latest/flame/components.html#querying-child-components
    * Component.childrenはQueryableOrderedSetのサブクラスとなる。(QueryableOrderedSet<Component>)
        * QueryableOrderedSetは、bluefireteam/ordered_setという外部パッケージのクラスとなる。
        * bluefireteamはFlutterやFlame関連に貢献しているコミュニティのようだ
    * QueryableOrderedSetはTypeをキーとしたMapを保持していて、registerメソッドによってComponentのリストを登録できる。
    * queryによってこれらを取得できる。
        * queryは結果が取得できなければ、regsiterで登録をする。
        * strictをtrue(デフォルトはfalse)にすると結果が取得できない時点で例外となる
* _state
    * コンポーネントの状態を表現する。
    * 状態遷移ごとにステートを順番に遷移させるようなものではなく、ビットとして各桁の１つ１つをフラグとしている。
        * フラグが有効かどうかのチェック(各isXXX)は、&演算でマスクすることによって行っている。
    * _initial
        * 初期の値
    * _loading
        * onLoadの処理中。完了するとクリアされる
    * _loaded
        * onLoadの完了後
    * _mounted
        * コンポーネントのツリーへマウントされたとき。
        * ツリーからremoveされるとクリアされる
    * _removing
        * コンポーネントが次回の最も早いタイミングでremoveされる予定であることを表す
        * removeされるとクリアされる


# (参考)GameWidgetState内部の実装
* 下記のようにFutureBuilderでgame.load()をawaitしている。
    ```dart
    @override
    Widget build(BuildContext context) {
        Future<void> get loaderFuture => _loaderFuture ??= (() async {
            //...
            await game.load();
            game.mount();
        //...
        })();
        // ...
        retrun FutureBuilder(
                        future: loaderFuture,
                        //...
        );
        //...
    }
    ```
* RenderGameWidget
    * 実際にゲームのレンダリングを担うウィジェット。
    * RenderObjectWidgetの派生クラスでレンダリング（RenderObjectの生成）を担う
    ```dart
    void gameLoopCallback(double dt) {// これはattach時のGameLoop生成時にcallbackとして渡される
        assert(attached);
        if (!attached) {
            return;
        }
        game.update(dt);//FlameGame.updateはchildrenのupdateTreeを呼び出す
        markNeedsPaint();
    }

    @override
    void paint(PaintingContext context, Offset offset) {
        context.canvas.save();
        context.canvas.translate(offset.dx, offset.dy);
        game.render(context.canvas);//FlameGame.renderはchildrenのrenderTreeを呼び出す
        context.canvas.restore();
    }
    ```
* GameLoop
    * 内部でTickerを保持し、コールバックを実行する
    ```dart
    class GameLoop {
        GameLoop(this.callback) {
            _ticker = Ticker(_tick);
        }
        void Function(double dt) callback;
        //...
        void _tick(Duration timestamp) {
            final durationDelta = timestamp - _previous;
            final dt = durationDelta.inMicroseconds / Duration.microsecondsPerSecond;
            _previous = timestamp;
            callback(dt);// RenderGameWidget.gameLoopCallbackが実行される
        }
        //...
        void start() {
            if (!_ticker.isActive) {
                _ticker.start();
            }
        }
        //...
    }
    ```
* FlameGame
```dart
  @override
  @mustCallSuper
  void render(Canvas canvas) {
    if (parent == null) {
      renderTree(canvas);
    }
  }

  @override
  void renderTree(Canvas canvas) {
    if (parent != null) {
      render(canvas);
    }
    for (final component in children) {
      component.renderTree(canvas);// Component.renderTreeはrenderを呼び、子のrenderTreeを呼ぶ
    }
  }

  @override
  @mustCallSuper
  void update(double dt) {
    if (parent == null) {
      updateTree(dt);
    }
  }

  @override
  void updateTree(double dt) {
    processLifecycleEvents();
    if (parent != null) {
      update(dt);
    }
    for (final component in children) {
      component.updateTree(dt);// Component.updateTreeはupdateを呼び、子のupdateTreeを呼ぶ
    }
    processRebalanceEvents();
  }
```
* ComponentTreeRoot(※ FlameGameが派生クラスとなっている)
```dart
  void processLifecycleEvents() {
    //...
      for (final event in _queue) {
        //...
        final status = switch (event.kind) {
          final child = event.child!;
          final parent = event.parent!;
          _LifecycleEventKind.add => child.handleLifecycleEventAdd(parent),
          _LifecycleEventKind.remove =>
            child.handleLifecycleEventRemove(parent),
          //...
        };
        //...
      }
    //...
  }
```
* Component
```dart
    // 親がマウントされていて、かつロードが完了している場合はマウントする。
    // 親がマウントされていて、かつロードを開始していない場合はロードを開始する。
    // --> ただし、_addChildの最後に_startLoading()を実行しているため、addを実行したタイミングで基本的にはローディングは即時開始されていると考えられる。
    //     そのため、この分岐がどういったケースで通るのかは理解できていない。
    // 親がマウントされていない場合 または ローディングが完了していない場合は、マウントを見送る
    //   ※ この場合はキューから削除されないため、次回のゲームループの際に再度処理される。（ロードが終わるまではキューに残り続ける）
  @internal
  LifecycleEventStatus handleLifecycleEventAdd(Component parent) {
    assert(!isMounted);
    if (parent.isMounted && isLoaded) {
      _parent ??= parent;
      _mount();
      return LifecycleEventStatus.done;
    } else {
      if (parent.isMounted && !isLoading) {
        _startLoading();
      }
      return LifecycleEventStatus.block;
    }
  }

  @internal
  LifecycleEventStatus handleLifecycleEventRemove(Component parent) {
    if (_parent == null) {
      parent._children?.remove(this);
    } else {
      _remove();
      assert(_parent == null);
    }
    return LifecycleEventStatus.done;
  }

  // 親のchildrenから自身を削除して、
  // 自身と子孫についてすべて親をnull、onRemoveを呼ぶ、removedフラグを立てる、等の処理をしている。(再帰的に_removedを実行するわけではない。) 
  void _remove() {
    //...
    _parent!.children.remove(this);
    propagateToChildren(
      (Component component) {
        component
          ..onRemove()
          .._unregisterKey()
          .._clearMountedBit()
          .._clearRemovingBit()
          .._setRemovedBit()
          .._removeCompleter?.complete()
          .._removeCompleter = null
          .._parent!.onChildrenChanged(component, ChildrenChangeType.removed)
          .._parent = null;
        return true;
      },
      includeSelf: true,
    );
  }
```


# 参考: FlameGameのonRemove処理
* FlameGameのonRemoveはComponent.onRemoveとは呼び出し元が異なる点について検証したコード
```dart
import 'package:flame/components.dart';
import 'package:flame/events.dart';
import 'package:flame/flame.dart';
import 'package:flame/game.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    const MaterialApp(
      home: MyWidget(),
    ),
  );
}

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  bool useGame = true;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: useGame ? GameWidget(game: MyGame()) : null,
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          useGame = !useGame;
          setState(
            () {},
          );
        },
        child: Text(useGame ? "detach game" : "attach game"),
      ),
    );
  }
}

class MyGame extends FlameGame with TapCallbacks {
  MyGame() {
    debugMode = true;
  }

  late Parent p;

  @override
  Future<void> onLoad() async {
    add(p = Parent(size: size));
  }

  @override
  void onTapUp(TapUpEvent event) {
    // remove処理はゲームループにおいて、FlameGame.updateTreeによって
    // キューによって処理される。
    // この処理の際は全ての子孫のonRemoveも実行される。
    remove(p);
  }

  @override
  void onAttach() {
    // attach(GameWidgetがFlutterのツリーにアタッチされた時)した際に呼ばれる
    // ※ GameRenderBox.attachから呼ばれる
    print("onAttach MyGame");
  }

  @override
  void onDetach() {
    // detach(GameWidgetがFlutterのツリーからデタッチされる時)した際に呼ばれる
    // ※ GameRenderBox.detachから呼ばれる
    // onDetachは(なぜか)アタッチの前にも呼ばれる
    print("onDetach MyGame");
  }

  @override
  void onDispose() {
    // GameWidget.disposeの際に呼ばれる。
    print("onDispose MyGame");
  }

  @override
  void onRemove() {
    // 通常のコンポーネントのonRemoveとは異なり、この関数はゲームループ内の
    // キュー処理で呼ばれるものではない点に注意。
    // Game.onRemoveはGameWidget.disposeやGameWidget.didUpdateWidgetから呼ばれる。
    // 　※ didUpdateWidgetの場合は新旧のオブジェクトが異なる時のみ。
    print("onRemove MyGame");

    // 上記の事由により、onRemove内でremoveを子に対して呼ぶのみでは、
    // 実際の削除・クリーンアップ処理は(キュー処理によって)実行されないため注意。
    //  ※ 下記のprocessLifecycleEvents以降をコメントアウトして実行すると
    //     子孫のonRemove内のprintが出力されないことからも分かる。
    // これは、GameWidgetがdisposeされると、ゲームループ自体が終了するため。
    // 配下のchildrenのクリーンアップも全て実行したいのであれば
    // 下記のように実行する。
    // https://docs.flame-engine.org/latest/flame/game.html#lifecycle
    removeAll(children); //キューにいれる
    processLifecycleEvents();//キューの処理関数を直接実行する。
    Flame.images.clearCache();
    Flame.assets.clearCache();
  }
}

class Parent extends PositionComponent {
  Parent({super.size});
  late Child ember;

  @override
  Future<void> onLoad() async {
    add(ember = Child(position: Vector2(size.x / 2, size.y / 2)));
  }

  @override
  void onRemove() {
    print("onRemove Parent");
  }
}

class Child extends TextComponent {
  Child({super.position, super.key})
      : super(
          size: Vector2.all(100),
          anchor: Anchor.center,
        );

  @override
  Future<void> onLoad() async {
    text = "child";
    add(GrandChild());
  }

  @override
  void onRemove() {
    print("onRemove Child");
  }
}

class GrandChild extends PositionComponent {
  @override
  void onRemove() {
    print("onRemove GrandChild");
  }
}
```


# 非同期処理をまたがるgameの参照の注意
* FlutterのBuildContextの似ているが、非同期処理の完了後にgameを参照できるとは限らない。
* これは、(Flutterのdisposeのように)コンポーネントがremoveされることでツリー上に存在しなくなり、gameを参照できなくなるためである。
```dart
void main() {
  runApp(
    SafeArea(child: GameWidget(game: Example())),
  );
}

class Example extends FlameGame {
  @override
  Future<void> onLoad() async {
    late Component component;
    add(
      component = MyComponent(),
    );
    remove(component);
  }

  // 上記の例では、一つの関数内で同じコンポーネントをadd/removeしているが、
  // 例えば下記のように別のコンポーネントのremoveとaddを行っているケースも注意が必要である。
  // この関数が、次回のゲームループまでに2回以上呼ばれると、同一の処理内(イベントループ内)で同一コンポーネントが
  // add/removeされるため、同じ問題が発生する。
  // void resetChildren() {
  //   removeAll(children);
  //   add(
  //     MyComponent(),
  //   );
  // }
}

class MyComponent extends Component with HasGameRef {
  MyComponent();

  @override
  Future<void> onLoad() async {
    await super.onLoad();// awaitが完了する頃には、removeの処理まで完了している。
    print(game);// assert error: Could not find Game instance ...
  }
}
```
* 非同期処理をまたがってgameを参照する必要がある場合
  * 例えば、下記のようにisMountedをチェックするとよいだろう。
  ```dart
  await /* ... */
  if (!isMounted) return;
  print(game);
  ```
## flame_forge2dのBodyComponent
* flame_forge2dのBodyComponentの実装では下記のように、awaitの後にgameを参照している
  * したがって１回のゲームループ内でBodyComponentをadd/removeしないように注意する必要がある。
  ```dart
  // flame_forge2d-0.18.2/lib/body_component.dart
  Body createBody() {
    assert(
      bodyDef != null,
      'Ensure this.bodyDef is not null or override createBody',
    );
    final body = world.createBody(bodyDef!);
    fixtureDefs?.forEach(body.createFixture);
    return body;
  }

  @mustCallSuper
  @override
  Future<void> onLoad() async {
    await super.onLoad();
    body = createBody();
  }

  Forge2DWorld get world => game.world;
  ```
