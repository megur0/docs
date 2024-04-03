- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


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

# Assert
* https://dart.dev/language/error-handling#assert
* 開発中に、結果がfalseになる場合に処理を中断させる。
* Flutter はデバッグ モードでアサーションを有効にする。
```
// Make sure the variable has a non-null value.
assert(text != null);

// Make sure the value is less than 100.
assert(number < 100);

// Make sure this is an https URL.
assert(urlString.startsWith('https'));
```

# その他
* Dartで定義されているError, Exceptionクラス
    * 参考
        * https://www.cresc.co.jp/tech/java/Google_Dart2/language/exceptions/exceptions.html

