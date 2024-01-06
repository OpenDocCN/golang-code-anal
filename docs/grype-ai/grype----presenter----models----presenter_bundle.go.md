# `grype\grype\presenter\models\presenter_bundle.go`

```
// 导入所需的包
import (
	"github.com/anchore/clio"
	"github.com/anchore/grype/grype/match"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/grype/grype/vulnerability"
	"github.com/anchore/syft/syft/sbom"
)

// 定义 PresenterConfig 结构体，包含各种字段用于展示配置
type PresenterConfig struct {
	// ID 用于标识展示配置
	ID               clio.Identification
	// Matches 包含匹配项
	Matches          match.Matches
	// IgnoredMatches 包含被忽略的匹配项
	IgnoredMatches   []match.IgnoredMatch
	// Packages 包含软件包信息
	Packages         []pkg.Package
	// Context 包含软件包的上下文信息
	Context          pkg.Context
	// MetadataProvider 用于提供漏洞元数据
	MetadataProvider vulnerability.MetadataProvider
	// SBOM 用于表示软件构建材料清单
	SBOM             *sbom.SBOM
	// AppConfig 用于表示应用程序配置
	AppConfig        interface{}
	// DBStatus 用于表示数据库状态
	DBStatus         interface{}
}
这是一个代码块的结束符号，表示前面的函数或者循环的结束。
```