# `kubo\core\commands\config_test.go`

```go
# 导入测试模块
import "testing"

# 定义测试函数 TestScrubMapInternalDelete
func TestScrubMapInternalDelete(t *testing.T) {
    # 调用 scrubMapInternal 函数，传入参数 nil, nil, true
    m, err := scrubMapInternal(nil, nil, true)
    # 如果返回错误，输出错误信息
    if err != nil {
        t.Error(err)
    }
    # 如果返回的 map 为空，输出错误信息
    if m == nil {
        t.Errorf("expecting an empty map, got nil")
    }
    # 如果返回的 map 长度不为 0，输出错误信息
    if len(m) != 0 {
        t.Errorf("expecting an empty map, got a non-empty map")
    }
}
```