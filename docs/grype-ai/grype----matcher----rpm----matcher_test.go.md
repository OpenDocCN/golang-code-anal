# `grype\grype\matcher\rpm\matcher_test.go`

```
package rpm

import (
    "testing"  // 导入测试包

    "github.com/google/uuid"  // 导入 Google UUID 包
    "github.com/stretchr/testify/assert"  // 导入 testify 断言包
    "github.com/stretchr/testify/require"  // 导入 testify 需求包

    "github.com/anchore/grype/grype/distro"  // 导入 distro 包
    "github.com/anchore/grype/grype/match"  // 导入 match 包
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
)

func intRef(x int) *int {
    return &x  // 返回整数的指针
}

func TestMatcherRpm(t *testing.T) {
    tests := []struct {
        name            string  // 测试名称
        p               pkg.Package  // 包对象
        setup           func() (vulnerability.Provider, *distro.Distro, Matcher)  // 设置函数返回漏洞提供者、发行版和匹配器
        expectedMatches map[string]match.Type  // 期望的匹配结果
        wantErr         bool  // 是否期望错误
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试
            store, d, matcher := test.setup()  // 获取漏洞存储、发行版和匹配器
            actual, err := matcher.Match(store, d, test.p)  // 进行匹配
            if err != nil {  // 如果有错误
                t.Fatal("could not find match: ", err)  // 输出错误信息
            }

            assert.Len(t, actual, len(test.expectedMatches), "unexpected matches count")  // 断言匹配结果数量

            for _, a := range actual {  // 遍历匹配结果
                if val, ok := test.expectedMatches[a.Vulnerability.ID]; !ok {  // 如果匹配结果不在期望的结果中
                    t.Errorf("return unknown match CVE: %s", a.Vulnerability.ID)  // 输出未知匹配结果的 CVE
                    continue
                } else {
                    require.NotEmpty(t, a.Details)  // 断言匹配结果的详情不为空
                    for _, de := range a.Details {  // 遍历匹配结果的详情
                        assert.Equal(t, val, de.Type)  // 断言匹配结果的类型与期望的类型相等
                    }
                }

                assert.Equal(t, test.p.Name, a.Package.Name, "failed to capture original package name")  // 断言匹配结果的包名称与原始包名称相等
                for _, detail := range a.Details {  // 遍历匹配结果的详情
                    assert.Equal(t, matcher.Type(), detail.Matcher, "failed to capture matcher type")  // 断言匹配结果的类型与匹配器类型相等
                }
            }

            if t.Failed() {  // 如果测试失败
                t.Logf("discovered CVES: %+v", actual)  // 输出发现的 CVE
            }
        })
    }
}

func Test_addZeroEpicIfApplicable(t *testing.T) {
    # 定义测试用例的结构体数组，包含版本号和期望结果
    tests := []struct {
        version  string
        expected string
    }{
        {
            version:  "3.26.0-6.el8",
            expected: "0:3.26.0-6.el8",
        },
        {
            version:  "7:3.26.0-6.el8",
            expected: "7:3.26.0-6.el8",
        },
    }
    # 遍历测试用例数组
    for _, test := range tests {
        # 使用测试版本号作为子测试的名称
        t.Run(test.version, func(t *testing.T) {
            # 调用被测试的函数，获取实际版本号
            actualVersion := addZeroEpicIfApplicable(test.version)
            # 断言实际版本号与期望结果相等
            assert.Equal(t, test.expected, actualVersion)
        })
    }
# 闭合前面的函数定义
```