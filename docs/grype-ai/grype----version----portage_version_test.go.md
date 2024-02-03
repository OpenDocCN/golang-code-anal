# `grype\grype\version\portage_version_test.go`

```go
// 导入测试包
import (
    "testing"
)

// 定义版本比较测试函数
func TestVersionPortage(t *testing.T) {
    // 定义版本比较测试用例
    tests := []struct {
        v1     string  // 版本1
        v2     string  // 版本2
        result int     // 期望结果
    }{
        {"1", "1", 0},           // 版本相同
        {"12.2.5", "12.2b", 1},  // 版本1大于版本2
        {"12.2a", "12.2b", -1},  // 版本1小于版本2
        {"12.2", "12.2.0", -1},  // 版本1小于版本2
        {"1.01", "1.1", -1},     // 版本1小于版本2
        {"1_p1", "1_p0", 1},     // 版本1大于版本2
        {"1_p0", "1", 1},        // 版本1大于版本2
        {"1-r1", "1", 1},        // 版本1大于版本2
        {"1.2.3-r2", "1.2.3-r1", 1},  // 版本1大于版本2
        {"1.2.3-r1", "1.2.3-r2", -1},  // 版本1小于版本2
    }

    // 遍历测试用例
    for _, test := range tests {
        // 拼接测试名称
        name := test.v1 + "_vs_" + test.v2
        // 运行单个测试用例
        t.Run(name, func(t *testing.T) {
            // 创建版本对象
            v1 := newPortageVersion(test.v1)
            v2 := newPortageVersion(test.v2)

            // 执行版本比较
            actual := v1.compare(v2)

            // 检查实际结果是否与期望结果一致
            if actual != test.result {
                t.Errorf("bad result: %+v (expected: %+v)", actual, test.result)
            }
        })
    }
}
```