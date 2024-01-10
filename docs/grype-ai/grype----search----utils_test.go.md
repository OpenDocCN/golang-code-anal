# `grype\grype\search\utils_test.go`

```
package search

import (
    "testing"  // 导入测试包

    "github.com/go-test/deep"  // 导入深度比较包
    "github.com/stretchr/testify/assert"  // 导入断言包

    "github.com/anchore/grype/grype/match"  // 导入匹配包
    "github.com/anchore/grype/grype/vulnerability"  // 导入漏洞包
)

func assertMatchesUsingIDsForVulnerabilities(t testing.TB, expected, actual []match.Match) {
    t.Helper()  // 标记当前测试函数为辅助函数
    assert.Len(t, actual, len(expected))  // 使用断言检查实际匹配数量是否与预期匹配数量相同
    for idx, a := range actual {
        // 只比较漏洞的 ID，不比较其他内容
        a.Vulnerability = vulnerability.Vulnerability{ID: a.Vulnerability.ID}
        for _, d := range deep.Equal(expected[idx], a) {
            t.Errorf("diff idx=%d: %+v", idx, d)  // 输出不同之处的索引和详细信息
        }
    }
}
```