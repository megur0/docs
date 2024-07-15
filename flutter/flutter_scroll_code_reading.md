[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# 注意
* 以下は筆者がFlutterのコードリーディングを2023年の7月頃に行って理解した内容である。
* 筆者の理解であり内容に誤りが含まれている可能性、情報が古くなっている可能性がある点に注意。

# スクロール処理についてコードリーディングを行ったモチベーション
* 開発中に以下のような発見があり、内部の動作がどのようになっているか興味を持ったことが動機となる。
* ListViewを使った画面のデバッグ中にオフセットの最大値(maxScrollExtent)の値が変動することがわかった
    * 検索すると関連するIssueがいくつかあることがわかった。
* ScrollController.postionの各種メソッドを利用した時に、APIのドキュメントのみでは予測できない挙動と遭遇した
    * ScrollPosition.jumpTo()はスクロールアクションが強制的に停止されて指定のオフセットへジャンプする一方、
    * correctBy()でオフセットを変更してみるとスクロールアクションは停止しないが、直後に変更前のオフセット位置に強制的に戻されてしまう。
    * おそらく何らかのアニメーション処理が発生していることは予測できたものの、具体的な中身を知りたくなった。

# クラス図の表記について
* [クラス図の表記方法](../common/about_class_diagram.md)

# レンダリングに関するクラス、メソッド・プロパティの色付け
* [アーキテクチャ(レンダリング)](./flutter_arch_code_reading.md) を参照

# ScrollableState, ScrollPosition, ScrollActivity

![](./svg/scroll_code_reading/flutter_scroll_activity_class.drawio.svg)


## ScrollableStateのビルド
* ScrollableStateのビルドの要点として、下記が実行される。
    * InitState
        * ScrollControllerの生成
            * Scrollableウィジェットがコントローラーを保持しているならば（これはListViewで指定したコントローラーとなる）それを利用する
    * build
        * ViewPortウィジェットの生成
            * ScrollableウィジェットのviewportBuilderを使用して生成する。
            * ViewPortについては別章で触れる。
        * Listenerウィジェットの生成
            * マウスなどによるポインタイベントを監視し、ハンドラーを実行する。
        * RawGestureDetectorウィジェットの生成
            * ドラッグイベントを監視し、ハンドラーを作成する。
        * linux, macOS, windowsの場合は、Scrollbarウィジェットを生成され上記のウィジェットを包含する。
    * didChangeDependencies
        * ScrollPositionの生成とコントローラーへのアタッチが行われる
    * didUpdateWidget
        * コントローラーが前回のビルド時と違うオブジェクトの場合のみ、ScrollPositionを再度コントローラーへアタッチする
    * ScrollPositionのコントローラーへのアタッチ
        * ScrollController._positonsへセットされる。
        * ScrollPositionへaddListener(ScrollController.notifyListeners)される。
            * ここでScrollController.notifyListenersが指定されることで、開発者がコントローラーへaddListenerで指定したコールバックが実行されることになる。

## ドラッグ操作によるアニメーションとレンダリング
* 流れとしては以下のようになる
    1. ホールド・ドラッグスタート(ドラッグ操作のために画面へタッチ)
    2. ドラッグアップデート(ドラッグ操作)
    3. ドラッグ終了/キャンセル（ドラッグが終了）
    4. スクロールアニメーション
* ScrollState内の子ウィジェットであるRawGestureDetectorによってドラッグなどの操作を検出され、ScrollPosition(ScrollPositionWithSingleContext)からDragContollerオブジェクトが生成される。
* ドラッグ中はDragContoller -> 委譲クラスのScrollActivityDelegate(具象はScrollPositionWithSingleContext).applyUserOffset()によってオフセットが更新されて再レイアウト・レンダリングされる。
* ドラッグを終了した時点で下記が行われる。
    * その区間の時間と距離から速度(velocity)が算出され、以降の時間経過に対するスクロールの物理的な移動距離を算出する Simulation オブジェクトが生成される。
    * BallisticScrollActivityが生成され、そのオブジェクトでAnimationオブジェクトのライフサイクル（生成・開始・破棄）の管理と委譲クラス（ScrollActivityDelegate）を通じてスクロールのオフセットのアップデートを行う
    * ※ 尚、ホールドやドラッグ開始、キャンセルでもそれぞれアクティビティが生成されるがこれらの説明や表記は割愛・省略する。
* Animationオブジェクト（(実際にはAnimationを実装したAnimationController）は、引数として受け取ったSimulationオブジェクトおよび経過時間に基づいてフレーム毎に値(value)を更新する
    * これはTickerというフレーム毎の処理をアニメーションフレームとして抽象化したクラスにコールバックとして渡す。
    * 内部ではTickerはコールバックを_transientCallbacksとして登録する。
        * アクティブである限りコールバック実行後に再度コールバックを_transientCallbacksへ登録することで連続したフレームで処理が行われる。
    * _transientCallbacksは_persistentCallbacksの前（リビルド・再レイアウト・レンダリング）に実行される。
    * https://api.flutter.dev/flutter/scheduler/Ticker-class.html
* Animation.valueの値を使って、BallisticScrollActivityは(委譲クラスを通じて)スクロールのオフセットのアップデート（ScrollActivityDelegate.setPixels()）を行う。
* オフセットがアップデートされると、ChangeNotifierを継承したScrollPosition.notifyListenerが呼ばれ、マウント時にaddListenerされているmarkNeedsLayoutが呼ばれて再レイアウト・レンダリングされて画面へ反映される。
* このAnimationオブジェクトによる値の更新とオフセットのアップデート・markNeedsLayoutとレンダリングが毎フレーム繰り返されることで、スクロール画面が動いているように見える。

## (参考)ScrollController.onAttach
* ScrollContollerを使ったScrollPositionの各種メソッドの実行
    * スクロールのオフセット情報を参照する標準の方法は、ScrollControllerを利用することである。
    * ただ、ScrollContoller.positionの参照にあたっては注意しなければならない点として、ScrollPositionがアタッチされる前に参照するとエラーとなってしまう事がある。
        * アタッチ：ScrollController._postionsのリストに生成されたScrollPositionがaddされること。
        ```
        // packages/flutter/lib/src/widgets/scroll_controller.dart
        ScrollPosition get position {
            assert(_positions.isNotEmpty, 'ScrollController not attached to any scroll views.');
            assert(_positions.length == 1, 'ScrollController attached to multiple scroll views.');
            return _positions.single;
        }
        ```
* そのため、よくある手段としてWidgetsBinding.instance.addPostFrameCallbackで渡すコールバック内でScrollContoller.positionを参照する処理を行う手段がある。
* しかし、この時点ではすでにリビルド・再レイアウト・画面へのレンダリングは終わっている点に注意する必要がある。
    * WidgetsBinding.instance._postFrameCallbacksはフレームの最後に実行される
    * ほとんどのユースケースでは上記は問題にならず、WidgetsBinding.instance.addPostFrameCallbackで十分。
* 例えば描画前にスクロールのオフセットを変更したい（※）といった場合は、再レイアウトの前に実行する必要がある。
    * ※ 基本的にアプリケーション開発でこのような要件が発生することは少ないかもしれない。
    * 実際にaddPostFrameCallbackでオフセットを変更してみると1フレーム遅れてオフセットの変更が画面に反映されることが確認できた。
* アタッチ〜再レイアウトの前に実行する方法
    * ScrollController.onAttachを利用する。
        * ScrollPostionがアタッチされた直後に実行されるため、確実にエラーを回避できる。
    * (参考)
        * これについて調べていた当初(2023/7月)は当該機能はmasterへマージされていたもののリリースはされていなかった。
            * https://github.com/flutter/flutter/pull/124823
            * https://github.com/flutter/flutter/commit/27caa7fed9e1f1d8555a53cf814d66851e9e9dfe
    * 注意点: 初回のみの実行だけではなく、繰り返し実行したいユースケースの場合
        * 例えば条件に応じて継続的にオフセットを補正したい、等
        * アタッチは、didChangeDependenciesとdidUpdateWidgetから呼ばれる。
        * ただしdidUpdateWidgetではScrollControllerが同一のオブジェクトの場合は、attachが呼ばれない。
        * もしdidUpdateWidgetサイクルで実行したい場合は、(少し無理矢理だが)ScrollControllerを再生成して渡す、といったやり方が考えられる。
        * (参考)「positionがattachされた後、かつビルドサイクルで呼ばれ、かつ、レンダリング前にコールバックを実行する」APIはあるか？
            * 恐らく無い?
            * もし実現したいのであればScrollableクラスを拡張する必要がある。（費用対効果の観点から、Scrollableクラスの拡張は行うことはまずないだろう）           

## (参考)スクロールのオフセットと進行中のアクティビティを補正するプログラム
* https://github.com/flutter/flutter/issues/99158#issuecomment-1637020182
* 動的にロードされる子のサイズ変更に伴ってmaxScrollExtentが変わった際に、SingleChildScrollViewのスクロールのオフセットがずれてしまうことを回避したプログラム。
    * ListViewに関しては動的にmaxScrollExtentが変わることの考慮も必要なため書いていない。
* ScrollPositionを拡張した独自クラスでapplyNewDimensionsをオーバーライドして、maxScrollExtentが前回の値から変わったかどうかをチェックする。
* 変わっている場合は、その差分だけオフセットを修正(ScrollPosition.correctPixels())し、ScrollActivityを新しいシミュレーション結果を持たせたオブジェクトで上書きしている。
* 単にスクロールの位置を修正するだけであれば、ScrollPosition.correctPixels()やcorrectBy()などのメソッドで修正可能だが以下の問題がある
    * 通常、ScrollPositionを参照する場合は（ScrollPositionがアタッチされていることを保証する必要があるため)addPostFrameCallbackを使う。このコールバックによってcorrectByを実行するとすでに画面へのペイントが完了しているため、1フレームのチラつきが発生してしまう。
    * スクロール中の場合、事前にシミュレーションされた結果によって各フレーム毎にオフセットが上書きされ続けてしまうため、オフセットを補正する場合は合わせてそちらも補正する必要がある。



# ScrollView(ListView)とViewport

## ツリー構造・クラス図・処理の流れ

* 下記の図はビルド・レイアウトフェーズにおいてListViewのbuildメソッド以降で生成されるWidgetとRenderObjectの階層を表した図となる。<br/><br/>

![](./svg/scroll_code_reading/flutter_scroll_view_build.drawio.svg)

<br/>

* 下記の図はListViewを起点として生成されるViewportやSliver、それらのElementとRenderObject、関連するクラスと処理の流れを表記している。<br/><br/>

![](./svg/scroll_code_reading/flutter_scroll_view_class.drawio.svg)

* 前提としてListViewに関連するクラスの一部を以下に示す。
    * ScrollView
        * https://api.flutter.dev/flutter/widgets/ScrollView-class.html
        * ScrollViewを構成する3つの要素は下記となる。
            * Scrollableウィジェット
            * Viewportウィジェット(もしくはShrinkWrappingViewportウィジェット)
            * 1 つ以上のスライバー

    * Scrollable
        * https://api.flutter.dev/flutter/widgets/Scrollable-class.html
        * 1次元でのスクロールを管理し、コンテンツが表示されるビューポートに通知するウィジェット
        * Scrollable はジェスチャ認識を含む、スクロール可能なウィジェットの対話モデルを実装する。
        * 実際に子を表示するビューポートがどのように構築されるかは責務外

    * Viewport
        * https://api.flutter.dev/flutter/widgets/Viewport-class.html
        * ビューポートという用語は一般的には「表示領域」を指す。
        * コンテンツがビューポートを超えるサイズとなる場合は、スクロールによって表示領域を制御することがUIにおいて一般的
        * Viewportクラスは、Scrollableクラスと組み合わせて利用する。
        * 自身の領域サイズとオフセットに基づいて子のサブセットを表示する。
        * ビューポートにはボックスレイアウトのウィジェットを直接子とすることはできない。SliverList, SliverFixedExtentList, SliverGrid, SliverToBoxAdapter を 利用する必要がある。
            * ボックスレイアウトのウィジェットは、RenderBoxを継承するRenderObjectにてレイアウトされるウィジェット。
            * https://api.flutter.dev/flutter/rendering/RenderBox-class.html

    * Sliver
        * https://api.flutter.dev/flutter/widgets/CustomScrollView/slivers.html
        > sliver (noun): a small, thin piece of something.
        > A sliver is a widget backed by a RenderSliver subclass, i.e. one that implements the constraint/geometry protocol that uses SliverConstraints and SliverGeometry.
        *  RenderSliv​​erサブクラスによってサポートされるウィジェット
        * SliverConstraints およびSliverGeometryを使用する制約/ジオメトリ プロトコルを実装する

    * ListView
        * https://api.flutter.dev/flutter/widgets/ListView-class.html
        * 最も一般的に使用されるスクロール ウィジェット
        * スライバーとしてSliverListウィジェットを採用している。
        * SliverListウィジェットは表示領域（+キャッシュ領域）のウィジェットのみをビルド・描画することによって性能向上を図っている

    * CustomScrollView
        * https://api.flutter.dev/flutter/widgets/CustomScrollView-class.html
        * 独自のスライバーウィジェットを利用する事ができる。
        * ListViewはCustomScrollView.sliversプロパティに1つのSliverListウィジェットを持つCustomScrollViewと考える事ができる。

* ListViewのビルド・レイアウト・マウント処理における実装上の要点は下記となる。
    * ListViewはSliverListを子として利用しており、SliverListはListViewからSliverChildDelegateを受け取る。
    * ListViewが渡すSliverChildDelegateは指定のインデックスに対しウィジェットを返すビルダーの役割を持つ。
        * SliverChildDelegateの具象はListViewの無名コンストラクタで指定されるchildrenにインデックスでアクセスするか、もしくはListView.builderの場合はitemBuilderをそのまま利用するか、どちらかになる。
    * RenderSliverListは、レイアウトフェーズ(performLayout)にてSliverMultiBoxAdaptorElementを経由して、SliverChildDelegateを呼び出す事で子ウィジェットを生成する。
        * 領域内の子のみ生成される。
        * 領域内から領域外となった子は破棄（アンマウント）される。
    * RenderSliverList.performLayoutは最後にmax scroll offsetの算出を行う。
        * これがScrollPostion.maxScrollExtentで取得できる値の基となる。
    * ListView(ScrollView)はScrollableStateを生成し、そのStateクラスであるScrollableStateのbuild内でViewportが生成される。
        * ScrollableStateがScrollPostionの情報を保有しており、ViewportはそれをViewportOffsetクラスという抽象として生成時に受け取る。
        * RenderViewportは、RenderSliverを保持している。
    * レンダリングフェーズにおいて、RenderViewport.performLayout()は以下を実行する
        1. ViewportOffset.applyViewportDimension()にてViewport自身のサイズを確立する。
            * https://api.flutter.dev/flutter/widgets/ScrollPosition/applyViewportDimension.html
        2. レイアウト処理を行う(_attemptLayout())
            * この処理の中で子のレイアウト処理を行う（RenderViewportBase.layoutChildSequence())
            * 子のRenderSliver.layout()を実行する。
            * ListViewの場合はRenderSliverの具象はRenderSliverListであり、ここでRenderSliverList.performLayoutが実行される。
        3. 上記のレイアウト処理によってオフセットの修正が発生した場合は補正を行う。
        4. レイアウト処理で取得した寸法情報からmaxScrollExtent, minScrollExtentを算出して、ViewportOffset.applyContentDimensions()にてコンテンツの表示領域のサイズを確立する。
            * https://api.flutter.dev/flutter/widgets/ScrollPosition/applyContentDimensions.html
            * この際、オフセットの補正が必要となる場合があり、その場合は1.に戻る。(補正が不要になるか回数制限を超過するまで繰り返し実行される)

