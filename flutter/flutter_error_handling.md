[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# 公式ドキュメント
* https://docs.flutter.dev/testing/errors
* https://api.flutter.dev/flutter/widgets/ErrorWidget-class.html


# エラーハンドラー

* Flutterではグローバルなエラーをハンドリングする以下のAPIが用意されている。

|No|ハンドリングするエラーの種類|ハンドリングするためのAPI|
|-|-|-|
|1|フレームワークの処理によるエラー|FlutterError.onError|
|2|フレームワーク外のエラー|PlatformDispatcher.instance.onError|
|3|レンダリング失敗時<br/>※デフォルトの挙動をカスタマイズ|ErrorWidget.builderで 挙動をカスタマイズ|
* ErrorWidget.builderのデフォルトの挙動
    * リリースモードの場合はエラーメッセージは出力されないようになっている。
    ```
    // flutter/lib/src/widgets/framework.dart
    static ErrorWidgetBuilder builder = _defaultErrorWidgetBuilder;
    //...
    static Widget _defaultErrorWidgetBuilder(FlutterErrorDetails details) {
        String message = '';
        assert(() {
            message = '${_stringify(details.exception)}\nSee also: https://flutter.dev/docs/testing/errors';
            return true;
        }());
        final Object exception = details.exception;
        return ErrorWidget.withDetails(message: message, error: exception is FlutterError ? exception : null);// 渡しているexceptionは インスペクタ表示用で画面に表示はされない。
    }
    ```
* ErrorWidget.builder の実行時には、FlutterError.onErrorも呼び出しされる。
* サンプルコード
```
void main() {
    // ErrorWidget.builderの実行時にはFlutterError.onErrorも呼ばれるため、この中でエラーレポートは不要。
    ErrorWidget.builder = (FlutterErrorDetails details) {
        if (kDebugMode) {
            return ErrorWidget(details.exception);
        }
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


