- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)



# 公式ドキュメント
* https://docs.flutter.dev/testing/errors
* https://api.flutter.dev/flutter/widgets/ErrorWidget-class.html


# エラーハンドラー

* Flutterではグローバルなエラーをハンドリングする以下のAPIが用意されている。

|No|ハンドリングするエラーの種類|ハンドリングするためのAPI|
|-|-|-|
|1|フレームワークの処理によるエラー|FlutterError.onError|
|2|フレームワーク外のエラー|PlatformDispatcher.instance.onError|
|3 *1|レンダリング失敗時<br/>※デフォルトの挙動をカスタマイズ|ErrorWidget.builderで 挙動をカスタマイズ|
* ErrorWidget.builderのデフォルトの挙動
    * デフォルトでは全画面表示でエラーが表示される。
    * 画面はえんじ色、エラー内容が黄色文字で表示される
    * (IMO)基本的にはErrorWidget.builderはリリースモードではエラー内容は表示しないようにカスタマイズしておく方が良いと考えられる
* サンプルコード
```
void main() {
    ErrorWidget.builder = (FlutterErrorDetails details) {
        if (kDebugMode) {
            return ErrorWidget(details.exception);
        }
        // エラーレポート送信等の処理
        // ... 
        return Container(
            alignment: Alignment.center,
            child: Text(
                'Something error happened!',
                style: const TextStyle(color: Colors.yellow),
                textAlign: TextAlign.center,
                textDirection: TextDirection.ltr,
            ),
        );
    };
    FlutterError.onError = (errorDetails) {
        debugPrint("Flutter framework error occured: $errorDetails");
        // エラーレポート送信等の処理
        // ...
    };

    PlatformDispatcher.instance.onError = (error, stack) {
        debugPrint("Error occured: $error");
        // エラーレポート送信等の処理
        // ...
        return true;
    };
    // ...
}
```

# (参考)ErrorWidget.builderの内部処理
* 内部ではdart:uiのParagraphを使ってpaintされていた。
```
class ErrorWidget extends LeafRenderObjectWidget {
    // ...
    RenderBox createRenderObject(BuildContext context) => RenderErrorBox(message);
    static ErrorWidgetBuilder builder = _defaultErrorWidgetBuilder;
    static Widget _defaultErrorWidgetBuilder(FlutterErrorDetails details) {
        // ...
        final Object exception = details.exception;
        return ErrorWidget.withDetails(message: message, error: exception is FlutterError ? exception : null);
        // ...
    }
    // ...
}
```
```
class RenderErrorBox extends RenderBox {
    // ...
    late final ui.Paragraph? _paragraph;
    void paint(PaintingContext context, Offset offset) {
        // ...
        context.canvas.drawParagraph(_paragraph, offset + Offset(left, top));
    }
    // ...
}
```
