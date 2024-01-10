# `kubebench-aquasecurity\check\check.go`

```
// 代码版权声明和许可证信息
// 该代码受 Apache 许可证版本 2.0 的许可
// 可以在遵守许可证的情况下使用该文件
// 可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则按"原样"分发软件
// 没有任何明示或暗示的担保或条件
// 请参阅许可证以获取特定语言的权限和限制

// 导入所需的包
package check

import (
    "bytes"
    "fmt"
    "os/exec"
    "strings"

    "github.com/golang/glog"
)

// NodeType 表示节点的类型（主节点、工作节点）
type NodeType string

// State 是控制检查的状态
type State string

const (
    // PASS 检查通过
    PASS State = "PASS"
    // FAIL 检查失败
    FAIL State = "FAIL"
    // WARN 无法执行检查
    WARN State = "WARN"
    // INFO 信息消息
    INFO State = "INFO"

    // SKIP 用于跳过检查
    SKIP = "skip"

    // MASTER 主节点
    MASTER NodeType = "master"
    // NODE 工作节点
    NODE NodeType = "node"
    // FEDERATED 联合部署
    FEDERATED NodeType = "federated"

    // ETCD etcd 节点
    ETCD NodeType = "etcd"
    // CONTROLPLANE 控制平面节点
    CONTROLPLANE NodeType = "controlplane"
    // POLICIES 用于运行策略的节点
    POLICIES NodeType = "policies"
    // MANAGEDSERVICES 用于运行托管服务的节点
    MANAGEDSERVICES = "managedservices"

    // MANUAL 检查类型
    MANUAL string = "manual"
)

// Check 包含 CIS Kubernetes 文档中推荐的信息
type Check struct {
    ID                string   `yaml:"id" json:"test_number"`
    Text              string   `json:"test_desc"`
    Audit             string   `json:"audit"`
    # 定义一个字符串类型的字段，用于存储审计环境
    AuditEnv string `yaml:"audit_env"`
    # 定义一个字符串类型的字段，用于存储审计配置
    AuditConfig string `yaml:"audit_config"`
    # 定义一个字符串类型的字段，用于存储类型
    Type string `json:"type"`
    # 定义一个指向tests结构体的指针类型的字段，用于存储测试
    Tests *tests `json:"-"`
    # 定义一个布尔类型的字段，用于表示是否设置
    Set bool `json:"-"`
    # 定义一个字符串类型的字段，用于存储修复信息
    Remediation string `json:"remediation"`
    # 定义一个字符串数组类型的字段，用于存储测试信息
    TestInfo []string `json:"test_info"`
    # 定义一个状态类型的字段，用于存储状态
    State `json:"status"`
    # 定义一个字符串类型的字段，用于存储实际值
    ActualValue string `json:"actual_value"`
    # 定义一个布尔类型的字段，用于表示是否计分
    Scored bool `json:"scored"`
    # 定义一个布尔类型的字段，用于表示是否使用多个值
    IsMultiple bool `yaml:"use_multiple_values"`
    # 定义一个字符串类型的字段，用于存储期望结果
    ExpectedResult string `json:"expected_result"`
    # 定义一个字符串类型的字段，用于存储原因
    Reason string `json:"reason,omitempty"`
    # 定义一个字符串类型的字段，用于存储审计输出
    AuditOutput string `json:"-"`
    # 定义一个字符串类型的字段，用于存储审计环境输出
    AuditEnvOutput string `json:"-"`
    # 定义一个字符串类型的字段，用于存储审计配置输出
    AuditConfigOutput string `json:"-"`
    # 定义一个布尔类型的字段，用于表示是否禁用环境测试
    DisableEnvTesting bool `json:"-"`
// Runner 接口包装了基本的 Run 方法
type Runner interface {
    // Run 运行给定的检查并返回执行状态
    Run(c *Check) State
}

// NewRunner 构造一个默认的 Runner
func NewRunner() Runner {
    return &defaultRunner{}
}

type defaultRunner struct{}

func (r *defaultRunner) Run(c *Check) State {
    return c.run()
}

// Run 执行检查中指定的审核命令并输出结果
func (c *Check) run() State {
    // 如果这是一个有分数的检查，且类型为空且没有测试，则返回警告以提醒用户需要关注这个检查
    if c.Scored && strings.TrimSpace(c.Type) == "" && c.Tests == nil {
        c.Reason = "There are no tests"
        c.State = WARN
        return c.State
    }

    // 如果检查类型为跳过，则强制结果为 INFO
    if c.Type == SKIP {
        c.Reason = "Test marked as skip"
        c.State = INFO
        return c.State
    }

    // 如果检查类型为手动，则强制结果为 WARN
    if c.Type == MANUAL {
        c.Reason = "Test marked as a manual test"
        c.State = WARN
        return c.State
    }

    // 如果没有定义任何测试，则为 FAIL 或 WARN
    if c.Tests == nil || len(c.Tests.TestItems) == 0 {
        c.Reason = "No tests defined"
        if c.Scored {
            c.State = FAIL
        } else {
            c.State = WARN
        }
        return c.State
    }

    // 命令行参数会覆盖配置文件中的设置，所以如果从审核命令得到了良好的结果，那就是我们需要运行的全部内容
    var finalOutput *testOutput
    var lastCommand string

    lastCommand, err := c.runAuditCommands()
    if err == nil {
        finalOutput, err = c.execute()
    }
}
    # 如果最终输出不为空
    if finalOutput != nil:
        # 如果最终输出的测试结果为真
        if finalOutput.testResult:
            # 设置状态为通过
            c.State = PASS
        else:
            # 如果需要评分
            if c.Scored:
                # 设置状态为失败
                c.State = FAIL
            else:
                # 设置状态为警告
                c.State = WARN

        # 设置实际值为最终输出的实际结果
        c.ActualValue = finalOutput.actualResult
        # 设置期望结果为最终输出的期望结果
        c.ExpectedResult = finalOutput.ExpectedResult

    # 如果有错误发生
    if err != nil:
        # 设置原因为错误信息
        c.Reason = err.Error()
        # 如果需要评分
        if c.Scored:
            # 设置状态为失败
            c.State = FAIL
        else:
            # 设置状态为警告
            c.State = WARN

    # 如果最终输出不为空
    if finalOutput != nil:
        # 输出日志信息，包括检查ID、命令、测试结果、状态
        glog.V(3).Infof("Check.ID: %s Command: %q TestResult: %t State: %q \n", c.ID, lastCommand, finalOutput.testResult, c.State)
    else:
        # 输出日志信息，包括检查ID、命令，测试结果为空
        glog.V(3).Infof("Check.ID: %s Command: %q TestResult: <<EMPTY>> \n", c.ID, lastCommand)

    # 如果存在原因
    if c.Reason != "":
        # 输出原因信息
        glog.V(2).Info(c.Reason)
    # 返回状态
    return c.State
}

func (c *Check) runAuditCommands() (lastCommand string, err error) {
    // 始终在需要时运行 auditEnvOutput
    if c.AuditEnv != "" {
        c.AuditEnvOutput, err = runAudit(c.AuditEnv)
        if err != nil {
            return c.AuditEnv, err
        }
    }

    // 运行 audit 命令和 auditConfig 命令（如果存在）
    c.AuditOutput, err = runAudit(c.Audit)
    if err != nil {
        return c.Audit, err
    }

    c.AuditConfigOutput, err = runAudit(c.AuditConfig)
    return c.AuditConfig, err
}

func (c *Check) execute() (finalOutput *testOutput, err error) {
    finalOutput = &testOutput{}

    ts := c.Tests
    res := make([]testOutput, len(ts.TestItems))
    expectedResultArr := make([]string, len(res))

    glog.V(3).Infof("%d tests", len(ts.TestItems))
    for i, t := range ts.TestItems {

        t.isMultipleOutput = c.IsMultiple

        // 首先尝试使用 auditOutput，如果找不到，则尝试使用 auditConfigOutput
        t.auditUsed = AuditCommand
        result := *(t.execute(c.AuditOutput))
        if !result.found {
            t.auditUsed = AuditConfig
            result = *(t.execute(c.AuditConfigOutput))
            if !result.found && t.Env != "" {
                t.auditUsed = AuditEnv
                result = *(t.execute(c.AuditEnvOutput))
            }
        }
        res[i] = result
        expectedResultArr[i] = res[i].ExpectedResult
    }

    var result bool
    // 如果未指定二进制操作，则默认为 AND
    switch ts.BinOp {
    default:
        glog.V(2).Info(fmt.Sprintf("unknown binary operator for tests %s\n", ts.BinOp))
        finalOutput.actualResult = fmt.Sprintf("unknown binary operator for tests %s\n", ts.BinOp)
        return finalOutput, fmt.Errorf("unknown binary operator for tests %s", ts.BinOp)
    // 当操作符为 and 时
    case and, "":
        // 初始化结果为 true
        result = true
        // 遍历结果数组
        for i := range res {
            // 逻辑与操作，更新结果
            result = result && res[i].testResult
        }
        // 生成一个 AND 期望结果
        finalOutput.ExpectedResult = strings.Join(expectedResultArr, " AND ")

    // 当操作符为 or 时
    case or:
        // 初始化结果为 false
        result = false
        // 遍历结果数组
        for i := range res {
            // 逻辑或操作，更新结果
            result = result || res[i].testResult
        }
        // 生成一个 OR 期望结果
        finalOutput.ExpectedResult = strings.Join(expectedResultArr, " OR ")
    }

    // 将最终结果赋给 finalOutput 的 testResult
    finalOutput.testResult = result
    // 将第一个结果的实际结果赋给 finalOutput 的 actualResult
    finalOutput.actualResult = res[0].actualResult

    // 输出日志信息
    glog.V(3).Infof("Returning from execute on tests: finalOutput %#v", finalOutput)
    // 返回 finalOutput 和空错误
    return finalOutput, nil
# 定义一个名为 runAudit 的函数，接受一个字符串参数 audit，并返回两个字符串，一个错误
func runAudit(audit string) (output string, err error) {
    # 创建一个字节缓冲区
    var out bytes.Buffer

    # 去除字符串两端的空白字符
    audit = strings.TrimSpace(audit)
    # 如果去除空白字符后的字符串长度为0，则返回空字符串和错误
    if len(audit) == 0 {
        return output, err
    }

    # 创建一个执行命令的对象，命令为 "/bin/sh"
    cmd := exec.Command("/bin/sh")
    # 将 audit 字符串作为标准输入
    cmd.Stdin = strings.NewReader(audit)
    # 将命令的标准输出和标准错误输出都重定向到 out 缓冲区
    cmd.Stdout = &out
    cmd.Stderr = &out
    # 运行命令，并将错误赋值给 err
    err = cmd.Run()
    # 将 out 缓冲区的内容转换为字符串赋值给 output
    output = out.String()

    # 如果运行命令出现错误
    if err != nil {
        # 格式化错误信息，包括运行的命令、输出和错误信息
        err = fmt.Errorf("failed to run: %q, output: %q, error: %s", audit, output, err)
    } else {
        # 打印日志信息，包括运行的命令和输出
        glog.V(3).Infof("Command %q\n - Output:\n %q", audit, output)

    }
    # 返回输出和错误
    return output, err
}
```