---
title: "制御構造 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 制御構造



## インデント
* golangは特別な理由がなければインデントはタブを使う。（フォーマッタを実行するとそのようになる）
* https://go.dev/doc/effective_go#formatting

## 変数スコープ
```go
func main() {
	r := "a "
	if true {
		r := "b " // 同じ変数を宣言できる
		print(r) // より内側の変数が 優先される
		r = "c " // より内側の変数が 優先される 
		print(r) // より内側の変数が 優先される 
	}
	print(r)
}
// b c a 
```
* ただ、紛らわしいので自分は基本的に内側のブロックでも同じ変数名は使わない。


## switch
* goのswitchは自動的にbreakする（breakを書く必要がない。）
* 注意として、何も書かなかった場合でもbreakするので注意。
* fallthroughと書くと次のケースを続けて実行する。
```go
func checkNumber(i int) {
	switch i {
	case 1: // これは何も実行されないので注意。
	case 2, 3, 5, 7:
		fmt.Println("primary number")
		fallthrough // これはdefaultブロックも実行される。
	default:
		fmt.Println("good number")
	}
}
```

## if
* 括弧不要。中括弧は必要。
  * if x < 0 {〜 とか for sum < 1000 { 
* if with ショートステートメント（ ifの中だけで使える変数が宣言できる）
  ```go
  if v := math.Pow(x, n); v < lim {
    return v
  }
  ```
  * https://go-tour-jp.appspot.com/concurrency/10
    * 配列の中身が入ってる場合のみ処理
    ```go
    if res, ok := f[url]; ok {
      return res.body, res.urls, nil
    }
    ```

## for
* 括弧不要
```go
for i := 0; i < 10; i++ {
  sum += i
}
```
* 初期化と後処理ステートメントの記述は任意（;を省略もできる。したがってwhileも兼ねている。）
```go
sum := 1
for ; sum < 1000; {
  sum += sum
}
```
* 無限ループがシンプルに書ける
```go
for {
}
```
* range（インデックスも一緒に使える。）
```go
for i, v := range pow {
  fmt.Printf("2**%d = %d\n", i, v)
}
```
* インデックスや値は、 " _ "(アンダーバー) へ代入することで捨てることができる。
```go
for i, _ := range pow
for _, value := range pow
```
* これも行けた。
```go
for range []string{"a", "b", "c"} {
  fmt.Println("test")// testが3回出力
}
```
* 外側のforのbreak
```go
func main() {
outer:
	for a := 1; a < 10; a++ {
		fmt.Println(a)
		switch a {
		case 4:
			break // この場合はswitchをbreakするのみ
		case 5:
			break outer
		default:
			continue
		}
	}
}
```
