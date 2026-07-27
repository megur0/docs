---
title: "Objective-C"
updated: 2026-07-26
---

[TOP(About this memo))](../README.md) > Objective-C


# 参考
* (参考) https://gist.github.com/iyuuya/3231301
* (参考) https://qiita.com/kidach1/items/866c7ea6ce7eaf02c35c


# ブラケット記法（メッセージ式、メッセージング）
* `[greeter sayHello];`
    * `greeter.sayHello();` と同じ意味。
* `[@"checkPermissionStatus" isEqualToString:call.method];`
    * `@"checkPermissionStatus".isEqualToString(call.method);` と同じ意味(?)。
* なお、ブラケット記法だとデリゲート (委譲)などの機能があるらしい。詳細については把握していない(TODO)。


# Blocks
* (参考) https://obc-fight.blogspot.com/2013/07/Block-basic.html
* クロージャ的なもの(?)。
```objectivec
// 宣言: 戻り値(^関数名)(引数)
void (^blocksTest1)(void);

// 代入: 関数名の後ろにイコール(=)を入れ、^(引数){}
blocksTest1 = ^(void) {
    NSLog(@"blocksTest1");
};

// 実行
blocksTest1();

// 直ぐに実行
^() {
    NSLog(@"テスト");
}();

// 宣言, 代入
void (^blocksTest2)(void) = ^(void) {
    NSLog(@"blocksTest2");
};

// 返り値あり
int (^blocksTest3)(void) = ^(void) {
    return 10;
};
void (^blocksTest5)(int x, int y) = ^(int x, int y) {
    NSLog(@"%d", x + y);
};

// 実行、NSLog()でログ出力
NSLog(@"%d", blocksTest3());
```
## メソッドの引数がBlocks
* 定義
```objectivec
// 「(int(^) (int a, int b))bt7」の部分が、a, bを引数にとってintを返すBlocksとなる
-(int)blocksTest7:(int(^) (int a, int b))bt7 x:(int)x y:(int)y {
    return bt7(x, y);
}
```
* 呼び出し
```objectivec
int i = [self blocksTest7:^int(int a, int b){
    return a * b;
} x:10 y:10];
```


# @マーク
```objectivec
@1;     // => [NSNumber numberWithInteger:1]
@0.5;   // => [NSNumber numberWithDouble:0.5]
@YES;   // => [NSNumber numberWithBool:YES]
@"文字列" // => キャスト(?)
@[@1];  // => [NSArray arrayWithObjects:{@1} count:1]
@{@"hoge": @"fuga"}; // [NSDictionary dictionaryWithObjects:{@"fuga"} forKeys:{@"hoge"} count:1]
```
* (参考) https://akuraru.hatenadiary.jp/entry/20130427/p1


# 型
* NSObject
    * ルートクラス的なもの。
    * 他の言語だとObject型に相当する。
* id
    * 動的型(dynamic type)的なもの(?)。


# 構造体
```objectivec
typedef struct Person {
    float height;
    float weight;
    int birthYear;
} Person; // struct PersonにPersonというaliasを切る。

Person a; // 変数宣言
a.height = 170.5;
```


# クラスのヘッダファイル（.h）
* @propertyで宣言すると自動でsetter, getterが設定される。
    * 基本的には.mファイルの@synthesizeとセットで使用する。
* デフォルトで、コンストラクタはinit（NSObject）が設定されている。
    * 上書きすることも出来る。
```objectivec
@interface Person : Mammal

@protected
    // メンバ定義
    int _life;

    // なお、ここにメンバ変数を定義しなくてもよい
    // (@synthesize 時に実体となる変数を定義できるため)

// アクセサ
@property int life;
@property int lifeb;
@property int lifec;
@property int lifed;

@end
```


# クラスの実装ファイル（.m）
* @synthesize
    * @propertyで定義したプロパティと、メンバ変数をsynthesize(合成)するもの。
* メソッド
    * Objective-Cでは第二引数以降はラベルをつけるのが一般的。
    * `- (戻り値の型)メソッド名:(引数の型)引数1 ラベル:(引数の型)引数2 ラベル:(引数の型)引数3`
    * ラベルは省略できる。
    * ラベルは呼び出す側が引数の名前を指定する際に使う（他の言語で言う名前付き引数的なもの(?)）。
    * なお、第一引数のラベルについては、メソッド名が第一引数のラベルも兼ねている。
```objectivec
#import "Person.h"
@implementation Person

// インスタンスメソッド定義
- (int)someMethod
{
    return life;
}
- (void)someMethod2:(int)xxx
{
    life += xxx;
    NSLog(@"life: %d", life);
    NSLog(@"life: %d", _life); // 直接、実体を参照しても結果は同じ。
    return;
}

// クラスメソッド定義
+ (int)someMethod3
{
    // ...
    return;
}

@synthesize life  = _life;  // 既に定義されているメンバ変数を指定
@synthesize lifeb = _lifeb; // 新しくメンバ変数を定義して指定
@synthesize lifec  = aaaa;  // 実体を指すメンバ変数の名前に決まりはない
@synthesize lifed;          // 実体を省略すると同名のメンバ変数が指定されたことになる

@end
```


# インスタンス生成
* alloc
    * インスタンスの生成に必要なメモリ容量を確保する。
* init
    * メモリ上にロードされたインスタンスに対して実行する。
    * つまり、インスタンスがデータを保存出来る状態にする。コンストラクタを呼ぶ。
* new
    * alloc + init を一度に行う。
```objectivec
Person *tarou = [[Person alloc] init];
// メンバセット
tarou->life = 10;
// メソッド呼び出し
[tarou someMethod];
[tarou someMethod2:5];
// クラスメソッド呼び出し
[Person someMethod3];
```


# ラベルがついたメソッドの呼び出し例
* (参考) https://qiita.com/Pinehead/items/a9476dcac6e39f33c282
* 宣言
```objectivec
@interface MyTest : NSObject
- (void)consoleWrightWithHeight:(NSInteger)h weight:(NSInteger)w;
@end
```
* 実装
```objectivec
@implementation MyTest
- (void)consoleWrightWithHeight:(NSInteger)h weight:(NSInteger)w {
    NSLog(@"Your Height=%ld Weight=%ld", h, w);
}
@end
```
* 呼び出し
```objectivec
MyTest *test = [[MyTest alloc] init];
[test consoleWrightWithHeight:174 weight:65];
// もし、ラベルがない場合だと、以下のようになる。
// [test consoleWrightWithHeight :174 :65];
```
