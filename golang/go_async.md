---
title: "非同期 - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > 非同期




# Goroutines (ゴルーチン)
* Goのランタイムに管理される軽量なスレッド
* go f(x, y, z)()
  * 新しいgoroutineが実行される
  * 評価はcurrentのgoroutinで行われる。
```go
go func() {
    ////
}()
```
* 同じアドレス空間で実行されるため、共有メモリへのアクセスは必ず同期する必要がある。
* 子のゴルーチンの終了を親は待たない。
  * mainがexitしたら、その下のgoroutineは強制的に終わる。
  * main -> go func1 とか、main -> func2 -> go func1って感じで呼び出したとき、（当たり前だが）func1の終了をmainは待たない。
  * https://go.dev/play/p/WNKSd0LjS1o




# チャネル
* チャネルとは、値の送受信ができるもの。
  * 送る側と受ける側が準備できるまで、 送受信はブロックされる。
  * したがって、チャネルを使うことでgoroutineの同期を可能にする。
## 生成
* ch := make(chan int)
* 生成せずに使うとデッドロックするので注意。（これは何も送れないし受け取れないからすべてが待機になってしまうから？）
```go
// all goroutines are asleep - deadlock!
func main() {
	var syn chan interface{}
	go func() {
		syn <- struct{}{}
		fmt.Println("g1 end")
	}()
	<-syn
}
```
* サイズが0だとデッドロックする。
```go
fmt.Println("main start")
	syn := make(chan interface{}, 0)
	syn <- struct{}{}//これが送信できないのでロック。
	fmt.Println("main end")
```
## 送信
* ch <- v
  * v をチャネル ch へ送信する
* とりあえずシグナルだけ送りたい場合は空の構造体を使う
  * struct{}{}。空の構造体はメモリサイズが0
  * https://zenn.dev/mstn_/articles/76ae3f3e65d207
## 受信
* <-ch
    * 受信、もしくは close したことだけ検知するパターン。非同期処理の終了を待つ場合に使うことが多い
* v := <-ch
  * 値を取得するパターン。 close された場合はnilが入る
* v, ok := <-ch
  * 値が入ったのか close されたのかを判別するパターン。 
  * ok が false なら close 。
  ```go
  v, ok := <-ch
  if !ok {
      // closed
  }
  ```
  * https://hori-ryota.com/blog/golang-channel-pattern/
## 自分で送信したものも受け取れる。
* つまり、自分が送ったかどうかではなく、とにかく一番先頭で受信を待機しているスレッドが受信できる。
```go
syn := make(chan int, 1)
syn <- 5
print(<-syn)// 5
```
# 受信（select）
* 複数の case　においていずれかが準備ができるまで、ブロックする。
  * 複数のcaseが準備ができているときはランダムに選択される。
  * defaultはどのcaseも準備ができていないときに実行される。
* https://go-tour-jp.appspot.com/concurrency/5
  * この例では、fibonacci側はquitチャネルに受信しない限りいつまでもchに送り続ける。quitを受け取った場合、returnする。
  * 呼び出し側は好きな個数分 fibonacciから値を受け取り、その後quitチャネルに送信する。
## バッファ
* チャネルはバッファとして使える。
  * `ch := make(chan int, 100)`
* バッファが詰まった時は、チャネルへの送信をブロック。
* バッファが空の時には、チャネルからの受信をブロック。
## close
* 送り手は、これ以上の送信する値がないことを示すため、チャネルを close できる。（受け手はできない。やるとpanicになる）
  * 受け手：　v, ok := <-ch
    * 受信する値がない、かつ、チャネルが閉じているなら、 ok の変数は、 falseになる。
  * 通常はチャネルをcloseする必要はない。受け手が知る必要があるときにだけ。これ以上値が来ないことを受け手が知る必要があるときにだけです。
    * 例えば、 range ループを終了するという場合。
## range
* rangeで受け取る事が可能。（closeが来るまでループ）
  ```go
  c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
  ```
## 送信専用、受信専用
```go
// ch 送信専用
func test1(ch chan<- string, wg *sync.WaitGroup) {
	defer wg.Done()
	ch <- "aaaa"
}

// ch 受信専用
func test2(ch <-chan string, wg *sync.WaitGroup) {
	defer wg.Done()
	m := <-ch
	fmt.Println(m)
}
```