* ListView.buildで指定される子は、ビルドフェーズではなくレイアウトフェーズで構築される。
    * 上記の説明の通り、SliverList.performLayoutの中でウィジェットおよびそのエレメント、レンダーオブジェクトが構築されている事がわかる。    
    * 階層図
        * 左図のウィジェットツリーのViewportまではビルドフェーズで生成され、SliverPadding以降がperformLayoutの際に生成される。
        * 右図のレンダーツリーのRenderSliverListまではビルドフェーズで生成され、RenderIndexedSemantics以降がperformLayoutの際に生成される。

* ListView.builderの場合のmaxScrollExtent
    * itemCountが指定されていない場合、スクロールが終端要素に到達するまではmaxScrollExtentはisInfiniteとなる。
        * 終端要素はitemBuilderでnullを返す箇所。これがキャッシュ領域も含めた領域内に入るとisInfiniteが実際の値に変わる。
    * 無名コンストラクタの場合は、childrenとして渡したウィジェットのリストのlengthが利用される。
        * コードとしては、SliverChildListDelegate.estimatedChildCount()が該当する。
    
* キャッシュ領域
    * https://api.flutter.dev/flutter/widgets/ScrollView/cacheExtent.html
        ```
        The total extent, which the viewport will try to cover with children, is cacheExtent before the leading edge + extent of the main axis + cacheExtent after the trailing edge.
        ```
    * ScrollViewで指定されるcacheExtentは、viewportの画面外の項目を何pixelをキャッシュ領域するか、という値となる。
        * デフォルト値はRenderViewportBaseのコンストラクタで250.0として設定されている。
    * RenderSliverListはViewportの高さ+cacheExtent*2の範囲内の子のみマウント・レイアウト・レンダリングを行う。
    * 範囲内であれば画面外の子でもマウントおよびレイアウト処理が行われる。

