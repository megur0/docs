[TOP(About this memo))](../README.md) > [一覧(Flutter)](./README.md) > 画像・アセット


# 公式ドキュメント
* https://docs.flutter.dev/ui/assets/assets-and-images



# 解像度を考慮した画像アセット
* https://docs.flutter.dev/ui/assets/assets-and-images#resolution-aware
* Flutter は、現在のデバイスのピクセル比に適した解像度の画像を読み込むことができる
* devicePixelRatio
    * https://api.flutter.dev/flutter/dart-ui/FlutterView/devicePixelRatio.html
    > The number of device pixels for each logical pixel for the screen this view is displayed on.
    * 各論理ピクセルのデバイスピクセル数
    ```dart
    final mediaQuery = MediaQuery.of(context);
    print("height:${mediaQuery.size.height}, width:${mediaQuery.size.width} devicePixceRatio:${mediaQuery.devicePixelRatio}");
    ```
* ある画像について、解像度別に規則に従ってフォルダに入れておくことで自動でFlutterが最も近い画像を選定する。
* 具体例
    * devicePixelRatio
        * 3.0
    * デバイスのピクセル解像度
        * 2,532 x 1,170
    * MediaQuery.of(context).size（論理的な画面の大きさ）
        * height:844.0, width:390.0
    * 画像フォルダ
        * assets/a.png
            * 100x100pixelの画像
        * assets/2.0x/a.png
            * 200x200pixelの画像
        * assets/3.0x/a.png 
            * 300x300pixelの画像
    * 表示
        * 論理的な画像の大きさ: 100x100pixel
        * デバイスへ表示される画像: assets/3.0x/a.png
        * デバイスに表示される大きさ：300x300pixel


# 起動画面のカスタマイズ
* https://docs.flutter.dev/ui/assets/assets-and-images#ios-1
* https://docs.flutter.dev/platform-integration/ios/launch-screen