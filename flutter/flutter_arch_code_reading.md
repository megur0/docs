- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# 注意
* 以下は筆者がFlutterのコードリーディングを2023年の7月頃に行って理解した内容である。
* あくまでも筆者の理解であり内容に誤りが含まれている可能性もある点に注意。

# クラス図の表記について
* [クラス図の表記方法](../common/about_class_diagram.md)

# レンダリングに関するクラス、メソッド・プロパティの色付け
* レンダリングに関するプロパティやメソッドに文字色をつけている
* クラスの見出しの背景色
    * <span style="background-color: #E1D5E7; ">Widget</span>
        * RenderObjectのWidgetは<span style="background-color: #76608A; color:white">紫色</span>としている
    * <span style="background-color: #D5E8D4; ">Element</span>
    * <span style="background-color: #F8CECC; ">RenderObject</span>
* プロパティやメソッドの文字色
    * **黒太字** 
        * マウント系の処理
        * runApp()から実行される各Bindings.initInstances()による初期処理・初期マウント処理
    * <span style="color: blue; ">青文字</span>
        * リビルド・リレイアウト・リレンダリング系の処理
        * リレンダリング(paint)系は背景色を白としている
        * リビルドからはマウント系の処理も適宜実行されるが、それらは青文字にはしていない。
            * それらの処理は<span style="color: blue; font-weight:bold">青文字かつ太字</span>にする方がより正確だが、大変であるためやっていない。
    * <span style="color: green; ">緑文字</span>
        * 状態をdirtyにする、およびそれによって次回のフレーム処理をスケジューリングするメソッド

# WidgetとElementのクラス図

![](./svg/arch_code_reading/flutter_widget_element_class.drawio.svg)

# フレーム処理
## runApp実行時の流れ
1. 以下が順に実行される。
    * ensureInitialized(各**BindingのinitInstances()の処理)
    * mount系の処理（図中で**黒太文字**となっている箇所）
    * ensureVisualUpdate()
    * scheduleWarmUpFrame()
2. 初回のdrawFrameの処理（図中で<span style="color: blue; ">青文字</span>となっている箇所）
    * initInstancesの処理にて、addPersistentFrameCallbackによって、_persistentCallbacksへdrawFrameを実行する関数が登録される。
        * _handleDrawFrameの中で_persistentCallbacksが実行される。
        * _persistentCallbacksは名前の通り永続的なコールバックであり、（_transientCallbacksや_postFrameCallbacksとは違って）_handleDrawFrameの実行時には毎回実行される。
    * そして_persistentCallbacksを呼ぶhandleDrawFrameは、scheduleWarmUpFrameによってすぐに呼ばれる。（Timer.run()で呼んでいてイベントキューに従って実行される）
        * もしくは、エンジン側で次回フレームが先に到来した場合にはそちらからdrawFrameが実行される。これは上記のensureVisualUpdate()によって_handleDrawFrame関数をエンジンに対して次回フレームで呼び出すようにスケジューリングをしているためである。
3. 以降のdrawFrameの処理
    * Element.markNeedsBuild()（※）が呼ばれることで、ensureVisualUpdate()が呼ばれて_handleDrawFrameがエンジンのコールバックへスケジューリングされることで実行される。
        * ※ 状態をdirtyにする処理。<span style="color: green; ">緑文字</span>にしている。StateのsetState()関数等から呼ばれる

## drawFrameの処理の流れ

* 以下はSchedulerBindingsの各フェーズとdrawFrameの各処理の概要を図にしたものである。

<img src="./svg/arch_code_reading/flutter_frame_abst.drawio.svg" width="50%"><br/><br/>  

1. dirtyな各Elmentに対して、rebuild()によってツリーがアップデートされる。
    * 具体的な処理は各ElementのperformRebuildメソッドの実装に従う。
    * RenderObjectElementであればupdateRenderObjectメソッドによってレンダーツリーがアップデートされる。
