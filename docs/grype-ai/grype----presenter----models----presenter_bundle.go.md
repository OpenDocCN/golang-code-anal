# `grype\grype\presenter\models\presenter_bundle.go`

```
package models

import (
    "github.com/anchore/clio"  // 导入 clio 包
    "github.com/anchore/grype/grype/match"  // 导入 match 包
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
    "github.com/anchore/syft/syft/sbom"  // 导入 sbom 包
)

type PresenterConfig struct {
    ID               clio.Identification  // 定义 ID 字段，类型为 clio.Identification
    Matches          match.Matches  // 定义 Matches 字段，类型为 match.Matches
    IgnoredMatches   []match.IgnoredMatch  // 定义 IgnoredMatches 字段，类型为 match.IgnoredMatch 切片
    Packages         []pkg.Package  // 定义 Packages 字段，类型为 pkg.Package 切片
    Context          pkg.Context  // 定义 Context 字段，类型为 pkg.Context
    MetadataProvider vulnerability.MetadataProvider  // 定义 MetadataProvider 字段，类型为 vulnerability.MetadataProvider
    SBOM             *sbom.SBOM  // 定义 SBOM 字段，类型为 *sbom.SBOM
    AppConfig        interface{}  // 定义 AppConfig 字段，类型为 interface{}
    DBStatus         interface{}  // 定义 DBStatus 字段，类型为 interface{}
}
```