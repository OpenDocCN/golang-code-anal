# `grype\grype\presenter\json\presenter.go`

```
package json

import (
    "encoding/json"  // 导入处理 JSON 数据的包
    "io"  // 导入处理输入输出的包

    "github.com/anchore/clio"  // 导入 anchore/clio 包
    "github.com/anchore/grype/grype/match"  // 导入 anchore/grype/grype/match 包
    "github.com/anchore/grype/grype/pkg"  // 导入 anchore/grype/grype/pkg 包
    "github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 anchore/grype/grype/vulnerability 包
)

// Presenter is a generic struct for holding fields needed for reporting
type Presenter struct {
    id               clio.Identification  // 用于标识的字段
    matches          match.Matches  // 匹配结果的字段
    ignoredMatches   []match.IgnoredMatch  // 忽略的匹配结果的字段
    packages         []pkg.Package  // 包信息的字段
    context          pkg.Context  // 上下文信息的字段
    metadataProvider vulnerability.MetadataProvider  // 元数据提供者的字段
    appConfig        interface{}  // 应用配置的字段
    dbStatus         interface{}  // 数据库状态的字段
}

// NewPresenter creates a new JSON presenter
func NewPresenter(pb models.PresenterConfig) *Presenter {
    return &Presenter{
        id:               pb.ID,  // 初始化标识字段
        matches:          pb.Matches,  // 初始化匹配结果字段
        ignoredMatches:   pb.IgnoredMatches,  // 初始化忽略的匹配结果字段
        packages:         pb.Packages,  // 初始化包信息字段
        metadataProvider: pb.MetadataProvider,  // 初始化元数据提供者字段
        context:          pb.Context,  // 初始化上下文信息字段
        appConfig:        pb.AppConfig,  // 初始化应用配置字段
        dbStatus:         pb.DBStatus,  // 初始化数据库状态字段
    }
}

// Present creates a JSON-based reporting
func (pres *Presenter) Present(output io.Writer) error {
    doc, err := models.NewDocument(pres.id, pres.packages, pres.context, pres.matches, pres.ignoredMatches, pres.metadataProvider,
        pres.appConfig, pres.dbStatus)  // 创建新的文档
    if err != nil {
        return err  // 如果创建文档出错，返回错误
    }

    enc := json.NewEncoder(output)  // 创建 JSON 编码器
    // prevent > and < from being escaped in the payload
    enc.SetEscapeHTML(false)  // 设置不转义 > 和 < 字符
    enc.SetIndent("", " ")  // 设置缩进格式
    return enc.Encode(&doc)  // 编码并输出文档
}
```