* (参考)ViewportsクラスとSliverListsについてレンダーツリーの構築の処理の流れをDartコードで書いたもの
    * https://gist.github.com/megur0/8b2eea0d377f5f4c929536025d794aac

* (参考)ScrollPosition.applyContentDimensions()
    * https://api.flutter.dev/flutter/rendering/ViewportOffset/applyContentDimensions.html
    * スクロールの表示領域のextentを設定することができるものの、以下のような仕様を理解して使う必要がある。
        1. このAPIは、RenderViewport.performLayout()から呼ばれている。（再レイアウトのたびに実行）
        2. レイアウトフェーズ（performLayoutの実行中）で呼ばないとエラーとなる関数を内部で呼んでいる。
            * ScrollPosition.applyContentDimensions ->  ScrollPositionWithSingleContext.applyNewDimensions -> context.setCanDrag -> replaceGestureRecognizers
            ```
            void replaceGestureRecognizers(Map<Type, GestureRecognizerFactory> gestures) {
                assert(() {
                if (!context.findRenderObject()!.owner!.debugDoingLayout) {
                    throw FlutterError.fromParts(<DiagnosticsNode>[
                    ErrorSummary('Unexpected call to replaceGestureRecognizers() method of RawGestureDetectorState.'),
                    ErrorDescription('The replaceGestureRecognizers() method can only be called during the layout phase.'),
                    // ...
                }
                return true;
                }());
                // ...
            }
            ```
        3. applyContentDimensionsの結果によってスクロール オフセットが変更が必要となる場合は再度レイアウトが必要になる。
    * 上記の仕様のため、build関数内（ビルドフェーズ）で呼ぶことは通常無い。
    * ScrollPositionを拡張したクラスを作成する場合等は、利用する可能性がある。したがって通常のアプリケーション開発ではほとんど無いと考えられる。

