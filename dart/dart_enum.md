[TOP(About this memo))](../README.md) > [一覧](./README.md) >



# Enum
* https://dart.dev/language/enums
* enum のすべてのインスタンスは先頭で宣言する必要がある。
* インスタンス変数はmixin によって追加されたものも含めて、finalである必要がある。
* すべてのコンストラクターは定数である必要がある。
* ファクトリコンストラクターは、固定された既知の enum インスタンスの 1 つだけを返すことができる。
* index、hashCode、等号演算子 == をオーバーライドすることはできない。
* valuesという名前のメンバをenumで宣言することはできない。
  * valuesメソッドと衝突するため。
* 親クラスをextendsすることはできない。
  * implementsすることは可能
* enumをextendsすることはできない。
* .values.byName(String)でenumを取得可能
* .nameによって名前を取得できる。
* extensionをEnum(すべてのenumが継承する抽象クラス)に適用できる
* ※ 上記の機能はDart2.17以降(Enhanced Enumsによってenumの機能が拡張された)の前提となる。
```
void main () {
  print(E1.values);
  print(E1.values.byName("A"));
  print(E1.A.name);
  //[E1.A, E1.B]
  //E1.A
  //A
}
enum E1 {
  A(1),
  B(2);
  final int f1;// finalである必要がある。
  const E1(this.f1);// constである必要がある。
}

/* 
enum E2 {
  A(DateTime.now());// constではない値を使うことはできない。
  final DateTime f1;
  const E2(this.f1);
}
*/
```

# enumの判定はswitchを使う
* https://dart.dev/language/branches#exhaustiveness-checking
* enumやsealed classは取りうる値が完全に列挙可能であるため、switchを利用することで漏れを静的に検出可能。
```
enum FriendStatus {
  notFriend,
  friend,
}
```
```
if (status ==  FriendStatus.notFriend) {
 // ...
} else {
 // ...
}
// こちらの場合は網羅性を担保できる。
switch (status) {
  case FriendStatus.notFriend:
    // ...
  case FriendStatus.friend:
    // ...
  // もしFriendStatusに新しい値が追加された場合に、コンパイルエラーとして修正箇所を把握できる。
}
```


# byXXXの実装
```
enum MyEnum {
  a(10),
  b(20);

  const MyEnum(this.code);

  static MyEnum byCode(int code) {
    return MyEnum.values.where((rt) => rt.code == code).toList().first;
  }

  final int code;
}
```