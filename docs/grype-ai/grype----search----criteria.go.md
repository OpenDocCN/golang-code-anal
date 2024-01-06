# `grype\grype\search\criteria.go`

```
// 导入所需的包
package search

import (
	"github.com/anchore/grype/grype/distro"  // 导入操作系统发行版相关的包
	"github.com/anchore/grype/grype/match"   // 导入匹配相关的包
	"github.com/anchore/grype/grype/pkg"     // 导入软件包相关的包
	"github.com/anchore/grype/grype/vulnerability"  // 导入漏洞相关的包
	"github.com/anchore/grype/internal/log"  // 导入日志相关的包
)

// 定义全局变量
var (
	ByCPE          Criteria = "by-cpe"  // 根据 CPE 进行搜索的标准
	ByLanguage     Criteria = "by-language"  // 根据语言进行搜索的标准
	ByDistro       Criteria = "by-distro"  // 根据操作系统发行版进行搜索的标准
	CommonCriteria          = []Criteria{  // 常用的搜索标准
		ByLanguage,  // 根据语言进行搜索
	}
)

// 定义搜索标准类型
type Criteria string
# 根据给定的条件从漏洞提供者中获取匹配的漏洞信息
func ByCriteria(store vulnerability.Provider, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType, criteria ...Criteria) ([]match.Match, error) {
    # 初始化一个空的匹配结果列表
    matches := make([]match.Match, 0)
    # 遍历每个给定的条件
    for _, c := range criteria {
        # 根据不同的条件进行不同的处理
        switch c {
            # 如果条件是 ByCPE
            case ByCPE:
                # 调用 ByPackageCPE 函数获取匹配的漏洞信息
                m, err := ByPackageCPE(store, d, p, upstreamMatcher)
                # 如果出现错误，记录日志并继续下一个条件
                if err != nil {
                    log.Warnf("could not match by package CPE (package=%+v): %v", p, err)
                    continue
                }
                # 将匹配结果添加到匹配列表中
                matches = append(matches, m...)
            # 如果条件是 ByLanguage
            case ByLanguage:
                # 调用 ByPackageLanguage 函数获取匹配的漏洞信息
                m, err := ByPackageLanguage(store, d, p, upstreamMatcher)
                # 如果出现错误，记录日志并继续下一个条件
                if err != nil {
                    log.Warnf("could not match by package language (package=%+v): %v", p, err)
                    continue
                }
                # 将匹配结果添加到匹配列表中
                matches = append(matches, m...)
            # 如果条件是 ByDistro
            case ByDistro:
			// 通过包的发行版匹配函数获取匹配结果
			m, err := ByPackageDistro(store, d, p, upstreamMatcher)
			// 如果发生错误，记录警告日志并继续下一次循环
			if err != nil {
				log.Warnf("could not match by package distro (package=%+v): %v", p, err)
				continue
			}
			// 将匹配结果追加到匹配列表中
			matches = append(matches, m...)
		}
	}
	// 返回匹配结果列表和空错误
	return matches, nil
}
```