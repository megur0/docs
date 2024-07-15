[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# dart:ioライブラリのHttpClientと httpパッケージ
* HttpClientはlowレベルのhttpのAPI
  * https://github.com/dart-lang/sdk/blob/ab2d19c93da6eaf14adc4d3161c8b19a5029cf8d/sdk/lib/io/io.dart#L63
  > The classes [HttpClient] and [HttpServer] provide low-level HTTPfunctionality.
* httpパッケージ
  * https://pub.dev/packages/http
  * dart.devによる公式パッケージ
* コード中には、開発者フレンドリーなhttpパッケージの利用の検討を勧めている
  > Instead of using these classes directly, consider using more developer-friendly and composable APIs found in packages
  > For HTTP clients, look at `package:http`.

# httpパッケージのRequestクラスによるcharsetの付与
* https://github.com/dart-lang/http/issues/184#issuecomment-511945225
* dart.devの公式パッケージであるhttpパッケージのRequestでは、content-typeに暗黙的にcharset=utf-8を付与する。
* 実装を確認すると下記のようになっている。
  ```
  // http-1.1.0/lib/src/request.dart
  class Request extends BaseRequest {
    //...
    Encoding get encoding {
      if (_contentType == null ||
          !_contentType!.parameters.containsKey('charset')) {
        return _defaultEncoding;
      }
      return requiredEncodingForCharset(_contentType!.parameters['charset']!);
    }
    //...
  }
  ```
* 上記は上書きは出来るが「charsetを指定しない」という選択はできない。
* 代替策として、httpパッケージではなくdart:ioライブラリのHttpClientを直接利用する手段が下記で紹介されている。
  * https://github.com/dart-lang/http/issues/184#issuecomment-673189476

