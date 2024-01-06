# `grype\grype\presenter\json\presenter.go`

```
// 导入 json 包
package json

// 导入必要的包
import (
	"encoding/json"  // 导入 json 编码/解码包
	"io"  // 导入输入/输出包

	"github.com/anchore/clio"  // 导入 anchore/clio 包
	"github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
	"github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
)

// Presenter 是一个用于保存报告所需字段的通用结构体
type Presenter struct {
	id               clio.Identification  // 用于标识的字段
	matches          match.Matches  // 匹配结果的字段
	ignoredMatches   []match.IgnoredMatch  // 忽略的匹配结果的字段
	packages         []pkg.Package  // 软件包信息的字段
	context          pkg.Context  // 上下文信息的字段
// metadataProvider vulnerability.MetadataProvider - 定义了一个名为metadataProvider的变量，类型为vulnerability.MetadataProvider
// appConfig        interface{} - 定义了一个名为appConfig的变量，类型为interface{}
// dbStatus         interface{} - 定义了一个名为dbStatus的变量，类型为interface{}

// NewPresenter creates a new JSON presenter - NewPresenter函数用于创建一个新的JSON presenter

// 创建一个新的Presenter对象，使用传入的PresenterConfig参数
func NewPresenter(pb models.PresenterConfig) *Presenter {
	// 返回一个指向Presenter对象的指针
	return &Presenter{
		// 使用传入的PresenterConfig参数初始化Presenter对象的各个字段
		id:               pb.ID,
		matches:          pb.Matches,
		ignoredMatches:   pb.IgnoredMatches,
		packages:         pb.Packages,
		metadataProvider: pb.MetadataProvider,
		context:          pb.Context,
		appConfig:        pb.AppConfig,
		dbStatus:         pb.DBStatus,
	}
}

// Present creates a JSON-based reporting - Present函数用于创建基于JSON的报告
// Present 方法用于将数据呈现到输出流中
func (pres *Presenter) Present(output io.Writer) error {
    // 创建一个新的文档对象
    doc, err := models.NewDocument(pres.id, pres.packages, pres.context, pres.matches, pres.ignoredMatches, pres.metadataProvider,
        pres.appConfig, pres.dbStatus)
    if err != nil {
        return err
    }

    // 创建一个 JSON 编码器
    enc := json.NewEncoder(output)
    // 防止在数据中转义 > 和 <
    enc.SetEscapeHTML(false)
    // 设置缩进格式
    enc.SetIndent("", " ")
    // 将文档对象编码并写入输出流
    return enc.Encode(&doc)
}
```