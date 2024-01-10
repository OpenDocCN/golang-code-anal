# `grype\grype\db\match_exclusion_provider.go`

```
package db

import (
    "fmt"

    grypeDB "github.com/anchore/grype/grype/db/v5"
    "github.com/anchore/grype/grype/match"
)

// MatchExclusionProvider 实现了 match.ExclusionProvider 接口
var _ match.ExclusionProvider = (*MatchExclusionProvider)(nil)

// MatchExclusionProvider 结构体
type MatchExclusionProvider struct {
    reader grypeDB.VulnerabilityMatchExclusionStoreReader
}

// NewMatchExclusionProvider 创建一个 MatchExclusionProvider 实例
func NewMatchExclusionProvider(reader grypeDB.VulnerabilityMatchExclusionStoreReader) *MatchExclusionProvider {
    return &MatchExclusionProvider{
        reader: reader,
    }
}

// 从 VulnerabilityMatchExclusion 构建 IgnoreRule 数组
func buildIgnoreRulesFromMatchExclusion(e grypeDB.VulnerabilityMatchExclusion) []match.IgnoreRule {
    var ignoreRules []match.IgnoreRule

    if len(e.Constraints) == 0 {
        ignoreRules = append(ignoreRules, match.IgnoreRule{Vulnerability: e.ID})
        return ignoreRules
    }

    for _, c := range e.Constraints {
        ignoreRules = append(ignoreRules, match.IgnoreRule{
            Vulnerability: e.ID,
            Namespace:     c.Vulnerability.Namespace,
            FixState:      string(c.Vulnerability.FixState),
            Package: match.IgnoreRulePackage{
                Name:     c.Package.Name,
                Language: c.Package.Language,
                Type:     c.Package.Type,
                Location: c.Package.Location,
                Version:  c.Package.Version,
            },
        })
    }

    return ignoreRules
}

// GetRules 从 MatchExclusionProvider 获取规则
func (pr *MatchExclusionProvider) GetRules(vulnerabilityID string) ([]match.IgnoreRule, error) {
    matchExclusions, err := pr.reader.GetVulnerabilityMatchExclusion(vulnerabilityID)
    if err != nil {
        return nil, fmt.Errorf("match exclusion provider failed to fetch records for vulnerability id='%s': %w", vulnerabilityID, err)
    }

    var ignoreRules []match.IgnoreRule

    for _, e := range matchExclusions {
        rules := buildIgnoreRulesFromMatchExclusion(e)
        ignoreRules = append(ignoreRules, rules...)
    }

    return ignoreRules, nil
}
```