* (参考)ScrollViewの子がリビルドされると、親もリビルドされる。
    * RenderObject.markNeedLayoutでは RenderObject._relayoutBoundary が自身では無い場合(親のRenderObjectが設定されている場合)markParentNeedsLayout()が呼ばれ、親のRenderViewportもdirtyになると思われる。
        * 実際には RenderSliverList -> RenderPadding -> RenderViewport、といった伝搬によってdirtyになると考えられる。


## (参考)最大スクロールオフセットの算出の実装
* ScrollViewで利用しているSliverListのRenderObjectであるRenderSliverListは、キャッシュ領域にある子の寸法から１つあたりの平均値を算出し、スクロールの最大のオフセットを「現在のキャッシュ領域の末端 + 領域外の子数 * 平均値」にて算出する。
* キャッシュ領域(cacheExtent)は、RenderViewport.performLayout()において下記のように算出されている。
    ```
    switch (cacheExtentStyle) {
      case CacheExtentStyle.pixel:// デフォルト値
        _calculatedCacheExtent = cacheExtent;
      case CacheExtentStyle.viewport:
        _calculatedCacheExtent = mainAxisExtent * _cacheExtent;
    }

    final double fullCacheExtent = mainAxisExtent + 2 * _calculatedCacheExtent!;
    ```
