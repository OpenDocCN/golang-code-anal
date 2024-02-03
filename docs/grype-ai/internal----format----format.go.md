# `grype\internal\format\format.go`

```go
package format

import (
    "strings"
)

const (
    UnknownFormat   Format = "unknown"  // 未知格式
    JSONFormat      Format = "json"     // JSON 格式
    TableFormat     Format = "table"    // 表格格式
    CycloneDXFormat Format = "cyclonedx"  // CycloneDX 格式
    CycloneDXJSON   Format = "cyclonedx-json"  // CycloneDX JSON 格式
    CycloneDXXML    Format = "cyclonedx-xml"   // CycloneDX XML 格式
    SarifFormat     Format = "sarif"    // Sarif 格式
    TemplateFormat  Format = "template"  // 模板格式

    // DEPRECATED <-- TODO: remove in v1.0
    EmbeddedVEXJSON Format = "embedded-cyclonedx-vex-json"  // 嵌入式 CycloneDX VEX JSON 格式
    EmbeddedVEXXML  Format = "embedded-cyclonedx-vex-xml"   // 嵌入式 CycloneDX VEX XML 格式
)

// Format is a dedicated type to represent a specific kind of presenter output format.
type Format string

func (f Format) String() string {
    return string(f)
}

// Parse returns the presenter.format specified by the given user input.
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
    case strings.ToLower(CycloneDXFormat.String()):
        return CycloneDXFormat
    case strings.ToLower(CycloneDXJSON.String()):
        return CycloneDXJSON
    case strings.ToLower(CycloneDXXML.String()):
        return CycloneDXXML
    case strings.ToLower(EmbeddedVEXJSON.String()):
        return CycloneDXJSON
    case strings.ToLower(EmbeddedVEXXML.String()):
        return CycloneDXFormat
    default:
        return UnknownFormat
    }
}

// AvailableFormats is a list of presenter format options available to users.
var AvailableFormats = []Format{
    JSONFormat,
    TableFormat,
    CycloneDXFormat,
    CycloneDXJSON,
    SarifFormat,
    TemplateFormat,
}

// DeprecatedFormats TODO: remove in v1.0
var DeprecatedFormats = []Format{
    EmbeddedVEXJSON,
    EmbeddedVEXXML,
}
```