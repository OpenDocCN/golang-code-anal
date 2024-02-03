# `grype\grype\db\v5\pkg\resolver\java\resolver_test.go`

```go
package java
// 导入所需的包
import (
    "testing" // 测试包

    "github.com/google/uuid" // UUID 生成包
    "github.com/stretchr/testify/assert" // 断言包

    grypePkg "github.com/anchore/grype/grype/pkg" // 引入 grypePkg 别名

)

// 测试函数：Normalize
func TestResolver_Normalize(t *testing.T) {
    // 测试用例
    tests := []struct {
        packageName string // 包名
        normalized  string // 标准化后的包名
    }{
        {
            packageName: "PyYAML", // 包名
            normalized:  "pyyaml", // 标准化后的包名
        },
        {
            packageName: "oslo.concurrency", // 包名
            normalized:  "oslo.concurrency", // 标准化后的包名
        },
        {
            packageName: "", // 空包名
            normalized:  "", // 空
        },
        {
            packageName: "test---1", // 包名
            normalized:  "test---1", // 标准化后的包名
        },
        {
            packageName: "AbCd.-__.--.-___.__.--1234____----....XyZZZ", // 包名
            normalized:  "abcd.-__.--.-___.__.--1234____----....xyzzz", // 标准化后的包名
        },
    }

    resolver := Resolver{} // 创建 Resolver 对象

    // 遍历测试用例
    for _, test := range tests {
        resolvedNames := resolver.Normalize(test.packageName) // 标准化包名
        assert.Equal(t, resolvedNames, test.normalized) // 断言标准化后的包名是否符合预期
    }
}

// 测试函数：Resolve
func TestResolver_Resolve(t *testing.T) {
    // 测试用例
    tests := []struct {
        name     string // 名称
        pkg      grypePkg.Package // grypePkg 包
        resolved []string // 解析后的结果
    }

    resolver := Resolver{} // 创建 Resolver 对象

    // 遍历测试用例
    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            resolvedNames := resolver.Resolve(test.pkg) // 解析包
            assert.ElementsMatch(t, resolvedNames, test.resolved) // 断言解析后的结果是否符合预期
        })
    }
}
```