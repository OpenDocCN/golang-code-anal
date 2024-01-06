# `grype\grype\vex\openvex\implementation_test.go`

```
package openvex

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/require"  // 导入断言包
)

func TestIdentifiersFromDigests(t *testing.T) {  // 定义测试函数
	for _, tc := range []struct {  // 遍历测试用例
		sut      string  // 定义被测对象
		expected []string  // 定义预期结果
	}{
		{  // 测试用例1
			"alpine@sha256:124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126",  // 设置被测对象
			[]string{  // 设置预期结果
				"alpine@sha256:124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126",
				"pkg:oci/alpine@sha256%3A124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126?repository_url=index.docker.io/library",
				"124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126",
			},  // 结束测试用例1
# 循环遍历测试用例
for _, tc := range []struct {
    # 测试用例的被测对象
    "cgr.dev/chainguard/curl@sha256:9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc",
    # 期望的结果
    []string{
        "cgr.dev/chainguard/curl@sha256:9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc",
        "pkg:oci/curl@sha256%3A9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc?repository_url=cgr.dev/chainguard",
        "9543ed09a38605c25c75486573cf530bd886615b993d5e1d1aa58fe5491287bc",
    },
},
{
    # 测试用例的被测对象
    "alpine",
    # 期望的结果
    []string{"alpine"},
} {
    # 调用被测函数，获取结果
    res := identifiersFromDigests([]string{tc.sut})
    # 断言结果是否符合期望
    require.Equal(t, tc.expected, res)
}
```