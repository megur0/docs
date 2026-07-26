---
title: "ログ - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > ログ




# スタックトレース
* Go1.13の時にxerrorsから輸入されたが、リリース前に消された、という経緯がある。
* https://zenn.dev/nekoshita/articles/097e00c6d3d1c9


# ログの自前実装 の 参考にしたもの。
* https://zenn.dev/tharu/articles/8c2ec139615fc4


# slog
* https://zenn.dev/88888888_kota/articles/7e97ff874083cf



# logrus
* logrus.Infolnなどは、内部でstd.Infoln(args...)していて、
    * 最終的に、Logger.Out.Writeが実行されている。
* このstdは下記のように初期化されており、つまり出力先はデフォで標準エラー出力になっている、というわけだ。（logパッケージと一緒。）
    * 標準エラー出力や標準出力はその実行環境によるが、
    * たとえばtty（ターミナルなど）とか、特定のログファイルが出力先になっている感じ。
```go
var (
	// std is the name of the standard logger in stdlib `log`
	std = New()
)
func New() *Logger {
	return &Logger{
		Out:          os.Stderr, //os.Stderrは、io.Writerを実装。
		Formatter:    new(TextFormatter),
		Hooks:        make(LevelHooks),
		Level:        InfoLevel,
		ExitFunc:     os.Exit,
		ReportCaller: false,
	}
}
```
# echoでlogrus
* https://echo.labstack.com/middleware/logger/

  


