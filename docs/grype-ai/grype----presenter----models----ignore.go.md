# `grype\grype\presenter\models\ignore.go`

```
// 导入 match 包，用于匹配
import "github.com/anchore/grype/grype/match"

// 定义 IgnoredMatch 结构体，包含 Match 和 AppliedIgnoreRules 字段
type IgnoredMatch struct {
	Match
	AppliedIgnoreRules []IgnoreRule `json:"appliedIgnoreRules"`
}

// 定义 IgnoreRule 结构体，包含 Vulnerability、Reason、FixState、Package、VexStatus 和 VexJustification 字段
type IgnoreRule struct {
	Vulnerability    string             `json:"vulnerability,omitempty"`
	Reason           string             `json:"reason,omitempty"`
	FixState         string             `json:"fix-state,omitempty"`
	Package          *IgnoreRulePackage `json:"package,omitempty"`
	VexStatus        string             `json:"vex-status,omitempty"`
	VexJustification string             `json:"vex-justification,omitempty"`
}

// 定义 IgnoreRulePackage 结构体，包含 Name 字段
type IgnoreRulePackage struct {
	Name     string `json:"name,omitempty"`
```
以上代码是一个 Go 语言的模型定义，定义了一些结构体和它们的字段，用于表示匹配和忽略规则。
// 定义结构体字段 Version，表示版本号，omitempty表示在序列化时如果为空则忽略
Version  string `json:"version,omitempty"`
// 定义结构体字段 Type，表示类型，omitempty表示在序列化时如果为空则忽略
Type     string `json:"type,omitempty"`
// 定义结构体字段 Location，表示位置，omitempty表示在序列化时如果为空则忽略
Location string `json:"location,omitempty"`
}

// 根据给定的 match.IgnoreRule 创建一个新的 IgnoreRule 对象
func newIgnoreRule(r match.IgnoreRule) IgnoreRule {
	// 定义一个指向 IgnoreRulePackage 结构体的指针
	var ignoreRulePackage *IgnoreRulePackage

	// 只有当存在要填充的值时，才设置规则的包部分不为 nil
	if p := r.Package; p.Name != "" || p.Version != "" || p.Type != "" || p.Location != "" {
		// 如果满足条件，创建一个 IgnoreRulePackage 对象，并赋值给 ignoreRulePackage 指针
		ignoreRulePackage = &IgnoreRulePackage{
			Name:     r.Package.Name,
			Version:  r.Package.Version,
			Type:     r.Package.Type,
			Location: r.Package.Location,
		}
	}

	// 返回一个新的 IgnoreRule 对象
	return IgnoreRule{
		Vulnerability:    r.Vulnerability,
		Reason:           r.Reason,  # 将r的Reason字段赋值给Reason字段
		FixState:         r.FixState,  # 将r的FixState字段赋值给FixState字段
		Package:          ignoreRulePackage,  # 将ignoreRulePackage赋值给Package字段
		VexStatus:        r.VexStatus,  # 将r的VexStatus字段赋值给VexStatus字段
		VexJustification: r.VexJustification,  # 将r的VexJustification字段赋值给VexJustification字段
	}
}

func mapIgnoreRules(rules []match.IgnoreRule) []IgnoreRule {
	var result []IgnoreRule  # 创建一个空的IgnoreRule切片

	for _, rule := range rules {  # 遍历rules切片中的每个元素
		result = append(result, newIgnoreRule(rule))  # 将newIgnoreRule函数处理后的rule添加到result切片中
	}

	return result  # 返回处理后的result切片
}
```