# `grype\grype\vex\processor.go`

```
package vex

import (
    "fmt"

    gopenvex "github.com/openvex/go-vex/pkg/vex"  // 导入 openvex 包

    "github.com/anchore/grype/grype/match"  // 导入 match 包
    "github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
    "github.com/anchore/grype/grype/vex/openvex"  // 导入 openvex 包
)

type Status string  // 定义 Status 类型为字符串

const (
    StatusNotAffected        Status = Status(gopenvex.StatusNotAffected)  // 定义 StatusNotAffected 常量
    StatusAffected           Status = Status(gopenvex.StatusAffected)  // 定义 StatusAffected 常量
    StatusFixed              Status = Status(gopenvex.StatusFixed)  // 定义 StatusFixed 常量
    StatusUnderInvestigation Status = Status(gopenvex.StatusUnderInvestigation)  // 定义 StatusUnderInvestigation 常量
)

type Processor struct {  // 定义 Processor 结构体
    Options ProcessorOptions  // Processor 结构体包含 Options 字段
    impl    vexProcessorImplementation  // Processor 结构体包含 vexProcessorImplementation 字段
}

type vexProcessorImplementation interface {  // 定义 vexProcessorImplementation 接口
    // ReadVexDocuments 接收 vex 文件名列表，返回表示 VEX 信息的单个值，如果无法处理文件，则返回错误
    ReadVexDocuments(docs []string) (interface{}, error)

    // FilterMatches 接收底层 VEX 实现的 VEX 数据和扫描上下文以及匹配结果，过滤已修复和未受影响的结果，并将其移动到忽略匹配列表中
    FilterMatches(interface{}, []match.IgnoreRule, *pkg.Context, *match.Matches, []match.IgnoredMatch) (*match.Matches, []match.IgnoredMatch, error)

    // AugmentMatches 从加载的文档中读取已知受影响的 VEX 产品，并在 VEX 数据中标记产品为受影响时，将新结果添加到扫描结果中
    AugmentMatches(interface{}, []match.IgnoreRule, *pkg.Context, *match.Matches, []match.IgnoredMatch) (*match.Matches, []match.IgnoredMatch, error)
}

// getVexImplementation 函数返回 vex 处理器实现，可以在某个时刻读取选项并选择用户配置的实现
func getVexImplementation() vexProcessorImplementation {
    return openvex.New()  // 返回 openvex 包的新实例
}
// NewProcessor 返回一个新的 VEX 处理器。目前，默认为唯一的 vex 实现：OpenVEX
func NewProcessor(opts ProcessorOptions) *Processor {
    return &Processor{
        Options: opts,
        impl:    getVexImplementation(),
    }
}

// ProcessorOptions 捕获了 VEX 处理器的选项
type ProcessorOptions struct {
    Documents   []string
    IgnoreRules []match.IgnoreRule
}

// ApplyVEX 接收扫描运行的结果，并应用在 grype 调用中指定的文件中的任何 VEX 信息。任何被过滤的结果将被移动到被忽略的匹配切片中。
func (vm *Processor) ApplyVEX(pkgContext *pkg.Context, remainingMatches *match.Matches, ignoredMatches []match.IgnoredMatch) (*match.Matches, []match.IgnoredMatch, error) {
    var err error

    // 如果没有加载 VEX 文档，则直接通过匹配，相当于 NOOP
    if len(vm.Options.Documents) == 0 {
        return remainingMatches, ignoredMatches, nil
    }

    // 从所有传递的文档中读取 VEX 数据
    rawVexData, err := vm.impl.ReadVexDocuments(vm.Options.Documents)
    if err != nil {
        return nil, nil, fmt.Errorf("parsing vex document: %w", err)
    }

    vexRules := extractVexRules(vm.Options.IgnoreRules)

    remainingMatches, ignoredMatches, err = vm.impl.FilterMatches(
        rawVexData, vexRules, pkgContext, remainingMatches, ignoredMatches,
    )
    if err != nil {
        return nil, nil, fmt.Errorf("checking matches against VEX data: %w", err)
    }

    remainingMatches, ignoredMatches, err = vm.impl.AugmentMatches(
        rawVexData, vexRules, pkgContext, remainingMatches, ignoredMatches,
    )
    if err != nil {
        return nil, nil, fmt.Errorf("checking matches to augment from VEX data: %w", err)
    }

    return remainingMatches, ignoredMatches, nil
}

// extractVexRules 是一个实用函数，它接受一组忽略规则，并提取那些作用于 VEX 状态的规则。
# 从给定的规则列表中提取 Vex 规则
func extractVexRules(rules []match.IgnoreRule) []match.IgnoreRule:
    # 创建一个新的规则列表
    newRules := []match.IgnoreRule{}
    # 遍历原规则列表
    for _, r := range rules:
        # 如果规则的 Vex 状态不为空
        if r.VexStatus != "":
            # 将规则添加到新规则列表中
            newRules = append(newRules, r)
            # 设置新规则列表中最后一个规则的命名空间为 "vex"
            newRules[len(newRules)-1].Namespace = "vex"
    # 返回新规则列表
    return newRules
```