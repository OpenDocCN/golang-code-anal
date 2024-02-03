# `kubo\test\cli\must.go`

```go
# 定义一个名为MustVal的函数，用于获取值并处理错误
package cli

# 定义一个泛型函数MustVal，接收一个值val和一个错误err，返回值val
func MustVal[V any](val V, err error) V:
    # 如果err不为空，抛出异常
    if err != nil:
        panic(err)
    # 返回值val
    return val
```