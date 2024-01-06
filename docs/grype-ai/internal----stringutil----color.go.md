# `grype\internal\stringutil\color.go`

```
// 定义一个名为 stringutil 的包，用于字符串处理
package stringutil

// 导入 fmt 包，用于格式化输出
import "fmt"

// 定义颜色常量
const (
	DefaultColor Color = iota + 30
	Red
	Green
	Yellow
	Blue
	Magenta
	Cyan
	White
)

// 定义颜色类型
type Color uint8

// 格式化输出字符串并添加颜色，不跨平台（例如 windows）
func (c Color) Format(s string) string {
	return fmt.Sprintf("\x1b[%dm%s\x1b[0m", c, s)
}
这是一个代码块的结束标记，表示前面的函数或者循环的结束。
```