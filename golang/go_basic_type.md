---
title: "基本型 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 基本型




# Types
* https://go.dev/ref/spec#Types
* 型は、値のセットと、それらの値に固有の操作およびメソッドを決定する
* 仕様書には下記のような分類がされている。
	* Predeclared types
		* 事前定義された型
	* defined types
		* Predeclared types あるいは、 type declarations か type parameter listsによって定義された型
	* named types
		* Predeclared types, defined types を named typesと呼ぶ
	* Composite types
		* type literalsを利用して構築するもの
		* array, struct, pointer, function, interface, slice, map, and channel types
		
# Assignability
* https://go.dev/ref/spec#Assignability
* V型の値xがT型の変数に代入可能である（「xはTに代入可能である」）のは、以下の条件のいずれかに当てはまる場合
	* VとTが同一(identical)である。
	* VとTは同一のunderlying typesを持つが、type parameters ではなく、VまたはTの少なくとも一方はnamed typeではない。
	* VとTは同一の element typesを持つchannel typesで、Vは双方向channelであり、VまたはTの少なくとも一方はnamed typeではない。
	* Tはinterface型であるが type parameterではなく、xはTを実装している。
	* xは事前に宣言された識別子nilであり、Tはポインタ、関数、スライス、マップ、チャネル、インタフェース型であるが、type parameter ではない。
	* xはT型の値で表現可能な型付けされていない定数である。


# 組み込み型(Predeclared types)
* https://go.dev/ref/spec#Predeclared_identifiers
* https://go-tour-jp.appspot.com/basics/11
```go
bool
string
int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr
byte // uint8 のエイリアス
rune // int32 のエイリアス
    // Unicode のコードポイントを表す
float32 float64
complex64 complex128 //複素数？
```
* int, uint, uintptr 型は、32-bitのシステムでは32 bitで、64-bitのシステムでは64 bit


# 複合型(Composite types)の制約
## Can I convert a []T to an []interface{}?
* https://blog.merovius.de/posts/2018-06-03-why-doesnt-go-have-variance-in/
* 複合型においては、言語仕様としてanyの複合型へ代入することができない。
* anyを他言語のObject型あるいはdynamic型のように捉えてしまうと齟齬が生じるだろう。
```go
func main() {
	var a []any
	//a = []string{"a", "b", "c"} // error
	//a = [6]int{2, 3, 5, 7, 11, 13} // error

	var b any
	b = "1"

	var c map[int]int = map[int]int{1:1}
	//f(c) // error
	var d map[any]any = map[any]any{}
	for k, v := range c {// named typeはanyへ代入可能
		d[k] = v
	}
	f(d)
}

func f(a map[any]any) {

}

```
## Can I convert []T1 to []T2 if T1 and T2 have the same underlying type?
* 同一なunderlying typeをT1, T2において
	* T1をT2へ変換することはできる(つまり、メソッドセットを変換することができる)
	* []T1 を []T2 へ変換することはできない
```go
type T1 int
type T2 int

func (t T1) M(){}

func main() {
	var t1 T1
	var x = T2(t1) // OK
	var st1 []T1
	var sx = ([]T2)(st1) // NOT OK
}
```
* https://go.dev/doc/faq#convert_slice_with_same_underlying_type


# Goの用語で「オブジェクト化」「インスタンス化」は何にあたる？
* specだと明示的にオブジェクトやインスタンスは使われていない。
* 「Allocation」が適切っぽい。
	* つまり、メモリ割り当て。
	* https://go.dev/ref/spec
* 参考
* https://www.reddit.com/r/golang/comments/x8utok/dumb_question_of_vocabulary_is_a_struct_instance/


# 文字列
* ""で囲むとエスケープシーケンスが利用できる
	* \nや\r
	* \uはunicodeのコードポイント
		```
		println("\u0061")//a
		println("\u3042")//あ
		```
	* \xはUTF-8の各バイト
		```
		println("\x61")// a
		println("\xe3\x81\x82")// あ（※「あ」はUTF-8では「E3 81 82」の３バイトで符号化される）
		println("\xe3\x81")// UTF-8として無効なバイト列のため文字化けする。
		```
