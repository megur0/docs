---
title: "その他パッケージ - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > その他パッケージ




## sort.Slice
* 比較関数 `func(i int, j int) bool`において
	* iの値 < jの値  の時に 真とする: 昇順
	* iの値 > jの値  の時に 真とする: 降順
```go
package main

import (
	"fmt"
	"sort"
	"time"
)

func main() {
	time1, _ := time.Parse("2006-01-02T15:04:05", "2024-01-01T10:00:00")
	time2, _ := time.Parse("2006-01-02T15:04:05", "2024-01-01T10:01:00")
	time3, _ := time.Parse("2006-01-02T15:04:05", "2024-01-01T10:02:00")

	target := []struct {
		CreatedAt time.Time
	}{
		{
			CreatedAt: time3,
		},
		{
			CreatedAt: time1,
		},
		{
			CreatedAt: time2,
		},
	}

	fmt.Println(target)
	sort.Slice(target, func(i, j int) bool { return target[i].CreatedAt.Before(target[j].CreatedAt) })
	fmt.Println(target)
	sort.Slice(target, func(i, j int) bool { return target[i].CreatedAt.After(target[j].CreatedAt) })
	fmt.Println(target)
}

/*

[{2024-01-01 10:02:00 +0000 UTC} {2024-01-01 10:00:00 +0000 UTC} {2024-01-01 10:01:00 +0000 UTC}]
[{2024-01-01 10:00:00 +0000 UTC} {2024-01-01 10:01:00 +0000 UTC} {2024-01-01 10:02:00 +0000 UTC}]
[{2024-01-01 10:02:00 +0000 UTC} {2024-01-01 10:01:00 +0000 UTC} {2024-01-01 10:00:00 +0000 UTC}]

*/

```


## sort.SliceStable
* sort.Sliceは等しい値でも、オリジナルの順番が保証されないが、sort.SliceStableの場合は保証される。
	> SliceStable sorts the slice x using the provided less function, keeping equal elements in their original order. 
* なお、内部で要素数が12個まではsort.Sliceもステーブルの処理を行う。
	* https://github.com/golang/go/blob/go1.21.4/src/sort/gen_sort_variants.go#L245
```go
package main

import (
	"fmt"
	"sort"
)

type Score struct {
	Name  string
	Score int
}

func getTarget() []Score {

	return []Score{
		{
			"1", 3,
		},
		{
			"2", 3,
		},
		{
			"3", 2,
		},
		{
			"4", 1,
		},
		{
			"5", 3,
		},
		{
			"6", 1,
		},
		{
			"7", 1,
		},
		{
			"8", 1,
		},
		{
			"9", 1,
		},
		{
			"10", 1,
		},
		{
			"11", 1,
		},
		{
			"12", 1,
		},
		{
			"13", 1,
		},
	}
}

func main() {
	target := getTarget()
	fmt.Println(target)
	sort.Slice(target, func(i, j int) bool { return target[i].Score < target[j].Score })
	fmt.Println(target)

	target2 := getTarget()
	sort.SliceStable(target2, func(i, j int) bool { return target2[i].Score < target2[j].Score })
	fmt.Println(target2)
}

/*

[{1 3} {2 3} {3 2} {4 1} {5 3} {6 1} {7 1} {8 1} {9 1} {10 1} {11 1} {12 1} {13 1}]
[{7 1} {4 1} {6 1} {8 1} {9 1} {10 1} {11 1} {12 1} {13 1} {3 2} {2 3} {5 3} {1 3}]
[{4 1} {6 1} {7 1} {8 1} {9 1} {10 1} {11 1} {12 1} {13 1} {3 2} {1 3} {2 3} {5 3}]

*/

```
