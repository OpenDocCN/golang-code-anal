# `grype\grype\vex\processor.go`

```
package vex

import (
	"fmt"

	gopenvex "github.com/openvex/go-vex/pkg/vex"  // 导入 openvex 包中的 go-vex 模块

	"github.com/anchore/grype/grype/match"  // 导入 anchore 包中的 grype 模块中的 match 子模块
	"github.com/anchore/grype/grype/pkg"  // 导入 anchore 包中的 grype 模块中的 pkg 子模块
	"github.com/anchore/grype/grype/vex/openvex"  // 导入 anchore 包中的 grype 模块中的 vex 子模块中的 openvex 子模块
)

type Status string  // 定义一个名为 Status 的字符串类型

const (
	StatusNotAffected        Status = Status(gopenvex.StatusNotAffected)  // 定义一个名为 StatusNotAffected 的常量，并赋值为 openvex 包中的 StatusNotAffected 常量
	StatusAffected           Status = Status(gopenvex.StatusAffected)  // 定义一个名为 StatusAffected 的常量，并赋值为 openvex 包中的 StatusAffected 常量
	StatusFixed              Status = Status(gopenvex.StatusFixed)  // 定义一个名为 StatusFixed 的常量，并赋值为 openvex 包中的 StatusFixed 常量
	StatusUnderInvestigation Status = Status(gopenvex.StatusUnderInvestigation)  // 定义一个名为 StatusUnderInvestigation 的常量，并赋值为 openvex 包中的 StatusUnderInvestigation 常量
)
// 定义 Processor 结构体，包含 Options 和 impl 两个字段
type Processor struct {
	Options ProcessorOptions
	impl    vexProcessorImplementation
}

// 定义 vexProcessorImplementation 接口，包含 ReadVexDocuments、FilterMatches 和 AugmentMatches 三个方法
type vexProcessorImplementation interface {
	// ReadVexDocuments 方法接收 vex 文件名列表，返回表示 VEX 信息的单个值，如果无法处理文件则返回错误
	ReadVexDocuments(docs []string) (interface{}, error)

	// FilterMatches 方法接收 VEX 实现的 VEX 数据和扫描上下文以及匹配结果，过滤 fixed 和 not_affected 结果，并将其移动到忽略匹配列表中
	FilterMatches(interface{}, []match.IgnoreRule, *pkg.Context, *match.Matches, []match.IgnoredMatch) (*match.Matches, []match.IgnoredMatch, error)

	// AugmentMatches 方法从加载的文档中读取已知受影响的 VEX 产品，并在 VEX 数据中将产品标记为受影响时，将新结果添加到扫描结果中
// AugmentMatches is a function that takes in an interface, a list of ignore rules, a context, a set of matches, and a list of ignored matches, and returns an augmented set of matches, a list of ignored matches, and an error.

// getVexImplementation is a function that returns the VEX processor implementation. It may read options and choose a user-configured implementation at some point.

// NewProcessor is a function that returns a new VEX processor with the given options. For now, it defaults to the only VEX implementation: OpenVEX.

// ProcessorOptions is a struct that captures the options of the VEX processor.
// Documents is a slice of strings that holds the paths to the VEX documents
// IgnoreRules is a slice of IgnoreRule objects that specify rules for ignoring matches

// ApplyVEX receives the results from a scan run and applies any VEX information
// in the files specified in the grype invocation. Any filtered results will
// be moved to the ignored matches slice.
func (vm *Processor) ApplyVEX(pkgContext *pkg.Context, remainingMatches *match.Matches, ignoredMatches []match.IgnoredMatch) (*match.Matches, []match.IgnoredMatch, error) {
	var err error

	// If no VEX documents are loaded, just pass through the matches, effectively NOOP
	if len(vm.Options.Documents) == 0 {
		return remainingMatches, ignoredMatches, nil
	}

	// Read VEX data from all passed documents
	rawVexData, err := vm.impl.ReadVexDocuments(vm.Options.Documents)
	if err != nil {
		return nil, nil, fmt.Errorf("parsing vex document: %w", err)
	}
# 从虚拟机选项中提取忽略规则
vexRules := extractVexRules(vm.Options.IgnoreRules)

# 使用虚拟机实现的方法过滤匹配项，返回剩余匹配项和被忽略的匹配项
remainingMatches, ignoredMatches, err = vm.impl.FilterMatches(
    rawVexData, vexRules, pkgContext, remainingMatches, ignoredMatches,
)
if err != nil {
    return nil, nil, fmt.Errorf("checking matches against VEX data: %w", err)
}

# 使用虚拟机实现的方法增强匹配项，返回剩余匹配项和被忽略的匹配项
remainingMatches, ignoredMatches, err = vm.impl.AugmentMatches(
    rawVexData, vexRules, pkgContext, remainingMatches, ignoredMatches,
)
if err != nil {
    return nil, nil, fmt.Errorf("checking matches to augment from VEX data: %w", err)
}

# 返回剩余匹配项和被忽略的匹配项
return remainingMatches, ignoredMatches, nil
// extractVexRules是一个实用函数，它接受一组忽略规则，并提取那些作用于VEX状态的规则。
func extractVexRules(rules []match.IgnoreRule) []match.IgnoreRule {
	// 创建一个新的规则切片
	newRules := []match.IgnoreRule{}
	// 遍历原始规则切片
	for _, r := range rules {
		// 如果规则的VEX状态不为空
		if r.VexStatus != "" {
			// 将规则添加到新的规则切片中
			newRules = append(newRules, r)
			// 设置新规则的命名空间为"vex"
			newRules[len(newRules)-1].Namespace = "vex"
		}
	}
	// 返回新的规则切片
	return newRules
}
```