* この値は加工をした上で、子であるRenderSliver.layout()を実行する際に制約としてSliverConstraintsオブジェクトにして渡される。
* RenderSliverList.performLayout()の最後では SliverGeometry を決定している。
    ```
    geometry = SliverGeometry(
      scrollExtent: estimatedMaxScrollOffset,
      paintExtent: paintExtent,
      cacheExtent: cacheExtent,
      maxPaintExtent: estimatedMaxScrollOffset,
      // Conservative to avoid flickering away the clip during scroll.
      hasVisualOverflow: (targetLastIndexForPaint != null && lastIndex >= targetLastIndexForPaint)
        || constraints.scrollOffset > 0.0,
    );
    ```
* このSliverGeometryはRenderViewport側（正確にはRenderPaddingを間に経由する）で_maxScrollExtent（この値はビューポート自体の高さも含む）を算出するために使われている。
    ```
    final SliverGeometry childLayoutGeometry = child.geometry!;
    // ...
    updateOutOfBandData(growthDirection, childLayoutGeometry);
    // ...
    void updateOutOfBandData(GrowthDirection growthDirection, SliverGeometry childLayoutGeometry) {
        switch (growthDirection) {
        case GrowthDirection.forward:
            _maxScrollExtent += childLayoutGeometry.scrollExtent;
        case GrowthDirection.reverse:
            _minScrollExtent -= childLayoutGeometry.scrollExtent;
        }
        if (childLayoutGeometry.hasVisualOverflow) {
        _hasVisualOverflow = true;
        }
    }
    ```
* このSliverGeometry内の設定値の内容は下記の通り。
    * firstIndex（indexOf(firstChild!)）　
        * （レイアウト）範囲内にある一番最初の子のインデックス。
    * lastIndex（indexOf(lastChild!)）
        * 範囲内にある一番最後の子のインデックス。
    * leadingScrollOffset(childScrollOffset(firstChild!)) 
        * 範囲内にある一番最初の子のスクロールオフセット
    * trailingScrollOffset(endScrollOffset)  
        * endScrollOffsetは下記のように定義されている。childにはループ処理で子を順次入れていく
            * `double endScrollOffset = childScrollOffset(child)! + paintExtentOf(child);`
        * 現在の子のスクロールオフセット + その子のレイアウト上のエクステント(child.size.height or width)
            * これはオフセットが終端にたどり着いている場合は、実際に描画される部分のみのエクステントとなる。
    * estimatedMaxScrollOffset  
        * 最大のスクロールオフセットの計算値。
        * 終端にリーチしている場合: 終端にリーチしている場合は現在の子の末端
        * それ以外の場合
            * アイテム数が不明の場合: double.infinity
            * アイテム数が明らかな場合: 範囲の高さを項目数で割って項目の高さの平均値を算出し、trailingScrollOffset + 範囲外の項目数 * 平均高さ として算出。
        * コードは下記となる。
            ```
            // ...
            if (reachedEnd) { // 終端にリーチしている場合
                estimatedMaxScrollOffset = endScrollOffset;
            } else {
                estimatedMaxScrollOffset = childManager.estimateMaxScrollOffset(〜〜);
                // ...
            }
            // ...

            double estimateMaxScrollOffset(
                SliverConstraints? constraints, {〜〜}) {
                final int? childCount = estimatedChildCount;
                if (childCount == null) {
                    return double.infinity; // nullの場合はinfinityとなる。
                }
                return 〜〜 _extrapolateMaxScrollOffset(〜〜);
            }

            static double _extrapolateMaxScrollOffset(
                int firstIndex,
                int lastIndex,
                double leadingScrollOffset,
                double trailingScrollOffset,
                int childCount,
            ) {
                if (lastIndex == childCount - 1) {
                    return trailingScrollOffset;
                }
                final int reifiedCount = lastIndex - firstIndex + 1;
                final double averageExtent = (trailingScrollOffset - leadingScrollOffset) / reifiedCount;
                final int remainingCount = childCount - lastIndex - 1;
                return trailingScrollOffset + averageExtent * remainingCount;
            }
            ```
        * この値を毎回算出しているため、アイテムの高さが均一ではない場合は、全体の高さが現在の領域内の子によって変動してしまう。
            * すべてのアイテムが同じ大きさの項目の場合は変動しない。
            * 一方、項目によって大きさが違う場合は現在の範囲内の項目の平均高さが変わるため、算出された全体の高さも変わってしまう。
        * 関連する公式のissuer
            * https://github.com/flutter/flutter/issues/97676
    * paintExtent 
        * paint対象の範囲?
    * cacheExtent
        * この値はScrollViewで指定されるcacheExtentとは別物
        * これはこのperfromLayoutを実行したレンダーオブジェクトの実際のキャッシュ範囲のように考えられたが、理解できていない。
        * なお、RenderSliver.calculateCacheOffset()で値が算出されており、trailingScrollOffset(endScrollOffset) - eadingScrollOffset(childScrollOffset(firstChild!)) の値とconstraintsをベースに算出しているようだった。


