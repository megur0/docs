- [TOP](./README.md)
- [このメモ・独自表記について](../README.md)


# クロージャ
* Dartの関数はすべてクロージャである。
> Dart is a lexically scoped language, which means that the scope of variables is determined statically, simply by the layout of the code.
* 例
```
void main() {
  print(data());// 1
  print(data(increment:true));// 2
  print(data(increment:true));// 3
  print(data(reset:true));// 1
}
final data = () {
    int page = 1;
    int inc({bool increment = false, bool reset = false}) {
      if (reset) {
        return page = 1;
      }
      return increment ? ++page : page;
    }
    return inc;
  }();
```

# 同名の変数は近くのスコープが優先される。
```
main(){
  int a = 3;
  (int a) {
      a = a;// このaはどちらも引数のaを指す
  }(5);
  print(a);// 3
}
```

# 関数は第一級オブジェクト
* 関数を変数のようにあつかうことができる
```
final a = () => print("a");
a();
```

# 無名関数
* 下記の「(item) {return item.toUpperCase();}」 や 「(item) => print('$item: ${item.length}')」 は無名関数
```
const list = ['apples', 'bananas', 'oranges'];
list.map((item) {
  return item.toUpperCase();
}).forEach((item) => print('$item: ${item.length}'));
```

# アロー関数
* ショートハンドとしてアローのシンタックスが書ける。
* expressionは書けるがstatementは書けない

# parameters
## Optional parameters
* 任意の引数を指す。
* Dartには以下のOptional parametersがある。
  * Optional positional parameters
  * Optional named parameters
## Positional parameters
* 順序にしたがって必ず指定する必要がある引数
```
void positionalParamTest(String a, String b) {
  print("positionalParamTest: $a, $b");
}
positionalParamTest("aaa", "bbb");
//positionalParamTest("aaa");//compile error
```
```
void positionalNullableParamTest(String a, String? b) {
  print("positionalNullableParamTest: $a, $b");
}
positionalNullableParamTest("aaa", null);
// positionalNullableParamTest("aaa");// compile error

void positionalNullableParamTest2(String? a, String? b) {
  print("positionalNullableParamTest2: $a, $b");
}
positionalNullableParamTest2(null, null);
//positionalNullableParamTest2();//compile error
```
* Optional positional parameters
> If you don’t provide a default value, their types must be nullable as 
> their default value will be null
```
void OptionalPositionalParamTest(String? a, String? b, [String c="ccc", String? d]) {
  print("OptionalPositionalParamTest: $a, $b, $c, $d");
}
OptionalPositionalParamTest("aaa", "bbb",);
OptionalPositionalParamTest("aaa", "bbb", "ccc");
OptionalPositionalParamTest("aaa", "bbb", "ccc", "ddd");
```

## Named parameters
* 名前付き引数
* Optional Named parameters
  * requiredをつけない場合は任意の引数となる。
* requredをつけると必須の引数となる。
```
/* 
// コンパイルエラー
// nullableではない場合は必須とする必要がある。
void namedParamTest({String a, String b}) {
  print("namedParamTest: $a, $b");
}
*/
void namedParamTest2({String? a, String? b, required String? c, String d="ddd"}) {
  print("namedParamTest2: $a, $b");
}
namedParamTest2(a:"aaa", b:"bbb", c:"ccc");
namedParamTest2(c:null);
```
> Named parameters are optional unless they’re explicitly marked as required.
> If you don’t provide a default value or mark a named parameter as required, 
> their types must be nullable as their default value will be null.
```
//namedParamTest2("aaa"); //compile error
//namedParamTest2("aaa", "bbb"); //compile error
//namedParamTest2(a:"aaa", b:"bbb");//compile error
```
* その他検証
```
void namedParamTest3(String? a, String? b, {required String? c, String d="ddd"}) {
  print("namedParamTest3: $a, $b, $c, $d");
}
namedParamTest3("aaa", "bbb", c:"ccc");
namedParamTest3(c:"ccc", "aaa", "bbb");
//namedParamTest3("aaa", "bbb");//compile error

/* こちらはエラー
  * void namedParamTest4(String? a, String? b, [String? e], {required String? c, String d="ddd"}) {
  print("namedParamTest4: $a, $b, $c, $d");
}*/ 
  
/* こちらもエラー
void namedParamTest5(String? a, String? b, {required String? c, String d="ddd"}, [String? e]) {
  print("namedParamTest5: $a, $b, $c, $d");
}*/

/* こちらもエラー
void namedParamTest6({required String? c, String d="ddd"}, String? a, String? b) {
  print("namedParamTest6: $a, $b, $c, $d");
}*/
```






