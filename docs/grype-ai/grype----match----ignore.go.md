# `grype\grype\match\ignore.go`

```go
package match

import (
    "github.com/bmatcuk/doublestar/v2"
)

// An IgnoredMatch is a vulnerability Match that has been ignored because one or more IgnoreRules applied to the match.
type IgnoredMatch struct {
    Match

    // AppliedIgnoreRules are the rules that were applied to the match that caused Grype to ignore it.
    AppliedIgnoreRules []IgnoreRule
}

// An IgnoreRule specifies criteria for a vulnerability match to meet in order
// to be ignored. Not all criteria (fields) need to be specified, but all
// specified criteria must be met by the vulnerability match in order for the
// rule to apply.
type IgnoreRule struct {
    Vulnerability    string            `yaml:"vulnerability" json:"vulnerability" mapstructure:"vulnerability"`
    Reason           string            `yaml:"reason" json:"reason" mapstructure:"reason"`
    Namespace        string            `yaml:"namespace" json:"namespace" mapstructure:"namespace"`
    FixState         string            `yaml:"fix-state" json:"fix-state" mapstructure:"fix-state"`
    Package          IgnoreRulePackage `yaml:"package" json:"package" mapstructure:"package"`
    VexStatus        string            `yaml:"vex-status" json:"vex-status" mapstructure:"vex-status"`
    VexJustification string            `yaml:"vex-justification" json:"vex-justification" mapstructure:"vex-justification"`
}

// IgnoreRulePackage describes the Package-specific fields that comprise the IgnoreRule.
type IgnoreRulePackage struct {
    Name     string `yaml:"name" json:"name" mapstructure:"name"`
    Version  string `yaml:"version" json:"version" mapstructure:"version"`
    Language string `yaml:"language" json:"language" mapstructure:"language"`
    Type     string `yaml:"type" json:"type" mapstructure:"type"`
    Location string `yaml:"location" json:"location" mapstructure:"location"`
}

// ApplyIgnoreRules iterates through the provided matches and, for each match,
// 根据提供的 IgnoreRules 判断是否应该忽略匹配项。如果任何规则适用于匹配项，则将所有适用规则附加到匹配项以形成 IgnoredMatch。
// ApplyIgnoreRules 返回两个集合：未被忽略的匹配项和被忽略的匹配项。
func ApplyIgnoreRules(matches Matches, rules []IgnoreRule) (Matches, []IgnoredMatch) {
    // 初始化被忽略的匹配项集合
    var ignoredMatches []IgnoredMatch
    // 初始化未被忽略的匹配项集合
    remainingMatches := NewMatches()

    // 遍历排序后的匹配项
    for _, match := range matches.Sorted() {
        // 初始化适用规则集合
        var applicableRules []IgnoreRule

        // 遍历规则，判断是否应该忽略匹配项
        for _, rule := range rules {
            if shouldIgnore(match, rule) {
                applicableRules = append(applicableRules, rule)
            }
        }

        // 如果有适用规则，则将匹配项和适用规则添加到被忽略的匹配项集合中
        if len(applicableRules) > 0 {
            ignoredMatches = append(ignoredMatches, IgnoredMatch{
                Match:              match,
                AppliedIgnoreRules: applicableRules,
            })

            continue
        }

        // 否则将匹配项添加到未被忽略的匹配项集合中
        remainingMatches.Add(match)
    }

    // 返回未被忽略的匹配项和被忽略的匹配项集合
    return remainingMatches, ignoredMatches
}

// 判断是否应该忽略匹配项
func shouldIgnore(match Match, rule IgnoreRule) bool {
    // VEX 规则由 vex 处理器处理
    if rule.VexStatus != "" {
        return false
    }

    // 获取规则的忽略条件
    ignoreConditions := getIgnoreConditionsForRule(rule)
    if len(ignoreConditions) == 0 {
        // 如果规则未指定任何条件，则不适用于匹配项
        return false
    }

    // 遍历忽略条件，判断是否应该忽略匹配项
    for _, condition := range ignoreConditions {
        if !condition(match) {
            // 一旦一个规则条件不适用，我们就知道这个规则不适用于匹配项
            return false
        }
    }

    // 所有规则条件都适用于匹配项
    return true
}

// 如果忽略规则具有可能导致匹配项被忽略的条件，则返回 true
func (ir IgnoreRule) HasConditions() bool {
    return len(getIgnoreConditionsForRule(ir)) == 0
}
// ignoreCondition 是一个函数类型，返回一个布尔值，指示是否应忽略给定的匹配项
type ignoreCondition func(match Match) bool

// 根据 IgnoreRule 获取应用于规则的忽略条件列表
func getIgnoreConditionsForRule(rule IgnoreRule) []ignoreCondition {
    var ignoreConditions []ignoreCondition

    // 如果规则中存在漏洞信息，则添加相应的忽略条件
    if v := rule.Vulnerability; v != "" {
        ignoreConditions = append(ignoreConditions, ifVulnerabilityApplies(v))
    }

    // 如果规则中存在命名空间信息，则添加相应的忽略条件
    if ns := rule.Namespace; ns != "" {
        ignoreConditions = append(ignoreConditions, ifNamespaceApplies(ns))
    }

    // 如果规则中存在包名信息，则添加相应的忽略条件
    if n := rule.Package.Name; n != "" {
        ignoreConditions = append(ignoreConditions, ifPackageNameApplies(n))
    }

    // 如果规则中存在包版本信息，则添加相应的忽略条件
    if v := rule.Package.Version; v != "" {
        ignoreConditions = append(ignoreConditions, ifPackageVersionApplies(v))
    }

    // 如果规则中存在包语言信息，则添加相应的忽略条件
    if l := rule.Package.Language; l != "" {
        ignoreConditions = append(ignoreConditions, ifPackageLanguageApplies(l))
    }

    // 如果规则中存在包类型信息，则添加相应的忽略条件
    if t := rule.Package.Type; t != "" {
        ignoreConditions = append(ignoreConditions, ifPackageTypeApplies(t))
    }

    // 如果规则中存在包位置信息，则添加相应的忽略条件
    if l := rule.Package.Location; l != "" {
        ignoreConditions = append(ignoreConditions, ifPackageLocationApplies(l))
    }

    // 如果规则中存在修复状态信息，则添加相应的忽略条件
    if fs := rule.FixState; fs != "" {
        ignoreConditions = append(ignoreConditions, ifFixStateApplies(fs))
    }

    return ignoreConditions
}

// 根据修复状态信息返回相应的忽略条件函数
func ifFixStateApplies(fs string) ignoreCondition {
    return func(match Match) bool {
        return fs == string(match.Vulnerability.Fix.State)
    }
}

// 根据漏洞信息返回相应的忽略条件函数
func ifVulnerabilityApplies(vulnerability string) ignoreCondition {
    return func(match Match) bool {
        return vulnerability == match.Vulnerability.ID
    }
}

// 根据命名空间信息返回相应的忽略条件函数
func ifNamespaceApplies(namespace string) ignoreCondition {
    return func(match Match) bool {
        return namespace == match.Vulnerability.Namespace
    }
}

// 根据包名信息返回相应的忽略条件函数
func ifPackageNameApplies(name string) ignoreCondition {
    return func(match Match) bool {
        return name == match.Package.Name
    }
}
# 根据给定的版本号创建一个忽略条件函数
func ifPackageVersionApplies(version string) ignoreCondition {
    # 返回一个函数，判断匹配对象的包版本是否符合给定的版本号
    return func(match Match) bool {
        return version == match.Package.Version
    }
}

# 根据给定的语言创建一个忽略条件函数
func ifPackageLanguageApplies(language string) ignoreCondition {
    # 返回一个函数，判断匹配对象的包语言是否符合给定的语言
    return func(match Match) bool {
        return language == string(match.Package.Language)
    }
}

# 根据给定的类型创建一个忽略条件函数
func ifPackageTypeApplies(t string) ignoreCondition {
    # 返回一个函数，判断匹配对象的包类型是否符合给定的类型
    return func(match Match) bool {
        return t == string(match.Package.Type)
    }
}

# 根据给定的位置创建一个忽略条件函数
func ifPackageLocationApplies(location string) ignoreCondition {
    # 返回一个函数，判断匹配对象的包位置是否符合给定的位置
    return func(match Match) bool {
        return ruleLocationAppliesToMatch(location, match)
    }
}

# 判断给定位置是否适用于匹配对象
func ruleLocationAppliesToMatch(location string, match Match) bool {
    # 遍历匹配对象的包位置，判断给定位置是否适用于实际路径或访问路径
    for _, packageLocation := range match.Package.Locations.ToSlice() {
        if ruleLocationAppliesToPath(location, packageLocation.RealPath) {
            return true
        }

        if ruleLocationAppliesToPath(location, packageLocation.AccessPath) {
            return true
        }
    }

    return false
}

# 判断给定位置是否适用于给定路径
func ruleLocationAppliesToPath(location, path string) bool {
    # 使用 doublestar 包判断给定位置是否匹配给定路径
    doesMatch, err := doublestar.Match(location, path)
    if err != nil {
        return false
    }

    return doesMatch
}
```