# Scrollbar
* 以下はスクロールバーに関連するクラスと処理の流れを表した図となる。<br/><br/>
![](./svg/scroll_code_reading/flutter_scroll_bar_class.drawio.svg)

<br/>

* 以下はRawScrollbarState.build()によって生成されるウィジェットの階層を表した図となる。<br/><br/>

<img src="./svg/scroll_code_reading/flutter_scroll_bar_build_widget.drawio.svg" width="70%">

<br/>


* ScrollPostionでは、スクロールのオフセットが更新される際にScrollNotificationを送信してウィジェットツリーへ伝搬させる。
* RawScrollbarStateはこのNotificationオブジェクトを監視して、その内容によってスクロールバーの描画情報（ScrollbarPainter）を更新してmarkNeedsPaintを呼び再描画する
* また、スクロールバーのThumb(つまみ)をドラッグする事でScrollPostionのオフセットが更新されるが、これはRawScrollbarStateがScrollPosition.jump()を呼ぶ事で更新を行っている。
    * ScrollPostion.jumpTo()（実際にはその具象クラスのjumpTo()）によって
        * ScrollPosition.pixelsがアップデートされてmarkNeedsLayoutが呼ばれ次回フレームで再レイアウト・レンダリングされる。
        * ScrollNotificationが伝搬されてRawScrollbarStateが受け取り次回フレームでレンダーされる。

* CupertinoApp, MaterialAppでは、実行環境がlinux, macOS, windowsの場合は明示的に宣言しなくてもスクロールバーが表示される。
    * 下記ではScrollbarウィジェットを指定していないが、上記のOS上で実行するとスクロールバーが表示される事が確認できる。(上記以外は表示されない。)
        ```
        main() => runApp(MaterialApp(
        home: ListView.builder(
            itemCount: 100, itemBuilder: (_, int i) => Text("$i")),
        ));
        ```
    * これはMaterialScrollBehavior, CupertinoScrollBehaviorのbuildScrollbar()によってScrollBarウィジェットが挿入されているためである。
    * もちろん、上記以外のOSでScrollbarのchildにListViewを置くことで、Scrollbarを表示することができる。
        ```
            main() => runApp(MaterialApp(
            home: Scrollbar(
                child: ListView.builder(
                    itemCount: 100, itemBuilder: (_, int i) => Text("$i")))));
        ```
        * また、上記のOSでScrollbarを明示的に指定した場合でも重複してスクロールバーが表示されることはない。
        * これは実行時のウィジェット生成はScrollableState.build()からbuildScrollbar()を実行することで行われ、利用するScrollBehaviorオブジェクトは先祖の最も近い1つのみを利用するためである。
 
* maxScrollExtentがisInfiniteの場合は、スクロールバーは非表示となる。
    * ソースコード上では下記のように分岐処理されている。
    ```
    class ScrollbarPainter extends ChangeNotifier implements CustomPainter {
        //...
        @override
        void paint(Canvas canvas, Size size) {
            //...
            if (_lastMetrics!.maxScrollExtent.isInfinite) {
                return;
            }
            // ...
        }
        // ...
    ```
    

## (参考)その他
* AppBarはスクロールを開始位置から動かすと背景色が透過するが、これはAppBar内部で下記のように処理されている。

    ![](./svg/scroll_code_reading/flutter_scroll_app_bar.drawio.svg)

