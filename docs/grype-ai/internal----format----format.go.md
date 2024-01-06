# `grype\internal\format\format.go`

```
package format

import (
	"strings"  // 导入 strings 包，用于处理字符串
)

const (
	UnknownFormat   Format = "unknown"  // 定义常量 UnknownFormat，并赋值为 "unknown"
	JSONFormat      Format = "json"  // 定义常量 JSONFormat，并赋值为 "json"
	TableFormat     Format = "table"  // 定义常量 TableFormat，并赋值为 "table"
	CycloneDXFormat Format = "cyclonedx"  // 定义常量 CycloneDXFormat，并赋值为 "cyclonedx"
	CycloneDXJSON   Format = "cyclonedx-json"  // 定义常量 CycloneDXJSON，并赋值为 "cyclonedx-json"
	CycloneDXXML    Format = "cyclonedx-xml"  // 定义常量 CycloneDXXML，并赋值为 "cyclonedx-xml"
	SarifFormat     Format = "sarif"  // 定义常量 SarifFormat，并赋值为 "sarif"
	TemplateFormat  Format = "template"  // 定义常量 TemplateFormat，并赋值为 "template"

	// DEPRECATED <-- TODO: remove in v1.0
	EmbeddedVEXJSON Format = "embedded-cyclonedx-vex-json"  // 定义常量 EmbeddedVEXJSON，并赋值为 "embedded-cyclonedx-vex-json"
	EmbeddedVEXXML  Format = "embedded-cyclonedx-vex-xml"  // 定义常量 EmbeddedVEXXML，并赋值为 "embedded-cyclonedx-vex-xml"
)
// Format 是一个专门用来表示特定类型的展示输出格式的类型。
type Format string

// String 方法用来返回格式的字符串表示。
func (f Format) String() string {
	return string(f)
}

// Parse 根据用户输入返回指定的 presenter.format。
func Parse(userInput string) Format {
	switch strings.ToLower(userInput) {
	case "":
		return TableFormat
	case strings.ToLower(JSONFormat.String()):
		return JSONFormat
	case strings.ToLower(TableFormat.String()):
		return TableFormat
	case strings.ToLower(SarifFormat.String()):
		return SarifFormat
	case strings.ToLower(TemplateFormat.String()):
		return TemplateFormat
	}
}
// 根据给定的字符串格式返回相应的格式枚举值
switch format {
    // 如果是 TemplateFormat，则返回 TemplateFormat 格式
    case strings.ToLower(TemplateFormat.String()):
        return TemplateFormat
    // 如果是 CycloneDXFormat，则返回 CycloneDXFormat 格式
    case strings.ToLower(CycloneDXFormat.String()):
        return CycloneDXFormat
    // 如果是 CycloneDXJSON，则返回 CycloneDXJSON 格式
    case strings.ToLower(CycloneDXJSON.String()):
        return CycloneDXJSON
    // 如果是 CycloneDXXML，则返回 CycloneDXXML 格式
    case strings.ToLower(CycloneDXXML.String()):
        return CycloneDXXML
    // 如果是 EmbeddedVEXJSON，则返回 CycloneDXJSON 格式
    case strings.ToLower(EmbeddedVEXJSON.String()):
        return CycloneDXJSON
    // 如果是 EmbeddedVEXXML，则返回 CycloneDXFormat 格式
    case strings.ToLower(EmbeddedVEXXML.String()):
        return CycloneDXFormat
    // 如果是其他未知格式，则返回 UnknownFormat 格式
    default:
        return UnknownFormat
    }
}

// AvailableFormats 是用户可用的演示格式选项列表
var AvailableFormats = []Format{
    JSONFormat,  // JSON 格式
    TableFormat,  // 表格格式
    // 其他格式...
}
// 定义一个枚举类型，包含CycloneDXFormat、CycloneDXJSON、SarifFormat和TemplateFormat四种格式
var Formats = []Format{
    CycloneDXFormat,
    CycloneDXJSON,
    SarifFormat,
    TemplateFormat,
}

// DeprecatedFormats TODO: remove in v1.0
// 定义一个枚举类型，包含EmbeddedVEXJSON和EmbeddedVEXXML两种格式，用于标记已废弃的格式
var DeprecatedFormats = []Format{
    EmbeddedVEXJSON,
    EmbeddedVEXXML,
}
```