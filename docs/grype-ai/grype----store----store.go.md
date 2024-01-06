# `grype\grype\store\store.go`

```
// 导入所需的包
package store

import (
	"github.com/anchore/grype/grype/match"  // 导入匹配相关的包
	"github.com/anchore/grype/grype/vulnerability"  // 导入漏洞相关的包
)

// 定义 Store 结构体，包含漏洞提供者、元数据提供者和排除提供者
type Store struct {
	vulnerability.Provider  // 漏洞提供者
	vulnerability.MetadataProvider  // 元数据提供者
	match.ExclusionProvider  // 排除提供者
}
```