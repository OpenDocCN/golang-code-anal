# `grype\grype\presenter\json\presenter_test.go`

```go
package json

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "flag"   // 导入 flag 包，用于解析命令行参数
    "regexp" // 导入 regexp 包，用于正则表达式匹配
    "sort"   // 导入 sort 包，用于排序
    "testing"    // 导入 testing 包，用于单元测试

    "github.com/stretchr/testify/assert"  // 导入 testify 包，用于断言

    "github.com/anchore/clio"  // 导入 clio 包
    "github.com/anchore/go-testutils"  // 导入 go-testutils 包
    "github.com/anchore/grype/grype/match"  // 导入 grype 包中的 match 模块
    "github.com/anchore/grype/grype/pkg"    // 导入 grype 包中的 pkg 模块
    "github.com/anchore/grype/grype/presenter/internal"  // 导入 grype 包中的 presenter/internal 模块
    "github.com/anchore/grype/grype/presenter/models"    // 导入 grype 包中的 presenter/models 模块
    "github.com/anchore/syft/syft/linux"    // 导入 syft 包中的 linux 模块
    "github.com/anchore/syft/syft/source"   // 导入 syft 包中的 source 模块
)

var update = flag.Bool("update", false, "update the *.golden files for json presenters")  // 定义命令行参数 update
var timestampRegexp = regexp.MustCompile(`"timestamp":\s*"[^"]+"`)  // 定义正则表达式 timestampRegexp

func TestJsonImgsPresenter(t *testing.T) {
    var buffer bytes.Buffer  // 定义字节流 buffer
    _, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.ImageSource)  // 调用 GenerateAnalysis 函数获取分析结果

    pb := models.PresenterConfig{  // 定义 PresenterConfig 结构体
        ID: clio.Identification{  // 定义 Identification 结构体
            Name:    "grype",  // 设置名称
            Version: "[not provided]",  // 设置版本
        },
        Matches:          matches,  // 设置匹配结果
        Packages:         packages,  // 设置包信息
        Context:          context,   // 设置上下文
        MetadataProvider: metadataProvider,  // 设置元数据提供者
    }

    pres := NewPresenter(pb)  // 创建新的 Presenter 对象

    // 运行 presenter
    if err := pres.Present(&buffer); err != nil {  // 调用 Present 方法
        t.Fatal(err)  // 输出错误信息
    }
    actual := buffer.Bytes()  // 获取字节流内容
    actual = redact(actual)   // 对内容进行处理

    if *update {  // 如果命令行参数 update 为真
        testutils.UpdateGoldenFileContents(t, actual)  // 更新 golden 文件内容
    }

    var expected = testutils.GetGoldenFileContents(t)  // 获取 golden 文件内容

    assert.JSONEq(t, string(expected), string(actual))  // 使用断言比较内容

    // TODO: add me back in when there is a JSON schema
    // validateAgainstDbSchema(t, string(actual))
}

func TestJsonDirsPresenter(t *testing.T) {
    var buffer bytes.Buffer  // 定义字节流 buffer

    _, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.DirectorySource)  // 调用 GenerateAnalysis 函数获取分析结果
    # 创建一个 PresenterConfig 结构体实例，包含 ID、Matches、Packages、Context、MetadataProvider 等字段
    pb := models.PresenterConfig{
        ID: clio.Identification{
            Name:    "grype",
            Version: "[not provided]",
        },
        Matches:          matches,
        Packages:         packages,
        Context:          context,
        MetadataProvider: metadataProvider,
    }

    # 根据 PresenterConfig 实例创建一个 Presenter 实例
    pres := NewPresenter(pb)

    # 运行 Presenter 实例，将结果写入 buffer
    if err := pres.Present(&buffer); err != nil {
        t.Fatal(err)
    }
    # 获取 buffer 中的字节内容
    actual := buffer.Bytes()
    # 对 actual 进行数据脱敏处理
    actual = redact(actual)

    # 如果需要更新 golden 文件，则更新
    if *update {
        testutils.UpdateGoldenFileContents(t, actual)
    }

    # 获取预期的 golden 文件内容
    var expected = testutils.GetGoldenFileContents(t)

    # 断言实际结果与预期结果是否相等
    assert.JSONEq(t, string(expected), string(actual))

    # TODO: 当有 JSON schema 时，将验证 actual 是否符合数据库 schema
    # validateAgainstDbSchema(t, string(actual))
func TestEmptyJsonPresenter(t *testing.T) {
    // 期望得到一个空的 JSON 数组
    var buffer bytes.Buffer

    // 创建一个空的匹配对象
    matches := match.NewMatches()

    // 创建一个上下文对象
    ctx := pkg.Context{
        Source: &source.Description{},
        Distro: &linux.Release{
            ID:      "centos",
            IDLike:  []string{"rhel"},
            Version: "8.0",
        },
    }

    // 创建一个 PresenterConfig 对象
    pb := models.PresenterConfig{
        ID: clio.Identification{
            Name:    "grype",
            Version: "[not provided]",
        },
        Matches:          matches,
        Packages:         nil,
        Context:          ctx,
        MetadataProvider: nil,
    }

    // 创建一个 Presenter 对象
    pres := NewPresenter(pb)

    // 运行 Presenter
    if err := pres.Present(&buffer); err != nil {
        t.Fatal(err)
    }
    actual := buffer.Bytes()
    actual = redact(actual)

    // 如果需要更新，更新 golden 文件内容
    if *update {
        testutils.UpdateGoldenFileContents(t, actual)
    }

    // 获取预期结果
    var expected = testutils.GetGoldenFileContents(t)

    // 断言 JSON 内容相等
    assert.JSONEq(t, string(expected), string(actual))

}

func TestPresenter_Present_NewDocumentSorted(t *testing.T) {
    // 生成分析所需的各种对象
    _, matches, packages, context, metadataProvider, appConfig, dbStatus := internal.GenerateAnalysis(t, internal.ImageSource)
    // 创建一个新的文档对象
    doc, err := models.NewDocument(clio.Identification{}, packages, context, matches, nil, metadataProvider, appConfig, dbStatus)
    if err != nil {
        t.Fatal(err)
    }

    // 检查匹配是否已排序
    if !sort.IsSorted(models.MatchSort(doc.Matches)) {
        t.Errorf("expected matches to be sorted")
    }
}

func redact(content []byte) []byte {
    // 替换内容中的时间戳
    return timestampRegexp.ReplaceAll(content, []byte(`"timestamp":""`))
}
```