# `grype\internal\format\presenter.go`

```go
package format

import (
    "github.com/wagoodman/go-presenter"

    "github.com/anchore/grype/grype/presenter/cyclonedx"
    "github.com/anchore/grype/grype/presenter/json"
    "github.com/anchore/grype/grype/presenter/models"
    "github.com/anchore/grype/grype/presenter/sarif"
    "github.com/anchore/grype/grype/presenter/table"
    "github.com/anchore/grype/grype/presenter/template"
    "github.com/anchore/grype/internal/log"
)

type PresentationConfig struct {
    TemplateFilePath string
    ShowSuppressed   bool
}

// GetPresenter retrieves a Presenter that matches a CLI option
func GetPresenter(format Format, c PresentationConfig, pb models.PresenterConfig) presenter.Presenter {
    switch format {
    case JSONFormat:
        return json.NewPresenter(pb)  // 返回一个新的 JSON 格式的 Presenter
    case TableFormat:
        return table.NewPresenter(pb, c.ShowSuppressed)  // 返回一个新的 Table 格式的 Presenter，根据 ShowSuppressed 参数决定是否显示被抑制的结果

    // NOTE: cyclonedx is identical to EmbeddedVEXJSON
    // The cyclonedx library only provides two BOM formats: JSON and XML
    // These embedded formats will be removed in v1.0
    case CycloneDXFormat:
        return cyclonedx.NewXMLPresenter(pb)  // 返回一个新的 CycloneDX XML 格式的 Presenter
    case CycloneDXJSON:
        return cyclonedx.NewJSONPresenter(pb)  // 返回一个新的 CycloneDX JSON 格式的 Presenter
    case CycloneDXXML:
        return cyclonedx.NewXMLPresenter(pb)  // 返回一个新的 CycloneDX XML 格式的 Presenter
    case SarifFormat:
        return sarif.NewPresenter(pb)  // 返回一个新的 Sarif 格式的 Presenter
    case TemplateFormat:
        return template.NewPresenter(pb, c.TemplateFilePath)  // 返回一个新的 Template 格式的 Presenter，根据模板文件路径决定格式

    // DEPRECATED TODO: remove in v1.0
    case EmbeddedVEXJSON:
        log.Warn("embedded-cyclonedx-vex-json format is deprecated and will be removed in v1.0")  // 发出警告信息，说明该格式将在 v1.0 版本中被移除
        return cyclonedx.NewJSONPresenter(pb)  // 返回一个新的 CycloneDX JSON 格式的 Presenter
    case EmbeddedVEXXML:
        log.Warn("embedded-cyclonedx-vex-xml format is deprecated and will be removed in v1.0")  // 发出警告信息，说明该格式将在 v1.0 版本中被移除
        return cyclonedx.NewXMLPresenter(pb)  // 返回一个新的 CycloneDX XML 格式的 Presenter
    default:
        return nil  // 返回空值
    }
}
```