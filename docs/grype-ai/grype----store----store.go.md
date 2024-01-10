# `grype\grype\store\store.go`

```
# 导入 store 包所需的依赖包
import (
    "github.com/anchore/grype/grype/match"  # 导入 match 包
    "github.com/anchore/grype/grype/vulnerability"  # 导入 vulnerability 包
)

# 定义 Store 结构体，包含 vulnerability.Provider、vulnerability.MetadataProvider 和 match.ExclusionProvider 接口
type Store struct {
    vulnerability.Provider  # Store 结构体实现了 vulnerability.Provider 接口
    vulnerability.MetadataProvider  # Store 结构体实现了 vulnerability.MetadataProvider 接口
    match.ExclusionProvider  # Store 结构体实现了 match.ExclusionProvider 接口
}
```