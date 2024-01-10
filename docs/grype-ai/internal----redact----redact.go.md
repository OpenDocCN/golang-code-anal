# `grype\internal\redact\redact.go`

```
package redact

import "github.com/anchore/go-logger/adapter/redact"

// 定义全局变量 store，用于存储 redact.Store
var store redact.Store

// 设置 redact.Store
func Set(s redact.Store) {
    // 如果已经存在 store，则抛出异常，因为不应该替换已有的 store
    if store != nil {
        panic("replace existing redaction store (probably unintentional)")
    }
    store = s
}

// 获取 redact.Store
func Get() redact.Store {
    return store
}

// 添加需要屏蔽的值到 redact.Store
func Add(vs ...string) {
    // 如果 store 为空，则抛出异常，因为不应该在没有 store 的情况下添加值
    if store == nil {
        panic("cannot add redactions without a store")
    }
    store.Add(vs...)
}

// 应用 redact.Store 中的屏蔽规则到给定的值
func Apply(value string) string {
    // 如果 store 为空，则抛出异常，因为不应该在没有 store 的情况下应用屏蔽规则
    if store == nil {
        panic("cannot apply redactions without a store")
    }
    return store.RedactString(value)
}
```