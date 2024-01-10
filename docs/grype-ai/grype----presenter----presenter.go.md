# `grype\grype\presenter\presenter.go`

```
package presenter

import (
    "github.com/wagoodman/go-presenter"  // 导入外部包 go-presenter

    "github.com/anchore/grype/grype/presenter/models"  // 导入外部包 models
    "github.com/anchore/grype/internal/format"  // 导入外部包 format
)

// GetPresenter retrieves a Presenter that matches a CLI option.
// Deprecated: this will be removed in v1.0
func GetPresenter(f string, templatePath string, showSuppressed bool, pb models.PresenterConfig) presenter.Presenter {
    // 调用 format 包中的 GetPresenter 函数，传入解析后的 f 字符串、PresentationConfig 结构体和 pb 参数
    return format.GetPresenter(format.Parse(f), format.PresentationConfig{
        TemplateFilePath: templatePath,  // 设置模板文件路径
        ShowSuppressed:   showSuppressed,  // 设置是否显示被抑制的信息
    }, pb)  // 返回获取到的 Presenter
}
```