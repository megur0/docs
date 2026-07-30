---
title: "math - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > math




## rand
* セキュアな疑似乱数の必要があるなら（パフォーマンスは落ちるが）この関数やmath/randではなく、crypto/randを使う。

## rand.Seed
* Go 1.20以降はrand.Seed関数は非推奨となった。
    * https://github.com/golang/go/issues/54880
* Seed関数を呼ばなくても固定値ではないSeed値が設定され、擬似的なランダム性はデフォルトで担保されるようになった。
    * 1.19以前は、Seed関数を呼ばない場合は、Seed(1)がされていた
    * なお、GODEBUG=randautoseed=0 を設定することで この自動のSeed設定は以前と同じようにSeed(1)となる。
* rand.Seedを廃止した事由
    * Seed値の設定が漏れてしまうユーザーが多くて、rand.Seed自体が無い方が間違いを犯しづらいだろう、という判断？
    * あえて、テスト目的でrand.Seed値を固定値にしたい、あるいは自前のシード値生成器を使いたいというややレアケースでは、後述のようにrand.Sourceから作るような既存のやり方もあるので、rand.Seed自体が要らない、という感じ?
* 再現性が必要な場合や別のシード値を使いたい場合は、下記のプログラムのように、rand.Sourceから作る感じになる。（これはもともと1.19以前でも存在する手段）
    * このrand.Sourceはスレッドセーフではないので注意。つまり グローバルに var source := rand.NewSource(5) みたいに設定してそれを複数のスレッドで参照するとpanicになりうる、といった感じ。なので使い回しさえしなければ全く問題ない。
* 参考
    * https://future-architect.github.io/articles/20230123a/
```go
func TestRand(t *testing.T) {
	/*t.Run("assert_rand_Seed", func(t *testing.T) {
		rand.Seed(time.Now().UnixNano())
		for i := 0; i < 10; i++ {
			fmt.Println(rand.Intn(10))
		}
	})*/


	t.Run("assert_rand", func(t *testing.T) {
		//rng := rand.New(rand.NewSource(time.Now().UnixNano()))
		rng := rand.New(rand.NewSource(5))
		fmt.Println("deterministic rand: ")
		for i := 0; i < 10; i++ {
			fmt.Println(rng.Intn(10))
		}
		fmt.Println("non-deterministic rand: ")
		for i := 0; i < 10; i++ {
			fmt.Println(rand.Intn(10))
		}
	})
}

```