2. レンダーツリーにしたがってレイアウト処理（サイズの決定、ポジションの決定）が行われる。
    * これは_nodesNeedingLayoutの中に入っているRenderObjectに対して行われる。
    * 初回のdrawFrameメソッドではツリーのルートが入っている。
        * RenderViewがinitInstancesメソッドにて設定される。
    * 具体的にはRenderObjectのlayoutメソッドが実行され、その内部の実装は各RenderObjectのperformLayoutメソッドやperformResizeメソッドの実装に従う。
        * なお、レイアウトは自身で決まるのではなく、親から受け取ったConstraintsおよび自身の子のsizeを以て自身のsizeが算出され、最終的に親側によって子のポジションとsizeが決定されることが原則
        * したがって、レイアウト処理は親から子へ一斉に実行される。
3. paintメソッドが実行される
    * _nodesNeedingPaintの中の各オブジェクトに対して実行され、depthが深いものから実行されていく。
    * 具体的な処理は各RenderObjectのpaint()メソッドの実装に従う。

## レンダーツリー構築に焦点を当てた図
* 以下はrunApp(ウィジェット)を実行した際に内部で行われるレンダーツリー構築の流れを図にしたものである。

![](./svg/arch_code_reading/flutter_render_tree.drawio.svg)

## クラス図と処理の流れ

![](./svg/arch_code_reading/flutter_run_app_flow.drawio.svg)

* RenderObject
    * あるサブツリー上のRenderObjectは同じownerとなる。
    * layout()で受け取るConstraintsは基本的にフィールドに設定されるのみの処理で、制約を基にしたサイジングは派生クラスのperformLayout()関数に委ねられていると思われる。
    * RenderObject派生クラスはRenderBoxを基本的に継承する。
        * RenderSliver等の部品系等で継承しないものもある。
        * 各レンダリング系のウィジェットクラスはCreateRenderObject()でこれらの派生クラスを生成する。
* RenderObject.performResize()
    * この関数をオーバーライドし、独自の実装を入れているのは下記のクラスのみだった
        * RenderBox, RenderAndroidView, RenderTwoDimensionalViewport,_RenderDeferredLayoutBox
* RenderObjectは以下のmixin, classを使って子を構成する。
    * RenderObjectWithChildMixin
        * ひとつの子をもつRenderObjectの構成のためのmixin。
    * ParentDataWidget, ParentData
        * 複数の子を持つ RenderObjectWidget に子ごとの構成（ParentData）を提供するための仕組み。
    * ContainerRenderObjectMixin / ContainerParentDataMixin
        * ContainerRenderObjectMixin
            * 複数の子のRenderObjectを含むRenderObjectが子をリンクドリストとして管理するためのmixin。（RenderObjectのコンテナ）
            * 定義は下記のようになっている
                ```
                ContainerRenderObjectMixin<ChildType extends RenderObject, ParentDataType extends ContainerParentDataMixin<ChildType>> on RenderObject {
                    //..
                }
                ```
        * ContainerParentDataMixin
            * ParentDataにリンクドリストの機能をもたせるmixin。
            * 定義は下記のようになっている
                ```
                ContainerParentDataMixin<ChildType extends RenderObject> on ParentData {
                    //...
                }
                ```
