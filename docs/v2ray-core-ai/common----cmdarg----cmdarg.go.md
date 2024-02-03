# `v2ray-core\common\cmdarg\cmdarg.go`

```go
// 导入字符串处理包
import "strings"

// 定义 Arg 结构体，用于 flag 包接受多个参数
type Arg []string

// 实现 String 方法，将 Arg 结构体转换为字符串
func (c *Arg) String() string {
    return strings.Join([]string(*c), " ")
}

// 实现 Set 方法，用于 flag 包调用，向 Arg 结构体中添加数值
func (c *Arg) Set(value string) error {
    *c = append(*c, value)
    return nil
}
```