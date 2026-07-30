---
title: "uuid - Go"
updated: 2026-07-25
---

[TOP(About this memo))](../README.md) > [一覧(Go)](./README.md) > uuid




## サンプルコード
```go
import (
	"testing"

	"github.com/google/uuid"
	"github.com/stretchr/testify/assert"
)

func GetLast(a ...any) any {
	return a[len(a)-1]
}

func TestUuid(t *testing.T) {
	t.Run("assert_uuid", func(t *testing.T) {
		// 直接作成する場合、parseに失敗するとpanicとなる。
		//u := uuid.UUID([]byte("aaaa"))

		assert.NotEqual(t, GetLast(uuid.Parse("aaaa")), nil)
		assert.Exactly(t, uuid.Nil.String(), "00000000-0000-0000-0000-000000000000")
	})
}

```