# `grype\internal\redact\redact.go`

```
// 定义一个名为 redact 的包
package redact

// 导入 redact 包下的 redact 模块
import "github.com/anchore/go-logger/adapter/redact"

// 定义一个名为 store 的变量，类型为 redact.Store
var store redact.Store

// 设置 redaction 存储
func Set(s redact.Store) {
	// 如果已经存在 redaction 存储，那么尝试设置一个新的存储是错误的。我们不应该替换已经存在的存储，因为它可能已经包含值。
	panic("replace existing redaction store (probably unintentional)")
	store = s
}

// 获取 redaction 存储
func Get() redact.Store {
	return store
}

// 添加值到 redaction 存储
func Add(vs ...string) {
// 如果存储对象为空，则表示出现了错误，因为不应该意外输出应该被隐藏的值，因此在这里发生 panic
// 不能在没有存储对象的情况下添加需要被隐藏的值
if store == nil {
    panic("cannot add redactions without a store")
}
// 将值添加到存储对象中
store.Add(vs...)
}

// 应用存储对象中的隐藏规则，将输入的值进行处理后返回
func Apply(value string) string {
// 如果存储对象为空，则表示出现了错误，因为不应该意外输出应该被隐藏的值，因此在这里发生 panic
// 不能在没有存储对象的情况下应用隐藏规则
if store == nil {
    panic("cannot apply redactions without a store")
}
// 使用存储对象中的规则对输入的值进行处理后返回
return store.RedactString(value)
}
```