* ``(バッククォート)で囲むとヒアドキュメントとなる
	* 改行がそのまま反映される
	* エスケープシーケンスは評価されない
* シングルクオテーションは文字列ではないため注意。
	* rune型として評価される？(未検証)
	* https://go.dev/ref/spec#Conversions_to_and_from_a_string_type
* 文字列結合：+
* interpolation（補間、内挿）機能は Goにはない。
  * interpolationは "$user" みたいに埋め込めるやつ。
  * https://qiita.com/KEINOS/items/baef1be88f15515026ec
* string型
  * string型は文字列実体へのポインタと文字列長を表すint型とで構成
  * https://qiita.com/hnw/items/ec3da327c37e3ad8c875
## Goの文字列はbyteのsliceのような形式。
* UTF-8文字列は3バイト。
  * fmt.Println(len("aaaaa"))//5
  * fmt.Println(len("あああああ"))//15
* UTF-8の文字列としての長さを知りたい場合はutf8.RuneCountInStringを使う必要がある。
  * fmt.Println(utf8.RuneCountInString("あああああ"))//5
## 切り出し
* ASCIIコードであればよいが、UTF-8文字列は1バイト単位で切ってしまうと、不正なバイト列になってしまう。
* なので、runeのsliceにキャストしてから切り出してstringへキャストする。runeのスライスはUTF-8一文字ごとになっている。
  * fmt.Println(string([]rune(str)[:2]))
* https://qiita.com/catatsuy/items/4586597246264e4674e1
## stringとintの相互変換
```go
import (
	"fmt"
	"strconv"
)

func main() {
	s := "1"
	si, _ := strconv.Atoi(s)
	fmt.Println(si)

	i := 2
	is := strconv.Itoa(i)
	fmt.Println(is)
}
```
## サンプルコード: UTF-8（rune）, バイト, 文字
```go
// 文字 -> unicode
fmt.Printf("%U\n", []rune("ああ\n\r")) // [U+3042 U+3042 U+000A U+000A]
fmt.Printf("%U\n", []rune(`ああ\n`))// [U+3042 U+3042 U+005C U+006E U+005C U+0072] // \nは改行ではないことに注意。
fmt.Printf("%U\n", []rune("\u0000\u0001\u0002\u0003\u0004\b\t\n\f\r\u0019あ"))// [U+0000 U+0001 U+0002 U+0003 U+0004 U+0008 U+0009 U+000A U+000C U+000D U+0019 U+3042]

// unicode -> 文字
println(string([]rune{0x3042, 0x3044, 0x3046, 0x3048, 0x304A})) // あいうえお
println("\u3042\u3044\u3046\u3048\u304A") //あいうえお
println("あ\u0000あ") //ああ（\u0000は NUL）

// バイト列 -> 文字
println("\xe3\x81\x82\xe3\x81\x84\xe3\x81\x86\xe3\x81\x88\xe3\x81\x8a")
println(string([]byte{0xe3, 0x81, 0x82, 0xe3, 0x81, 0x84, 0xe3, 0x81, 0x86, 0xe3, 0x81, 0x88, 0xe3, 0x81, 0x8a}))
println("あ\x00あ") //ああ（\x00は NUL）
println(`\xe3\x81\x82` + "\xe3\x81\x82")//\xe3\x81\x82あ
```
* 参考
  	* https://ja.wikipedia.org/wiki/Unicode%E4%B8%80%E8%A6%A7_0000-0FFF
	* https://go.dev/ref/spec#Conversions_to_and_from_a_string_type
	* https://qiita.com/masakielastic/items/01a4fb691c572dd71a19
