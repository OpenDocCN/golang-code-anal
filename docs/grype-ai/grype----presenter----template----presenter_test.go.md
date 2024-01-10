# `grype\grype\presenter\template\presenter_test.go`

```
package template

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "flag"   // 导入 flag 包，用于解析命令行参数
    "os"     // 导入 os 包，提供操作系统函数
    "path"   // 导入 path 包，用于处理文件路径
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/stretchr/testify/assert"   // 导入 testify 断言包，用于编写测试断言
    "github.com/stretchr/testify/require"  // 导入 testify 断言包，用于编写测试断言

    "github.com/anchore/go-testutils"  // 导入测试工具包
    "github.com/anchore/grype/grype/presenter/internal"  // 导入内部 presenter 包
    "github.com/anchore/grype/grype/presenter/models"  // 导入 presenter 模型包
)

var update = flag.Bool("update", false, "update the *.golden files for template presenters")  // 定义命令行参数 update，用于更新模板 presenter 的 golden 文件

func TestPresenter_Present(t *testing.T) {
    _, matches, packages, context, metadataProvider, appConfig, dbStatus := internal.GenerateAnalysis(t, internal.ImageSource)  // 生成分析数据

    workingDirectory, err := os.Getwd()  // 获取当前工作目录
    if err != nil {  // 如果获取失败
        t.Fatal(err)  // 输出错误信息并终止测试
    }
    templateFilePath := path.Join(workingDirectory, "./test-fixtures/test.template")  // 拼接模板文件路径

    pb := models.PresenterConfig{  // 创建 PresenterConfig 结构体
        Matches:          matches,  // 匹配项
        Packages:         packages,  // 包信息
        Context:          context,   // 上下文
        MetadataProvider: metadataProvider,  // 元数据提供者
        AppConfig:        appConfig,  // 应用配置
        DBStatus:         dbStatus,   // 数据库状态
    }

    templatePresenter := NewPresenter(pb, templateFilePath)  // 创建模板 presenter

    var buffer bytes.Buffer  // 创建字节缓冲区
    if err := templatePresenter.Present(&buffer); err != nil {  // 调用模板 presenter 的 Present 方法
        t.Fatal(err)  // 输出错误信息并终止测试
    }

    actual := buffer.Bytes()  // 获取缓冲区中的字节数据

    if *update {  // 如果命令行参数 update 为真
        testutils.UpdateGoldenFileContents(t, actual)  // 更新 golden 文件内容
    }
    expected := testutils.GetGoldenFileContents(t)  // 获取 golden 文件内容

    assert.Equal(t, string(expected), string(actual))  // 断言实际结果与期望结果相等
}

func TestPresenter_SprigDate_Fails(t *testing.T) {
    _, matches, packages, context, metadataProvider, appConfig, dbStatus := internal.GenerateAnalysis(t, internal.ImageSource)  // 生成分析数据
    workingDirectory, err := os.Getwd()  // 获取当前工作目录
    require.NoError(t, err)  // 断言没有错误发生

    // this template has the generic sprig date function, which is intentionally not supported for security reasons
    templateFilePath := path.Join(workingDirectory, "./test-fixtures/test.template.sprig.date")  // 拼接模板文件路径
}
    # 创建一个 PresenterConfig 结构体实例，包含匹配项、包、上下文、元数据提供者、应用配置和数据库状态
    pb := models.PresenterConfig{
        Matches:          matches,
        Packages:         packages,
        Context:          context,
        MetadataProvider: metadataProvider,
        AppConfig:        appConfig,
        DBStatus:         dbStatus,
    }

    # 使用 PresenterConfig 实例和模板文件路径创建一个模板 Presenter 实例
    templatePresenter := NewPresenter(pb, templateFilePath)

    # 创建一个字节缓冲区
    var buffer bytes.Buffer
    # 使用模板 Presenter 实例将内容呈现到缓冲区中
    err = templatePresenter.Present(&buffer)
    # 断言错误信息中包含特定的字符串
    require.ErrorContains(t, err, `function "now" not defined`)
# 闭合前面的函数定义
```