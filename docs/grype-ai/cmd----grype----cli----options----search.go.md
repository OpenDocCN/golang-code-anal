# `grype\cmd\grype\cli\options\search.go`

```
package options

import (
	"fmt"  // 导入 fmt 包，用于格式化输出等操作

	"github.com/anchore/clio"  // 导入 anchore/clio 包，用于命令行交互
	"github.com/anchore/syft/syft/pkg/cataloger"  // 导入 anchore/syft/syft/pkg/cataloger 包，用于包目录的索引
	"github.com/anchore/syft/syft/source"  // 导入 anchore/syft/syft/source 包，用于包源的操作
)

type search struct {
	Scope                    string `yaml:"scope" json:"scope" mapstructure:"scope"`  // 定义 search 结构体的 Scope 字段，并指定其在 yaml、json、mapstructure 中的映射
	IncludeUnindexedArchives bool   `yaml:"unindexed-archives" json:"unindexed-archives" mapstructure:"unindexed-archives"`  // 定义 search 结构体的 IncludeUnindexedArchives 字段，并指定其在 yaml、json、mapstructure 中的映射
	IncludeIndexedArchives   bool   `yaml:"indexed-archives" json:"indexed-archives" mapstructure:"indexed-archives"`  // 定义 search 结构体的 IncludeIndexedArchives 字段，并指定其在 yaml、json、mapstructure 中的映射
}

var _ clio.PostLoader = (*search)(nil)  // search 结构体实现了 clio.PostLoader 接口

func defaultSearch(scope source.Scope) search {
	c := cataloger.DefaultSearchConfig()  // 获取默认的包目录搜索配置
# 返回一个包含搜索范围、是否包含未索引的存档和是否包含已索引的存档的结构体
return search{
    Scope:                    scope.String(),
    IncludeUnindexedArchives: c.IncludeUnindexedArchives,
    IncludeIndexedArchives:   c.IncludeIndexedArchives,
}

# 在加载后对搜索进行处理的方法
func (cfg *search) PostLoad() error {
    # 获取搜索范围的选项
    scopeOption := cfg.GetScope()
    # 如果搜索范围的选项是未知的，则返回错误
    if scopeOption == source.UnknownScope {
        return fmt.Errorf("bad scope value %q", cfg.Scope)
    }
    # 否则返回空
    return nil
}

# 获取搜索范围的方法
func (cfg search) GetScope() source.Scope {
    return source.ParseScope(cfg.Scope)
}

# 将搜索转换为配置的方法
func (cfg search) ToConfig() cataloger.Config {
# 返回一个Config对象，其中包含了SearchConfig和ExcludeBinaryOverlapByOwnership的配置信息
return cataloger.Config{
    # 设置SearchConfig对象的IncludeIndexedArchives属性为cfg.IncludeIndexedArchives的值
    Search: cataloger.SearchConfig{
        IncludeIndexedArchives:   cfg.IncludeIndexedArchives,
        # 设置SearchConfig对象的IncludeUnindexedArchives属性为cfg.IncludeUnindexedArchives的值
        IncludeUnindexedArchives: cfg.IncludeUnindexedArchives,
        # 设置SearchConfig对象的Scope属性为cfg.GetScope()的返回值
        Scope:                    cfg.GetScope(),
    },
    # 设置ExcludeBinaryOverlapByOwnership属性为true
    ExcludeBinaryOverlapByOwnership: true,
}
```