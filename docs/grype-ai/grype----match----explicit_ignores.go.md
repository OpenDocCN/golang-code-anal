# `grype\grype\match\explicit_ignores.go`

```go
package match

import (
    "github.com/anchore/grype/internal/log"
)

// 定义全局变量，用于存储显式忽略规则
var explicitIgnoreRules []IgnoreRule

// 初始化函数，在包被导入时执行
func init() {
    // 定义结构体，用于存储忽略规则的类型、漏洞和软件包
    type ignoreValues struct {
        typ             string
        vulnerabilities []string
        packages        []string
    }

    // 遍历显式忽略规则列表
    for _, ignore := range explicitIgnores {
        // 遍历漏洞列表
        for _, vulnerability := range ignore.vulnerabilities {
            // 遍历软件包列表
            for _, packageName := range ignore.packages {
                // 将忽略规则添加到全局变量中
                explicitIgnoreRules = append(explicitIgnoreRules, IgnoreRule{
                    Vulnerability: vulnerability,
                    Package: IgnoreRulePackage{
                        Name: packageName,
                        Type: ignore.typ,
                    },
                })
            }
        }
    }
}

// ApplyExplicitIgnoreRules 根据定义的规则过滤匹配项，并过滤掉 grype 数据库中的匹配项
func ApplyExplicitIgnoreRules(provider ExclusionProvider, matches Matches) (Matches, []IgnoredMatch) {
    // 定义忽略规则列表
    var ignoreRules []IgnoreRule
    // 将显式忽略规则添加到忽略规则列表中
    ignoreRules = append(ignoreRules, explicitIgnoreRules...)

    // 遍历匹配项
    for _, m := range matches.Sorted() {
        // 获取匹配项对应的忽略规则
        r, err := provider.GetRules(m.Vulnerability.ID)

        if err != nil {
            // 输出警告日志
            log.Warnf("unable to get ignore rules for vuln id=%s", m.Vulnerability.ID)
            continue
        }

        // 将匹配项对应的忽略规则添加到忽略规则列表中
        ignoreRules = append(ignoreRules, r...)
    }

    // 应用忽略规则，返回过滤后的匹配项和被忽略的匹配项
    return ApplyIgnoreRules(matches, ignoreRules)
}
```