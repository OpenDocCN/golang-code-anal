# `grype\grype\presenter\cyclonedx\presenter.go`

```
package cyclonedx

import (
	"io"

	"github.com/CycloneDX/cyclonedx-go"  // 导入CycloneDX库

	"github.com/anchore/clio"  // 导入clio库
	"github.com/anchore/grype/grype/match"  // 导入match模块
	"github.com/anchore/grype/grype/pkg"  // 导入pkg模块
	"github.com/anchore/grype/grype/presenter/models"  // 导入models模块
	"github.com/anchore/grype/grype/vulnerability"  // 导入vulnerability模块
	"github.com/anchore/syft/syft/format/common/cyclonedxhelpers"  // 导入cyclonedxhelpers模块
	"github.com/anchore/syft/syft/sbom"  // 导入sbom模块
	"github.com/anchore/syft/syft/source"  // 导入source模块
)

// Presenter writes a CycloneDX report from the given Matches and Scope contents
type Presenter struct {
	id               clio.Identification  // 定义Presenter结构体，包含id字段
results          match.Matches  // 声明一个名为 results 的变量，类型为 match.Matches
packages         []pkg.Package  // 声明一个名为 packages 的变量，类型为 pkg.Package 的切片
src              *source.Description  // 声明一个名为 src 的变量，类型为指向 source.Description 类型的指针
metadataProvider vulnerability.MetadataProvider  // 声明一个名为 metadataProvider 的变量，类型为 vulnerability.MetadataProvider
format           cyclonedx.BOMFileFormat  // 声明一个名为 format 的变量，类型为 cyclonedx.BOMFileFormat
sbom             *sbom.SBOM  // 声明一个名为 sbom 的变量，类型为指向 sbom.SBOM 类型的指针
}

// NewPresenter is a *Presenter constructor
func NewJSONPresenter(pb models.PresenterConfig) *Presenter {
	return &Presenter{
		id:               pb.ID,  // 初始化 id 字段为传入参数 pb 的 ID 值
		results:          pb.Matches,  // 初始化 results 字段为传入参数 pb 的 Matches 值
		packages:         pb.Packages,  // 初始化 packages 字段为传入参数 pb 的 Packages 值
		metadataProvider: pb.MetadataProvider,  // 初始化 metadataProvider 字段为传入参数 pb 的 MetadataProvider 值
		src:              pb.Context.Source,  // 初始化 src 字段为传入参数 pb 的 Context 中的 Source 值
		sbom:             pb.SBOM,  // 初始化 sbom 字段为传入参数 pb 的 SBOM 值
		format:           cyclonedx.BOMFileFormatJSON,  // 初始化 format 字段为 cyclonedx.BOMFileFormatJSON
	}
}
// NewPresenter 是一个 *Presenter 构造函数
func NewXMLPresenter(pb models.PresenterConfig) *Presenter {
	// 创建并返回一个 Presenter 对象，其中包含了传入的 PresenterConfig 的相关信息
	return &Presenter{
		id:               pb.ID,  // 设置 Presenter 对象的 id 属性为传入的 PresenterConfig 的 ID
		results:          pb.Matches,  // 设置 Presenter 对象的 results 属性为传入的 PresenterConfig 的 Matches
		packages:         pb.Packages,  // 设置 Presenter 对象的 packages 属性为传入的 PresenterConfig 的 Packages
		metadataProvider: pb.MetadataProvider,  // 设置 Presenter 对象的 metadataProvider 属性为传入的 PresenterConfig 的 MetadataProvider
		src:              pb.Context.Source,  // 设置 Presenter 对象的 src 属性为传入的 PresenterConfig 的 Context 的 Source
		sbom:             pb.SBOM,  // 设置 Presenter 对象的 sbom 属性为传入的 PresenterConfig 的 SBOM
		format:           cyclonedx.BOMFileFormatXML,  // 设置 Presenter 对象的 format 属性为 CycloneDX 的 XML 格式
	}
}

// Present 创建一个基于 CycloneDX 的报告
func (pres *Presenter) Present(output io.Writer) error {
	// 注意：这里使用了 syft cyclondx helpers 来创建一个一致的 CycloneDX BOM，用于在 syft 和 grype 中使用
	cyclonedxBOM := cyclonedxhelpers.ToFormatModel(*pres.sbom)  // 使用 syft cyclondx helpers 将 Presenter 对象的 sbom 转换为 CycloneDX 格式的 BOM
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

vulns := make([]cyclonedx.Vulnerability, 0)  // 创建一个空的漏洞列表
for _, m := range pres.results.Sorted() {  // 遍历结果列表
    v, err := NewVulnerability(m, pres.metadataProvider)  // 创建一个新的漏洞对象
    if err != nil {  // 如果创建过程中出现错误
        continue  // 继续下一个循环
    }
    vulns = append(vulns, v)  // 将新创建的漏洞对象添加到漏洞列表中
}
# 将 vulns 赋值给 cyclonedxBOM.Vulnerabilities
cyclonedxBOM.Vulnerabilities = &vulns
# 创建一个新的 BOM 编码器，指定输出和格式
enc := cyclonedx.NewBOMEncoder(output, pres.format)
# 设置编码器为美化输出
enc.SetPretty(true)
# 设置编码器为不转义 HTML
enc.SetEscapeHTML(false)

# 使用编码器将 cyclonedxBOM 编码为指定格式的输出
return enc.Encode(cyclonedxBOM)
```