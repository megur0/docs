[TOP(About this memo))](../README.md) > Google AdMob


## 公式
* 公式のドキュメントはiOS, Android, Unity, C++, Flutterのそれぞれの別のURL体系でドキュメントが用意されている。
    * C++は2024 年 6 月 17 日をもってサポートが終了
* iOS
    * https://developers.google.com/admob/ios/quick-start?hl=ja
* Android
    * https://developers.google.com/admob/android/banner?hl=ja
* Flutter
    * https://developers.google.com/admob/flutter/banner?hl=ja#ios
    * https://codelabs.developers.google.com/codelabs/admob-ads-in-flutter?hl=ja#0
    * (IME) 日本語ページの翻訳が正確ではないため、英語ページを翻訳した方が良いかもしれない。


# はじめる
* AdMobのサイトからログイン（電話番号確認が必要）
    * https://admob.google.com/intl/ja/home/
* 支払い情報を登録する
* 審査が行われる
    * 筆者の場合、翌日には承認のメールが届いた

# アプリを登録する
* AdMobの管理画面上で登録を行う。
* 登録して発行されたApp IDは後述のInfo.plistへ設定する必要がある
* Storeへリリース前でも登録可能。

# テスト広告・テストデバイスの設定
* デモ広告(テスト広告、サンプル広告とも呼ばれている)
    * 理由は分からないが、それぞれのドキュメントで指定される広告IDが異なっている。
    * Android
        * https://developers.google.com/admob/android/test-ads?hl=ja
        * 各フォーマットごとにでも広告IDが用意されている。
    * iOS
        * ※ (24/10/22時点でこのページへアクセスするとサーバーエラーとなる。)
        * https://developers.google.com/admob/ios/test-ads?hl=ja
    * Flutter
        * https://developers.google.com/admob/flutter/test-ads?hl=ja
        * 各フォーマットごとではなくiOSとAndroidの２つのみで区分されている。
* テストデバイスの登録
    * https://developers.google.com/admob/android/test-ads?hl=ja#enable_test_devices
    * iOSのIDFAを取得して、AdMobの管理画面で登録する
        * iOSは、デフォルトでIDFAを取得する機能はない。
        * 筆者はApp Storeで検索して表示されるアプリを利用した
    * 注意
        * アプリがIDFAの情報を送信するためには、アプリがATTの許可をされている必要がる。
        * したがって、対象のテストデバイスでアプリをテストする際にATTの許可をした状態でテストする必要がある。
    

# （任意）AdMobを Firebaseにリンクする
* https://firebase.google.com/docs/admob/ios/quick-start?hl=ja

# Info.plistの設定
* https://developers.google.com/admob/ios/quick-start?hl=ja#update_your_infoplist
* AdMobのApp IDの設定
    * AdMobのApp IDをplistのGADApplicationIdentifierに設定する
* SKAdNetworkの設定
    * 対象のキーはGoogleのもの１つと、現時点(24/8/24)ではサードパーティのものをあわせて40程度が存在する。
        * https://developers.google.com/admob/ios/3p-skadnetworks?hl=ja
    * 筆者はGoogleのもの１つのみInfo.plistへ設定した。(特に動作上問題はなかった)

# コードの実装
* (参考)バナー広告の種類
    * https://support.google.com/admob/answer/9993556?hl=ja
    * 標準
        * 指定されたバナーサイズのみに適合
    * アダプティブ
        * アンカー
            * 常に画面の上部または下部に固定された状態で画面に表示されることが想定されたバナー
        * インライン
            * スクロール可能なコンテンツ内に配置される。
            * 通常、ユーザーがスクロールして通り過ぎる際に大きなサイズで表示される場合がある
    * スマート(非推奨)
        * 現在、サポートは終了。代わりにアダプティブ バナーを実装することが推奨されている。
        * 固定された高さで、デバイスの画面の全幅に表示される
* ※ 以降のリンクはFlutter用のドキュメントのリンク
* 固定サイズ・カスタムサイズ
    * https://developers.google.com/admob/flutter/banner?hl=ja
