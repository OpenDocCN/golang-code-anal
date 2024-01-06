# `grype\grype\matcher\portage\matcher_test.go`

```
package portage

import (
	"testing"  // 导入测试包
	"github.com/google/uuid"  // 导入用于生成 UUID 的包
	"github.com/stretchr/testify/assert"  // 导入断言包
	"github.com/stretchr/testify/require"  // 导入断言包

	"github.com/anchore/grype/grype/distro"  // 导入用于操作发行版信息的包
	"github.com/anchore/grype/grype/pkg"  // 导入用于操作软件包信息的包
	"github.com/anchore/grype/internal/stringutil"  // 导入用于操作字符串的包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入用于操作软件包信息的包

)

func TestMatcherPortage_Match(t *testing.T) {
	matcher := Matcher{}  // 创建 Matcher 对象
	p := pkg.Package{  // 创建 Package 对象
		ID:      pkg.ID(uuid.NewString()),  // 生成一个新的 UUID 作为 Package 的 ID
		Name:    "app-misc/neutron",  // 设置 Package 的名称
	// 定义软件包的版本和类型
	Version: "2014.1.3",
	Type:    syftPkg.PortagePkg,
	}

	// 创建一个新的发行版对象
	d, err := distro.New(distro.Gentoo, "", "")
	if err != nil {
		t.Fatal("could not create distro: ", err)
	}

	// 创建一个模拟的存储提供者对象
	store := newMockProvider()
	// 使用匹配器对象匹配存储提供者中的漏洞信息和发行版信息
	actual, err := matcher.Match(store, d, p)
	assert.NoError(t, err, "unexpected err from Match", err)

	// 断言实际匹配结果的数量为1
	assert.Len(t, actual, 1, "unexpected indirect matches count")

	// 创建一个集合用于存储找到的漏洞ID
	foundCVEs := stringutil.NewStringSet()
	// 遍历实际匹配结果，将漏洞ID添加到集合中
	for _, a := range actual {
		foundCVEs.Add(a.Vulnerability.ID)

		// 断言漏洞详情不为空
		require.NotEmpty(t, a.Details)
		// 使用断言检查 p.Name 和 a.Package.Name 是否相等，如果不相等则输出错误信息
		assert.Equal(t, p.Name, a.Package.Name, "failed to capture original package name")
		// 遍历 a.Details 数组，对每个 detail 进行断言检查其 matcher 是否与 matcher.Type() 相等，如果不相等则输出错误信息
		for _, detail := range a.Details {
			assert.Equal(t, matcher.Type(), detail.Matcher, "failed to capture matcher type")
		}
	}

	// 遍历字符串数组，对每个 id 进行检查
	for _, id := range []string{"CVE-2014-fake-2"} {
		// 如果 foundCVEs 不包含当前 id，则输出错误信息
		if !foundCVEs.Contains(id) {
			t.Errorf("missing discovered CVE: %s", id)
		}
	}
	// 如果测试失败，则输出已发现的 CVE 列表
	if t.Failed() {
		t.Logf("discovered CVES: %+v", foundCVEs)
	}
}
```