* RenderObjectToWidgetElementクラス
    > The root of the element tree that is hosted by a [RenderObject]. This element class is the instantiation of a [RenderObjectToWidgetAdapter] widget. It can be used only as the root of an [Element] tree (it cannot be mounted into another [Element]; it's parent must be null). In typical usage, it will be instantiated for a [RenderObjectToWidgetAdapter] whose container is the [RenderView] that connects to the Flutter engine. In this usage, it is normally instantiated by the bootstrapping logic in the [WidgetsFlutterBinding] singleton created by [runApp].
    * このElementのparentは必ずnullになる。
* イベントキューと、フレームについて（筆者の理解）
    * フレームはエンジンにおいてレンダリングを行う間隔の単位。この間隔は実行環境の性能やレンダリングエンジンに依存する
    * イベントループはDartが単一のスレッドで並行処理をするための仕組み。イベントキュー上の各イベントを処理するサイクルは実行環境の性能やDart VMに依存する。
    * フレーム毎にエンジンからdart:uiのPlatformDispatcher.onDrawFrameやonBeginFrameが呼ばれるが、通常のDartコードと同様にDartのイベントループに基づいて順次実行される。
    * 各フレームで実行される一連の処理は、ひとつのイベントで実行される。
        * たとえば今回のフレームではなく後続で実行したい処理は`Timer.run`や`scheduleMicrotask`などを使っている処理がフレームワークの中で随所で確認できる。
        * アプリケーションコードでもStateのライフサイクルメソッド内などで、`Future((){ 処理 })`といった形で処理を書くとそのメソッドが実行されているフレームよりも後のイベントで実行される。
        * 注意)この解釈は推測が混じっている。
* フレームへのスケジューリングは重複しないか?
    * SchedulerBindingの実装を確認すると、フレームへのスケジューリングは次回のフレームのみに対して行われ重複はしないように実装されている
    * たとえば1フレームより短い時間でエンジンから次々と入力イベントが届いたとしても、リビルド・リレイアウト・リレンダーは次のフレームで1回実行されるのみとなる。
* 再描画の具体例
    * 例えば、スクロール処理を例に考える。
    * スクロール処理ではプラットフォーム上からでの入力イベントを（Embbederを介して）エンジンが検知しそれをDart Frameworkへ伝達しスクロールのオフセットが更新されてnotifyListenerメソッドによって（事前にaddListnerされた）markNeedsLayoutが呼ばれる。
    * これによって_nodesNeedingLayoutへ登録され、次回のDrawFrameがスケジューリングされるため次回のフレームでdrawFrameが実行され、（リビルドの有無とは関係なく）_nodesNeedingLayoutに含まれるRenderObjectが再レンダリングされる。
    * なお、リビルドされる時（setState等からmarkNeedsBuildをした場合）は、RenderObjectElementからRenderObjectWidgetのupdateRenderObjectが呼ばれ、RenderObjectに何らかの更新がかかる。
    * この更新によってRenderObjectが_nodesNeedingLayoutへ登録するかどうかはRenderObjectの実装に委ねられるものの、通常はダーティにするため基本的にはリビルドにはリレンダーも含まれると考えて良い（と筆者は理解している）

* 以下はStateに関連するクラスとメソッド、処理を表記した図

    ![](./svg/arch_code_reading/flutter_class_relate_to_state.drawio.svg)

    * Elementのupdate
        * Stateクラスのライフサイクルにおいて使われるメソッドState.didUpdateWidgetは、このupdateメソッドから呼ばれる。
        * updateメソッドは親ウィジェットのperformRebuildメソッドの実行時にupdateChildメソッド経由で条件を満たしたときに呼ばれる。
            ```
            if (hasSameSuperclass && child.widget == newWidget) {
                // ...
            } else if (hasSameSuperclass && Widget.canUpdate(child.widget, newWidget)) {
                // ...
                child.update(newWidget);
            }
            ```
    
    
