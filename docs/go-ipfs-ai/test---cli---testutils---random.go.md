# `kubo\test\cli\testutils\random.go`

```
# 导入 testutils 包
package testutils

# 导入 crypto/rand 包
import "crypto/rand"

# 生成指定长度的随机字节序列
func RandomBytes(n int) []byte:
    # 创建一个长度为 n 的字节数组
    bytes := make([]byte, n)
    # 从加密随机数生成器中读取随机字节序列，将结果存储在字节数组中
    _, err := rand.Read(bytes)
    # 如果发生错误，则抛出异常
    if err != nil:
        panic(err)
    # 返回生成的随机字节序列
    return bytes

# 生成指定长度的随机字符串
func RandomStr(n int) string:
    # 将随机字节序列转换为字符串并返回
    return string(RandomBytes(n))
```