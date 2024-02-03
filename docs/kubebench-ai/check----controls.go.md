# `kubebench-aquasecurity\check\controls.go`

```go
// 版权声明和许可证信息
// 2017年版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0版授权
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则不得分发软件
// 根据许可证的规定分发的软件是基于"按原样"的基础分发的
// 没有任何明示或暗示的担保或条件
// 请参阅许可证以了解特定语言的权限和限制

// 包声明
package check

// 导入所需的包
import (
    "bytes"
    "encoding/json"
    "encoding/xml"
    "fmt"
    "time"

    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/service/securityhub"
    "github.com/golang/glog"
    "github.com/onsi/ginkgo/reporters"
    "github.com/spf13/viper"
    "gopkg.in/yaml.v2"
)

// 常量声明
const (
    // 未知状态，当无法找到AWS账户时使用
    UNKNOWN = "Unknown"
    // AWS Security Hub服务的ARN
    ARN = "arn:aws:securityhub:%s::product/aqua-security/kube-bench"
    // AWS Security Hub服务的模式
    SCHEMA = "2018-10-08"
    // Security Hub发现的类型
    TYPE = "Software and Configuration Checks/Industry and Regulatory Standards/CIS Kubernetes Benchmark"
)

// OverallControls 结构体定义
type OverallControls struct {
    Controls []*Controls
    Totals   Summary
}

// Controls 结构体定义
type Controls struct {
    ID      string   `yaml:"id" json:"id"` // 控制项ID
    Version string   `json:"version"` // 版本信息
    Text    string   `json:"text"` // 控制项描述
    Type    NodeType `json:"node_type"` // 节点类型
    Groups  []*Group `json:"tests"` // 控制项组
    Summary // 继承Summary结构体
}

// Group 结构体定义
type Group struct {
    ID     string   `yaml:"id" json:"section"` // 组ID
    Skip   bool     `yaml:"skip" json:"skip"` // 是否跳过
    Pass   int      `json:"pass"` // 通过数量
    Fail   int      `json:"fail"` // 失败数量
    Warn   int      `json:"warn"` // 警告数量
    // 其他字段...
}
    # 定义一个名为Info的整数类型字段，并使用json标签指定在JSON中的名称为"info"
    Info   int      `json:"info"`
    # 定义一个名为Text的字符串类型字段，并使用json标签指定在JSON中的名称为"desc"
    Text   string   `json:"desc"`
    # 定义一个名为Checks的Check类型切片字段，并使用json标签指定在JSON中的名称为"results"
    Checks []*Check `json:"results"`
// Summary 是控制检查运行结果的摘要
type Summary struct {
    Pass int `json:"total_pass"`  // 通过的数量
    Fail int `json:"total_fail"`  // 失败的数量
    Warn int `json:"total_warn"`  // 警告的数量
    Info int `json:"total_info"`  // 信息的数量
}

// Predicate 是对给定的 Group 和 Check 参数的断言
type Predicate func(group *Group, check *Check) bool

// NewControls 实例化一个新的主控制对象
func NewControls(t NodeType, in []byte) (*Controls, error) {
    c := new(Controls)

    // 从输入的字节流中解析 YAML 数据到控制对象
    err := yaml.Unmarshal(in, c)
    if err != nil {
        return nil, fmt.Errorf("failed to unmarshal YAML: %s", err)
    }

    // 检查控制对象的类型是否与指定的类型匹配
    if t != c.Type {
        return nil, fmt.Errorf("non-%s controls file specified", t)
    }

    return c, nil
}

// RunChecks 运行具有给定 Runner 的检查。只有过滤器 Predicate 返回 `true` 的检查才会运行
func (controls *Controls) RunChecks(runner Runner, filter Predicate, skipIDMap map[string]bool) Summary {
    var g []*Group
    m := make(map[string]*Group)
    controls.Summary.Pass, controls.Summary.Fail, controls.Summary.Warn, controls.Info = 0, 0, 0, 0
    // 遍历控件组列表
    for _, group := range controls.Groups {
        // 遍历每个控件组中的检查项
        for _, check := range group.Checks {

            // 如果不符合过滤条件，则跳过当前检查项
            if !filter(group, check) {
                continue
            }

            // 检查当前控件组和检查项是否在跳过ID映射中
            _, groupSkippedViaCmd := skipIDMap[group.ID]
            _, checkSkippedViaCmd := skipIDMap[check.ID]

            // 如果控件组被标记为跳过，或者在跳过ID映射中，或者检查项在跳过ID映射中，则将检查项类型设置为跳过
            if group.Skip || groupSkippedViaCmd || checkSkippedViaCmd {
                check.Type = SKIP
            }

            // 运行当前检查项，获取其状态
            state := runner.Run(check)

            // 将检查项的补充信息追加到测试信息中
            check.TestInfo = append(check.TestInfo, check.Remediation)

            // 检查是否已经添加了当前检查组
            if v, ok := m[group.ID]; !ok {
                // 创建一个具有相同信息的新组
                w := &Group{
                    ID:     group.ID,
                    Text:   group.Text,
                    Skip:   group.Skip,
                    Checks: []*Check{},
                }

                // 将当前检查项添加到新组中
                w.Checks = append(w.Checks, check)
                // 对新组进行总结
                summarizeGroup(w, state)

                // 将新组添加到已访问的组中
                m[w.ID] = w
                g = append(g, w)
            } else {
                // 将当前检查项添加到已存在的组中
                v.Checks = append(v.Checks, check)
                // 对已存在的组进行总结
                summarizeGroup(v, state)
            }

            // 对控件进行总结
            summarize(controls, state)
        }
    }

    // 更新控件组列表
    controls.Groups = g
    // 返回控件的总结信息
    return controls.Summary
// JSON encodes the results of last run to JSON.
func (controls *Controls) JSON() ([]byte, error) {
    return json.Marshal(controls)
}

// JUnit encodes the results of last run to JUnit.
func (controls *Controls) JUnit() ([]byte, error) {
    // 创建 JUnit 测试套件对象
    suite := reporters.JUnitTestSuite{
        Name:      controls.Text,
        TestCases: []reporters.JUnitTestCase{},
        Tests:     controls.Summary.Pass + controls.Summary.Fail + controls.Summary.Info + controls.Summary.Warn,
        Failures:  controls.Summary.Fail,
    }
    // 遍历控件组
    for _, g := range controls.Groups {
        // 遍历每个控件组的检查项
        for _, check := range g.Checks {
            jsonCheck := ""
            // 将检查项转换为 JSON 字符串
            jsonBytes, err := json.Marshal(check)
            if err != nil {
                // 如果转换失败，记录错误信息
                jsonCheck = fmt.Sprintf("Failed to marshal test into JSON: %v. Test as text: %#v", err, check)
            } else {
                // 如果转换成功，获取 JSON 字符串
                jsonCheck = string(jsonBytes)
            }
            // 创建 JUnit 测试用例对象
            tc := reporters.JUnitTestCase{
                Name:      fmt.Sprintf("%v %v", check.ID, check.Text),
                ClassName: g.Text,

                // 将整个 JSON 序列化作为系统输出，以便在需要更深层次调试时不丢失数据
                SystemOut: jsonCheck,
            }

            // 根据检查项的状态进行处理
            switch check.State {
            case FAIL:
                // 如果状态为 FAIL，记录失败信息
                tc.FailureMessage = &reporters.JUnitFailureMessage{Message: check.Remediation}
            case WARN, INFO:
                // 如果状态为 WARN 或 INFO，表示跳过测试，记录为跳过状态
                tc.Skipped = &reporters.JUnitSkipped{}
            case PASS:
                // 如果状态为 PASS，不做任何处理
            default:
                // 如果状态未识别，记录警告信息
                glog.Warningf("Unrecognized state %s", check.State)
            }

            // 将测试用例添加到测试套件中
            suite.TestCases = append(suite.TestCases, tc)
        }
    }

    // 创建字节缓冲区
    var b bytes.Buffer
    // 创建 XML 编码器
    encoder := xml.NewEncoder(&b)
    // 设置缩进格式
    encoder.Indent("", "    ")
    // 将测试套件对象编码为 XML 格式，并写入到字节缓冲区中
    err := encoder.Encode(suite)
    # 如果发生错误，返回空和错误信息
    if err != nil:
        return nil, fmt.Errorf("Failed to generate JUnit report: %s", err.Error())
    # 返回生成的 JUnit 报告字节流和空错误信息
    return b.Bytes(), nil
// ASFF encodes the results of last run to AWS Security Finding Format(ASFF).
// 将上次运行的结果编码为 AWS 安全发现格式（ASFF）

func (controls *Controls) ASFF() ([]*securityhub.AwsSecurityFinding, error) {
    // 初始化一个空的 AWS 安全发现结果切片
    fs := []*securityhub.AwsSecurityFinding{}
    // 获取 AWS 账号配置
    a, err := getConfig("AWS_ACCOUNT")
    if err != nil {
        return nil, err
    }
    // 获取集群 ARN 配置
    c, err := getConfig("CLUSTER_ARN")
    if err != nil {
        return nil, err
    }
    // 获取 AWS 区域配置
    region, err := getConfig("AWS_REGION")
    if err != nil {
        return nil, err
    }
    // 根据 AWS 区域构建 ARN
    arn := fmt.Sprintf(ARN, region)

    // 获取当前时间并格式化为 RFC3339 格式
    ti := time.Now()
    tf := ti.Format(time.RFC3339)
    }
    // 返回 AWS 安全发现结果切片和空错误
    return fs, nil
}

// 根据配置名称获取配置值
func getConfig(name string) (string, error) {
    // 从 viper 中获取配置值
    r := viper.GetString(name)
    // 如果配置值为空，返回错误
    if len(r) == 0 {
        return "", fmt.Errorf("%s not set", name)
    }
    // 返回配置值和空错误
    return r, nil
}

// 根据状态对控制进行汇总
func summarize(controls *Controls, state State) {
    // 根据状态对控制进行汇总
    switch state {
    case PASS:
        controls.Summary.Pass++
    case FAIL:
        controls.Summary.Fail++
    case WARN:
        controls.Summary.Warn++
    case INFO:
        controls.Summary.Info++
    default:
        glog.Warningf("Unrecognized state %s", state)
    }
}

// 根据状态对组进行汇总
func summarizeGroup(group *Group, state State) {
    // 根据状态对组进行汇总
    switch state {
    case PASS:
        group.Pass++
    case FAIL:
        group.Fail++
    case WARN:
        group.Warn++
    case INFO:
        group.Info++
    default:
        glog.Warningf("Unrecognized state %s", state)
    }
}
```