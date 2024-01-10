# `grype\internal\stringutil\color.go`

```
# 定义名为 stringutil 的包
package stringutil

# 导入 fmt 包
import "fmt"

# 定义常量 DefaultColor 为 Color 类型的枚举值，初始值为 30
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

# 定义类型 Color 为 uint8 类型
type Color uint8

# 定义 Color 类型的方法 Format，用于格式化字符串并添加颜色
# TODO: not cross platform (windows...)
func (c Color) Format(s string) string:
    # 返回格式化后的带颜色的字符串
    return fmt.Sprintf("\x1b[%dm%s\x1b[0m", c, s)
```