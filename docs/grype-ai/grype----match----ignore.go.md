# `grype\grype\match\ignore.go`

```
// 包 match 包含了与匹配相关的功能
import (
	"github.com/bmatcuk/doublestar/v2" // 导入 doublestar 包
)

// IgnoredMatch 是一个被忽略的漏洞匹配，因为一个或多个 IgnoreRules 应用于匹配
type IgnoredMatch struct {
	Match // 继承自 Match 结构体

	// AppliedIgnoreRules 是应用于匹配的规则
	AppliedIgnoreRules []IgnoreRule // 忽略规则的切片
}

// IgnoreRule 指定了被忽略的漏洞匹配需要满足的条件
type IgnoreRule struct {
	Vulnerability    string            `yaml:"vulnerability" json:"vulnerability" mapstructure:"vulnerability"` // 漏洞名称
	// 其他字段可以根据需要添加注释
}
// Reason字段存储忽略规则的原因
Reason string `yaml:"reason" json:"reason" mapstructure:"reason"`

// Namespace字段存储忽略规则的命名空间
Namespace string `yaml:"namespace" json:"namespace" mapstructure:"namespace"`

// FixState字段存储忽略规则的修复状态
FixState string `yaml:"fix-state" json:"fix-state" mapstructure:"fix-state"`

// Package字段存储忽略规则的包信息
Package IgnoreRulePackage `yaml:"package" json:"package" mapstructure:"package"`

// VexStatus字段存储忽略规则的Vex状态
VexStatus string `yaml:"vex-status" json:"vex-status" mapstructure:"vex-status"`

// VexJustification字段存储忽略规则的Vex理由
VexJustification string `yaml:"vex-justification" json:"vex-justification" mapstructure:"vex-justification"`

// IgnoreRulePackage描述了IgnoreRule的特定于包的字段
type IgnoreRulePackage struct {
    // Name字段存储包的名称
    Name string `yaml:"name" json:"name" mapstructure:"name"`
    // Version字段存储包的版本
    Version string `yaml:"version" json:"version" mapstructure:"version"`
    // Language字段存储包的语言
    Language string `yaml:"language" json:"language" mapstructure:"language"`
    // Type字段存储包的类型
    Type string `yaml:"type" json:"type" mapstructure:"type"`
    // Location字段存储包的位置
    Location string `yaml:"location" json:"location" mapstructure:"location"`
}

// ApplyIgnoreRules遍历提供的匹配项，并对每个匹配项进行评估，确定是否应忽略该匹配项，
// 通过评估是否任何提供的忽略规则适用于该匹配项。如果任何规则适用于该匹配项，则所有
// ```
// 提供的规则都适用于该匹配项，所有
//```
// 根据给定的规则对匹配项进行忽略处理
// ApplyIgnoreRules 返回两个集合：未被忽略的匹配项和被忽略的匹配项
func ApplyIgnoreRules(matches Matches, rules []IgnoreRule) (Matches, []IgnoredMatch) {
    // 存储被忽略的匹配项
    var ignoredMatches []IgnoredMatch
    // 存储未被忽略的匹配项
    remainingMatches := NewMatches()

    // 遍历所有匹配项
    for _, match := range matches.Sorted() {
        // 存储适用的规则
        var applicableRules []IgnoreRule

        // 遍历所有规则，判断是否适用于当前匹配项
        for _, rule := range rules {
            if shouldIgnore(match, rule) {
                applicableRules = append(applicableRules, rule)
            }
        }

        // 如果有适用的规则，则将匹配项和适用的规则添加到被忽略的匹配项集合中
        if len(applicableRules) > 0 {
            ignoredMatches = append(ignoredMatches, IgnoredMatch{
                Match:              match,
                AppliedIgnoreRules: applicableRules,
		})

		// 继续下一次循环
		continue
	}

	// 将匹配项添加到剩余匹配列表中
	remainingMatches.Add(match)
}

// 返回剩余匹配列表和被忽略的匹配列表
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
		// 此规则未指定任何条件，因此不适用于匹配项
		// 如果没有条件，则返回 false
		return false
	}

	// 遍历忽略条件列表
	for _, condition := range ignoreConditions {
		// 如果有一个条件不满足，则该规则不适用于该匹配项
		if !condition(match) {
			return false
		}
	}

	// 所有规则中的条件都适用于该匹配项
	return true
}

// HasConditions 返回 true 如果忽略规则有条件
// 可以导致匹配项被忽略
func (ir IgnoreRule) HasConditions() bool {
	// 返回忽略规则的条件数量是否为 0
	return len(getIgnoreConditionsForRule(ir)) == 0
}
// ignoreCondition 是一个函数类型，它返回一个布尔值，指示给定的 Match 是否应该被忽略
type ignoreCondition func(match Match) bool