* Notification
    * https://api.flutter.dev/flutter/widgets/Notification-class.html
    * ウィジェットツリー上で伝播させることができるクラス
    * NotificationListenerを使う事でリッスンすることができる。
    * 通知の区別やパラメータを持たせるにはNotificationクラスの派生クラスを作成する。
    * NotifiableElementMixin
        * https://api.flutter.dev/flutter/widgets/NotifiableElementMixin-mixin.html
        * https://api.flutter.dev/flutter/widgets/NotifiableElementMixin/onNotification.html
        > Mixin this class to allow receiving Notification objects dispatched by child elements.
        > Return true to cancel the notification bubbling. Return false to allow the notification to continue to be dispatched to further ancestors.
        * 伝播中の処理や伝播を条件によってキャンセルさせたい際に利用すると考えられる。
        * Flutter Framework上で"NotifiableElementMixin"を検索すると、使っている箇所は以下の4つのElementだった。
            * _NotificationElement
                * こちらはNotificationListener(ProxyWidget派生クラス)によってcreateElementされる。
            * _ViewportElement
            * _SingleChildViewportElement
            * _TwoDimensionalViewportElement

* MouseRegion
    * https://api.flutter.dev/flutter/widgets/MouseRegion-class.html
    > MouseRegion is used when it is needed to compare the list of objects that a mouse pointer is hovering over between this frame and the last frame. This means entering events, exiting events, and mouse cursors. 


# スクロール関連のIssueと考察
* ListViewのアイテムが非常に多い場合、スクロールバーを手動で一気に動かしたりanimateToで大きく移動させると著しく動作が遅くなる事由
    * これはSliverList（RenderSliverList）がレイアウトの際にレンダーツリーの生成を行っていることが原因と考えられる。
        * 通常のウィジェットはビルドの際にレンダーツリーが構築されて、レイアウト処理とは分けられている。
            * したがってレイアウトのみが変わった場合は通常はリビルド処理は行われず、ツリーの再構築も行われない。
        * スクロールバーを移動することで、markNeedsLayoutが呼ばれ次フレームでレイアウトされるときにレンダリングツリーが生成される。スクロールバーを一気に移動すると連続したフレームでこの生成処理が発生して動作が遅くなっていると考えられる。
        * なお、issue内でrecommendDeferredLoadingForContextプロパティでの対策が触れられているが、筆者は利用方法がわからなかった。
    * 公式のissue
        * https://github.com/flutter/flutter/issues/75399
        * https://github.com/flutter/flutter/issues/52207

* ListViewでスクロールするたびに ScrollPosition.maxScrollExtent（スクロールのオフセットの最大値）が変化する理由
    * 例えば下記のようなコードのprint出力でmaxScrollExtentを確認すると、スクロールするたびに値が変動することがわかる。
        * 動作するコード: https://gist.github.com/megur0/27a6d1cd69fa79c79738846f8fe96298 
        ```
        ScrollController? _scrollController;
        @override
        void initState() {
            super.initState();
            _scrollController = ScrollController();
            _scrollController!.addListener(() {
            debugPrint(
                "offset:${_scrollController!.position.pixels},  minScrollExtent:${_scrollController!.position.minScrollExtent}, maxScrollExtent: ${_scrollController!.position.maxScrollExtent}, viewportDimension: ${_scrollController!.position.viewportDimension}");
            });
        }

        @override
        Widget build(BuildContext context) => Scaffold(
                body: ListView(
                controller: _scrollController,
                children: List.generate(widget.totalNumberOfData, (i) => i)
                    .map<Widget>((e) => Text("item $e ${'\n' * Random().nextInt(5)}"))
                    .toList(),
                ),
            );
        ```
    * これはListViewが表示領域(+キャッシュ領域)のウィジェットのみ構築しているため、領域外のウィジェットの情報は知り得ず、領域内の子のサイズからmaxScrollExtentを算出しているためである。
        * したがって、上記のようにmaxScrollExtentは現在の領域の子の寸法によって変わる。
        * すべての要素が同じサイズであれば算出した結果は常に同じ値となり正確な寸法となる。

* maxScrollExtentが変動することによるスクロールバーへの影響
    * 下記のコードでは、スクロールバーを動かすたびにスクロールバーが上下に揺れて不安定となることが確認できる。
    ```
    @override
    Widget build(BuildContext context) {
        return Scaffold(
        body: Scrollbar(
            child: ListView(
                children: List.generate(300, (i) => i)
                .map<Widget>((e) => Text("item $e ${'\n' * Random().nextInt(5)}"))
                .toList(),
        )));
    }
    ```
    * 動作コード
        * https://gist.github.com/megur0/b4679216391382a6e443cc84ffe333aa
    * 公式issue
        * https://github.com/flutter/flutter/issues/25652

