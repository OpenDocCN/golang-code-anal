# `grype\grype\presenter\template\presenter_test.go`

```
package template

import (
	"bytes"  // 导入 bytes 包，用于操作字节
	"flag"   // 导入 flag 包，用于命令行参数解析
	"os"     // 导入 os 包，提供操作系统函数
	"path"   // 导入 path 包，用于路径操作
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/stretchr/testify/assert"  // 导入 testify 断言包，用于编写测试断言
	"github.com/stretchr/testify/require"  // 导入 testify 断言包，用于编写测试断言

	"github.com/anchore/go-testutils"  // 导入 go-testutils 包，用于编写测试工具
	"github.com/anchore/grype/grype/presenter/internal"  // 导入 grype presenter 内部包
	"github.com/anchore/grype/grype/presenter/models"  // 导入 grype presenter 模型包
)

var update = flag.Bool("update", false, "update the *.golden files for template presenters")  // 定义一个布尔型标志，用于更新模板展示器的 *.golden 文件

func TestPresenter_Present(t *testing.T) {  // 定义一个测试函数
	// 使用内部函数生成分析所需的参数
	_, matches, packages, context, metadataProvider, appConfig, dbStatus := internal.GenerateAnalysis(t, internal.ImageSource)

	// 获取当前工作目录
	workingDirectory, err := os.Getwd()
	if err != nil {
		// 如果出现错误，记录并终止测试
		t.Fatal(err)
	}
	// 拼接模板文件的路径
	templateFilePath := path.Join(workingDirectory, "./test-fixtures/test.template")

	// 创建 PresenterConfig 对象
	pb := models.PresenterConfig{
		Matches:          matches,
		Packages:         packages,
		Context:          context,
		MetadataProvider: metadataProvider,
		AppConfig:        appConfig,
		DBStatus:         dbStatus,
	}

	// 使用 PresenterConfig 对象和模板文件路径创建 Presenter 对象
	templatePresenter := NewPresenter(pb, templateFilePath)

	// 创建一个字节缓冲区
	var buffer bytes.Buffer
// 如果模板展示器出现错误，则记录错误并终止测试
if err := templatePresenter.Present(&buffer); err != nil {
	t.Fatal(err)
}

// 获取实际结果的字节流
actual := buffer.Bytes()

// 如果需要更新测试数据，则更新测试数据文件
if *update {
	testutils.UpdateGoldenFileContents(t, actual)
}

// 获取预期结果的字节流
expected := testutils.GetGoldenFileContents(t)

// 断言实际结果和预期结果是否相等
assert.Equal(t, string(expected), string(actual))
}

// 测试 SprigDate 函数失败的情况
func TestPresenter_SprigDate_Fails(t *testing.T) {
	// 生成分析所需的参数
	_, matches, packages, context, metadataProvider, appConfig, dbStatus := internal.GenerateAnalysis(t, internal.ImageSource)
	// 获取当前工作目录
	workingDirectory, err := os.Getwd()
	require.NoError(t, err)

	// 此模板包含通用的 SprigDate 函数，出于安全原因故意不支持
# 将工作目录和模板文件路径拼接成完整的模板文件路径
templateFilePath := path.Join(workingDirectory, "./test-fixtures/test.template.sprig.date")

# 创建一个 PresenterConfig 结构体对象，并初始化其属性
pb := models.PresenterConfig{
    Matches:          matches,
    Packages:         packages,
    Context:          context,
    MetadataProvider: metadataProvider,
    AppConfig:        appConfig,
    DBStatus:         dbStatus,
}

# 使用 PresenterConfig 对象和模板文件路径创建一个新的 Presenter 对象
templatePresenter := NewPresenter(pb, templateFilePath)

# 创建一个字节缓冲区
var buffer bytes.Buffer

# 调用 Presenter 对象的 Present 方法，将结果存储到 buffer 中
err = templatePresenter.Present(&buffer)

# 断言错误信息中包含特定的字符串
require.ErrorContains(t, err, `function "now" not defined`)
```