# `grype\grype\presenter\cyclonedx\presenter.go`

```go
// 导入所需的包
package cyclonedx

import (
    "io"

    "github.com/CycloneDX/cyclonedx-go"

    "github.com/anchore/clio"
    "github.com/anchore/grype/grype/match"
    "github.com/anchore/grype/grype/pkg"
    "github.com/anchore/grype/grype/presenter/models"
    "github.com/anchore/grype/grype/vulnerability"
    "github.com/anchore/syft/syft/format/common/cyclonedxhelpers"
    "github.com/anchore/syft/syft/sbom"
    "github.com/anchore/syft/syft/source"
)

// Presenter 结构体用于生成 CycloneDX 报告
type Presenter struct {
    id               clio.Identification
    results          match.Matches
    packages         []pkg.Package
    src              *source.Description
    metadataProvider vulnerability.MetadataProvider
    format           cyclonedx.BOMFileFormat
    sbom             *sbom.SBOM
}

// NewJSONPresenter 是 Presenter 结构体的构造函数，返回一个 *Presenter 实例
func NewJSONPresenter(pb models.PresenterConfig) *Presenter {
    return &Presenter{
        id:               pb.ID,
        results:          pb.Matches,
        packages:         pb.Packages,
        metadataProvider: pb.MetadataProvider,
        src:              pb.Context.Source,
        sbom:             pb.SBOM,
        format:           cyclonedx.BOMFileFormatJSON,
    }
}

// NewXMLPresenter 是 Presenter 结构体的构造函数，返回一个 *Presenter 实例
func NewXMLPresenter(pb models.PresenterConfig) *Presenter {
    return &Presenter{
        id:               pb.ID,
        results:          pb.Matches,
        packages:         pb.Packages,
        metadataProvider: pb.MetadataProvider,
        src:              pb.Context.Source,
        sbom:             pb.SBOM,
        format:           cyclonedx.BOMFileFormatXML,
    }
}

// Present 方法用于生成基于 CycloneDX 的报告
func (pres *Presenter) Present(output io.Writer) error {
    // 注意：这里使用了 syft cyclondx helpers 来创建一个一致的 cyclondx BOM，用于 syft 和 grype
    cyclonedxBOM := cyclonedxhelpers.ToFormatModel(*pres.sbom)
    // 清空工具元数据并添加 grype 元数据
    cyclonedxBOM.Metadata.Tools = &cyclonedx.ToolsChoice{
        Components: &[]cyclonedx.Component{
            {
                Type:    cyclonedx.ComponentTypeApplication,  // 设置组件类型为应用程序
                Author:  "anchore",  // 设置作者为 anchore
                Name:    pres.id.Name,  // 设置名称为 pres.id.Name
                Version: pres.id.Version,  // 设置版本为 pres.id.Version
            },
        },
    }

    // 创建一个空的漏洞列表
    vulns := make([]cyclonedx.Vulnerability, 0)
    // 遍历 pres.results.Sorted() 中的每个元素
    for _, m := range pres.results.Sorted() {
        // 根据 m 和 pres.metadataProvider 创建一个新的漏洞对象
        v, err := NewVulnerability(m, pres.metadataProvider)
        // 如果创建过程中出现错误，则跳过当前循环
        if err != nil {
            continue
        }
        // 将新创建的漏洞对象添加到漏洞列表中
        vulns = append(vulns, v)
    }
    // 将漏洞列表赋值给 cyclonedxBOM.Vulnerabilities
    cyclonedxBOM.Vulnerabilities = &vulns
    // 创建一个 BOM 编码器
    enc := cyclonedx.NewBOMEncoder(output, pres.format)
    // 设置编码器为美化输出
    enc.SetPretty(true)
    // 设置编码器不转义 HTML
    enc.SetEscapeHTML(false)

    // 使用编码器将 cyclonedxBOM 编码为输出格式，并返回结果
    return enc.Encode(cyclonedxBOM)
# 闭合前面的函数定义
```