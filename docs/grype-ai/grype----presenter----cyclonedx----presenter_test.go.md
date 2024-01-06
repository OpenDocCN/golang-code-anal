# `grype\grype\presenter\cyclonedx\presenter_test.go`

```
package cyclonedx
# 导入所需的包

import (
	"bytes"
	"flag"
	"testing"

	"github.com/stretchr/testify/require"

	"github.com/anchore/clio"
	"github.com/anchore/go-testutils"
	"github.com/anchore/grype/grype/presenter/internal"
	"github.com/anchore/grype/grype/presenter/models"
)
# 引入所需的包

var update = flag.Bool("update", false, "update the *.golden files for cyclonedx presenters")
# 定义一个命令行标志，用于更新 cyclonedx 展示器的 *.golden 文件

func TestCycloneDxPresenterImage(t *testing.T) {
# 定义一个测试函数，用于测试 CycloneDxPresenterImage

	var buffer bytes.Buffer
# 声明一个字节缓冲区变量
# 从内部生成分析结果，返回的结果包括 sbom, matches, packages, context, metadataProvider, 以及两个未使用的变量
sbom, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.ImageSource)

# 创建一个 PresenterConfig 结构体对象 pb，包括 ID、Matches、Packages、Context、MetadataProvider 和 SBOM 字段
pb := models.PresenterConfig{
    ID: clio.Identification{
        Name:    "grype",
        Version: "[not provided]",
    },
    Matches:          matches,
    Packages:         packages,
    Context:          context,
    MetadataProvider: metadataProvider,
    SBOM:             sbom,
}

# 使用 PresenterConfig 结构体对象 pb 创建一个 JSON 格式的 Presenter 对象 pres
pres := NewJSONPresenter(pb)

# 运行 Presenter 对象，将结果输出到 buffer 中
err := pres.Present(&buffer)
if err != nil {
    # 如果出现错误，输出错误信息并终止测试
    t.Fatal(err)
}
// 将缓冲区内容转换为字节流
actual := buffer.Bytes()
// 如果更新标志为真，则更新测试用例的黄金文件内容
if *update {
    testutils.UpdateGoldenFileContents(t, actual)
}

// 获取黄金文件的内容
var expected = testutils.GetGoldenFileContents(t)

// 移除动态值，这些值会单独进行测试
actual = internal.Redact(actual)
expected = internal.Redact(expected)

// 使用 require.JSONEq 比较两个 JSON 字符串是否相等
require.JSONEq(t, string(expected), string(actual))
}

// 测试 CycloneDxPresenterDir 函数
func TestCycloneDxPresenterDir(t *testing.T) {
    // 创建一个字节缓冲区
    var buffer bytes.Buffer
    // 生成分析结果
    sbom, matches, packages, ctx, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.DirectorySource)
    // 创建 PresenterConfig 结构体
    pb := models.PresenterConfig{
        ID: clio.Identification{
            Name:    "grype",
		// 创建一个新的结构体实例，包含给定的字段和值
		pb := &PresenterBundle{
			Version: "[not provided]",
		},
		// 设置匹配项
		Matches:          matches,
		// 设置包
		Packages:         packages,
		// 设置上下文
		Context:          ctx,
		// 设置元数据提供者
		MetadataProvider: metadataProvider,
		// 设置软件构建物料清单
		SBOM:             sbom,
	}

	// 创建一个新的 JSON 格式的 Presenter 实例
	pres := NewJSONPresenter(pb)

	// 运行 Presenter
	err := pres.Present(&buffer)
	// 如果出现错误，记录并终止测试
	if err != nil {
		t.Fatal(err)
	}

	// 获取实际结果的字节流
	actual := buffer.Bytes()
	// 如果需要更新测试数据，更新黄金文件内容
	if *update {
		testutils.UpdateGoldenFileContents(t, actual)
	}
    // 获取预期结果的黄金文件内容
	var expected = testutils.GetGoldenFileContents(t)

	// 移除动态值，这些值会单独进行测试
    // 对实际结果进行数据脱敏处理
	actual = internal.Redact(actual)
    // 对预期结果进行数据脱敏处理
	expected = internal.Redact(expected)

    // 使用 JSONEq 断言比较实际结果和预期结果
	require.JSONEq(t, string(expected), string(actual))
}
```