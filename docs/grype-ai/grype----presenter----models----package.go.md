# `grype\grype\presenter\models\package.go`

```
// 导入所需的包
import (
	"github.com/anchore/grype/grype/internal/packagemetadata"
	"github.com/anchore/grype/grype/pkg"
	"github.com/anchore/syft/syft/file"
	syftPkg "github.com/anchore/syft/syft/pkg"
)

// Package 结构体用于显示单个 pkg.Package 对象的字段，用于 JSON 展示
type Package struct {
	// 包的唯一标识符
	ID           string             `json:"id"`
	// 包的名称
	Name         string             `json:"name"`
	// 包的版本
	Version      string             `json:"version"`
	// 包的类型
	Type         syftPkg.Type       `json:"type"`
	// 包的位置信息
	Locations    []file.Coordinates `json:"locations"`
	// 包的语言
	Language     syftPkg.Language   `json:"language"`
	// 包的许可证信息
	Licenses     []string           `json:"licenses"`
	// 包的 CPE（通用产品标识符）信息
	CPEs         []string           `json:"cpes"`
	// 包的 PURL（可持久资源定位符）信息
	PURL         string             `json:"purl"`
}
// Upstreams 是一个 UpstreamPackage 类型的切片，用于存储上游包信息
// MetadataType 是一个字符串类型，用于存储元数据类型，omitempty 表示在 JSON 中如果为空则忽略
// Metadata 是一个空接口类型，用于存储元数据，omitempty 表示在 JSON 中如果为空则忽略
type Package struct {
	Upstreams    []UpstreamPackage  `json:"upstreams"` // 存储上游包信息的切片
	MetadataType string             `json:"metadataType,omitempty"` // 元数据类型
	Metadata     interface{}        `json:"metadata,omitempty"` // 元数据
}

// UpstreamPackage 是一个包含名称和版本信息的结构体
type UpstreamPackage struct {
	Name    string `json:"name"` // 包名称
	Version string `json:"version,omitempty"` // 包版本，omitempty 表示在 JSON 中如果为空则忽略
}

// newPackage 是一个函数，用于将 pkg.Package 转换为 Package 类型
func newPackage(p pkg.Package) Package {
	var cpes = make([]string, 0) // 创建一个空的字符串切片
	for _, c := range p.CPEs { // 遍历 p.CPEs 切片
		cpes = append(cpes, c.BindToFmtString()) // 将 c.BindToFmtString() 的结果追加到 cpes 切片中
	}

	licenses := p.Licenses // 获取 p.Licenses
	if licenses == nil { // 如果 licenses 为空
		licenses = make([]string, 0) // 创建一个空的字符串切片
	}
}
# 创建一个空的坐标数组
var coordinates = make([]file.Coordinates, 0)
# 将位置转换为切片
locations := p.Locations.ToSlice()
# 遍历位置切片，将坐标添加到坐标数组中
for _, l := range locations {
    coordinates = append(coordinates, l.Coordinates)
}

# 创建一个空的上游包数组
var upstreams = make([]UpstreamPackage, 0)
# 遍历上游包，将名称和版本添加到上游包数组中
for _, u := range p.Upstreams {
    upstreams = append(upstreams, UpstreamPackage{
        Name:    u.Name,
        Version: u.Version,
    })
}

# 返回一个包含特定属性的包对象
return Package{
    ID:           string(p.ID),
    Name:         p.Name,
    Version:      p.Version,
    Locations:    coordinates,
		# 将许可证信息添加到包的元数据中
		Licenses:     licenses,
		# 将语言信息添加到包的元数据中
		Language:     p.Language,
		# 将类型信息添加到包的元数据中
		Type:         p.Type,
		# 将CPE信息添加到包的元数据中
		CPEs:         cpes,
		# 将PURL信息添加到包的元数据中
		PURL:         p.PURL,
		# 将上游信息添加到包的元数据中
		Upstreams:    upstreams,
		# 将元数据类型信息添加到包的元数据中
		MetadataType: packagemetadata.JSONName(p.Metadata),
		# 将元数据添加到包的元数据中
		Metadata:     p.Metadata,
	}
}
```