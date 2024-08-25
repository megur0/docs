[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# 注意
* 以下は筆者がFlutterのコードリーディングを2023年の7月頃に行って書いたメモを基に再構成したメモである。
* あくまでも筆者の理解であり内容に誤りが含まれている可能性もある点に注意。
* 更新
    * (2024/5) ルートのウィジェットがRenderObjectToWidgetAdapterからRootWidgetに変わっていたためアップデート

# 表記
* クラス図の表記について
    * [クラス図の表記方法](../common/about_class_diagram.md)
* クラスの色付け
    * <span style="background-color: #E1D5E7; ">Widget</span>
        * RenderObjectのWidgetは<span style="background-color: #76608A; color:white">紫色</span>としている
    * <span style="background-color: #D5E8D4; ">Element</span>
    * <span style="background-color: #F8CECC; ">RenderObject</span>

# 全体像
* 以下はフレーム処理とレンダリングパイプラインの概要を表した図となる。
    * 本コードリーディングでは、Build phaseとLayout phaseを中心に扱っている。

<img src="./svg/arch_code_reading/flutter_pipeline_abst.drawio.svg" width="70%"><br/><br/>  

* WidgetsBinding.drawFrame
    * https://api.flutter.dev/flutter/widgets/WidgetsBinding/drawFrame.html
        > Pump the build and rendering pipeline to generate a frame.
    * パイプライン処理の構成
        * Build phase
            * ウィジェット ツリー内のすべてのダーティなElementが再構築される。
        * Layout phase
            * システム内のすべてのダーティなRenderObjectがレイアウトされる。
        * Compositing bits phase
            * ダーティな RenderObjectオブジェクトの合成ビットが更新される
        * Paint phase
            * システム内のすべてのダーティなRenderObjectが再ペイントされる。
            * これにより、レイヤーツリーが生成される。
            * https://api.flutter.dev/flutter/rendering/RenderObject/paint.html
        * Compositing phase
            * レイヤー ツリーがSceneに変換され、GPUに送信される。
        * Semantics phase
            * システム内のすべてのダーティRenderObjectのセマンティクスが更新される

# runApp実行時の流れ
* 以降の図では、下記のようにプロパティやメソッドの文字色をつけている。(何も色をつけていない図もある。)
    * <span style="background-color: #FFF2CC; ">背景色が薄黄色</span>
        * runAppによる同期的に行われる初期化処理
    * <span style="color: blue; ">青文字</span>
        * スケジューリングされたフレーム毎に実行される処理
        * ペイント(paint)系は<span style="background-color: #CCFFFF; ">背景色を水色</span>としている
    * <span style="color: green; ">緑文字</span>
        * 状態をdirtyにする、およびそれによって次回のフレーム処理をスケジューリングするメソッド

![](./svg/arch_code_reading/flutter_flow_run_app.drawio.svg)

* dart:ui/PlatformDispatcher
    * https://api.flutter.dev/flutter/dart-ui/PlatformDispatcher-class.html
        > The most basic interface to the host operating system's interface.This is the central entry point for platform messages and configuration events from the platform.It exposes the core scheduler API, the input event callback, the graphics drawing API, and other such core services.
    * onBeginFrame()
        * https://api.flutter.dev/flutter/dart-ui/PlatformDispatcher/onBeginFrame.html
        * フレームを開始するときに呼び出されるコールバック
        * このコールバックが最後に呼び出されてからPlatformDispatcher.scheduleFrameが呼び出された場合にのみ呼び出される。
        * https://api.flutter.dev/flutter/widgets/WidgetsBinding/drawFrame.html
            * onBeginFrameにて行われる具体的な処理として、ここですべてのアクティブなアニメーション オブジェクトがこの時点でTickされる。
            * このTickという処理は、アニメーションに関するvalueが更新されて再レイアウト・再ペイント対象となるということを指していると筆者は理解している。
    * onDrawFrame()
        * https://api.flutter.dev/flutter/dart-ui/PlatformDispatcher/onDrawFrame.html
        * フレームごとに呼び出される。onBeginFrameが完了しマイクロタスクキューが空になった後に呼ばれるコールバック
        * 最終的にこの中でFlutterView.renderが呼ばれる。
            * Flutter.renderはSceneオブジェクトをGPUへ送信し、ハードウェア上に描画される。
* SchedulerBinding.scheduleFrame()
    * https://api.flutter.dev/flutter/scheduler/SchedulerBinding/scheduleFrame.html
    * 必要に応じて、dart:ui.PlatformDispatcher.scheduleFrameを呼び出して新しいフレームをスケジュールする。
    * これが呼び出された後、エンジンは（最終的に）handleBeginFrameを呼び出す。
    * フレーム中にこれを呼び出すと、現在のフレームが未完了でも、別のフレームが強制的にスケジュールされる。
    * スケジューリングされたフレームは、オペレーティング・システムが提供する "Vsync "シグナルによってトリガーされたときに処理される
    * Vsync
        >  The "Vsync" signal, or vertical synchronization signal, was historically related to the display refresh, at a time when hardware physically moved a beam of electrons vertically between updates of the display. The operation of contemporary hardware is somewhat more subtle and complicated, but the conceptual "Vsync" refresh signal continue to be used to indicate when applications should update their rendering.
        * ディスプレイのリフレッシュの度に「Vsync」と呼ばれるシグナルが送られてきて、その際にエンジンがFlutter frameworkへフレーム生成を要求する、といった流れとして理解している。
    * debugPrintScheduleFrameStacks を trueとすると、scheduleFrameが呼ばれる度にスタックトレースを出力してデバッグが出来る。
    * (参考)PlatformConfigurationNativeApi::ScheduleFrame
        * SchedulerBinding.scheduleFrame()は、PlatformDispatcher.ScheduleFrame()を呼び、PlatformDispatcher.ScheduleFrame()は下記のようにPlatformConfigurationNativeApi::ScheduleFrame()を呼ぶ。
            ```
            void scheduleFrame() => _scheduleFrame();
            @Native<Void Function()>(symbol: 'PlatformConfigurationNativeApi::ScheduleFrame')
            external static void _scheduleFrame();
            ```
        * PlatformConfigurationNativeApi::ScheduleFrame()はエンジン側(C++)で下記のようにインターフェースが定義されている。
            * https://github.com/flutter/engine/blob/1feb9302050c5a6f7e822433915d7159365ad65c/lib/ui/window/platform_configuration.h#L558
        * おそらく内部では以下の関数あたりが実行されていると考えられる。
            * https://github.com/flutter/engine/blob/main/shell/common/animator.cc#L239
* (参考)engineのコード
    * https://github.com/flutter/engine/
    * プラットフォーム依存コード
        * コード
            * https://github.com/flutter/engine/tree/main/shell/platform
        * 共通embedder？
            * https://github.com/flutter/engine/tree/main/shell/platform/embedder
        * iOS
            * https://github.com/flutter/engine/tree/main/shell/platform/darwin/ios
        * fuchsia
            * https://github.com/flutter/engine/tree/main/shell/platform/fuchsia
* runAppによって以下が順に実行される。（これらの処理は同期的に行われる）
    * 1.ensureInitialized()
        * 各***BindingのinitInstances()が実行される。
    * 2.scheduleAttachRootWidget()
        * Timer.run(() {attachRootWidget()})
            * attachRootWidgetによって初回のツリー構築が行われる。
        * attachRootWidget()にはrunApp()で渡されたウィジェットをViewウィジェットでラップしたものを渡す
    * 3.scheduleWarmUpFrame()
        * https://api.flutter.dev/flutter/scheduler/SchedulerBinding/scheduleWarmUpFrame.html
            * Flutter エンジンは、OSからリクエストを受信すると、Flutterframeworkにフレームを生成するよう促す。
            * ただし、アプリの起動後 (またはホットリロード後) 数ミリ秒はこれが発生しない場合がある。
            * ツリーが最初に構成されてからエンジンが更新を要求するまでの時間を利用するため、フレームワークはウォームアップ フレームをスケジュールする。
            * これによって実際にエンジンがフレームを要求してきた際に最小限の追加作業でフレームの生成が可能。
            * ウォームアップフレームでは、ビルド、レイアウト、ペイントの各ステップを実行するがエンジン側のリクエストが未達の際は、画面へのレンダリングは行われない。
                * これは、コンテキスト(PlatformDispatcher.implicitViewの事を指している?)が未だ有効ではないため。
        * 以下のようにTimer.run()によって次のイベントにて実行
            * Timer.run((){handleBeginFrame()})
            * Timer.run((){handleDrawFrame()})
            * フェーズがSchedulerPhase.idle以外の場合は何もしない
                * これは、handleBeginFrame()やhandleDrawFrame()の処理が既に開始している場合。
            * フレームが既にスケジューリングされている(かつ未だ開始していない)場合は、handleDrawFrameを実行後にscheduleFrame()も実行する。
        * (IMO)
            * 2.のscheduleAttachRootWidget()にてTimer.run()で実行された attachRootWidget()ではensureVisualUpdate()によってscheduleFrame()が実行されるため、scheduleWarmUpFrame()の実行時点で少なくとも「フレームがスケジューリング済」の状態に必ずなっていると推測される。
            * なお、scheduleWarmUpFrame()の実行時点で、スケジュールされたフレームが既に実行開始されているかどうかは、エンジンからの要求タイミングに依存するため非決定的と考えられる。
* 初回のツリー構築(attachRootWidget())
    * 上記の同期的な処理が完了後、次のイベントにて初回のツリー構築が行われる。
    * 詳細は次節で掘り下げる。
    * ツリーの構築後にSchedulerBinding.instance.ensureVisualUpdate() によって scheduleFrame()を行う。
        * おそらくこのタイミング(初回のツリー構築が完了後、scheduleFrame()が完了した後)で初めて、エンジンの要求が来た時のdrawFrame()が行われるようになる。
        * もしこの前にエンジンからの要求が来た場合は特に何も実行されない。ウォームアップフレームにてレイアウトとペイント、レンダリングまで行われる(と筆者は理解している。)
* 初回のdrawFrameの処理
    * initInstances()の処理にてaddPersistentFrameCallback()によって、_persistentCallbacksへdrawFrame()を実行する関数が登録される。
        * _persistentCallbacksは名前の通り永続的なコールバックであり、（_transientCallbacksや_postFrameCallbacksとは違って）_handleDrawFrameの実行時には毎回実行される。
    * _handleDrawFrameを呼ぶhandleDrawFrameは、上記のscheduleWarmUpFrameによってセットされている通り、 attachRootWidget()の実行後に実行される。
    * 初回のdrawFrame()が実行開始される流れは以下のどちらかとなる。（どちらになるかは非決定的）
        * エンジン側からの要求によってonBeginFrame()およびonDrawFrame()が実行される。この場合はレンダリングまで行われる。
        * エンジン側からの要求より先にscheduleWarmUpFrame()によるonBeginFrame()およびonDrawFrame()が実行される。
            * この場合は完了後にscheduleFrame()を実行し、エンジン側からの要求が来てから(再度onBeginFrame()とonDrawFrame()の実行を経て)レンダリングが行われる。
    * いずれの場合にせよ、エンジン側の準備ができていない限り、レンダリングはされない。
    * (参考)(24/4/30時点)
        * β版(3.22.0-17.0.pre)時点で、ウォームアップ処理の方針が変わっている。
        * ウォームアップにて生成されるツリーは、エンジンからのリクエストが未達の場合は現行の仕様ではレンダリングはできないが、β版の方でエンジンからのリクエストを待たずにレンダリングまで行うように方針を変えている。
        * https://github.com/flutter/flutter/commit/be2544ab598bffcf8e0fd33f1e6668920f53c1d5
        * https://github.com/flutter/flutter/issues/142851
* 以降のdrawFrameの処理
    * Element.markNeedsBuild()が呼ばれることで、ensureVisualUpdate()が呼ばれて_handleDrawFrameがエンジンのコールバックへスケジューリングされることで実行される。
        * markNeedsBuildはStateのsetState()関数等から呼ばれる。<span style="color: green; ">緑文字</span>にしている。

* 以下はrunAppの実行順番を確認するサンプルコードである。
```dart
import "package:flutter/widgets.dart";

main() {
  debugPrintScheduleFrameStacks = true;
  debugPrintBuildScope = true;
  debugPrint("before runApp");
  Future(() => debugPrint("future before runApp"));
  runApp(const MyWidget());
  debugPrint("after runApp");// 初回のツリーの構築はTimer.runにて実行されるため、initStateやbuildよりもこちらのprintが先に実行される。
  Future(() => debugPrint("future after runApp")); 
  Future.microtask(() => debugPrint("microtask after runtApp"));// Timer.runより早く実行される。
}

class MyWidget extends StatefulWidget {
  const MyWidget({super.key});

  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  @override
  void initState() {
    debugPrint("initState");
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    debugPrint("build");
    return const Placeholder();
  }
}

/*
flutter: before runApp
flutter: after runApp
flutter: microtask after runtApp
flutter: future before runApp
flutter: buildScope called with context [root](dirty); dirty list is: []
flutter: scheduleFrame() called. Current phase is SchedulerPhase.idle.
flutter: #0      debugPrintStack (package:flutter/src/foundation/assertions.dart:1201:29)
flutter: #1      SchedulerBinding.scheduleFrame.<anonymous closure> (package:flutter/src/scheduler/binding.dart:875:9)
...
flutter: #25     BuildOwner.buildScope (package:flutter/src/widgets/framework.dart:2835:19)
flutter: #26     RootWidget.attach (package:flutter/src/widgets/binding.dart:1255:13)
flutter: #27     WidgetsBinding.attachToBuildOwner (package:flutter/src/widgets/binding.dart:1083:27)
flutter: #28     WidgetsBinding.attachRootWidget (package:flutter/src/widgets/binding.dart:1065:5)
flutter: #29     WidgetsBinding.scheduleAttachRootWidget.<anonymous closure> (package:flutter/src/widgets/binding.dart:1051:7)
flutter: #33     _RawReceivePort._handleMessage (dart:isolate-patch/isolate_patch.dart:184:12)
flutter: (elided 3 frames from class _Timer and dart:async-patch)
flutter: initState
flutter: build
flutter: buildScope finished
flutter: future after runApp
*/
```
## (IMO)フレームについて
* イベントキューと、フレームについて（筆者の理解）
    * フレームはエンジンにおいてレンダリングを行う間隔の単位。この間隔はハードウェアのリフレッシュレートに依存する。
    * DartアイソレートのイベントループはDartが単一のスレッドで並行処理をするための仕組み。イベントキュー上の各イベントを処理する速度は処理内容や実行環境の性能、Dart VMに依存する。
    * エンジンからの要求でdart:uiのPlatformDispatcher.onDrawFrame()やonBeginFrame()が呼ばれた際、それぞれ通常のDartコードと同様にDartのイベントループに基づいて順次実行される。
* フレームへのスケジューリングは重複しないか?
    * SchedulerBindingの実装を確認すると、フレームへのスケジューリングは次回のフレームのみに対して行われ重複はしないように実装されている
    * たとえば1フレームより短い時間でエンジンから次々と入力イベントが届いたとしても、リビルド・再レイアウト・レンダリングは次のフレームで1回実行されるのみとなる。


# ビルド(ツリーの構築)

## Widget, Element, RenderObject
* 以下はWidget, Element, Renderの各クラスを表記した図
    * メソッドは筆者が着目したものだけが表記されている。

![](./svg/arch_code_reading/flutter_class_widget_element_render_obj.drawio.svg)

* Widgetの親子同士の参照情報
    * 子はbuild()内で生成 or 親から受け取ったオブジェクトを利用
        * StatefulWidget(State)
        * StatelessWIdget
    * childとして子への参照を保持
        * SingleChildRenderObjectWidget
        * ProxyWidget
    * childrenとして子への参照を保持
        * MultiChildRenderObjectWidget

* Elementの親子同士の参照情報
    * childとして子への参照を保持
        * ComponentElement
        * SingleChildRenderObjectElement
    * childrenとして子への参照を保持
        * MultiChildRenderObjectElement
    * parent
        * 親への参照
    * slot
        * ある子が親が保持する子リストのどの位置に存在するかを表す

* RenderObjectの親子同士の参照情報
    * childとして子への参照を保持
        * RenderObjectWithChildMixin
    * 子が持つParentDataを使って親が子の情報を参照
        * ContainerRenderObjectMixin
    * parent
        * 親への参照


## 全体像
* 以下はWidgetツリー、Elementツリー、RenderObjectツリーが構築される流れである。

<img src="./svg/arch_code_reading/flutter_tree.drawio.svg" width="100%"><br/><br/>  

## Element
* 初回ビルド(マウント)
    * 初回のビルドは、以下のどちらかによって行われる。
    * 親のElementが子のElement.inflateWidget()を実行する
        * inflateWidget()の中でWidget.createElement()、mount()が実行される。
        * 基本的にはmount()の中でrebuild()やperformBuild()が実行される
    * (RootElementのみ)直接createElement()とmount()を実行
* リビルド
    * リビルド(ツリーの再構築)は以下のどちらかによって行われる。
        * 親のElementが子のElement.update()を呼ぶ
            * 基本的にはupdate()の中でrebuild()やperformRebuild()が呼ばれる
        * State.SetState()を呼ぶことでBuildOwner._dirtyElementsに追加され、次回のフレームでBuildOwnerが各Elementのrebuild()を呼ぶ。
* Element.mount(), deactivate(), activate(), unmount() と _ElementLifecycle
    * https://api.flutter.dev/flutter/widgets/Element-class.html
    * Element._lifecycleStateの初期状態は _ElementLifecycle.initial となる。
    * mount()では_ElementLifecycle.initial から  _ElementLifecycle.activeへ遷移する。また、Element.keyがnullではない場合はグローバルなマップにkeyとElementを保存する。
    * 画面への表示が必要なくなったウィジェットのElementはdeactivate()が実行されてElementLifecycle.inactiveに変更される。
    * グローバルキーによって共通のウィジェットを使用している場合、ElementLifecycle.inactiveなElementも、同ビルドフェーズ内であればツリー上の別の箇所で再利用される可能性があり、そのときはactivate()が実行されてinactiveからactiveへ変更される。 
    * _ElementLifecycle.inactiveであるElementはそのビルドフェーズの最後にまとめてunmountされる。
        * したがって一度deactivateされたElementのGlobalKeyによる再利用は、同一のフレーム内のビルドフェーズの中行われる必要があり、ビルドフェーズが終わるとunmountされて再利用はされない。
* Element.rebuild()
    * リビルドの起点。
    * コード概要
        ```
        rebuild()
            lifecycleがactiveではない または（!dirty && !force)の場合はreturn
            performRebuild()
        ```
* Element.performRebuild()
    * https://api.flutter.dev/flutter/widgets/Element/performRebuild.html
    ```dart
    void performRebuild() {
        _dirty = false;
    }
    ```
* Element.inflateWidget(Widget newWidget, Object? newSlot)
    * https://api.flutter.dev/flutter/widgets/Element/inflateWidget.html
    * 引数で与えられたウィジェットに紐づくElementを生成・マウントする。
        * 生成したElementは thisの子となる。
    * ウィジェットにGlobalKeyが設定されている、かつ対象のkeyに紐づくElementが存在し、かつアップデート可能(canUpdate()が真)な場合
        * activate()してElementを再利用する。
            * なお、この直前にelement._parent != nullの場合はnullにしてdeactivate()をしているので対象のelementは必ず非アクティブ状態になっている。
    * 上記以外
        * 対象のウィジェットのWidget.createElement()を実行してElementを生成する。
        * Element.mount()を実行する。
    * コード概要
        ```dart
        inflateWidget(Widget newWidget, Object? newSlot)
            Key? key = newWidget.key;
            if (key is GlobalKey) 
                final Element? newChild = _retakeInactiveElement(key, newWidget);
                if (newChild != null) 
                    newChild._activateWithParent(this, newSlot);
                    updatedChild = updateChild(newChild, newWidget, newSlot);
                    return updatedChild
            newWidget.createElement()
            newChild.mount(this, newSlot);
            return newChild;
        ```
        ```dart
        _retakeInactiveElement(GlobalKey key, Widget newWidget) 
            //...
            final Element? element = key._currentElement;
            if (element == null)  return null;
            if (!Widget.canUpdate(element.widget, newWidget)) return null;
            if (element._parent != null ) parent.forgetChild(element);parent.deactivateChild(element);
            owner!._inactiveElements.remove(element);
            return element;
        ```
* Element.mount(Element? parent, Object? newSlot)
    * https://api.flutter.dev/flutter/widgets/Element/mount.html
    * Elementを指定された親の指定されたスロット内のツリーに追加する。
* Element.update(covariant Widget newWidget)
    * https://api.flutter.dev/flutter/widgets/Element/update.html
    * 指定のウィジェットでElementを再構成する。指定されたウィジェットは古いウィジェットに対してcanUpdate()である必要がある。
    * ウィジェット自身がリビルドされた際には呼ばれるわけではなく、updateChild()を通して呼ばれる点に注意。
        * 例えばStateがsetStateを呼んでリビルドされても、rebuild()は呼ばれて子のupdate()は呼ばれる可能性があるが、自身のupdate()は呼ばれない。
    * コードの概要
        ```dart
        void update(covariant Widget newWidget)
            // ...
            _widget = newWidget;
        ```
* Element.updateChild(Element? child, Widget? newWidget, Object? newSlot)
    * https://api.flutter.dev/flutter/widgets/Element/updateChild.html
    * ウィジェットシステムの中核となるAPI
    * "updateChild"という名前だが、子の更新だけではなく、生成処理も含まれている。
        * 更新、生成、アクティベート・非アクティベートが行われる。
    * このメソッドは例えば、以下から呼ばれる
        * ComponentElement.performRebuild()
        * Element.updateChildren()
            * MultiChildRenderObjectElement.update() 等から呼ばれる
        * Element.inflateWidget()
        * SingleChildRenderObjectElement.mount()
        * SingleChildRenderObjectElement.update()
    * 前回ビルド時のElementと新しいウィジェットの両方が存在する場合は、両者を比較する。
        * この時に以下の条件を両方満たす場合にElement.update()が呼び、そうでは無い場合はinflateWidget()によって生成とmount()を行う。
        * 既存の子のウィジェットと新しいウィジェットが異なるオブジェクトである
            * オブジェクトがconst値の場合、必ず同一となるためupdateは呼ばれない。
        * canUpdate()が真となる場合
            * runtimeTypeが同一であり、かつ keyが一致する場合　に 真となる。
                * 両者のkeyがnullの場合も一致する。
                * したがってkeyがnullで、constでない場合は常にupdate()が実行される
    * なお、Widgetクラスの==メソッドはObjectの==を利用(同一のオブジェクトである場合のみtrue)していて、サブクラスでオーバーライドすると@nonVirtualによるエラーが表示される。
        ```dart
        abstract class Widget extends DiagnosticableTree {
            // ...
            @override
            @nonVirtual
            bool operator ==(Object other) => super == other; // Objectの==を利用。
            // ...
        }
        ```
    * コードの概要
        ```dart
        updateChild(Element? child, Widget? newWidget, Object? newSlot)
            if newWidget == null
                if (child != null)  deactivateChild(child);
                return null; 
                // ウィジェットがnullの場合は何もelementを返さない。
                // もし、Elementが存在する場合は非アクティブ化する。
                // TODO: この分岐は、おそらくupdateChildren()の方の経路から呼び出した際に通ると推測
            final Element newChild;
            if child != null
                // assert内でhotreload用の処理を行っている
                if child.widget == newWidget // 同一オブジェクトの場合
                    if (child.slot != newSlot) {
                        updateSlotForChild(child, newSlot);
                    }
                    newChild = child;
                else if Widget.canUpdate(child.widget, newWidget)
                    // 同一オブジェクトではない場合でもupdate扱いとなる分岐
                    if (child.slot != newSlot) {
                        updateSlotForChild(child, newSlot);
                    }
                    child.update(newWidget);
                    newChild = child;
                else  // 上記のどちらでもない場合はupdate対象外のため古いものをdeactivateして新しいものをinflateする
                    deactivateChild(child);
                    newChild = inflateWidget(newWidget, newSlot);
            else
                newChild = inflateWidget(newWidget, newSlot);
            return newChild
        ```
        ```
        static bool canUpdate(Widget oldWidget, Widget newWidget) {
            return oldWidget.runtimeType == newWidget.runtimeType
                && oldWidget.key == newWidget.key;
        }
        ```
* Element.updateChildren(List<Element> oldChildren, List<Widget> newWidgets, {Set<Element>? forgottenChildren, List<Object?>? slots})
    * https://api.flutter.dev/flutter/widgets/Element/updateChildren.html
    * 個々の子を更新するupdateChild()の便利なラッパーで、IndexedSlot<Element>をslotに設定する。
        * slotに関してはRenderObjectの方で説明する。
    * Element派生クラス(主にMultiChildRenderObjectElement)のupdate()メソッドから、子ごとにupdateChild()を呼び出すのではなくupdateChild()の代わりに呼ばれる事が多い。

* Element.unmount()
    * コードの概要
        ```dart
        unmount()
            assert(_lifecycleState == _ElementLifecycle.inactive);
            //...
            final Key? key = _widget?.key;
            if (key is GlobalKey) owner!._unregisterGlobalKey(key, this);
            _widget = null;
            _dependencies = null;
            _lifecycleState = _ElementLifecycle.defunct;
        ```


## StatefulElement/State
* 以下はStatefulElement/Stateに関連するクラスとメソッド、処理に焦点を当てて表記した図となる。

    ![](./svg/arch_code_reading/flutter_flow_state_widget.drawio.svg)

* State.didUpdateWidget()
    * Elementの具象であるStatefulElement.update()からState.didUpdateWidget()が呼ばれる。
* State.didChangeDependencies()
    * https://api.flutter.dev/flutter/widgets/State/didChangeDependencies.html
    * 以下のタイミングで呼ばれる。
        * mount()
            * State.initState()の後に実行される。
        * 依存先のInheritedElementに変更があった時のperformBuild()
            * あるInheritedElementを監視している場合、そのInheritedElementに変更が行われた際は監視する側のElement.didChangeDependencies()が呼ばれてStatefulElement._didChangedependenciesが真となる。この時、その後のStatefulElement.performRebuild()からState.didChangeDependencies()が実行さsれる。  
        * 依存先のInheritedElementがあり、activate()が実行された時
            * 何らかのInheritedElementに依存するElementにおいてactivate()が実行された場合、そのElementのdidChangeDependencies()が呼ばれ、その後のStatefulElement.performRebuild()からState.didChangeDependencies()が実行される。
    * APIの仕様にも書いてあるが、build()の手前で呼ばれて、かつ依存先に基づいて実行されるため、コストの高い処理などをbuild()よりも少ない頻度で実行したいケースでのみ利用する。したがって使う機会は少ない。
* State.dispose()
    * StatefulElement.unmount()から呼ばれる。

## RootElement
* ルートとなるRootElementからの処理は以下のようになる。  
<img src="./svg/arch_code_reading/flutter_flow_attach_root_widget.drawio.svg" width="85%"><br/><br/>  

* RootWidget
    * https://api.flutter.dev/flutter/widgets/RootWidget-class.html
        > A widget for the root of the widget tree.
    * RootWidget.attach()によって BuildOwnerへウィジェットツリーをattachするとともに、Elementツリーをビルドする。
* View
    * https://api.flutter.dev/flutter/widgets/View-class.html
        > Bootstraps a render tree that is rendered into the provided FlutterView.
    * https://api.flutter.dev/flutter/widgets/WidgetsBinding/wrapWithDefaultView.html
        > The View determines into what FlutterView the app is rendered into. 
        > This is currently PlatformDispatcher.implicitView from platformDispatcher.
        * 今現在、Flutterで使われているFlutterViewのオブジェクトは、PlatformDispatcher.implicitViewとなる。
* FlutterView
    * https://api.flutter.dev/flutter/dart-ui/FlutterView-class.html
        > A view into which a Flutter Scene is drawn.
        > Each FlutterView has its own layer tree that is rendered whenever render is called on it with a Scene.
    * render()
        * https://api.flutter.dev/flutter/dart-ui/FlutterView/render.html
        > Updates the view's rendering on the GPU with the newly provided Scene.

* 以下は上記に続く_RawViewからRenderViewが生成されるまでの処理の流れとなる  
<img src="./svg/arch_code_reading/flutter_flow_raw_view_to_render_view.svg" width="80%"><br/><br/>  

* PipelineOwnerのオブジェクトは下記のような構成となる。
    * ※ 現在(v3.19.6)ではrunAppからRendererBinding.pipelineOwnerが_deprecatedPipelineOwnerとして渡されている。
    * v4以降は_deprecatedPipelineOwnerは廃止されるかnullとなり_RawViewElement内で生成されたオブジェクトがeffectivePipelineOwnerへ設定されると考えられる。
    * debugDumpPipelineOwnerTree()によってPipelineOwnerの階層構造をプリントすることができる。

    <img src="./svg/arch_code_reading/flutter_pipeline_owner.svg" width="65%"><br/><br/>  

## RenderObject

<img src="./svg/arch_code_reading/flutter_class_relate_to_render_object.drawio.svg" width="85%"><br/><br/>  

* RenderObject
    * RenderObject派生クラスの多くはRenderBoxを継承する。
    * RenderSliver等の部品系等で継承しないものもある。
* RenderView
    * https://api.flutter.dev/flutter/rendering/RenderView-class.html
        > The root of the render tree.
    * 1つのRenderBoxオブジェクトを子として持つ
* RenderBox
    * https://api.flutter.dev/flutter/rendering/RenderBox-class.html
    * ボックスレイアウトでレイアウトを行うためのRenderObject
* RenderObject.parentData
    * ParentData
        * https://api.flutter.dev/flutter/rendering/ParentData-class.html
    * 自身の親のRenderObjectが子に対する情報を保存・扱うために使うプロパティ
    * 親側はRenderObject.setupParentData()をオーバーライドして、特定のParentData派生オブジェクトをparentDataへ設定させることができる。
        * RenderObject.setupParentData()はRenderObject.adoptChild()から呼ばれ、RenderObject.adoptChild()はRenderObject派生のRenderObjectWithChildMixinやContainerRenderObjectMixinで子の挿入の直前に呼ばれる。
* RenderObjectは以下のmixin, classを使って子を構成する。
    * RenderObjectWithChildMixin
        * ひとつの子をもつRenderObjectの構成のためのmixin。
    * ContainerRenderObjectMixin
        * 複数の子のRenderObjectを含むRenderObjectが子をリンクドリストとして管理するためのmixin。（RenderObjectのコンテナ）
        * 定義は下記のようになっている
            ```
            ContainerRenderObjectMixin<ChildType extends RenderObject, ParentDataType extends ContainerParentDataMixin<ChildType>> on RenderObject {
                //..
            }
            ```
        * RenderObject.parentDataは ContainerParentDataMixinを利用する
        * ContainerParentDataMixin
            * parentDataとしてリンクドリストの機能をもたせるmixin。
            * 定義は下記のようになっている
                ```
                ContainerParentDataMixin<ChildType extends RenderObject> on ParentData {
                    //...
                }
                ```
* Element.slot
    * Element.slot
        * 子Elementが親の持つ子リスト内においてどこに位置するか?という情報を保有する。
            * https://api.flutter.dev/flutter/widgets/Element/slot.html
                > Information set by parent to define where this child fits in its parent's child list.
        * 親の updateChild()によって子ウィジェットがinflateされた時に決定する。
    * IndexedSlot
        * https://api.flutter.dev/flutter/widgets/IndexedSlot-class.html
        * リスト内のどこに位置するかインデックスに加えて任意の値の
        * MultiChildRenderObjectElementにて利用されている。
        * 以下のように==がオーバーライドされている。
            ```dart
            bool operator ==(Object other) {
                if (other.runtimeType != runtimeType) {
                    return false;
                }
                return other is IndexedSlot
                    && index == other.index
                    && value == other.value;
            }
            ```
        * この比較はupdateChild内でslotの更新を判定する際に使われている。単にオブジェクト比較ではなく構成の変化(位置関係が変わった時)を判定するため。
            ```dart
            Element? updateChild(Element? child, Widget? newWidget, Object? newSlot) {
                // ...
                    if (child.slot != newSlot) {
                        updateSlotForChild(child, newSlot);
                    }
                // ...
            }
            ```
    * 実装を確認すると下記のように、複数の子を持つElementだけが具体的な値を設定している。
        * ComponentElement派生クラス
            * mount()やupdate()でのupdateChild()実行時に自身のslotをそのまま子に渡す
        * RenderObjectElement派生クラス
            * SingleChildRenderObject
                * mount()やupdate()でのupdateChild()実行時にslotはnullを渡す
            * MultiChildRenderObject
                * mount()でのinflateWidget()実行時にIndexedSlot<Element?>(子リストのインデックス, 子リスト上の順において手前の子) を渡す
                * update()でのupdateChildren()実行時には何も渡していない。(updateChildren()の中でIndexedSlotが生成されている。)
    * slotの値はどこで利用されるのか?
        * 具体的な活用としては、RenderObjectの子の順番が変わった際に、リンクドリストに挿入するためにこのslotが利用されている。
        * 下記のMultiChildRenderObjectElementがContainerParentDataMixinであるRenderObjectに対してinsert()を実行する際にIndexedSlotのデータが利用されている。
            ```
            @override
            void insertRenderObjectChild(RenderObject child, IndexedSlot<Element?> slot) {
                final ContainerRenderObjectMixin<RenderObject, ContainerParentDataMixin<RenderObject>> renderObject = this.renderObject;
                assert(renderObject.debugValidateChild(child));
                renderObject.insert(child, after: slot.value?.renderObject);// slot.valueには親Elementが保持する子リスト上の手前の子が設定されている。これによって、その子のRenderObjectのafterに挿入することを実現できている。
                assert(renderObject == this.renderObject);
            }
            ```
        * https://api.flutter.dev/flutter/widgets/Element/updateChildren.html
            > When the slot value of an Element changes, its associated renderObject needs to move to a new position in the child list of its parents. If that RenderObject organizes its children in a linked list (as is done by the ContainerRenderObjectMixin) this can be implemented by re-inserting the child RenderObject into the list after the RenderObject associated with the Element provided as IndexedSlot.value in the slot object.
            
            > Using the previous sibling as a slot is not enough, though, because child RenderObjects are only moved around when the slot of their associated RenderObjectElements is updated. When the order of child Elements is changed, some elements in the list may move to a new index but still have the same previous sibling. For example, when [e1, e2, e3, e4] is changed to [e1, e3, e4, e2] the element e4 continues to have e3 as a previous sibling even though its index in the list has changed and its RenderObject needs to move to come before e2's RenderObject. In order to trigger this move, a new slot value needs to be assigned to its Element whenever its index in its parent's child list changes. Using an IndexedSlot<Element> achieves exactly that and also ensures that the underlying parent RenderObject knows where a child needs to move to in a linked list by providing its new previous sibling.
        * https://api.flutter.dev/flutter/widgets/RenderObjectElement-class.html
            > Each child Element corresponds to a RenderObject which should be attached to this element's render object as a child.However, the immediate children of the element may not be the ones that eventually produce the actual RenderObject that they correspond to. For example, a StatelessElement (the element of a StatelessWidget) corresponds to whatever RenderObject its child (the element returned by its StatelessWidget.build method) corresponds to. Each child is therefore assigned a slot token. This is an identifier whose meaning is private to this RenderObjectElement node. When the descendant that finally produces the RenderObject is ready to attach it to this node's render object, it passes that slot token back to this node, and that allows this node to cheaply identify where to put the child render object relative to the others in the parent render object.A child's slot is determined when the parent calls updateChild to inflate the child (see the next section). It can be updated by calling updateSlotForChild.

## InheritedElement/didChangedDependencies
* 先祖のInheritedWidgetのサブクラスは、子孫のウィジェットからAPIを利用して監視・参照することが出来る。
* 具体的には子孫のウィジェットはBuildConext.dependOnInheritedWidgetOfExactType()で監視したり、BuildConext.getInheritedWidgetOfExactType()によって参照することができる。
* dependOnInheritedWidgetOfExactType()では監視する側のElementが、監視される側のInheritedElement._dependentsに追加される。
* 以下はInheritedElement.dependOnInheritedWidgetOfExactType()で_dependentsが設定される処理、および監視する側のElement.didChangeDependencies(), State.didChangedDependencies()が実行される処理の流れについて記載した図である。

<img src="./svg/arch_code_reading/flutter_flow_inherited_widget.drawio.svg" width="70%"><br/><br/>  



# レイアウト

![](./svg/arch_code_reading/flutter_class_relate_to_layout.drawio.svg)

* RenderObject
    * https://api.flutter.dev/flutter/rendering/RenderObject-class.html
    * RenderObjectクラスには、子に関する情報や座標系については定義されておらず、その派生クラスやmixinで定義されている。
    * RenderObjectのオブジェクトは不要になったらdispose()する。通常は作成者がRenderObjectElementであり、unmount時に作成したオブジェクトをdispose()する。
    * RenderObjectのdispose()は例えばPictureやImageといった高価なオブジェクトおよび自身が作成したSceneオブジェクトをクリーンナップする責務がある。
    * ほとんどのケースではRenderObjectの派生クラスを作成するのは過剰であり、RenderBoxの派生クラスを使うことが推奨されている
* RenderObject.layout()
    * https://api.flutter.dev/flutter/rendering/RenderObject/layout.html
    * 親が子供にレイアウト情報の更新を依頼するための主要なエントリ ポイント
    * 親はconstraintsオブジェクトを渡す
    * 実際のレイアウト、サイジングの作業はperformResize()とperformLayout()に委譲される。
    * サブクラスはlayout()ではなくperformResize()とperformLayout()をオーバーライドする。
* RenderObject.performLayout()
    * https://api.flutter.dev/flutter/rendering/RenderObject/performLayout.html
    * sizeByParentが true の場合、この関数は実際にこのレンダリング オブジェクトの寸法を変更してはいけない。
    * sizeByParentが false の場合、この関数はこのレンダリング オブジェクトの寸法を変更し、その子にレイアウトを指示する必要がある
* RenderObject.performResize()
    * https://api.flutter.dev/flutter/rendering/RenderObject/performResize.html
    * この関数は、 sizeByParentが true の場合にのみ呼び出される。
    * constraintsのみ利用してsizeをアップデートする
    * フレームワーク内でこの関数をオーバーライドし、独自の実装を入れているのは下記のクラスのみだった
        * RenderBox, RenderAndroidView, RenderTwoDimensionalViewport, _RenderDeferredLayoutBox
* RenderView
    * https://api.flutter.dev/flutter/rendering/RenderView-class.html
        > The root of the render tree.
    * RenderView.compositeFrame()
        * レンダーオブジェクトのツリーがdart:ui.Sceneオブジェクトへ変換され、dart:ui.FlutterView.render()によってGPUへ送信される。

## (参考) ParentDataWidget
<img src="./svg/arch_code_reading/flutter_class_relate_to_parent_data_widget.drawio.svg" width="100%"><br/><br/>  

* ColumnやRowウィジェットを利用した際のレイアウトの制約は、RenderFlex.performLayout()の処理によって実現されている。
* RenderFlex.performLayout()内の処理内では、子のRenderObject.parentData(FlexParentData).flex > 0 である場合と、そうでない場合で child.layout()の実行時に渡すConstraintsおよび自身のsizeの設定処理が異なっていることが分かる。
    * flex > 0の場合
        * 最大値の制約(maxHeight/maxWidth)にdouble.infiniteではない値を設定
        * これはFlex自身のConstraintなどを考慮した値?
    * そうではない場合
        * 最大値の制約(maxHeight/maxWidth)にdouble.infiniteを設定
* RenderViewport等では、このBoxConstraintsを受け取った時点でdouble.infiniteの場合にはエラーとなるようにAssertされている。
* したがってFlexの子孫としてRenderViewportなどのdouble.infiniteを許容しないRenderObjectが存在する場合、以下のどちらかを満たす必要がある。
    * 上位から来たdouble.infiniteの制約を許容しつつ、下位へ渡すConstraints内の最大値の制約がinfiniteではないRenderObjectを入れる。
    * parentData.flex > 0 となるRenderObjectが存在する。
* Flexible
    * https://api.flutter.dev/flutter/widgets/Flexible-class.html
    * Row, Column, またはFlexの子孫のflexを制御するウィジェット
    * ParentDataWidgetの派生クラスであり、内部処理としてParentDataWidget.applyParentData()によって最も近い子孫のRenderObjectのparentData.flexやfitを上書きする。
    * FlexibleウィジェットはRow, Column, またはFlexの子孫である必要がある。
        * FlexはsetupParentData()によってFlexParentDataを子のRenderObject.parentDataへセットしている。
        * これによってFlexibleウィジェットがapplyParentData()によってFlexParentDataを扱うことが出来るようになっている。
        * Flexの子孫ではない場合は下記のようにassertエラーとなる。
        ```dart
        main() => testWidgets("", (tester) async {
            await tester.pumpWidget(Flexible(child: Container()));
            });
        // The following assertion was thrown while applying parent data.:
        // Incorrect use of ParentDataWidget.
        // ...
        // RenderObjectElement._updateParentData.<anonymous closure> (package:flutter/src/widgets/framework.dart:6512:11)
        ```
* ParentDataWidget
    * https://api.flutter.dev/flutter/widgets/ParentDataWidget-class.html
    * ParentData情報をRenderObjectWidgetの子に フックするウィジェットの基本クラス
* ParentDataWidget.applyParentData()
    * このメソッドはParentDataWidgetに最も近い子孫のRenderObjectのmount()内のRenderObjectElement.attachRenderObject()によって呼ばれる。
    * それによってそのRendeObjectのparentDataを上書きすることが出来る。
```dart
main() {
  testWidgets("", (tester) async {
    await tester.pumpWidget(Directionality(
        textDirection: TextDirection.ltr,
        child: Column(
          children: [Flexible(child: ListView())],
        )));
    debugDumpRenderTree();
  });
}
// ...
// child: RenderFlex#99a23
// ...
//  └─child 1: RenderRepaintBoundary#b97ba relayoutBoundary=up1
// ...
//     │ parentData: offset=Offset(0.0, 0.0); flex=1; fit=FlexFit.loose
```

# GlobalKey
* 内部の実装としてはWidgetsBindingのシングルトンが持つマップを参照している。
    ```dart
    abstract class GlobalKey<T extends State<StatefulWidget>> extends Key {
        //...
        Element? get _currentElement => WidgetsBinding.instance.buildOwner!._globalKeyRegistry[this];
        BuildContext? get currentContext => _currentElement;
        //...
    }
    ```
* 上記の_globalKeyRegistryは以下のようにGlobalKeyとElementのマップで保存されている。
    ```dart
    class BuildOwner {
        // ..
        final Map<GlobalKey, Element> _globalKeyRegistry = <GlobalKey, Element>{};
        // ...
        void _registerGlobalKey(GlobalKey key, Element element) {
            //...
            _globalKeyRegistry[key] = element;
        }
        //...
    }
    ```
* 上記の仕組みによって、以下が実現されている
    * Elementの生成の際に、GlobalKeyによってグローバルなマップに保存されたElementを参照することでElementの再利用が可能か確認する。
        * ref. Element.inflateWidget()
    * 任意のウィジェットのkeyにGlobalKeyを設定することで、GlobalKey.currentContextのAPIによってBuildContextを取得することができる。
* (参考)(IME)GlobalKeyクラスとWidgetsBindingインスタンスの密結合について
    * この結合が筆者にとって問題となったシーンがある。
    * 通常のアプリケーション開発で行うことはまず無いが、FlutterではWidget.createElement()やElement.mount()のAPIを利用すれば自前でElementツリーやRenderObjectのツリーを作成することが可能である。
        ```dart
        // 例: ウィジェットのSizeを仮計算するコード

        final widget = .....

        // Create dummy BuildOwner.
        BuildOwner owner = BuildOwner(focusManager: FocusManager());

        // Create dummy render tree with inner widgets.
        // Using lockState, buildScope function to avoid assert error.
        _DummyTreeRootElement? element;
        owner.lockState(() {
            element = widget.createElement();
            element!.assignOwner(owner);
        });
        owner.buildScope(element!, () {
            element!.mount(null, null);
        });

        // Find _WrapColumn widget and get Size
        final columnSize = element!.computeDryLayout(const BoxConstraints());

        // Dispose dummy tree
        element!.deactivate();
        element!.unmount();
        ```
        * 全コード: https://gist.github.com/megur0/81edb7a49216b8998fe0802f63da4cd9#file-precompile_listview_extent-dart-L265
    * ただ、GlobalKey.currentContextのAPIは、直接 `WidgetsBinding.instance.buildOwner!._globalKeyRegistry[this];` を参照しており、runApp()が実行されている時のみ利用可能と成る。
    * runApp()が実行されていない場合は、WidgetsBinding.instanceの未初期化としてエラーとなってしまう。
        ```dart
        main() {
            GlobalKey().currentContext;
            // assert error: Binding has not yet been initialized.
        }
        ```
    * また、GlobalKeyの登録を行うElement.mount()ではWidget.keyをElement.owner._globalKeyRegistryに登録する一方、参照の際には上記のようにシングルトンのプロパティを参照する。
    * 上記のコードのように個別にElementを生成したい場合はBuildOwnerを個別に用意する必要がある(と筆者は考えている)ため、Widget.keyはそのBuildOwnerが保有するマップに登録される。
        * これが問題となるケースとして、標準ウィジェットの内部でGlobalKey.currentContextに対してnull assertionをしている場合がある。
        * 例えばListTileはInkクラスを内部で使っていて、_InkStateクラスは内部で下記のようにnull assertionをしている。
            ```
            final GlobalKey _boxKey = GlobalKey();
            // ...
            referenceBox: _boxKey.currentContext!.findRenderObject()! as RenderBox,
            ```
    * GlobalKeyおよびこれら標準のウィジェットはrunAppありきでビルドされる設計と考えて良いかもしれない。
    

# その他
## 備忘: コードリーディング未済
* 下記の内容についてはコードリーディングは未済
    * Paint〜Rendering
        * https://api.flutter.dev/flutter/dart-ui/FlutterView-class.html
        * https://api.flutter.dev/flutter/dart-ui/Scene-class.html
        * Sceneオブジェクト、GPU 上のビューのレンダリングの更新をするScene.renderメソッド等。
        * ツリー情報がSceneとしてGPUへ送信されて物理デバイスへ描画されるまでの流れ
    * ホットリロード関連(performReassemble)
## (参考) Stand-alone widget tree with multiple render trees to enable multi-view rendering
* 下記の変更によってRenderObjectToWidgetAdapter/RenderObjectToWidgetによってウィジェットからレンダーツリーへ橋渡しをしていた構成が、RootWidget内のViewウィジェット内の_RawViewウィジェットが担うように変更されている。
    * この変更では、元々単一だったRenderObjectのツリーのルートであるRenderViewを複数扱える構成に変更され、あわせてRenderViewオブジェクトに紐づくPipelineOwnerもそれぞれで新規生成されたオブジェクトを扱うことができる厚生となっている。
    * https://github.com/flutter/flutter/commit/6f09064e785b2bb600a390fe6d8be4ac6775b82b#diff-536040af34d7bdac5955bf3692693c96f3a0219c16c8130df4c8d980deb55751
* 変更前
    * RendererBinding.initInstances()によってRenderViewが生成され、RendererBinding.pipelineOwner.rootNodeに設定され、getter RenderBinding.renderViewを通して単一のRenderViewを参照、処理を行う。
    * 具体的にはdrawFrame()の最後にGPUへSceneを送信する際は、単一のRenderView.compositeFrame()を実行するという構成だった。
    * RenderObjectToWidgetAdapter.createRenderObject()の方は、既に生成されたRenderViewオブジェクトを参照するようになっていた。
* 変更の具体的内容は下記。
* RendererBindingsは複数のRenderViewを所持する。
    * RendererBinding.renderViewsの各オブジェクトに対してcompositeFrame()が実行されるようになっている。
    * このオブジェクトはRendererBinding.addRenderView()やremoveRenderView()によって追加・削除されるようになっている。
    * 具体的には、_RawView.createRenderObject()にてRenderViewが生成されるようになり、_RawViewElement.mount()内でaddRenderView()が呼ばれている。
    * ただし、現在(下位互換性を維持する間)は下記のようにrenderViewはRendererBinding.pipelineOwnerで生成されるものが使われている。
        ```dart
        class _RawView extends RenderObjectWidget {
        //...
            @override
            RenderObject createRenderObject(BuildContext context) {
                return _deprecatedRenderView ?? RenderView(
                view: view,
                );
            }
        //...
        }
        ```
        ```dart
        void runApp(Widget app) {
            // ...
            binding
                ..scheduleAttachRootWidget(binding.wrapWithDefaultView(app))
                ..scheduleWarmUpFrame();
        }
        ```
        ```dart
        mixin WidgetsBinding on BindingBase, ServicesBinding, SchedulerBinding, GestureBinding, RendererBinding, SemanticsBinding {
            //...
            Widget wrapWithDefaultView(Widget rootWidget) {
                return View(
                view: platformDispatcher.implicitView!,
                deprecatedDoNotUseWillBeRemovedWithoutNoticePipelineOwner: pipelineOwner, // これはRendererBinding.pipelineOwnerを指す
                deprecatedDoNotUseWillBeRemovedWithoutNoticeRenderView: renderView, // これは RendererBinding.renderViewを指す
                child: rootWidget,
                );
            }
            // ...
        }
        ```
        ```dart
        @Deprecated(/* ... */)
        late final PipelineOwner pipelineOwner = PipelineOwner(/* ... */);

        @Deprecated(/* ... */)
        // TODO(goderbauer): When this deprecated property is removed also delete the _ReusableRenderView class.
        late final RenderView renderView = _ReusableRenderView(
            view: platformDispatcher.implicitView!,
        );
        ```
* _RawViewElementごとにPipelineOwnerを保持するようになっている。
    * したがって複数の_RawViewElementすなわち複数のViewオブジェクト、RenderViewオブジェクトが存在すれば、それぞれにPipelineOwnerが存在することになる。
    * ただし、下位互換性を保証する間はrenderViewと同様にRendererBinding.pipelineOwnerを使いまわしするようになっている。
    ```dart
    class _RawViewElement extends RenderTreeRootElement {
        //...
        late final PipelineOwner _pipelineOwner = PipelineOwner(
            onSemanticsOwnerCreated: _handleSemanticsOwnerCreated,
            onSemanticsUpdate: _handleSemanticsUpdate,
            onSemanticsOwnerDisposed: _handleSemanticsOwnerDisposed,
        );

        PipelineOwner get _effectivePipelineOwner => (widget as _RawView)._deprecatedPipelineOwner ?? _pipelineOwner;
        //...

        @override
        void mount(Element? parent, Object? newSlot) {
            //...
            _effectivePipelineOwner.rootNode = renderObject;
            //...
        }
    ```




