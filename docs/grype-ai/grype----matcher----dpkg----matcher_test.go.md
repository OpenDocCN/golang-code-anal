# `grype\grype\matcher\dpkg\matcher_test.go`

```go
package dpkg

import (
    "testing"  // 导入测试包

    "github.com/google/uuid"  // 导入 UUID 生成包
    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包

    "github.com/anchore/grype/grype/distro"  // 导入发行版包
    "github.com/anchore/grype/grype/match"  // 导入匹配包
    "github.com/anchore/grype/grype/pkg"  // 导入包管理包
    "github.com/anchore/grype/internal/stringutil"  // 导入字符串工具包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 包
)

func TestMatcherDpkg_matchBySourceIndirection(t *testing.T) {
    matcher := Matcher{}  // 创建 Matcher 对象
    p := pkg.Package{  // 创建包对象
        ID:      pkg.ID(uuid.NewString()),  // 设置包 ID
        Name:    "neutron",  // 设置包名称
        Version: "2014.1.3-6",  // 设置包版本
        Type:    syftPkg.DebPkg,  // 设置包类型
        Upstreams: []pkg.UpstreamPackage{  // 设置上游包列表
            {
                Name: "neutron-devel",  // 设置上游包名称
            },
        },
    }

    d, err := distro.New(distro.Debian, "8", "")  // 创建 Debian 发行版对象
    if err != nil {  // 如果发生错误
        t.Fatal("could not create distro: ", err)  // 输出错误信息并终止测试
    }

    store := newMockProvider()  // 创建模拟提供者对象
    actual, err := matcher.matchUpstreamPackages(store, d, p)  // 调用匹配上游包方法
    assert.NoError(t, err, "unexpected err from matchUpstreamPackages", err)  // 断言没有错误发生

    assert.Len(t, actual, 2, "unexpected indirect matches count")  // 断言实际结果长度为2

    foundCVEs := stringutil.NewStringSet()  // 创建字符串集合对象
    for _, a := range actual {  // 遍历实际结果
        foundCVEs.Add(a.Vulnerability.ID)  // 将漏洞 ID 添加到字符串集合中

        require.NotEmpty(t, a.Details)  // 断言详情不为空
        for _, d := range a.Details {  // 遍历详情
            assert.Equal(t, match.ExactIndirectMatch, d.Type, "indirect match not indicated")  // 断言间接匹配类型为 ExactIndirectMatch
        }
        assert.Equal(t, p.Name, a.Package.Name, "failed to capture original package name")  // 断言包名称匹配
        for _, detail := range a.Details {  // 遍历详情
            assert.Equal(t, matcher.Type(), detail.Matcher, "failed to capture matcher type")  // 断言匹配类型匹配
        }
    }

    for _, id := range []string{"CVE-2014-fake-2", "CVE-2013-fake-3"} {  // 遍历漏洞 ID 列表
        if !foundCVEs.Contains(id) {  // 如果字符串集合中不包含当前漏洞 ID
            t.Errorf("missing discovered CVE: %s", id)  // 输出缺失的漏洞 ID
        }
    }
    if t.Failed() {  // 如果测试失败
        t.Logf("discovered CVES: %+v", foundCVEs)  // 输出发现的漏洞 ID
    }
}
```