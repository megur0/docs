[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# エラー処理
> Your Dart code can throw and catch exceptions. Exceptions are errors indicating that something unexpected happened. If the exception isn't caught, the isolate that raised the exception is suspended, and typically the isolate and its program are terminated.
* https://dart.dev/language/error-handling
* ExceptionやErrorだけではなく、任意のオブジェクトを投げることを可能。
* throw〜はstatementではなくexpressionとなる。
    * 例えばアロー関数の右辺に入れることもできる。
* rethrowはエラーをpropagateする。
* 例外の有無に関わらず最終的に実行したい処理をfinallyとして記述できる
    * catchが存在しない場合はfinallyを実行後にエラーがpropagateされる。

```
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e) {
  // No specified type, handles all
  print('Something really unknown: $e');
}
```
```
try {
  // ···
} on Exception catch (e) {
  print('Exception details:\n $e');
} catch (e, s) {
  print('Exception details:\n $e');
  print('Stack trace:\n $s');
}
```
```
void main() {
 a(); 
}
void a() {
  try {
    throw Exception("dummy");
    //print("try");
    return;
  } catch(e) {
    print(e);
    return;
  } finally {
    print("finally");
  }
  print("a end");
}
// Exception: dummy
// finally
```


# assert
* https://dart.dev/language/error-handling#assert
* assertはリリースビルドでは処理されず残しておいても問題ないため、コードテストに適している。
* Flutter はデバッグ モードでアサーションを有効にする。
```
// Make sure the variable has a non-null value.
assert(text != null);

// Make sure the value is less than 100.
assert(number < 100);

// Make sure this is an https URL.
assert(urlString.startsWith('https'));
```
* 例1
  * コンストラクタの引数について複雑な条件をチェック
    ```
    const ScrollView({
        ...
        this.controller,
        this.primary,
      }) : assert(
            !(controller != null && (primary ?? false)),
            'Primary ScrollViews obtain their ScrollController via inheritance '
            'from a PrimaryScrollController widget. You cannot both set primary to '
            'true and pass an explicit controller.',
          );

    ```
* 例2
  * 以下はFlutter内部のホットリロードを考慮したチェックとなる。
  * 変数のhasSameSuperclassのデフォルトがtrueとなるためassertの実行有無によって処理結果に影響が発生することはない。
  ```
  bool hasSameSuperclass = true;
  assert(() {
      final int oldElementClass = Element._debugConcreteSubtype(child);
      final int newWidgetClass = Widget._debugConcreteSubtype(newWidget);
      hasSameSuperclass = oldElementClass == newWidgetClass;
      return true;
  }());
  if (hasSameSuperclass && child.widget == newWidget) {
      ...
  } else if (hasSameSuperclass && Widget.canUpdate(child.widget, newWidget)) {
      ...
  } else {
      ...
  }
  ```

# その他
* StackTraceの取得
  * `main() => print(StackTrace.current);`
* Dartで定義されているError, Exceptionクラス
    * 参考
        * https://www.cresc.co.jp/tech/java/Google_Dart2/language/exceptions/exceptions.html

