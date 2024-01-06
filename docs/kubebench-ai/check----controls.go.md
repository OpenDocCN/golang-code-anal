# `kubebench-aquasecurity\check\controls.go`

```
// 版权声明，版权归Aqua Security Software Ltd.所有
// 根据Apache License, Version 2.0许可证授权
// 除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

// 导入所需的包
package check

import (
    "bytes"  // 导入bytes包，用于操作字节
    "encoding/json"  // 导入json包，用于JSON数据的编解码
    "encoding/xml"   // 导入xml包，用于XML数据的编解码
// 导入 fmt 包，用于格式化输入输出
// 导入 time 包，用于处理时间相关操作
// 导入 AWS SDK 包，用于与 AWS 服务进行交互
// 导入 glog 包，用于日志记录
// 导入 ginkgo 包，用于测试报告生成
// 导入 viper 包，用于处理配置文件
// 导入 yaml 包，用于处理 YAML 格式数据

// 定义常量 UNKNOWN，表示 AWS 账户未知
const UNKNOWN = "Unknown"
// 定义常量 ARN，表示 AWS Security Hub 服务的资源标识符
const ARN = "arn:aws:securityhub:%s::product/aqua-security/kube-bench"
// 定义常量 SCHEMA，表示 AWS Security Hub 服务的架构版本
const SCHEMA = "2018-10-08"
// 定义常量 TYPE，表示 Security Hub 发现的类型
const TYPE = "Software and Configuration Checks/Industry and Regulatory Standards/CIS Kubernetes Benchmark"
// OverallControls 结构体定义了一个包含 Controls 切片和 Summary 结构体的类型
type OverallControls struct {
	Controls []*Controls
	Totals   Summary
}

// Controls 结构体定义了用于检查主节点的所有控制项
type Controls struct {
	ID      string   `yaml:"id" json:"id"`  // 控制项的ID
	Version string   `json:"version"`       // 控制项的版本
	Text    string   `json:"text"`          // 控制项的文本描述
	Type    NodeType `json:"node_type"`     // 控制项的节点类型
	Groups  []*Group `json:"tests"`         // 控制项的测试组
	Summary                                  // 控制项的摘要信息
}

// Group 结构体定义了一组相似的检查
type Group struct {
	ID     string   `yaml:"id" json:"section"`  // 组的ID
```
// Skip 表示是否跳过某个检查
// Pass 表示通过的数量
// Fail 表示失败的数量
// Warn 表示警告的数量
// Info 表示信息的数量
// Text 表示描述信息
// Checks 表示检查结果的数组

// Summary 是对运行控制检查结果的总结
// Pass 表示总通过数量
// Fail 表示总失败数量
// Warn 表示总警告数量
// Info 表示总信息数量

// Predicate 是对给定的 Group 和 Check 参数的断言
// NewControls 实例化一个新的 Controls 对象。
func NewControls(t NodeType, in []byte) (*Controls, error) {
    c := new(Controls) // 创建一个新的 Controls 对象

    err := yaml.Unmarshal(in, c) // 使用 yaml 解析输入的字节流，并将结果存储到 c 中
    if err != nil {
        return nil, fmt.Errorf("failed to unmarshal YAML: %s", err) // 如果解析失败，返回错误信息
    }

    if t != c.Type { // 检查指定的类型是否与 Controls 对象的类型匹配
        return nil, fmt.Errorf("non-%s controls file specified", t) // 如果不匹配，返回错误信息
    }

    return c, nil // 返回创建的 Controls 对象
}

// RunChecks 使用给定的 Runner 运行检查。只有满足筛选条件 Predicate 返回 true 的检查才会运行。
func (controls *Controls) RunChecks(runner Runner, filter Predicate, skipIDMap map[string]bool) Summary {
    var g []*Group // 创建一个 Group 切片
    m := make(map[string]*Group) // 创建一个字符串到 Group 指针的映射
# 初始化控件的统计数据
controls.Summary.Pass, controls.Summary.Fail, controls.Summary.Warn, controls.Info = 0, 0, 0, 0

# 遍历控件组
for _, group := range controls.Groups:
    # 遍历控件组中的检查项
    for _, check := range group.Checks:
        # 如果不符合过滤条件，则跳过当前检查项
        if !filter(group, check):
            continue
        
        # 检查是否在跳过ID映射中
        _, groupSkippedViaCmd := skipIDMap[group.ID]
        _, checkSkippedViaCmd := skipIDMap[check.ID]

        # 如果控件组被跳过或者通过命令行跳过，或者检查项通过命令行跳过，则将检查项类型设置为跳过
        if group.Skip || groupSkippedViaCmd || checkSkippedViaCmd:
            check.Type = SKIP

        # 运行检查项，获取状态
        state := runner.Run(check)

        # 将检查项的修复信息追加到测试信息中
        check.TestInfo = append(check.TestInfo, check.Remediation)
// 检查我们是否已经添加了这个检查组。
if v, ok := m[group.ID]; !ok {
    // 创建一个具有相同信息的新组
    w := &Group{
        ID:     group.ID,
        Text:   group.Text,
        Skip:   group.Skip,
        Checks: []*Check{},
    }

    // 将此检查添加到新组
    w.Checks = append(w.Checks, check)
    summarizeGroup(w, state)

    // 将其添加到我们已经访问过的组中。
    m[w.ID] = w
    g = append(g, w)
} else {
    v.Checks = append(v.Checks, check)
    summarizeGroup(v, state)
}
		}

		// 对控件进行总结
		summarize(controls, state)
	}

	// 将控件组信息赋值给变量 g
	controls.Groups = g
	// 返回控件的总结信息
	return controls.Summary
}

