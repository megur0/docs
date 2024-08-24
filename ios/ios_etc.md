[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# WIP
* このメモは作成中
* Objective-C, UIKit, SwiftUI,周りや理解が乏しいため、取り急ぎ、分散した知識をここに追記していく。

# ユーザーに許可の設定状態の確認
## Appのバックグラウンド更新
* iOS7から利用できるようになった
* デフォルトでは有効だがユーザーが設定で無効化できる。
* UIApplication.sharedApplication.backgroundRefreshStatus から有効・無効状態を参照できる
## 通知
* 設定 > 対象のアプリ > 通知 以下の設定状況に該当する設定状況
* UNUserNotificationCenter.current().notificationSettings() から参照できる