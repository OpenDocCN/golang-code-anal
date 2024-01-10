# `grype\grype\match\explicit_ignores_test.go`

```
package match

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包

    "github.com/anchore/grype/grype/pkg"  // 导入 grype 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入漏洞包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 包并重命名为 syftPkg
)

type mockExclusionProvider struct {
    data map[string][]IgnoreRule  // 定义 mockExclusionProvider 结构体，包含一个数据字段
}

func newMockExclusionProvider() *mockExclusionProvider {
    d := mockExclusionProvider{  // 创建 mockExclusionProvider 实例
        data: make(map[string][]IgnoreRule),  // 初始化数据字段
    }
    d.stub()  // 调用 stub 方法
    return &d  // 返回 mockExclusionProvider 实例的指针
}

func (d *mockExclusionProvider) stub() {
}

func (d *mockExclusionProvider) GetRules(vulnerabilityID string) ([]IgnoreRule, error) {
    return d.data[vulnerabilityID], nil  // 返回指定漏洞 ID 对应的忽略规则
}

func Test_ApplyExplicitIgnoreRules(t *testing.T) {
    type cvePkg struct {
        cve string
        pkg string
    }
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        typ      syftPkg.Type  // 包类型
        matches  []cvePkg  // 匹配的漏洞包
        expected []string  // 期望结果
        ignored  []string  // 忽略的结果
    }

    p := newMockExclusionProvider()  // 创建 mockExclusionProvider 实例
}
    # 遍历测试用例
    for _, test := range tests {
        # 使用测试名称创建子测试
        t.Run(test.name, func(t *testing.T) {
            # 创建一个新的匹配对象
            matches := NewMatches()

            # 遍历测试用例中的匹配项
            for _, cp := range test.matches {
                # 将匹配项添加到匹配对象中
                matches.Add(Match{
                    # 设置包的ID、名称和类型
                    Package: pkg.Package{
                        ID:   pkg.ID(cp.pkg),
                        Name: cp.pkg,
                        Type: test.typ,
                    },
                    # 设置漏洞的ID
                    Vulnerability: vulnerability.Vulnerability{
                        ID: cp.cve,
                    },
                })
            }

            # 应用显式忽略规则，获取过滤后的匹配项和被忽略的匹配项
            filtered, ignores := ApplyExplicitIgnoreRules(p, matches)

            # 创建一个空数组用于存储找到的包名
            var found []string
            # 遍历过滤后的匹配项
            for match := range filtered.Enumerate() {
                # 将找到的包名添加到数组中
                found = append(found, match.Package.Name)
            }
            # 断言找到的包名与预期的包名数组相匹配
            assert.ElementsMatch(t, test.expected, found)

            # 如果有被忽略的包名
            if len(test.ignored) > 0 {
                # 创建一个空数组用于存储被忽略的包名
                var ignored []string
                # 遍历被忽略的匹配项
                for _, i := range ignores {
                    # 将被忽略的包名添加到数组中
                    ignored = append(ignored, i.Package.Name)
                }
                # 断言被忽略的包名与预期的被忽略包名数组相匹配
                assert.ElementsMatch(t, test.ignored, ignored)
            } else {
                # 如果没有被忽略的包名，断言忽略列表为空
                assert.Empty(t, ignores)
            }
        })
    }
# 闭合前面的函数定义
```