```go
func TestUnicode(t *testing.T) {
	t.Run("assert_utf8_valid", func(t *testing.T) {
		assert.Exactly(t, utf8.ValidString("aaaa"), true)
		assert.Exactly(t, utf8.ValidString("\x00"), true) // NUL
		assert.Exactly(t, utf8.ValidString("\xe3"), false)
		assert.Exactly(t, utf8.ValidString("\"\ufffd\u0000\u0001\u0002\u0003\u0004\b\t\n\f\r\u0019あ"), true)
		assert.Exactly(t, utf8.ValidString(string([]rune{0x3042})), true)           // 「あ」
		assert.Exactly(t, utf8.ValidString(string([]byte{0xe3, 0x81, 0x82})), true) // 「あ」
		assert.Exactly(t, utf8.ValidString(string([]byte{0xe3, 0x81})), false)      // これは文字化けする。
	})
}
```


# Goのエスケープシーケンス
```go
escaped_char = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
```
* https://go.dev/ref/spec
## 参考（url.Parseに出てくる関数）
* エスケープシーケンスのチェックをしている。
```go
func stringContainsCTLByte(s string) bool {
	for i := 0; i < len(s); i++ {
		b := s[i]
		if b < ' ' || b == 0x7f {//  ' 'は、0x20。0x7fはDEL。(?) なぜこの範囲なのかは分からない。
			return true
		}
	}
	return false
}
```



# stringの文字数上限
* 上限は無く、メモリの量が上限？
https://stackoverflow.com/questions/40675029/is-there-any-string-length-limit-in-golangs-string-map-key



# type
* https://go.dev/ref/spec#Type_declarations
```go
type a string // Type definitions:  これはstringとは別の型
type b = string // Alias declarations: これはstringの別名(C言語のtypedefに相当する)
```
* なお anyは`type any = interface{}`として定義されている
## Type definitionsでは、レシーバーは継承されない。
* 一方、structの埋め込みだと（継承ではないが）移譲的に利用できる。
```go
package main

import (
	"fmt"
)

type a string
type b a

func (s a) r1() string {
	return string(s)
}

type c struct{ a }

type d struct{ i1 }
type i1 interface {
	r1() string
}

func main() {
	fmt.Println(a("test").r1())
	
	// typeだとレシーバーは引き継がれない。
	fmt.Println(b("test"))
	//fmt.Println(b("test").r1()) //compile error
	//fmt.Println(interface{}(b("test")).(i1)) // panic:  main.b is not main.i1: missing method r1
	
	// 埋め込みの場合は（継承ではないが）使えるし、実装したことになる。
	fmt.Println(c{"test"}.r1()) 
	fmt.Println(c{"test"}.a)
	_, ok := interface{}(c{"test"}).(i1)
	fmt.Println(ok) // true

	// interfaceも埋め込みによって実装したことになる。ただし中身はnil
	fmt.Printf("%#v\n", d{}) // main.d{i1:main.i1(nil)}
	//fmt.Println(d{}.r1()) // 実装はしているが、中身はnilなのでpanicになる。
	fmt.Println(d{}.i1) // <nil>
	_, ok = interface{}(d{}).(i1)
	fmt.Println(ok) // true
}
```


# enum 
* Goはenumに相当する機能がない。
* 下記のようにiotaを利用して擬似的な表現は可能だが、enumに備わる値の制限といった機能はない。
	* enumというよりは代入する値のエイリアス、といった
```go
package main

import "fmt"

type MyValue int

// var a = iota // error: constにのみ利用可能

const (
	MyValueA MyValue = iota
	MyValueB
)

// print用にfmt.Stringerを実装したもの
func (m MyValue) String() string {
	switch m {
	case MyValueA:
		return "MyValueA"
	case MyValueB:
		return "MyValueB"
	case 5:// 定義した以外の値も分岐可能
		return "This is 5!!"
	default:
		return "not defined"
	}
}

func main() {
	a := MyValueA
	fmt.Println(a) // MyValueA
	a = MyValueB
	fmt.Println(a) // MyValueB
	a = 10         // 定義した以外の値を代入可能
	fmt.Println(a) // not defined
}
```
