# `grype\grype\presenter\presenter.go`

```
// 导入所需的包
package presenter

import (
	"github.com/wagoodman/go-presenter"  // 导入外部包

	"github.com/anchore/grype/grype/presenter/models"  // 导入内部包
	"github.com/anchore/grype/internal/format"  // 导入内部包
)

// GetPresenter 根据 CLI 选项获取匹配的 Presenter
// Deprecated: 在 v1.0 版本中将被移除
func GetPresenter(f string, templatePath string, showSuppressed bool, pb models.PresenterConfig) presenter.Presenter {
	// 调用 format 包中的 GetPresenter 函数，根据传入的参数获取对应的 Presenter
	return format.GetPresenter(format.Parse(f), format.PresentationConfig{
		TemplateFilePath: templatePath,  // 设置模板文件路径
		ShowSuppressed:   showSuppressed,  // 设置是否显示被抑制的信息
	}, pb)  // 返回获取的 Presenter
}
```