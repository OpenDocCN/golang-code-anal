# `grype\grype\presenter\models\package.go`

```go
package models

import (
    "github.com/anchore/grype/grype/internal/packagemetadata"  // 导入 packagemetadata 包
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
    "github.com/anchore/syft/syft/file"  // 导入 file 包
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包并重命名为 syftPkg
)

// Package 是用于在 JSON 展示器中显示单个 pkg.Package 对象时所需的字段
type Package struct {
    ID           string             `json:"id"`  // 包的 ID
    Name         string             `json:"name"`  // 包的名称
    Version      string             `json:"version"`  // 包的版本
    Type         syftPkg.Type       `json:"type"`  // 包的类型
    Locations    []file.Coordinates `json:"locations"`  // 包的位置
    Language     syftPkg.Language   `json:"language"`  // 包的语言
    Licenses     []string           `json:"licenses"`  // 包的许可证
    CPEs         []string           `json:"cpes"`  // 包的 CPEs
    PURL         string             `json:"purl"`  // 包的 PURL
    Upstreams    []UpstreamPackage  `json:"upstreams"`  // 上游包
    MetadataType string             `json:"metadataType,omitempty"`  // 元数据类型
    Metadata     interface{}        `json:"metadata,omitempty"`  // 元数据
}

type UpstreamPackage struct {
    Name    string `json:"name"`  // 上游包的名称
    Version string `json:"version,omitempty"`  // 上游包的版本
}

func newPackage(p pkg.Package) Package {
    var cpes = make([]string, 0)  // 创建一个空的 CPEs 切片
    for _, c := range p.CPEs {  // 遍历包的 CPEs
        cpes = append(cpes, c.BindToFmtString())  // 将格式化后的 CPE 添加到切片中
    }

    licenses := p.Licenses  // 获取包的许可证
    if licenses == nil {  // 如果许可证为空
        licenses = make([]string, 0)  // 创建一个空的许可证切片
    }

    var coordinates = make([]file.Coordinates, 0)  // 创建一个空的坐标切片
    locations := p.Locations.ToSlice()  // 获取包的位置切片
    for _, l := range locations {  // 遍历位置切片
        coordinates = append(coordinates, l.Coordinates)  // 将坐标添加到坐标切片中
    }

    var upstreams = make([]UpstreamPackage, 0)  // 创建一个空的上游包切片
    for _, u := range p.Upstreams {  // 遍历上游包
        upstreams = append(upstreams, UpstreamPackage{  // 将上游包添加到上游包切片中
            Name:    u.Name,  // 上游包的名称
            Version: u.Version,  // 上游包的版本
        })
    }
}
    # 返回一个Package对象，包含以下属性：
    # ID: 将p.ID转换为字符串类型
    # Name: p.Name
    # Version: p.Version
    # Locations: coordinates
    # Licenses: licenses
    # Language: p.Language
    # Type: p.Type
    # CPEs: cpes
    # PURL: p.PURL
    # Upstreams: upstreams
    # MetadataType: 使用packagemetadata.JSONName方法处理p.Metadata的类型
    # Metadata: p.Metadata
    return Package{
        ID:           string(p.ID),
        Name:         p.Name,
        Version:      p.Version,
        Locations:    coordinates,
        Licenses:     licenses,
        Language:     p.Language,
        Type:         p.Type,
        CPEs:         cpes,
        PURL:         p.PURL,
        Upstreams:    upstreams,
        MetadataType: packagemetadata.JSONName(p.Metadata),
        Metadata:     p.Metadata,
    }
# 闭合前面的函数定义
```