# サンプルコード
* 自分で色々試してみた。（値の受け渡し、closeの状態、デッドロック）
* https://go.dev/play/p/cECYnsK8LhX
```go
package main

import (
	"fmt"
	"time"
)

func chtest(ch chan interface{}) {
	fmt.Println("chtest running")
	time.Sleep(time.Second * 3)
	ch <- struct{}{}
	fmt.Println("chtest end")
}

func chtest2(ch chan interface{}) {
	defer close(ch)
	fmt.Println("chtest2 running")
	time.Sleep(time.Second * 3)
	fmt.Println("chtest2 end")
}

func chtest3(ch chan interface{}) {
	fmt.Println("chtest3 running")
	time.Sleep(time.Second * 3)
	fmt.Println("chtest3 end")
}

func chtest4(ch chan interface{}) {
	fmt.Println("chtest4 running")
	time.Sleep(time.Second * 3)
	ch <- 1
	fmt.Println("chtest4 end")
}

func main() {
	ch := make(chan interface{}, 1)
	fmt.Println("chtest call")
	go chtest(ch)
	v, ok := <-ch
	fmt.Println(v)
	fmt.Println(ok)

	/*
		// goroutineがすべて終了にしているのでデッドロックとなるためpanicになる。
		v, ok = <-ch 
		fmt.Println(v)
		fmt.Println(ok)
	*/

	//chはcloseしなければ使いまわしできる。
	fmt.Println("chtest call again")
	go chtest(ch)
	v, ok = <-ch
	fmt.Println(v)
	fmt.Println(ok)

	//closeされた場合はcloseしたタイミングでvはnil、okはfalseを受け取る。
	fmt.Println("chtest2 call")
	go chtest2(ch)
	v, ok = <-ch
	fmt.Println(v)
	fmt.Println(ok)

	/*
		// これはpanicになる。chがcloseしているのに、chtest2の最後に再度closeをしようとするから。
		fmt.Println("chtest2 call again")
		go chtest2(ch)
		v, ok = <-ch // これはすぐに後続へ進む。なぜならchはcloseされているから。
		fmt.Println(v)
		fmt.Println(ok)
	*/

	ch2 := make(chan interface{}, 1)
	/*
		//panicになる。chtest3ではチャネル送信をしていないため、待ち状態のまま（デッドロック）してしまうため。
		fmt.Println("chtest3 call")
		go chtest3(ch2)
		v, ok = <-ch2
		fmt.Println(v)
		fmt.Println(ok)
	*/

	fmt.Println("chtest4 call")
	go chtest4(ch2)
	v, ok = <-ch2
	fmt.Println(v)
	fmt.Println(ok)
}
``` 
  


# 排他制御（Mutex）
* チャネルによって通信間で同期を取ることもできるが、通信をしない場合に変数にアクセスするgoroutineを一つにするにはどうすればよいか？
  * このコンセプトを「排他制御」という。一般的にそのデータ構造をmutexという。
* GOで用意されている「sync.Mutex」型は、構造体に含めることで利用できる？（ここはチュートリアルには説明が書いてなかった）
*  defer でunlockを関数の一番最後に呼び出すようにできる。
  * この例だと、returnの後にunlockするように保証しているはず。（この例だと別にロックしなくても成立する気がするので少し分かりづらいが、読み込み側も読むときに更新されてほしくない、といったパターン？）
  * https://go-tour-jp.appspot.com/concurrency/9
* (未読)
https://www.ohitori.fun/entry/mutex-and-rwmutex-in-golang
https://qiita.com/gold-kou/items/8e5342d8a30ae8f34dff
https://qiita.com/tomokon/items/449c01b14507c9817ad7



# context
* https://pkg.go.dev/context
* キャンセルなどをサブルーチンに伝搬したり，限られたスコープ内で一貫してアクセスできるインメモリ KV ストア的な役目
* 具体的な使い方
  * 空のContextの生成
  * 値の伝播( WithValue, Value )
  * キャンセルの伝播( WithCancel )
  * タイムアウトの伝播( WithTimeout )
* https://www.wakuwakubank.com/posts/867-go-context/
* https://zenn.dev/hsaki/books/golang-context/viewer/definition
## 注意点
* 関数で許可されている場合でも、 nil Contextを渡さない。
* リクエスト スコープのデータにのみ使用する。（たとえば、httpリクエストそれぞれのスコープに閉じたデータ）
	* 間違った使い方として、関数のオプションパラメーターを渡すため 等
	* https://moneyforward-dev.jp/entry/2020/07/28/go-context/
## Value
* キーの衝突を避けるために個々のパッケージでそれぞれ独自型を定義してキーにする。
	* そうしないとリンターで注意されるはず。
	* 独自型は、struct{}{} をつかうと同じキーとしてみなされる罠がある。
		* あと、stringで独自型にしたときもちゃんと動作しなかった。
	* 自分はstruct{Key string}を使っている。
* contextはイミュータブルで、
	* ctx = context.WithValue(ctx, key, value)  みたいな感じで新しく生成する。
* https://tech.anti-pattern.co.jp/go-ctx-key/
## サーバーでの使い方
* [http](./go_http.md)を参照


# sleep と go routine
* 1マイクロセカンドでもsleepさせると、同じ処理なら後に終わる。
```go
func main() {
	for i := 0; i < 100; i++ {
		a := 0
		var wg sync.WaitGroup
		wg.Add(1)
		go func() {
			a = 1
			wg.Done()
		}()
		wg.Add(1)
		go func() {// 必ずこっちが後に終わる。
			time.Sleep(time.Microsecond * 1)
			a = 2
			wg.Done()
		}()
		wg.Wait()
		fmt.Print(a)
	}

}
```



# sync
* https://zenn.dev/yamato0211/articles/e3da679fd8dd6f
```go
// You can edit this code!
// Click here and start typing.
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	fmt.Println("main start")

	syn := make(chan interface{}, 1)
	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		fmt.Println("g1 start")
		fmt.Println("g1 waiting...")
		<-syn
		time.Sleep(time.Millisecond * 1000)
		fmt.Println("g1 exec done")
		syn <- struct{}{}
		fmt.Println("g1 end")
		wg.Done()
	}()

	wg.Add(1)
	go func() {
		fmt.Println("g2 start")
		time.Sleep(time.Millisecond * 1000)
		fmt.Println("g2 exec done")
		syn <- struct{}{}
		fmt.Println("g2 waiting...")
		<-syn
		fmt.Println("g2 end")
		wg.Done()
	}()

	wg.Wait()
	fmt.Println("main end")

/*

main start
g2 start
g1 start
g1 waiting...
g2 exec done
g2 waiting...
g1 exec done
g1 end
g2 end
main end

*/

}

```