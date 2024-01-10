# `grype\cmd\grype\cli\options\search.go`

```
package options

import (
    "fmt"

    "github.com/anchore/clio"
    "github.com/anchore/syft/syft/pkg/cataloger"
    "github.com/anchore/syft/syft/source"
)

type search struct {
    Scope                    string `yaml:"scope" json:"scope" mapstructure:"scope"`  // 定义搜索结构体，包含作用域和是否包含索引/未索引归档的选项
    IncludeUnindexedArchives bool   `yaml:"unindexed-archives" json:"unindexed-archives" mapstructure:"unindexed-archives"`  // 是否包含未索引的归档
    IncludeIndexedArchives   bool   `yaml:"indexed-archives" json:"indexed-archives" mapstructure:"indexed-archives"`  // 是否包含索引的归档
}

var _ clio.PostLoader = (*search)(nil)  // 确保 search 结构体实现了 clio.PostLoader 接口

func defaultSearch(scope source.Scope) search {
    c := cataloger.DefaultSearchConfig()  // 获取默认的搜索配置
    return search{
        Scope:                    scope.String(),  // 设置搜索结构体的作用域
        IncludeUnindexedArchives: c.IncludeUnindexedArchives,  // 设置搜索结构体是否包含未索引的归档
        IncludeIndexedArchives:   c.IncludeIndexedArchives,  // 设置搜索结构体是否包含索引的归档
    }
}

func (cfg *search) PostLoad() error {
    scopeOption := cfg.GetScope()  // 获取搜索结构体的作用域选项
    if scopeOption == source.UnknownScope {  // 如果作用域选项为未知
        return fmt.Errorf("bad scope value %q", cfg.Scope)  // 返回错误信息
    }
    return nil
}

func (cfg search) GetScope() source.Scope {
    return source.ParseScope(cfg.Scope)  // 解析搜索结构体的作用域
}

func (cfg search) ToConfig() cataloger.SearchConfig {
    return cataloger.SearchConfig{
        IncludeIndexedArchives:   cfg.IncludeIndexedArchives,  // 返回搜索配置，包含是否包含索引的归档和是否包含未索引的归档
        IncludeUnindexedArchives: cfg.IncludeUnindexedArchives,
        Scope:                    cfg.GetScope(),
    }
}
```