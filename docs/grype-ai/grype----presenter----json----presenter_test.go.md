# `grype\grype\presenter\json\presenter_test.go`

```
// 导入标准库中的包
package json

// 导入其他自定义包
import (
	"bytes // 用于操作字节的包
	"flag" // 用于解析命令行参数的包
	"regexp" // 用于正则表达式的包
	"sort" // 用于排序的包
	"testing" // 用于编写测试的包

	"github.com/stretchr/testify/assert" // 使用第三方库进行断言

	// 导入自定义包
	"github.com/anchore/clio" // 用于命令行交互的包
	"github.com/anchore/go-testutils" // 用于测试的工具包
	"github.com/anchore/grype/grype/match" // 用于匹配的包
	"github.com/anchore/grype/grype/pkg" // 用于包管理的包
	"github.com/anchore/grype/grype/presenter/internal" // 内部展示相关的包
	"github.com/anchore/grype/grype/presenter/models" // 用于展示模型的包
	"github.com/anchore/syft/syft/linux" // 用于处理 Linux 相关信息的包
	"github.com/anchore/syft/syft/source" // 用于处理源码的包
)
# 定义一个布尔类型的命令行参数，用于指定是否更新 *.golden 文件
var update = flag.Bool("update", false, "update the *.golden files for json presenters")
# 创建一个正则表达式对象，用于匹配包含特定格式的时间戳字符串
var timestampRegexp = regexp.MustCompile(`"timestamp":\s*"[^"]+"`)

# 定义一个测试函数，用于测试 JsonImgsPresenter
func TestJsonImgsPresenter(t *testing.T) {
    # 创建一个字节缓冲区
    var buffer bytes.Buffer
    # 调用 internal.GenerateAnalysis 函数，获取分析结果的相关信息
    _, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.ImageSource)

    # 创建一个 PresenterConfig 对象
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

    # 创建一个新的 Presenter 对象
    pres := NewPresenter(pb)
// 运行 presenter，将结果写入 buffer
if err := pres.Present(&buffer); err != nil {
    // 如果出现错误，记录错误并终止测试
    t.Fatal(err)
}
// 获取实际结果的字节流
actual := buffer.Bytes()
// 对实际结果进行数据脱敏处理
actual = redact(actual)

// 如果需要更新测试数据
if *update {
    // 更新测试数据的黄金文件内容
    testutils.UpdateGoldenFileContents(t, actual)
}

// 获取黄金文件的期望内容
var expected = testutils.GetGoldenFileContents(t)

// 断言实际结果与期望结果是否相等
assert.JSONEq(t, string(expected), string(actual))

// TODO: 当有 JSON 模式时，将我重新添加进来
// 针对数据库模式验证实际结果
// validateAgainstDbSchema(t, string(actual))
# 定义一个测试函数，用于测试JsonDirsPresenter
func TestJsonDirsPresenter(t *testing.T) {
    # 创建一个字节缓冲区
    var buffer bytes.Buffer

    # 调用GenerateAnalysis函数，获取分析结果的各种信息
    _, matches, packages, context, metadataProvider, _, _ := internal.GenerateAnalysis(t, internal.DirectorySource)

    # 创建一个PresenterConfig对象
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

    # 根据PresenterConfig对象创建一个Presenter对象
    pres := NewPresenter(pb)

    # 运行Presenter对象的Present方法，将结果写入缓冲区
    if err := pres.Present(&buffer); err != nil {
		// 如果发生错误，测试失败
		t.Fatal(err)
	}
	// 获取实际结果并进行数据脱敏处理
	actual := buffer.Bytes()
	actual = redact(actual)

	// 如果需要更新测试数据，更新黄金文件内容
	if *update {
		testutils.UpdateGoldenFileContents(t, actual)
	}

	// 获取黄金文件内容作为期望结果
	var expected = testutils.GetGoldenFileContents(t)

	// 断言实际结果与期望结果相等
	assert.JSONEq(t, string(expected), string(actual))

	// TODO: 当有 JSON 模式时，将我重新添加进来
	// validateAgainstDbSchema(t, string(actual))
}

func TestEmptyJsonPresenter(t *testing.T) {
	// 预期返回一个空的 JSON 数组
	var buffer bytes.Buffer
```

// 创建一个新的匹配对象
matches := match.NewMatches()

// 创建一个上下文对象，包括源描述和Linux发行版信息
ctx := pkg.Context{
	Source: &source.Description{},
	Distro: &linux.Release{
		ID:      "centos",
		IDLike:  []string{"rhel"},
		Version: "8.0",
	},
}

// 创建一个展示配置对象，包括标识信息、匹配对象、包信息和上下文信息
pb := models.PresenterConfig{
	ID: clio.Identification{
		Name:    "grype",
		Version: "[not provided]",
	},
	Matches:          matches,
	Packages:         nil,
	Context:          ctx,
}
		// 创建一个 MetadataProvider 的实例，此处为 nil
		MetadataProvider: nil,
	}

	// 创建一个 Presenter 的实例，传入参数为 pb
	pres := NewPresenter(pb)

	// 运行 presenter
	if err := pres.Present(&buffer); err != nil {
		// 如果出现错误，记录错误信息并终止测试
		t.Fatal(err)
	}
	// 获取实际结果的字节流
	actual := buffer.Bytes()
	// 对实际结果进行数据处理
	actual = redact(actual)

	// 如果需要更新测试数据
	if *update {
		// 更新测试数据的黄金文件内容
		testutils.UpdateGoldenFileContents(t, actual)
	}

	// 获取期望结果的黄金文件内容
	var expected = testutils.GetGoldenFileContents(t)

	// 断言实际结果与期望结果的 JSON 字符串是否相等
	assert.JSONEq(t, string(expected), string(actual))
# 测试函数，用于测试Presenter的Present方法，生成分析结果并检查是否按照预期排序
func TestPresenter_Present_NewDocumentSorted(t *testing.T) {
    # 生成分析结果所需的各种参数
	_, matches, packages, context, metadataProvider, appConfig, dbStatus := internal.GenerateAnalysis(t, internal.ImageSource)
    # 创建新的文档对象
	doc, err := models.NewDocument(clio.Identification{}, packages, context, matches, nil, metadataProvider, appConfig, dbStatus)
    # 检查是否有错误发生
	if err != nil {
		t.Fatal(err)
	}
    # 检查文档中的匹配项是否按照预期排序
	if !sort.IsSorted(models.MatchSort(doc.Matches)) {
		t.Errorf("expected matches to be sorted")
	}
}

# 用于替换内容中的时间戳的函数
func redact(content []byte) []byte {
    # 使用正则表达式替换内容中的时间戳为指定字符串
	return timestampRegexp.ReplaceAll(content, []byte(`"timestamp":""`))
}
```