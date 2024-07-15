[TOP(About this memo))](../README.md) > [一覧](./README.md) >


# DateTimeクラス
* https://api.dart.dev/stable/3.3.3/dart-core/DateTime-class.html
* DateTime は、エポック (1970-01-01 UTC) から最大 100,000,000 日離れた時刻値を表すことができる。
```
void main() => print(DateTime.now().microsecondsSinceEpoch);
```
* UTC タイム ゾーンで明示的に作成されない限り、オブジェクトDateTimeはローカル タイム ゾーンとなる。
```
void main() {
  print(DateTime.now().isUtc);// false
  print(DateTime.now().timeZoneName); // 例: 日本標準時
  print(DateTime.now().timeZoneOffset); // 例: 9:00:00.000000
  print(DateTime.now().toUtc().isUtc); // true;
  print(DateTime.now().toUtc().timeZoneName);// UTC
  print(DateTime.now().toUtc().timeZoneOffset);// 0:00:00.000000
  print(DateTime.now()); // 例: 1930-04-02 12:08:59.908  
  print(DateTime.now().toIso8601String()); // 例: 1930-04-02T03:08:59.908Z
  print(DateTime.parse("2000-02-02 00:00:00")); // 2000-02-02 00:00:00.000
  print(DateTime.parse("2000-02-02 00:00:00").timeZoneName); // 例: 日本標準時 ※ parseしたときはタイムゾーンがローカルタイムゾーンとなる。
  print(DateTime.parse('2023-09-04T15:54:04.274164+09:00').timeZoneName);// UTC ※ parseにおいてタイムゾーン付きで渡すとタイムゾーンがUTCとなる
  print(DateTime.parse('2023-09-04T15:54:04.274164+09:00'));// 2023-09-04 06:54:04.274Z
  print(DateTime.now().isAfter(DateTime.parse("1900-01-01 00:00:00"))); // true
}
```

# 日時の矯正
* [参照](./dart_fake_async.md)


# (参考) DartPad や Flutter Web 上ではparseの際にマイクロ秒の情報がロストする
```
print(DateTime.parse('2023-09-04T15:54:04.274164+09:00').toIso8601String());
// Flutter モバイルの環境で実行すると 　2023-09-04T06:54:04.274164Z
// DartpadやFlutter webの環境で実行すると　2023-09-04T06:54:04.274Z
```
* https://github.com/dart-lang/sdk/issues/44876
  


