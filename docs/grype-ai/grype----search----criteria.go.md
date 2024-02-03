# `grype\grype\search\criteria.go`

```go
package search

import (
    "github.com/anchore/grype/grype/distro"  // 导入 distro 包
    "github.com/anchore/grype/grype/match"   // 导入 match 包
    "github.com/anchore/grype/grype/pkg"      // 导入 pkg 包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
    "github.com/anchore/grype/internal/log"   // 导入 log 包
)

var (
    ByCPE          Criteria = "by-cpe"  // 定义 Criteria 类型的变量 ByCPE，并赋值为 "by-cpe"
    ByLanguage     Criteria = "by-language"  // 定义 Criteria 类型的变量 ByLanguage，并赋值为 "by-language"
    ByDistro       Criteria = "by-distro"  // 定义 Criteria 类型的变量 ByDistro，并赋值为 "by-distro"
    CommonCriteria          = []Criteria{  // 定义 CommonCriteria 变量为 Criteria 类型的切片，并赋值为包含 ByLanguage 的切片
        ByLanguage,
    }
)

type Criteria string  // 定义 Criteria 类型为 string

func ByCriteria(store vulnerability.Provider, d *distro.Distro, p pkg.Package, upstreamMatcher match.MatcherType, criteria ...Criteria) ([]match.Match, error) {
    matches := make([]match.Match, 0)  // 创建一个空的 match.Match 类型的切片 matches
    for _, c := range criteria {  // 遍历传入的 criteria 切片
        switch c {  // 根据 c 的值进行判断
        case ByCPE:  // 如果 c 的值为 ByCPE
            m, err := ByPackageCPE(store, d, p, upstreamMatcher)  // 调用 ByPackageCPE 函数
            if err != nil {  // 如果出现错误
                log.Warnf("could not match by package CPE (package=%+v): %v", p, err)  // 打印警告日志
                continue  // 继续下一次循环
            }
            matches = append(matches, m...)  // 将 m 切片追加到 matches 切片中
        case ByLanguage:  // 如果 c 的值为 ByLanguage
            m, err := ByPackageLanguage(store, d, p, upstreamMatcher)  // 调用 ByPackageLanguage 函数
            if err != nil {  // 如果出现错误
                log.Warnf("could not match by package language (package=%+v): %v", p, err)  // 打印警告日志
                continue  // 继续下一次循环
            }
            matches = append(matches, m...)  // 将 m 切片追加到 matches 切片中
        case ByDistro:  // 如果 c 的值为 ByDistro
            m, err := ByPackageDistro(store, d, p, upstreamMatcher)  // 调用 ByPackageDistro 函数
            if err != nil {  // 如果出现错误
                log.Warnf("could not match by package distro (package=%+v): %v", p, err)  // 打印警告日志
                continue  // 继续下一次循环
            }
            matches = append(matches, m...)  // 将 m 切片追加到 matches 切片中
        }
    }
    return matches, nil  // 返回 matches 切片和 nil
}
```