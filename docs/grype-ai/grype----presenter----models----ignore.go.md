# `grype\grype\presenter\models\ignore.go`

```
package models

import "github.com/anchore/grype/grype/match"

type IgnoredMatch struct {
    Match
    AppliedIgnoreRules []IgnoreRule `json:"appliedIgnoreRules"`
}

type IgnoreRule struct {
    Vulnerability    string             `json:"vulnerability,omitempty"`  // 漏洞名称
    Reason           string             `json:"reason,omitempty"`         // 忽略规则的原因
    FixState         string             `json:"fix-state,omitempty"`      // 修复状态
    Package          *IgnoreRulePackage `json:"package,omitempty"`        // 忽略规则的包信息
    VexStatus        string             `json:"vex-status,omitempty"`     // VEX 状态
    VexJustification string             `json:"vex-justification,omitempty"`  // VEX 证明
}

type IgnoreRulePackage struct {
    Name     string `json:"name,omitempty"`      // 包名称
    Version  string `json:"version,omitempty"`   // 包版本
    Type     string `json:"type,omitempty"`      // 包类型
    Location string `json:"location,omitempty"`  // 包位置
}

func newIgnoreRule(r match.IgnoreRule) IgnoreRule {
    var ignoreRulePackage *IgnoreRulePackage

    // 如果有值需要填充，才设置忽略规则的包信息
    if p := r.Package; p.Name != "" || p.Version != "" || p.Type != "" || p.Location != "" {
        ignoreRulePackage = &IgnoreRulePackage{
            Name:     r.Package.Name,
            Version:  r.Package.Version,
            Type:     r.Package.Type,
            Location: r.Package.Location,
        }
    }

    return IgnoreRule{
        Vulnerability:    r.Vulnerability,
        Reason:           r.Reason,
        FixState:         r.FixState,
        Package:          ignoreRulePackage,
        VexStatus:        r.VexStatus,
        VexJustification: r.VexJustification,
    }
}

func mapIgnoreRules(rules []match.IgnoreRule) []IgnoreRule {
    var result []IgnoreRule

    // 遍历忽略规则列表，将每个规则转换为 IgnoreRule 类型，并添加到结果列表中
    for _, rule := range rules {
        result = append(result, newIgnoreRule(rule))
    }

    return result
}
```