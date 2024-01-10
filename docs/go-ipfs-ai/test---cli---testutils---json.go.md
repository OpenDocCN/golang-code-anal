# `kubo\test\cli\testutils\json.go`

```
# 定义 testutils 包
package testutils

# 导入 encoding/json 包
import "encoding/json"

# 定义 JSONObj 类型，是一个键为字符串、值为接口的映射
type JSONObj map[string]interface{}

# 定义 ToJSONStr 函数，将 JSONObj 对象转换为 JSON 字符串
func ToJSONStr(m JSONObj) string:
    # 使用 json.Marshal 将 JSONObj 对象转换为 JSON 字节流
    b, err := json.Marshal(m)
    # 如果转换过程中出现错误，抛出异常
    if err != nil:
        panic(err)
    # 将 JSON 字节流转换为字符串并返回
    return string(b)
```