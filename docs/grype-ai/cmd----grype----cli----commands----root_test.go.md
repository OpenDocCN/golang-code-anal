# `grype\cmd\grype\cli\commands\root_test.go`

```
package commands

import (
	"testing"  // 导入测试包

	"github.com/stretchr/testify/assert"  // 导入断言包

	"github.com/anchore/grype/cmd/grype/cli/options"  // 导入命令行选项包
	"github.com/anchore/grype/grype/pkg"  // 导入 grype 包
)

func Test_applyDistroHint(t *testing.T) {
	ctx := pkg.Context{}  // 创建 pkg.Context 对象
	cfg := options.Grype{}  // 创建 options.Grype 对象

	applyDistroHint([]pkg.Package{}, &ctx, &cfg)  // 调用 applyDistroHint 函数，传入参数 pkg.Package 切片、pkg.Context 对象的指针和 options.Grype 对象的指针
	assert.Nil(t, ctx.Distro)  // 使用断言检查 ctx.Distro 是否为 nil

	// works when distro is nil
	cfg.Distro = "alpine:3.10"  // 设置 cfg.Distro 为 "alpine:3.10"
```
// 应用发行版提示到上下文中的发行版对象
applyDistroHint([]pkg.Package{}, &ctx, &cfg)
// 断言上下文中的发行版对象不为空
assert.NotNil(t, ctx.Distro)

// 断言发行版对象的名称为"alpine"
assert.Equal(t, "alpine", ctx.Distro.Name)
// 断言发行版对象的版本为"3.10"
assert.Equal(t, "3.10", ctx.Distro.Version)

// 覆盖现有的发行版对象
cfg.Distro = "ubuntu:latest"
applyDistroHint([]pkg.Package{}, &ctx, &cfg)
// 断言上下文中的发行版对象不为空
assert.NotNil(t, ctx.Distro)

// 断言发行版对象的名称为"ubuntu"
assert.Equal(t, "ubuntu", ctx.Distro.Name)
// 断言发行版对象的版本为"latest"
assert.Equal(t, "latest", ctx.Distro.Version)

// 当发行版为空时不会移除现有的发行版对象
cfg.Distro = ""
applyDistroHint([]pkg.Package{}, &ctx, &cfg)
// 断言上下文中的发行版对象不为空
assert.NotNil(t, ctx.Distro)

// 断言发行版对象的名称仍然为"ubuntu"
assert.Equal(t, "ubuntu", ctx.Distro.Name)
# 使用断言来检查上下文中的发行版版本是否为"latest"
assert.Equal(t, "latest", ctx.Distro.Version)
```