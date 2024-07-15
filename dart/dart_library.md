[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# ライブラリ
* https://dart.dev/language/libraries
* Dartではライブラリという単位で構成を管理される。
* ライブラリ単位でスクリプトにインポートしたり、ライブラリ単位で名前空間を分ける。
* クラスや変数をプライベートにすることで外部から見えなくなるが、その単位がライブラリとなる。
* importを用いて読み込むことのできるものはすべてライブラリである。
* すべてのDartファイルは宣言しなくてもライブラリとなる。
  > Every Dart file (plus its parts) is a library, even if it doesn’t use a library directive.
* ライブラリレベルでドキュメントを書かないのであればlibraryは省略することが推奨されている。
  * https://dart.dev/guides/libraries/create-packages#what-makes-a-library-package
  > Note: When the library directive isn’t specified, a unique tag is generated for each library based on its path and filename. Therefore, we suggest that you omit the library directive from your code unless you plan to generate library-level documentation.


# (参考) Dartの名前空間名前空間について
* 比較対象: Go
  * Go言語の場合はimportしたパッケージはデフォルトでパッケージ名で名前空間が分けられる。（アクセスの際はパッケージ名.xxxとなる）
* importしたライブラリは別々の名前空間とはならない。
* 衝突する場合はimportの際にプレフィクスを指定する必要がある。

# エントリーポイント
* エントリーポイントはmain関数となる
* mainは引数指定可能。
  * `void main(List<String> arguments) {/* ... */}`

# import と part
* import
  * ライブラリを読み込むために使用。
  * publicのみ読み込む
* part
  * 一つのライブラリを複数のファイルに分割するために使用
  * publicとprivateのどちらも読み込む
* 公式ではpartとpart ofでのファイル分割よりも、個別のライブラリとして分割してimportでアクセスすることが推奨されている。
  * https://dart.dev/guides/libraries/create-packages#organizing-a-package
  
# import
* ビルトインのライブラリのURIはdart:からはじまる。
* identifiersが衝突する場合はprefixを使うためにasを使う。
* ライブラリの一部のみ使う
  ```
  // Import only foo.
  import 'package:lib1/lib1.dart' show foo;
  // Import all names EXCEPT foo.
  import 'package:lib2/lib2.dart' hide foo;
  ```

# export
* 外部に公開するライブラリを指定するために使用する。メインのライブラリにexportを記述することで、メインのライブラリを読み込むだけでexportで指定されたライブラリにもアクセスできる。
```
lib/my_library.dart
  export 'src/foo.dart';
  export 'src/bar.dart';
```

# プライベート
* _をつけることでファイル（ライブラリ）単位でプライベートになる。
  * クラス内のオブジェクトがプライベートでも、同じライブラリ内であればアクセスできるので注意。
  ```
  void main() => print(Y()._private);
  class X {
    final int _private = 4;
    void x() {
      print(_private);
    }
  }
  class Y extends X {
    void y() {
      print(_private);// アクセスできる。
    }
  }
  ```
* Javaのようなアクセス修飾詞(public, private, protected)はない。
* (参考)アノテーションで@protectedはある。@privateはない。
  * https://github.com/dart-lang/sdk/issues/41089

# ライブラリの作成(未読)
* https://dart.dev/guides/libraries/create-library-packages