* maxScrollExtentの変動を回避するための手段(2023/7時点)
    * 以下のような手段はあるものの、アイテムの大きさが固定でない場合は、基本的にはmaxScrollExtentが変動することは許容する方向で検討する事が一番現実的かもしれない。
    * ListView.itemExtent
        * スクロール内の各アイテムが固定の高さになる。
        * この場合はmaxScrollExtentは変動しない。
    * ListView.prototypeItem
        * itemExtentと似ているが、これはウィジェットを指定してそのウィジェットと同じextentを子に強制する。
        * この場合もmaxScrollExtentは変動しない。
    * ListView.cacheExtentを大きくしていく
        * アイテムの個数に合わせてcacheExtentを大きくしていく。
        * ただし、この手段はListViewを使うメリットが無くなってしまうかもしれない。
    * SingleChildScrollViewを利用する。
        * SingleChildScrollViewは渡した単一の子ウィジェットを渡すため、ListViewのように部分的なElment・RenderObjectの生成ではなく、通常のウィジェットと同様にすべて生成される。
        * 大量のアイテムが存在した際は全てを生成するためボトルネックとなる可能性がある。
    * 事前にmaxScrollExtentを算出して利用する。
        * これはサードパッケージの利用や、あるいは自前でクラスを拡張して実装する等の手段が考えられる。
        * アイテムの個数が非常に多い場合は算出処理がボトルネックとなる場合があるかもしれない。
    * 参考
        * https://github.com/flutter/flutter/issues/25652
        * https://github.com/flutter/flutter/issues/22314#issuecomment-615308850
    
*  (参考) maxScrollExtentの事前算出を独自クラスで行うサンプルプログラム
    * https://gist.github.com/megur0/81edb7a49216b8998fe0802f63da4cd9
    * maxScrollExtentを事前計算してそれをScrollController経由で設定するようにした
        * これによってスクロールを動かすたびにmaxScrollExtentが変わることなく、正確な値を利用できるようになる。
    * 手動でレンダリングツリーを構築してmaxScrollExtentを算出
        * createElement()からmount()まで行ってツリーを生成する。
        * RenderFlex.computeDryLayoutを使ってSizeを取得する。
        * 不要となったツリーをunmountする
    * ScrollControllerを拡張した独自クラスで、createScrollPositionをオーバーライドして独自のScrollPositionクラスを返す。
    * ScrollPositionを拡張した独自クラスで、applyContentDimensionsをオーバーライドして事前に算出したmaxScrollExtentを利用するように上書きをする。

* 「Write your first app」に無限にスクロールするリストがあるが、そのリストにスクロールバーがない
    * https://github.com/flutter/flutter/issues/41434
        * ※ 「Write your first app」の内容はIssue作成時から変わっていて、本メモの作成時は既に題材には無限にスクロールするリストは無かった。
    * 現行、maxScrollExtentがisInfiniteの場合にスクロールバーがpaintされない実装となっているためこれは仕様となる。
    * これに関するPRは以下のものがあったが、いずれもクローズされておりマージはされていない。
        * https://github.com/flutter/flutter/pull/96769
            * ScrollbarPainter、RawScrollbar/RawScrollStateに関する修正がされており、paintの際に、_totalContentExtent関数の計算において、metrics.maxScrollExtent（これはScrollPositonからScrollNotificationのdispatchで送られる。）がisInfiniteの場合の分岐を入れて計算処理を追加している。
        * https://github.com/flutter/flutter/pull/96825
            * ThemeDataにinfiniteBehaviorというプロパティを入れ、その設定値によって無限スクロールのスクロールバーの動作の処理を変更するもの
            * なお、ここで扱われる「無限スクロール(InfiniteScroll)」は、maxScrollExtent.isInfiniteがtrue（double.infinity)となるものと考えられる。
            * 提案されている項目は下記となる。
                * InfiniteScrollBehavior.page
                    * これはthumbがスクロールバーの終わりの位置に来たら追加ロードする方式。
                * InfiniteScrollBehavior.continuous
                    * pageと似ているが、これはスクロールバーの終わりに来る前にロードされ、常にthumbの場所が定位置となる。
                * InfiniteScrollBehavior.none
                    * スクロールバーを非表示にする。
                    * 現行がこの動作となる。
                * https://docs.google.com/document/d/1TV2oF2iLIZtGd48336korxMTQrGizDAASAeZAeteMSc
            * 実装は、ScrollbarPainter.paintのthumbのポジションの算出処理と、thumbをドラッグした際のScrllPositonをjumpToするオフセット値の算出処理を、InfiniteScrollBehaviorによって分岐させている。

* ネットワークイメージがロードされた際にスクロール位置がずれてしまう問題
    * https://github.com/flutter/flutter/issues/99158
    * 現状、回避策は無いと考えられるが、動的なロードが必要なアイテムは可能であればサイズを固定にする方が良いと考えられる。
        * 例えばImage.networkのようなウィジェットを利用をする際ローディング中は固定サイズの画像を表示しロード完了したら同じサイズで画像を表示する方法が考えられる。

* PageScrollPhysicsに関する不具合
    * https://github.com/flutter/flutter/issues/35687

