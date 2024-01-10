# `grype\grype\matcher\portage\matcher_test.go`

```
package portage

import (
    "testing"  // 导入测试包
    "github.com/google/uuid"  // 导入用于生成 UUID 的包
    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包

    "github.com/anchore/grype/grype/distro"  // 导入发行版信息包
    "github.com/anchore/grype/grype/pkg"  // 导入包信息包
    "github.com/anchore/grype/internal/stringutil"  // 导入字符串工具包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syft 包

)

func TestMatcherPortage_Match(t *testing.T) {
    matcher := Matcher{}  // 创建 Matcher 对象
    p := pkg.Package{  // 创建包对象
        ID:      pkg.ID(uuid.NewString()),  // 生成 UUID 作为包的 ID
        Name:    "app-misc/neutron",  // 设置包的名称
        Version: "2014.1.3",  // 设置包的版本
        Type:    syftPkg.PortagePkg,  // 设置包的类型
    }

    d, err := distro.New(distro.Gentoo, "", "")  // 创建 Gentoo 发行版信息
    if err != nil {  // 如果创建发行版信息出错
        t.Fatal("could not create distro: ", err)  // 输出错误信息并终止测试
    }

    store := newMockProvider()  // 创建模拟的提供者对象
    actual, err := matcher.Match(store, d, p)  // 使用 Matcher 对象进行匹配
    assert.NoError(t, err, "unexpected err from Match", err)  // 断言匹配过程中没有错误

    assert.Len(t, actual, 1, "unexpected indirect matches count")  // 断言实际匹配结果的数量为 1

    foundCVEs := stringutil.NewStringSet()  // 创建字符串集合用于存储发现的 CVE
    for _, a := range actual {  // 遍历实际匹配结果
        foundCVEs.Add(a.Vulnerability.ID)  // 将发现的 CVE 添加到集合中

        require.NotEmpty(t, a.Details)  // 断言匹配结果的详情不为空
        assert.Equal(t, p.Name, a.Package.Name, "failed to capture original package name")  // 断言匹配结果的包名称与原始包名称相同
        for _, detail := range a.Details {  // 遍历匹配结果的详情
            assert.Equal(t, matcher.Type(), detail.Matcher, "failed to capture matcher type")  // 断言匹配结果的类型与 Matcher 类型相同
        }
    }

    for _, id := range []string{"CVE-2014-fake-2"} {  // 遍历预期的 CVE ID
        if !foundCVEs.Contains(id) {  // 如果发现的 CVE 集合中不包含预期的 CVE ID
            t.Errorf("missing discovered CVE: %s", id)  // 输出缺失的 CVE ID
        }
    }
    if t.Failed() {  // 如果测试失败
        t.Logf("discovered CVES: %+v", foundCVEs)  // 输出发现的 CVE 集合
    }
}
```