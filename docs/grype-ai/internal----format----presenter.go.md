# `grype\internal\format\presenter.go`

```
package format

import (
	"github.com/wagoodman/go-presenter"  // 导入外部包，用于格式化输出

	"github.com/anchore/grype/grype/presenter/cyclonedx"  // 导入用于生成CycloneDX格式输出的包
	"github.com/anchore/grype/grype/presenter/json"  // 导入用于生成JSON格式输出的包
	"github.com/anchore/grype/grype/presenter/models"  // 导入用于生成模型的包
	"github.com/anchore/grype/grype/presenter/sarif"  // 导入用于生成SARIF格式输出的包
	"github.com/anchore/grype/grype/presenter/table"  // 导入用于生成表格格式输出的包
	"github.com/anchore/grype/grype/presenter/template"  // 导入用于生成模板格式输出的包
	"github.com/anchore/grype/internal/log"  // 导入内部日志包
)

type PresentationConfig struct {
	TemplateFilePath string  // 模板文件路径
	ShowSuppressed   bool  // 是否显示被抑制的信息
}

// GetPresenter retrieves a Presenter that matches a CLI option  // 获取与CLI选项匹配的Presenter
# 根据不同的格式和配置参数，返回对应的 Presenter 对象
func GetPresenter(format Format, c PresentationConfig, pb models.PresenterConfig) presenter.Presenter {
    # 根据不同的格式选择不同的 Presenter 对象
    switch format {
    case JSONFormat:
        # 返回一个处理 JSON 格式的 Presenter 对象
        return json.NewPresenter(pb)
    case TableFormat:
        # 返回一个处理表格格式的 Presenter 对象，根据配置参数决定是否显示被抑制的信息
        return table.NewPresenter(pb, c.ShowSuppressed)

    # NOTE: cyclonedx is identical to EmbeddedVEXJSON
    # The cyclonedx library only provides two BOM formats: JSON and XML
    # These embedded formats will be removed in v1.0
    # 根据不同的 CycloneDX 格式选择对应的 Presenter 对象
    case CycloneDXFormat:
        # 返回一个处理 CycloneDX XML 格式的 Presenter 对象
        return cyclonedx.NewXMLPresenter(pb)
    case CycloneDXJSON:
        # 返回一个处理 CycloneDX JSON 格式的 Presenter 对象
        return cyclonedx.NewJSONPresenter(pb)
    case CycloneDXXML:
        # 返回一个处理 CycloneDX XML 格式的 Presenter 对象
        return cyclonedx.NewXMLPresenter(pb)
    case SarifFormat:
        # 返回一个处理 Sarif 格式的 Presenter 对象
        return sarif.NewPresenter(pb)
    case TemplateFormat:
        # 返回一个处理模板格式的 Presenter 对象，根据配置参数指定模板文件路径
        return template.NewPresenter(pb, c.TemplateFilePath)
    }
}
// 如果格式为EmbeddedVEXJSON，则记录警告信息并返回一个新的JSONPresenter对象
case EmbeddedVEXJSON:
    log.Warn("embedded-cyclonedx-vex-json format is deprecated and will be removed in v1.0")
    return cyclonedx.NewJSONPresenter(pb)
// 如果格式为EmbeddedVEXXML，则记录警告信息并返回一个新的XMLPresenter对象
case EmbeddedVEXXML:
    log.Warn("embedded-cyclonedx-vex-xml format is deprecated and will be removed in v1.0")
    return cyclonedx.NewXMLPresenter(pb)
// 如果格式不是以上两种格式，则返回空
default:
    return nil
}
```