# `grype\grype\db\v4\pkg\resolver\java\resolver_test.go`

```
package java

import (
    "testing"

    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"

    grypePkg "github.com/anchore/grype/grype/pkg"
)

func TestResolver_Normalize(t *testing.T) {
    tests := []struct {
        packageName string
        normalized  string
    }{
        {
            packageName: "PyYAML",
            normalized:  "pyyaml",
        },
        {
            packageName: "oslo.concurrency",
            normalized:  "oslo.concurrency",
        },
        {
            packageName: "",
            normalized:  "",
        },
        {
            packageName: "test---1",
            normalized:  "test---1",
        },
        {
            packageName: "AbCd.-__.--.-___.__.--1234____----....XyZZZ",
            normalized:  "abcd.-__.--.-___.__.--1234____----....xyzzz",
        },
    }

    resolver := Resolver{} // 创建 Resolver 对象

    for _, test := range tests { // 遍历测试用例
        resolvedNames := resolver.Normalize(test.packageName) // 调用 Normalize 方法对包名进行规范化
        assert.Equal(t, resolvedNames, test.normalized) // 使用断言检查规范化后的包名是否符合预期
    }
}

func TestResolver_Resolve(t *testing.T) {
    tests := []struct {
        name     string
        pkg      grypePkg.Package
        resolved []string
    }

    resolver := Resolver{} // 创建 Resolver 对象

    for _, test := range tests { // 遍历测试用例
        t.Run(test.name, func(t *testing.T) { // 使用子测试运行每个测试用例
            resolvedNames := resolver.Resolve(test.pkg) // 调用 Resolve 方法解析包
            assert.ElementsMatch(t, resolvedNames, test.resolved) // 使用断言检查解析后的包名列表是否符合预期
        })
    }
}
```