// getIgnoreConditionsForRule 函数根据给定的 IgnoreRule 返回一组 ignoreCondition 函数
func getIgnoreConditionsForRule(rule IgnoreRule) []ignoreCondition {
    var ignoreConditions []ignoreCondition

    // 如果规则中存在漏洞信息，则添加对应的 ignoreCondition 函数
    if v := rule.Vulnerability; v != "" {
        ignoreConditions = append(ignoreConditions, ifVulnerabilityApplies(v))
    }

    // 如果规则中存在命名空间信息，则添加对应的 ignoreCondition 函数
    if ns := rule.Namespace; ns != "" {
        ignoreConditions = append(ignoreConditions, ifNamespaceApplies(ns))
    }

    // 如果规则中存在包名信息，则添加对应的 ignoreCondition 函数
    if n := rule.Package.Name; n != "" {
        ignoreConditions = append(ignoreConditions, ifPackageNameApplies(n))
    }

    // 如果规则中存在包版本信息，则添加对应的 ignoreCondition 函数
    if v := rule.Package.Version; v != "" {
# 将 ifPackageVersionApplies(v) 的结果添加到 ignoreConditions 中
ignoreConditions = append(ignoreConditions, ifPackageVersionApplies(v))

# 如果 rule.Package.Language 不为空，将 ifPackageLanguageApplies(l) 的结果添加到 ignoreConditions 中
if l := rule.Package.Language; l != "":
    ignoreConditions = append(ignoreConditions, ifPackageLanguageApplies(l))

# 如果 rule.Package.Type 不为空，将 ifPackageTypeApplies(t) 的结果添加到 ignoreConditions 中
if t := rule.Package.Type; t != "":
    ignoreConditions = append(ignoreConditions, ifPackageTypeApplies(t))

# 如果 rule.Package.Location 不为空，将 ifPackageLocationApplies(l) 的结果添加到 ignoreConditions 中
if l := rule.Package.Location; l != "":
    ignoreConditions = append(ignoreConditions, ifPackageLocationApplies(l))

# 如果 rule.FixState 不为空，将 ifFixStateApplies(fs) 的结果添加到 ignoreConditions 中
if fs := rule.FixState; fs != "":
    ignoreConditions = append(ignoreConditions, ifFixStateApplies(fs))

# 返回 ignoreConditions
return ignoreConditions
# 定义一个函数，用于检查修复状态是否适用于给定的字符串
func ifFixStateApplies(fs string) ignoreCondition {
    # 返回一个函数，该函数接受一个匹配对象，并返回修复状态是否与给定字符串相等
    return func(match Match) bool {
        return fs == string(match.Vulnerability.Fix.State)
    }
}

# 定义一个函数，用于检查漏洞是否适用于给定的字符串
func ifVulnerabilityApplies(vulnerability string) ignoreCondition {
    # 返回一个函数，该函数接受一个匹配对象，并返回漏洞是否与给定字符串相等
    return func(match Match) bool {
        return vulnerability == match.Vulnerability.ID
    }
}

# 定义一个函数，用于检查命名空间是否适用于给定的字符串
func ifNamespaceApplies(namespace string) ignoreCondition {
    # 返回一个函数，该函数接受一个匹配对象，并返回命名空间是否与给定字符串相等
    return func(match Match) bool {
        return namespace == match.Vulnerability.Namespace
    }
}
# 根据包名判断是否适用于条件
func ifPackageNameApplies(name string) ignoreCondition {
    return func(match Match) bool {
        return name == match.Package.Name
    }
}

# 根据包版本判断是否适用于条件
func ifPackageVersionApplies(version string) ignoreCondition {
    return func(match Match) bool {
        return version == match.Package.Version
    }
}

# 根据包语言判断是否适用于条件
func ifPackageLanguageApplies(language string) ignoreCondition {
    return func(match Match) bool {
        return language == string(match.Package.Language)
    }
}

# 根据包类型判断是否适用于条件
func ifPackageTypeApplies(t string) ignoreCondition {
    return func(match Match) bool {
// 检查给定的类型是否与匹配对象的包类型相同
func ifPackageTypeApplies(t string) ignoreCondition {
	return func(match Match) bool {
		return t == string(match.Package.Type)
	}
}

// 检查包位置是否适用于给定的位置
func ifPackageLocationApplies(location string) ignoreCondition {
	return func(match Match) bool {
		return ruleLocationAppliesToMatch(location, match)
	}
}

// 检查规则位置是否适用于匹配对象的包位置
func ruleLocationAppliesToMatch(location string, match Match) bool {
	for _, packageLocation := range match.Package.Locations.ToSlice() {
		// 如果规则位置适用于包位置的实际路径，则返回 true
		if ruleLocationAppliesToPath(location, packageLocation.RealPath) {
			return true
		}

		// 如果规则位置适用于包位置的访问路径，则返回 true
		if ruleLocationAppliesToPath(location, packageLocation.AccessPath) {
			return true
		}
	}
}
# 返回布尔值 false
return false
}

# 判断规则位置是否适用于给定路径
func ruleLocationAppliesToPath(location, path string) bool {
    # 使用 doublestar 包中的 Match 函数判断规则位置是否匹配给定路径
    doesMatch, err := doublestar.Match(location, path)
    # 如果出现错误，返回 false
    if err != nil {
        return false
    }
    # 返回匹配结果
    return doesMatch
}
```