# `grype\grype\presenter\cyclonedx\presenter_test.go`

```go
package cyclonedx

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "flag" // 导入 flag 包，用于解析命令行参数
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/stretchr/testify/require" // 导入 testify 包，用于断言测试结果

    "github.com/anchore/clio" // 导入 clio 包
    "github.com/anchore/go-testutils" // 导入 go-testutils 包
    "github.com/anchore/grype/grype/presenter/internal" // 导入 grype 内部 presenter 包
    "github.com/anchore/grype/grype/presenter/models" // 导入 presenter 模型包
)

var update = flag.Bool("update", false, "update the *.golden files for cyclonedx presenters") // 定义命令行参数 update

func TestCycloneDxPresenterImage(t *testing.T) {
    var buffer bytes.Buffer // 声明一个字节流缓冲区变量 buffer

    // 生成分析结果
    sbom, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.ImageSource)
    pb := models.PresenterConfig{ // 创建 PresenterConfig 结构体
        ID: clio.Identification{ // 设置 ID 字段
            Name:    "grype",
            Version: "[not provided]",
        },
        Matches:          matches, // 设置 Matches 字段
        Packages:         packages, // 设置 Packages 字段
        Context:          context, // 设置 Context 字段
        MetadataProvider: metadataProvider, // 设置 MetadataProvider 字段
        SBOM:             sbom, // 设置 SBOM 字段
    }

    pres := NewJSONPresenter(pb) // 创建 JSONPresenter 实例
    // 运行 presenter
    err := pres.Present(&buffer) // 调用 Present 方法，将结果写入 buffer
    if err != nil { // 如果出现错误
        t.Fatal(err) // 终止测试并输出错误信息
    }

    actual := buffer.Bytes() // 获取 buffer 中的字节流
    if *update { // 如果命令行参数 update 为 true
        testutils.UpdateGoldenFileContents(t, actual) // 更新 golden 文件内容
    }

    var expected = testutils.GetGoldenFileContents(t) // 获取 golden 文件内容

    // 移除动态值，这些值会单独进行测试
    actual = internal.Redact(actual) // 调用 Redact 方法，移除动态值
    expected = internal.Redact(expected) // 调用 Redact 方法，移除动态值

    require.JSONEq(t, string(expected), string(actual)) // 断言两个 JSON 字符串是否相等
}

func TestCycloneDxPresenterDir(t *testing.T) {
    var buffer bytes.Buffer // 声明一个字节流缓冲区变量 buffer
    sbom, matches, packages, ctx, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.DirectorySource) // 生成分析结果
    pb := models.PresenterConfig{ // 创建 PresenterConfig 结构体
        ID: clio.Identification{ // 设置 ID 字段
            Name:    "grype",
            Version: "[not provided]",
        },
        Matches:          matches, // 设置 Matches 字段
        Packages:         packages, // 设置 Packages 字段
        Context:          ctx, // 设置 Context 字段
        MetadataProvider: metadataProvider, // 设置 MetadataProvider 字段
        SBOM:             sbom, // 设置 SBOM 字段
    }

    pres := NewJSONPresenter(pb) // 创建 JSONPresenter 实例
    // 运行 presenter
    // 使用Present方法将buffer中的内容呈现出来，并将错误信息赋给err变量
    err := pres.Present(&buffer)
    // 如果err不为空，则输出错误信息并终止测试
    if err != nil {
        t.Fatal(err)
    }

    // 将buffer中的内容转换为字节流并赋给actual变量
    actual := buffer.Bytes()
    // 如果update标记为true，则更新golden文件的内容
    if *update {
        testutils.UpdateGoldenFileContents(t, actual)
    }

    // 从golden文件中获取预期的内容并赋给expected变量
    var expected = testutils.GetGoldenFileContents(t)

    // 移除被独立测试的动态值
    actual = internal.Redact(actual)
    expected = internal.Redact(expected)

    // 使用require.JSONEq方法比较expected和actual的JSON字符串表示形式是否相等
    require.JSONEq(t, string(expected), string(actual))
# 闭合前面的函数定义
```