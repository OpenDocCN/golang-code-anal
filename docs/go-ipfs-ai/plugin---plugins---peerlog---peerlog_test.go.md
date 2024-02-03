# `kubo\plugin\plugins\peerlog\peerlog_test.go`

```go
package peerlog

import "testing"

func TestExtractEnabled(t *testing.T) {
    // 遍历测试用例
    for _, c := range []struct {
        name     string
        config   interface{}
        expected bool
    }{
        // 测试用例1：当配置为nil时返回false
        {
            name:     "nil config returns false",
            config:   nil,
            expected: false,
        },
        // 测试用例2：当配置不是字符串映射时返回false
        {
            name:     "returns false when config is not a string map",
            config:   1,
            expected: false,
        },
        // 测试用例3：当配置没有Enabled字段时返回false
        {
            name:     "returns false when config has no Enabled field",
            config:   map[string]interface{}{},
            expected: false,
        },
        // 测试用例4：当配置的Enabled字段为null时返回false
        {
            name:     "returns false when config has a null Enabled field",
            config:   map[string]interface{}{"Enabled": nil},
            expected: false,
        },
        // 测试用例5：当配置的Enabled字段为非布尔值时返回false
        {
            name:     "returns false when config has a non-boolean Enabled field",
            config:   map[string]interface{}{"Enabled": 1},
            expected: false,
        },
        // 测试用例6：返回Enabled字段的值
        {
            name:     "returns the value of the Enabled field",
            config:   map[string]interface{}{"Enabled": true},
            expected: true,
        },
    } {
        // 运行测试用例
        t.Run(c.name, func(t *testing.T) {
            // 提取Enabled字段的值
            isEnabled := extractEnabled(c.config)
            // 检查提取的值是否符合预期
            if isEnabled != c.expected {
                t.Fatalf("expected %v, got %v", c.expected, isEnabled)
            }
        })
    }
}
```