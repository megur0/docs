[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > permission_handler


# 注記
* メモにはiOSの内容のみ記載する
* 扱っているpermission_handlerのバージョンは11.3.1となる。

# permissino_handlerは権限の設定状況の確認とそれに応じたハンドリングを行うために利用
* 実行時にユーザーに対して権限のリクエストを行うプラグインは多い。
* しかし、これらのプラグインはユーザーが許可をすれば、正常に動作するが、拒否した状態の場合はエラーを返したり、何も実行しないといったそれぞれの振る舞いをする。
* 一度拒否した場合は、ユーザーが端末の設定から対象のアプリに対しての許可設定をユーザー自身で行う必要がある。
* そういった操作が必要なことを画面上でユーザーへガイダンスする方がユーザーにとっては分かりやすいケースも多い
* そのためには各プラグインを実行する手前で権限状態を確認してハンドリングする必要がある。この用途でpermissino_handlerが有用となる。
* permissino_handlerは権限のリクエストを行う機能も備えているため、リクエストもまとめて行なってしまっても良いだろう。

# ドキュメント
* https://pub.dev/packages/permission_handler
* セットアップ
    * https://pub.dev/packages/permission_handler#setup
    * ドキュメントに従って、アプリで権限を取得したい項目を以下に設定する
        * マクロの定義(ios/Podfile)
            * 不要な項目はコメントアウトする
        * Info.plist(ios/Runner/Info.plist)

# (参考)マクロをコメントアウトすると常に拒否、無効が返される。
* 例えばPERMISSION_APP_TRACKING_TRANSPARENCYであれば、下記のように値が定義されていない場合は0が設定される。
```
//ios/.symlinks/plugins/permission_handler_apple/ios/Classes/PermissionHandlerEnums.h
#ifndef PERMISSION_APP_TRACKING_TRANSPARENCY
    #define PERMISSION_APP_TRACKING_TRANSPARENCY 0
#endif
```
* そして、下記のように1の場合と0の場合で分岐していて、0の場合はUnknownPermissionStrategyが利用される。
* UnknownPermissionStrategyは、常にPermissionStatusDenied, PermissionStatusPermanentlyDenied, ServiceStatusDisabled を返すように実装されている。
```
// ios/.symlinks/plugins/permission_handler_apple/ios/Classes/strategies/AppTrackingTransparencyPermissionStrategy.h
#if PERMISSION_APP_TRACKING_TRANSPARENCY

#import <AppTrackingTransparency/AppTrackingTransparency.h>

@interface AppTrackingTransparencyPermissionStrategy : NSObject <PermissionStrategy>
@end

#else

#import "UnknownPermissionStrategy.h"

@interface AppTrackingTransparencyPermissionStrategy : UnknownPermissionStrategy
@end

#endif
```

# Info.plist（全項目について載せた雛形）
* https://github.com/Baseflow/flutter-permission-handler/blob/main/permission_handler/example/ios/Runner/Info.plist
* (参考)PermissionGroupNotificationが無いが無視して良い。
    * セットアップ手順を確認すると、PermissionGroup.notificationについて、Info.plistに該当するものが"PermissionGroupNotification"ということが表に記載されている。
        * https://pub.dev/packages/permission_handler#setup
    * 以下のissueを確認すると、同様の内容が質問されている。
        * https://github.com/Baseflow/flutter-permission-handler/issues/842
        * 結論(?)は通知に関してはInfo.plistへの記載は不要。
            * https://github.com/Baseflow/flutter-permission-handler/issues/842#issuecomment-1156644740
        * また、"PermissionGroupNotification"というキーワード自体が検索してもFlutterのpermission_handlerのトピックでのみヒットするため、iOS自体のkeyではないと考えられる。


# コードで確認する
* 下記のサンプルコードのPermissionHandlerWidgetをコピーして、自身のアプリケーションで出力すると権限の状況が把握できる。
* https://pub.dev/packages/permission_handler/example
* 許可した覚えのない項目でgrantedとなっている箇所は、デフォルトで有効になっている項目や、iOSの場合はpermisssion_handlerが常にgrantedを返している項目になる。
* 許可のリクエストが未済の項目はdeniedとなっている。
* マクロ上でコメントアウトした項目は(押下して)リクエストを行ってもpermanentlyDeniedが返る
* リクエストに対してユーザーが拒否をした場合は、permanentlyDeniedが返る
* PermissionWithService(Permissionのサブクラス)
    * Permission.locationなどはこのクラスとなっている。
    * これは例えばロケーションの場合は、アプリに対しての許可とは別に、位置情報サービス自体が設定で有効化されている必要があり、そういったサービスを利用する項目がこのクラスに該当するようだ。
    * なおマクロ上でコメントアウトした項目は、有効化状態の確認の際は常にdisabledが返る
## (参考)iOS上ではどの項目に対応するのか？
* 一覧の中でいくつかの項目について、iOSでどの項目に対応するのかわからなかったため、コードを直接確認した。
* 各Permissionごとの分岐
    ```objectivec
    // ios/.symlinks/plugins/permission_handler_apple/ios/Classes/PermissionManager.m
    + (id)createPermissionStrategy:(PermissionGroup)permission {
        switch (permission) {
            case PermissionGroupCalendar:
            case PermissionGroupCalendarWriteOnly:
            case PermissionGroupCalendarFullAccess:
                return [EventPermissionStrategy new];
            //...
    }
    ```
    * 上記では、それぞれのcaseで　NSObject<PermissionStrategy>を 実装したクラスを返している。
    * したがって、そのクラスを確認すれば実装内容が分かる。
* PermissionGroupBackgroundRefresh
    * UIApplication.sharedApplication.backgroundRefreshStatus(Appのバックグラウンド更新)を参照しているようだ。
    ```objectivec
    // ios/.symlinks/plugins/permission_handler_apple/ios/Classes/strategies/BackgroundRefreshStrategy.m
    + (PermissionStatus) permissionStatus {
        UIBackgroundRefreshStatus status = UIApplication.sharedApplication.backgroundRefreshStatus;
        switch (status) {
            case UIBackgroundRefreshStatusDenied:
                return PermissionStatusDenied;
            case UIBackgroundRefreshStatusRestricted:
                return PermissionStatusRestricted;
            case UIBackgroundRefreshStatusAvailable:
                return PermissionStatusGranted;
            default:
                return PermissionStatusDenied;
        }
    }
    ```
* PermissionGroupStorage
    * 常にGrantedを返しているようである。
    ```objectivec
    // ios/.symlinks/plugins/permission_handler_apple/ios/Classes/strategies/StoragePermissionStrategy.m
    + (PermissionStatus)permissionStatus {
        return PermissionStatusGranted;
    }
    ```
* PermissionGroupNotification
    * ios/.symlinks/plugins/permission_handler_apple/ios/Classes/strategies/NotificationPermissionStrategy.m
    * UNUserNotificationCenter.currentNotificationCenterから取得している。