## その他
* InheritedElement
    * Element.dependOnInheritedElementで監視(変更に伴って自身をリビルドしたいケース等）
        * InheritedElement._dependentsに追加される
    * Element.getElementForInheritedWidgetOfExactTypeをBuildContext経由で参照
    * dependOnInheritedWidgetOfExactTypeによって、_dependentsに追加されると、Element.updateの際に、ProxyElement.notifyDependentによってdependent.didChangeDependencies()が呼ばれて再ビルドされるという仕組み。
* ColumnやRowのレイアウト制約の仕組み
    * ColumnやRowウィジェットを利用した際のレイアウトの制約は、RenderFlexのperformLayout()の処理によって実現されている。
    * performLayout()内の処理が複雑だが、子のparentData(FlexParentData).flex > 0 である場合と、そうでない場合で child.layoutの実行時にわたすConstraintsおよび自身のsizeの設定処理が異なっていることが分かる。
    * おそらくflex > 0の場合はdouble.infiniteではない値（これはFlex自身のConstraintなどを考慮した値?）を設定し、そうではない場合は最大値の制約(maxHeight/maxWidth)がdouble.infiniteとなる。
    * RenderViewport等では、このBoxConstraintsを受け取った時点でdouble.infiniteの場合にはエラーとなるようにAssertされている。
    * したがって、Flexの子孫としてRenderViewportなどのdouble.infiniteのを許容しないRenderObjectが存在する場合、以下のどちらかを満たす必要がある。
        * 間にConstraintsの最大値がinfiniteではないRenderObjectを入れる。
        * parentData.flex > 0 となるRenderObjectが存在する。（= Flexibleウィジェットが間に存在する）
* ScheduleBinding
    * debugPrintScheduleFrameStacksプロパティをtrueとするとscheduleFrameが呼び出されたタイミングを出力して確認が可能。
* dart:ui/PlatformDispatcher
    * https://api.flutter.dev/flutter/dart-ui/PlatformDispatcher-class.html
    > The most basic interface to the host operating system's interface.This is the central entry point for platform messages and configuration events from the platform.It exposes the core scheduler API, the input event callback, the graphics drawing API, and other such core services.
    * PlatformConfigurationNativeApi::ScheduleFrame
        * https://github.com/flutter/engine/blob/main/lib/ui/window/platform_configuration.h#L496
        * https://github.com/flutter/engine/blob/main/lib/ui/window/platform_configuration.cc#L470
        * おそらくこの箇所を最終的に実行している？
            * https://github.com/flutter/engine/blob/main/shell/common/animator.cc#L199
* engineのコード
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
* (IMO)GlobalKeyクラスとWidgetsBindingインスタンスの結合
    * GlobalKey.currentContextでは、現在のBuildContextを取得することができる。
    * 内部の実装としてはWidgetsBindingのシングルトンが持つマップを参照している。
        ```
        abstract class GlobalKey<T extends State<StatefulWidget>> extends Key {
            //...
            Element? get _currentElement => WidgetsBinding.instance.buildOwner!._globalKeyRegistry[this];
            BuildContext? get currentContext => _currentElement;
            //...
        }
        ```
    * BuildOwnerクラスでは以下のように_globalKeyRegistryを定義していて_registerGlobalKeyによって登録を行っている。
        ```
        final Map<GlobalKey, Element> _globalKeyRegistry = <GlobalKey, Element>{};
        void _registerGlobalKey(GlobalKey key, Element element)
        ```
    * この結合が(筆者にとって)問題となったシーン
        * 通常のアプリケーション開発で行うことはまず無いが、FlutterではWidget.createElement()やElement.mount()を直接呼び出すと自前でレンダリングツリーを作ることが可能である。
        * ただ、そのウィジェットツリーの中で、上記のcurrentContextを参照しているウィジェットを利用するとエラーとなってしまう。 
        * 例えばListTileはInkクラスを内部で使っていて、その_InkStateクラスが以下の行でnull assertionでエラーとなってしまう。
            ```
            referenceBox: _boxKey.currentContext!.findRenderObject()! as RenderBox,// WidgetsBinding.instance.buildOwner!._globalKeyRegistry[this]には存在しないためエラーとなる。
            // ※ _boxKeyは、final GlobalKey _boxKey = GlobalKey();　として生成されている。
            ```
* 下記の内容については（体力が尽きたため）コードリーディングしていない。
    * activate, deactivate
    * アンマウント（dispose）系の処理
    * paintの具体的な処理