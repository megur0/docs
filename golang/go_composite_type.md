---
title: "複合型 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 複合型



## set型
* Goにはset型がない。
* sliceやmapを使えば表現可能。
* https://qiita.com/kotaroooo0/items/fe63ecc68723ab6ee81f


## 配列(Arrays)
* 例
  * var a [10]int
  * primes := [6]int{2, 3, 5, 7, 11, 13}
* 配列の長さは型の一部分。
* よって配列のサイズを変えることはできない。
* しかし、Goでは配列を扱うための便利なものが用意されている -> slices

## Slices
* https://go-tour-jp.appspot.com/moretypes/7
* https://go-tour-jp.appspot.com/moretypes/10
* 配列は固定長。一方で、スライスは可変長。
    * sliceは配列の参照のようなもの。
		* 内部的に配列を参照している。
		* https://go-tour-jp.appspot.com/concurrency/2
    * 既にある配列の参照にする or 作成する（作成すると、実際には配列を作成してそれを参照するスライスを作成する。）
* sliceは 長さと容量を持っている。
    * 動きがポインタみたい。開始位置をずらすと容量も変わる。
    * https://go-tour-jp.appspot.com/moretypes/11
* append
    * https://go-tour-jp.appspot.com/moretypes/15
* sliceは内部構造でポインタを保持している
  ```go
  type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
  }
  ```
  * 下記のような挙動になる。
    ```go
    s := []int{1, 2, 3}
    s2 := s                        // sliceのコピーはポインタとしてコピーされる
    fmt.Printf("%p, %p \n", s, s2) // 同じ値: 0xc000018018, 0xc000018018
    s[0] = 2                       // これはslice.arrayを変更する操作
    fmt.Println(s, s2)             // [2,2,3] [2,2,3]
    fmt.Printf("%p, %p \n", s, s2) // 同じ値: 0xc000018018, 0xc000018018
    ```
  * ただし、appendの場合はポインタが置き換わるため注意。
    ```go
    s := []int{1, 2, 3}
    s2 := s                        // sliceのコピーはポインタの値がコピーされる
    fmt.Printf("%p, %p \n", s, s2) // 同じ値: 0xc000018018, 0xc000018018
    s = append(s, 4)               // この場合はポインタが置き換わる。
    fmt.Println(s, s2)             // [1,2,3,4] [1,2,3]
    fmt.Printf("%p, %p \n", s, s2) // 異なる値: 0xc000104000, 0xc000018018
    ```
* sliceのデフォルト値はnil
  * これは内部構造のarrayがnilになっている？？
```go
primes := [6]int{2, 3, 5, 7, 11, 13}
var s []int = primes[1:4] 
fmt.Println(s)////[3 5 7] 3 5 7 11 ではないことに注意。

// makeをつかった作成
b := make([]int, 0, 5) // len(b)=0, cap(b)=5、cap値は省略できる。したがって容量上限をなしにできる。

// 最初から容量が分かっているなら下記のようにするのが効率的。
// 長さが容量を超えるとエラーになる。
a := make([]string, 10)
for i, c := range a { c[i] = "test"}

// 個数が分からないときはappendで。
// appendでは、元の配列 が、変数群を追加する際に容量が小さい場合は、より大きいサイズの配列を割り当て直す。
// その場合、戻り値となるスライスは、新しい割当先を示すようになる。
friend1 := []string{"a"}
friend2 := []string{"a", "b"}
friend3 := append(friend1, friend2...)

// sliceからsliceを作成
s = s[1:4]
fmt.Println(s)
s = s[:2]
fmt.Println(s)
s = s[1:]
fmt.Println(s)

// range でfor文
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
func main() {
  for i, v := range pow {
    fmt.Printf("2**%d = %d\n", i, v)
  }
}

// 以下は[6]bool{true, false, true, true, false, true}を作ってからそれを参照するスライスを作成している。
r := []bool{true, false, true, true, false, true}
```
* スライスのゼロ値はnilである。（参照先の配列を持っていない）
```go
var s []int
fmt.Println(s, len(s), cap(s))///[] 0 0
if s == nil {
    fmt.Println("nil!")///nil!
}
```
* 以下はすべて等価（aは10個の要素を持つとする。）
```go
a[0:10]
a[:10]
a[0:]
a[:]
```
* 多重slice
  * https://go-tour-jp.appspot.com/moretypes/14
* Exercise
  * https://go-tour-jp.appspot.com/moretypes/18


## sliceの要素の削除など
* https://go.dev/wiki/SliceTricks


## map、mapリテラル
* https://go-tour-jp.appspot.com/moretypes/19
```go
m := make(map[string]int)
m["aaa"] = 4
n := map[string]int{"aaa": 3}
n["bbb"] = 4
```
* mapリテラルと structリテラルは似ているが、mapはキー値が必要。
* 取得、delete、存在確認
  * https://go-tour-jp.appspot.com/moretypes/22
  * v, ok := m["Answer"] 
    * m に key があれば、変数 ok は true となり、存在しなければ、 ok は false
    * mapに key が存在しない場合、 vはmapの要素の型のゼロ値
* 注意：最初に makeしないで代入するとエラーになるので注意
  * make(map[string]int)
  ```go
  var m map[string]int
  m["Answer"] = 42 // panic: assignment to entry in nil map
  ```
* 具体例
  ```go
  type fakeFetcher map[string]*fakeResult
  type fakeResult struct {
    body string
    urls []string
  }
  ```
* range
```go
m := map[int]string{
		1: "value1",
		2: "value2",
		3: "value3",
	}

for key, value := range m {// 順番はランダムになるので注意。
  fmt.Println(key, value)
}
```


## 参照型のコピー
* Goのsliceやmapは値への参照を保持しているため、単純に値をコピーするだけではDeepCopyできない。（シャローコピーになってしまう）
* copyを使うのが正しい。


## (IMO) []Modelと[]*Model
* []*Modelを使う必要は無いはず。
* []Modelを渡すとスライスはコピーされるが、その参照先は共有されるため。
* なお、ChatGPTに聞いたら誤った回答が出てきた
  * https://chatgpt.com/share/681d4cb3-b920-8000-9e19-ad939f171c22