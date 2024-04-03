- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# Extension
* https://dart.dev/language/extension-methods
> Extension methods add functionality to existing libraries. You might use extension methods without even knowing it. For example, when you use code completion in an IDE, it suggests extension methods alongside regular methods.
```
extension NumberParsing on String {
  int parseInt() {
    return int.parse(this);
  }
  // ···
}
```
* extensionプライベートのフィールドにはライブラリ外からはアクセスが出来ないため注意
  ```
  extension Iso8601Ex on DateTime {
    String toIso8601StringEx() {
      String y =
          (year >= -9999 && year <= 9999) ? _fourDigits(year) : _sixDigits(year); // _fourDigitsや_sixDigitsは利用できないためエラーとなる。
    // ... 
    }
  }
  ```
* 静的なメソッドを定義することはできない
  * 関連
    * https://github.com/dart-lang/language/issues/723
  





