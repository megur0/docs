[TOP(About this memo))](../README.md) > Google AdMob


# WIP: 随時更新
* このメモは執筆中のため随時更新 

# 注意:iOS/Flutterの情報を記載
* 筆者は当該環境で開発しているため、このメモは:iOS/Flutterの情報のみ記載

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

# テスト広告(デモ広告)・テストデバイスの設定
* 各デモ広告ユニット
    * https://developers.google.com/admob/android/test-ads?hl=ja#demo_ad_units
* テストデバイスの登録
    * https://developers.google.com/admob/android/test-ads?hl=ja#enable_test_devices
    * 注意
        * アプリがIDFAを送信する必要がある。これはアプリからのATTの許可を開発者が行っている必要ががある。
    * iOSのIDFAを取得する
        * iOSは、デフォルトでIDFAを取得する機能はない。
        * 筆者はApp Storeで検索して表示されるアプリを利用した

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
* https://developers.google.com/admob/ios/quick-start?hl=ja
 * https://codelabs.developers.google.com/codelabs/admob-ads-in-flutter?hl=ja#0
## AdMobの禁止事項
* https://support.google.com/admob/answer/6275345?hl=ja
* いずれも、"偶発的クリック" を招くものは禁止
    * インタラクティブな要素の近くにある広告
    * アプリアイテムの間に挟まれた広告
    * アプリ コンテンツに重なった広告
## (任意)IDFAへのアクセス許可
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

