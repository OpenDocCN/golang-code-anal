# `grype\grype\vex\openvex\implementation_test.go`

```go
package openvex

import (
    "testing"

    "github.com/stretchr/testify/require"
)

func TestIdentifiersFromDigests(t *testing.T) {
    for _, tc := range []struct {  // 遍历测试用例
        sut      string  // 定义被测字符串
        expected []string  // 定义期望结果数组
    }{
        {
            "alpine@sha256:124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126",  // 测试用例1
            []string{  // 期望结果数组1
                "alpine@sha256:124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126",
                "pkg:oci/alpine@sha256%3A124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126?repository_url=index.docker.io/library",
                "124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126",
            },
        },
        {
            "cgr.dev/chainguard/curl@sha256:9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc",  // 测试用例2
            []string{  // 期望结果数组2
                "cgr.dev/chainguard/curl@sha256:9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc",
                "pkg:oci/curl@sha256%3A9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc?repository_url=cgr.dev/chainguard",
                "9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc",
            },
        },
        {
            "alpine",  // 测试用例3
            []string{"alpine"},  // 期望结果数组3
        },
    } {
        res := identifiersFromDigests([]string{tc.sut})  // 调用函数计算结果
        require.Equal(t, tc.expected, res)  // 断言结果与期望值相等
    }
}
```