// 将最后一次运行的结果编码为 JSON 格式
func (controls *Controls) JSON() ([]byte, error) {
	// 使用 json.Marshal 方法将控件对象编码为 JSON 格式的字节流
	return json.Marshal(controls)
}

// 将最后一次运行的结果编码为 JUnit 格式
func (controls *Controls) JUnit() ([]byte, error) {
	// 创建 JUnit 测试套件对象
	suite := reporters.JUnitTestSuite{
		Name:      controls.Text,
		TestCases: []reporters.JUnitTestCase{},
		// 计算测试总数，包括通过、失败、信息和警告
		Tests:     controls.Summary.Pass + controls.Summary.Fail + controls.Summary.Info + controls.Summary.Warn,
		// 计算失败的测试总数
		Failures:  controls.Summary.Fail,
	}
	// 遍历控制组
	for _, g := range controls.Groups {
		// 遍历每个控制组的检查
		for _, check := range g.Checks {
			// 将检查对象转换为 JSON 字符串
			jsonCheck := ""
			jsonBytes, err := json.Marshal(check)
			// 如果转换出错，记录错误信息和原始检查对象
			if err != nil {
				jsonCheck = fmt.Sprintf("Failed to marshal test into JSON: %v. Test as text: %#v", err, check)
			} else {
				// 否则，将 JSON 字节转换为字符串
				jsonCheck = string(jsonBytes)
			}
			// 创建 JUnitTestCase 对象
			tc := reporters.JUnitTestCase{
				// 设置测试用例的名称为检查的 ID 和文本
				Name:      fmt.Sprintf("%v %v", check.ID, check.Text),
				// 设置测试用例的类名为控制组的文本
				ClassName: g.Text,

				// 将整个 JSON 序列化作为系统输出，以便在需要更深层次调试时不丢失数据
				SystemOut: jsonCheck,
			}
// 根据 check.State 的不同值进行不同的处理
switch check.State {
    // 如果状态为 FAIL，则设置测试用例的失败消息为 check.Remediation
    case FAIL:
        tc.FailureMessage = &reporters.JUnitFailureMessage{Message: check.Remediation}
    // 如果状态为 WARN 或 INFO，则将测试用例标记为跳过
    case WARN, INFO:
        // WARN 和 INFO 都是跳过测试的不同版本。无论如何，以其他方式报告它都会是一个误报
        tc.Skipped = &reporters.JUnitSkipped{}
    // 如果状态为 PASS，则不做任何处理
    case PASS:
    // 如果状态为其他值，则记录警告日志，表示状态未被识别
    default:
        glog.Warningf("Unrecognized state %s", check.State)
}

// 将处理后的测试用例添加到测试套件中
suite.TestCases = append(suite.TestCases, tc)

// 创建一个字节缓冲区
var b bytes.Buffer
// 创建一个 XML 编码器，并设置缩进格式
encoder := xml.NewEncoder(&b)
encoder.Indent("", "    ")
// 使用编码器将测试套件 suite 编码为 XML 格式，并将结果存储在字节缓冲区中
err := encoder.Encode(suite)
	if err != nil {
		// 如果发生错误，返回一个格式化的错误信息
		return nil, fmt.Errorf("Failed to generate JUnit report: %s", err.Error())
	}

	// 返回生成的 JUnit 报告的字节流和一个空的错误
	return b.Bytes(), nil
}

// ASFF 将最后一次运行的结果编码为 AWS 安全发现格式（ASFF）
func (controls *Controls) ASFF() ([]*securityhub.AwsSecurityFinding, error) {
	// 创建一个空的 AWS 安全发现格式数组
	fs := []*securityhub.AwsSecurityFinding{}
	// 获取 AWS 账户配置
	a, err := getConfig("AWS_ACCOUNT")
	if err != nil {
		// 如果发生错误，返回错误信息
		return nil, err
	}
	// 获取集群 ARN 配置
	c, err := getConfig("CLUSTER_ARN")
	if err != nil {
		// 如果发生错误，返回错误信息
		return nil, err
	}
	// 获取 AWS 区域配置
	region, err := getConfig("AWS_REGION")
	if err != nil {
		// 如果发生错误，返回 nil 和错误信息
		return nil, err
	}
	// 根据 region 格式化 ARN
	arn := fmt.Sprintf(ARN, region)

	// 获取当前时间并格式化为 RFC3339 格式
	ti := time.Now()
	tf := ti.Format(time.RFC3339)
	// 遍历所有的控制组
	for _, g := range controls.Groups {
		// 遍历每个控制组的检查项
		for _, check := range g.Checks {
			// 如果检查状态为 FAIL 或 WARN
			if check.State == FAIL || check.State == WARN {
				// 如果实际值长度超过 1024，截取前 1023 个字符
				actualValue := check.ActualValue
				if len(check.ActualValue) > 1024 {
					actualValue = check.ActualValue[0:1023]
				}
				// 创建安全发现对象
				f := securityhub.AwsSecurityFinding{
					AwsAccountId:  aws.String(a),
					Confidence:    aws.Int64(100),
					GeneratorId:   aws.String(fmt.Sprintf("%s/cis-kubernetes-benchmark/%s/%s", arn, controls.Version, check.ID)),
					Id:            aws.String(fmt.Sprintf("%s%sEKSnodeID+%s%s", arn, a, check.ID, tf)),
					CreatedAt:     aws.String(tf),
# 设置描述字段为指定的文本
Description:   aws.String(check.Text),

# 设置产品 ARN 字段为指定的 ARN
ProductArn:    aws.String(arn),

# 设置模式版本字段为指定的模式版本
SchemaVersion: aws.String(SCHEMA),

# 设置标题字段为格式化后的检查 ID 和文本
Title:         aws.String(fmt.Sprintf("%s %s", check.ID, check.Text)),

# 设置更新时间字段为指定的时间戳
UpdatedAt:     aws.String(tf),

# 设置类型字段为指定的类型
Types:         []*string{aws.String(TYPE)},

# 设置严重性字段为高严重性
Severity: &securityhub.Severity{
    Label: aws.String(securityhub.SeverityLabelHigh),
},

# 设置修复建议字段为指定的修复建议文本
Remediation: &securityhub.Remediation{
    Recommendation: &securityhub.Recommendation{
        Text: aws.String(check.Remediation),
    },
},

# 设置产品字段为包含指定键值对的映射
ProductFields: map[string]*string{
    "Reason":          aws.String(check.Reason),
    "Actual result":   aws.String(actualValue),
    "Expected result": aws.String(check.ExpectedResult),
    "Section":         aws.String(fmt.Sprintf("%s %s", controls.ID, controls.Text)),
    "Subsection":      aws.String(fmt.Sprintf("%s %s", g.ID, g.Text)),
},
# 定义一个结构体，包含了一些字段和资源列表
type Finding struct {
    Id:   aws.String(c),  # 设置资源的 ID
    Type: aws.String(TYPE),  # 设置资源的类型
}

# 定义一个函数，用于获取配置信息
func getConfig(name string) (string, error) {
    # 从配置文件中获取指定名称的配置信息
    r := viper.GetString(name)
    # 如果获取的配置信息为空，返回错误
    if len(r) == 0 {
        return "", fmt.Errorf("%s not set", name)
    }
    # 返回获取的配置信息
    return r, nil
}
// summarize 函数用于根据状态对控件进行汇总统计
func summarize(controls *Controls, state State) {
	// 根据状态进行不同的处理
	switch state {
	case PASS:
		// 如果状态为 PASS，则将控件的 Pass 数量加一
		controls.Summary.Pass++
	case FAIL:
		// 如果状态为 FAIL，则将控件的 Fail 数量加一
		controls.Summary.Fail++
	case WARN:
		// 如果状态为 WARN，则将控件的 Warn 数量加一
		controls.Summary.Warn++
	case INFO:
		// 如果状态为 INFO，则将控件的 Info 数量加一
		controls.Summary.Info++
	default:
		// 如果状态未被识别，则记录警告日志
		glog.Warningf("Unrecognized state %s", state)
	}
}

// summarizeGroup 函数用于根据状态对组进行汇总统计
func summarizeGroup(group *Group, state State) {
	// 根据状态进行不同的处理
	switch state {
	case PASS:
		// 如果状态为 PASS，则将组的 Pass 数量加一
		group.Pass++
# 根据不同的状态对组内的计数器进行增加
case FAIL:
    # 如果状态为 FAIL，则将组内的 Fail 计数器加一
    group.Fail++
case WARN:
    # 如果状态为 WARN，则将组内的 Warn 计数器加一
    group.Warn++
case INFO:
    # 如果状态为 INFO，则将组内的 Info 计数器加一
    group.Info++
default:
    # 如果状态不是以上三种情况，则记录警告日志，表示状态未被识别
    glog.Warningf("Unrecognized state %s", state)
}
```