---
title: "time - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > time



* タイムゾーンの設定
```go
time.Local, err = time.LoadLocation("Asia/Tokyo")
if err != nil { /*ignore-coverage-panic*/
	panic(fmt.Sprintf("timezone init failed:%s", err))
}
```
* time.Time
* time.Now()
* Time.Add(time.Duration)
	* 引き算の場合はマイナス値を使う(Time.Subではないので注意)
```go
time.Now().Add(time.Hour)
time.Now().Add(-1 * time.Hour)
```
* Time.Sub(time.Time)
	* ２つのtime.Timeの差