* アダプティブ
    * https://developers.google.com/admob/flutter/banner/anchored-adaptive
        > アダプティブバナーは、業界標準の320x50バナーサイズと、それを置き換えるスマートバナーフォーマットの両方をドロップインで置き換えることができるように設計されています。   
        > これらのバナーサイズは、通常、画面の上部または下部に固定されるアンカーバナーとして一般的に使用されます。  
        > このようなアンカーバナーの場合、アダプティブバナーを使用する際のアスペクト比は、以下の3つの例で見ることができるように、標準の320x50広告と同様になります：(以下画像の例が記載)   
        > アダプティブ バナーは、利用可能な画面サイズをより有効に活用します。さらに、スマート バナーと比較すると、アダプティブ バナーは次の理由からより優れた選択肢となります。
        >* 幅を全画面に強制するのではなく、指定した任意の幅を使用するため、iOS の安全領域を考慮し、Android で切り抜きを表示できます。
        >* さまざまなサイズのデバイス間で高さを一定にするのではなく、特定のデバイスに最適な高さを選択し、デバイスの断片化の影響を軽減します。
    * 固定サイズではなく、指定された幅に合わせて最適化された広告の高さでバナーのサイズを決定
    * アスペクト比が元の広告と同じとなる。
    * アンカーアダプティブバナー
        * https://developers.google.com/admob/flutter/banner/anchored-adaptive#when_to_use_adaptive_banners
        * アンカー付きアダプティブバナー
        * アンカーバナーの高さは、デバイスの高さの15％または90DP(density independent pixels)のいずれか小さい方より大きくなることはなく、50DPより小さくなることはない。
        * 特定のデバイス上の特定の幅に対して返されるサイズは常に同じ事が保証される。
        * 「アンカー」ではあるものの、(すくなくともFlutterのSDKでは)バナー自体がアンカー機能を有してはおらず、固定されるかどうかはその実装に依存する。
            * 例えば下記のようにしても追随はせず、
            ```dart
            ListView(
                children: [
                    if (_anchoredAdaptiveAd != null && _isLoaded)
                    Container(
                        color: Colors.green,
                        width: _anchoredAdaptiveAd!.size.width.toDouble(),
                        height: _anchoredAdaptiveAd!.size.height.toDouble(),
                        child: AdWidget(ad: _anchoredAdaptiveAd!),
                    ),
                    ...(List.generate(100, (i) => Text("$i")).toList())
                ],
            )
            ```
            * 下記のように実装することで下部に固定される
            ```dart
            Stack(
                alignment: AlignmentDirectional.bottomCenter,
                children: <Widget>[
                    ListView(
                    padding: const EdgeInsets.symmetric(horizontal: 16.0),
                    children: [...(List.generate(100, (i) => Text("$i")).toList())],
                    ),
                    if (_anchoredAdaptiveAd != null && _isLoaded)
                    Container(
                        color: Colors.green,
                        width: _anchoredAdaptiveAd!.size.width.toDouble(),
                        height: _anchoredAdaptiveAd!.size.height.toDouble(),
                        child: AdWidget(ad: _anchoredAdaptiveAd!),
                    ),
                ],
            )
            ```
    * インラインアダプティブバナー
        * https://developers.google.com/admob/flutter/banner/inline-adaptive
        * インライン アダプティブ バナーは、アンカー アダプティブ バナーに比べて大きくて背の高いバナーとなる。
        * 高さは可変で、デバイスの画面と同じ高さになることもある。(maxHeightを指定することも可能)

# AdMobの禁止事項
* https://support.google.com/admob/answer/6275345?hl=ja
* "偶発的クリック" を招くものは避ける
    * インタラクティブな要素の近くにある広告(非推奨)
    * アプリアイテムの間に挟まれた広告(非推奨)
    * アプリ コンテンツに重なった広告(ポリシー違反)

# (任意)IDFAへのアクセス許可
* https://support.google.com/admob/answer/9997589?hl=ja
    > Apple が 2020 年 6 月に発表した更新に伴い、アプリが Apple の広告配信用の識別子（IDFA）にアクセスする際はユーザーに許可を求めることが必須となりました。これには、App Tracking Transparency（ATT）フレームワークと呼ばれるプロンプトを利用します。
* ユーザーがIDFAへのアクセスを許可しない場合は、広告の単価が低下する。
* また、テストデバイスはIDFAでGoogleが認識するため、ATTが許可されていない場合はテストデバイスを登録しても実際には機能しない。

# app-ads.txtの配置
* iOSアプリは、app-ads.txtというテキストファイルをApp Store申請時のマーケティングURLのルートに配置する必要がある。
* App Store申請時にはこのマーケティングURLに忘れずに設定しておく必要がある。
    * 設定していない場合は、設定時に再申請（新しいビルド）が必要となる点に注意。    
* AppStoreのマーケティングURLに設定しておくことで、AdMob側もそれによって対象のアプリを認識する。

# (WIP)(AppStoreへ公開後)対象のアプリにストア情報を設定する
* AdMobの管理画面上でストアから検索できるようになるまでのタイムラグがある。(半月程度？)
* 登録後、審査が行われる（数日？）

