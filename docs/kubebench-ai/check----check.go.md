# `kubebench-aquasecurity\check\check.go`

```
// 版权声明，声明代码版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0版本授权，除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

// 导入所需的包
package check

import (
    "bytes"  // 导入bytes包，用于操作字节切片
    "fmt"    // 导入fmt包，用于格式化输入输出
    "os/exec"  // 导入os/exec包，用于执行外部命令
// 导入字符串操作包
"strings"

// 导入日志记录包
"github.com/golang/glog"
)

// NodeType 表示节点的类型（主节点、从节点）
type NodeType string

// State 表示控制检查的状态
type State string

const (
	// PASS 表示检查通过
	PASS State = "PASS"
	// FAIL 表示检查失败
	FAIL State = "FAIL"
	// WARN 表示无法执行检查
	WARN State = "WARN"
	// INFO 表示信息性消息
	INFO State = "INFO"
// 定义常量 SKIP 用于跳过检查
SKIP = "skip"

// 定义枚举类型 NodeType，包括 MASTER（主节点）、NODE（普通节点）、FEDERATED（联邦部署）
MASTER NodeType = "master"
NODE NodeType = "node"
FEDERATED NodeType = "federated"

// 定义枚举类型 NodeType，包括 ETCD（etcd 节点）、CONTROLPLANE（控制平面节点）、POLICIES（运行策略的节点）、MANAGEDSERVICES（运行托管服务的节点）
ETCD NodeType = "etcd"
CONTROLPLANE NodeType = "controlplane"
POLICIES NodeType = "policies"
MANAGEDSERVICES = "managedservices"
// MANUAL 是一个常量，表示手动检查类型
MANUAL string = "manual"
)

// Check 包含了 CIS Kubernetes 文档中一个推荐的信息
type Check struct {
    ID                string   `yaml:"id" json:"test_number"`  // ID 表示测试编号
    Text              string   `json:"test_desc"`  // Text 表示测试描述
    Audit             string   `json:"audit"`  // Audit 表示审计信息
    AuditEnv          string   `yaml:"audit_env"`  // AuditEnv 表示审计环境
    AuditConfig       string   `yaml:"audit_config"`  // AuditConfig 表示审计配置
    Type              string   `json:"type"`  // Type 表示类型
    Tests             *tests   `json:"-"`  // Tests 表示测试
    Set               bool     `json:"-"`  // Set 表示设置
    Remediation       string   `json:"remediation"`  // Remediation 表示修复措施
    TestInfo          []string `json:"test_info"`  // TestInfo 表示测试信息
    State             `json:"status"`  // State 表示状态
    ActualValue       string `json:"actual_value"`  // ActualValue 表示实际值
    Scored            bool   `json:"scored"`  // Scored 表示是否计分
```

// 定义一个布尔类型的字段 IsMultiple，用于指示是否使用多个值
IsMultiple        bool   `yaml:"use_multiple_values"`

// 定义一个字符串类型的字段 ExpectedResult，用于存储预期结果
ExpectedResult    string `json:"expected_result"`

// 定义一个字符串类型的字段 Reason，用于存储原因，可选
Reason            string `json:"reason,omitempty"`

// 定义一个字符串类型的字段 AuditOutput，用于存储审计输出，不会被序列化为 JSON
AuditOutput       string `json:"-"`

// 定义一个字符串类型的字段 AuditEnvOutput，用于存储审计环境输出，不会被序列化为 JSON
AuditEnvOutput    string `json:"-"`

// 定义一个字符串类型的字段 AuditConfigOutput，用于存储审计配置输出，不会被序列化为 JSON
AuditConfigOutput string `json:"-"`

// 定义一个布尔类型的字段 DisableEnvTesting，用于指示是否禁用环境测试
DisableEnvTesting bool `json:"-"`

// 定义一个接口类型 Runner，包含一个 Run 方法
type Runner interface {
	// Run 方法接收一个 Check 对象，运行该检查并返回执行状态
	Run(c *Check) State
}

// NewRunner 函数用于构造一个默认的 Runner 对象
func NewRunner() Runner {
	// 返回一个 defaultRunner 对象
	return &defaultRunner{}
}
// 定义默认的运行器结构体
type defaultRunner struct{}

// 实现运行器接口的运行方法，执行检查并返回状态
func (r *defaultRunner) Run(c *Check) State {
	return c.run()
}

// run方法执行检查中指定的审计命令并输出结果
func (c *Check) run() State {
	// 如果这是一个有分数的检查，并且类型为空并且没有测试，则返回警告状态，提醒用户需要关注这个检查
	if c.Scored && strings.TrimSpace(c.Type) == "" && c.Tests == nil {
		c.Reason = "There are no tests"
		c.State = WARN
		return c.State
	}

	// 如果检查类型为跳过，则强制结果为信息状态
	if c.Type == SKIP {
		// 将测试标记为跳过
		c.Reason = "Test marked as skip"
		// 设置状态为信息
		c.State = INFO
		// 返回状态
		return c.State
	}

	// 如果检查类型为手动，则强制结果为警告
	if c.Type == MANUAL {
		// 将测试标记为手动测试
		c.Reason = "Test marked as a manual test"
		// 设置状态为警告
		c.State = WARN
		// 返回状态
		return c.State
	}

	// 如果没有定义任何测试，则为失败或警告
	if c.Tests == nil || len(c.Tests.TestItems) == 0 {
		// 没有定义测试的原因
		c.Reason = "No tests defined"
		// 如果有分数，则状态为失败
		if c.Scored {
			c.State = FAIL
		} else {
			// 否则状态为警告
			c.State = WARN
		}
		// 返回当前状态
		return c.State
	}

	// 命令行参数会覆盖配置文件中的设置，因此如果我们从 Audit 命令得到了一个好的结果，那么我们只需要运行这个命令
	var finalOutput *testOutput
	var lastCommand string

	// 运行 Audit 命令，并获取最后一个命令的结果
	lastCommand, err := c.runAuditCommands()
	if err == nil {
		// 如果没有错误，执行最后一个命令
		finalOutput, err = c.execute()
	}

	// 如果最终输出不为空
	if finalOutput != nil {
		// 如果测试结果为真
		if finalOutput.testResult {
			// 设置状态为通过
			c.State = PASS
		} else {
			// 如果测试结果为假，并且是有分数的
			if c.Scored {
				// 设置状态为失败
				c.State = FAIL
			} else {
				// 设置状态为警告
				c.State = WARN
		}
	}

	c.ActualValue = finalOutput.actualResult
	c.ExpectedResult = finalOutput.ExpectedResult
	// 设置实际值和期望值

	if err != nil {
		c.Reason = err.Error()
		// 如果有错误，设置原因为错误信息
		if c.Scored {
			c.State = FAIL
			// 如果已经评分，设置状态为失败
		} else {
			c.State = WARN
			// 如果未评分，设置状态为警告
		}
	}

	if finalOutput != nil {
		glog.V(3).Infof("Check.ID: %s Command: %q TestResult: %t State: %q \n", c.ID, lastCommand, finalOutput.testResult, c.State)
		// 如果最终输出不为空，记录日志信息
	} else {
		glog.V(3).Infof("Check.ID: %s Command: %q TestResult: <<EMPTY>> \n", c.ID, lastCommand)
		// 如果最终输出为空，记录空输出的日志信息
	}
// 如果检查结果包含原因，记录原因信息
if c.Reason != "" {
    glog.V(2).Info(c.Reason)
}
// 返回检查状态
return c.State
}

// 运行审计命令
func (c *Check) runAuditCommands() (lastCommand string, err error) {
    // 如果需要，始终运行审计环境输出
    if c.AuditEnv != "" {
        c.AuditEnvOutput, err = runAudit(c.AuditEnv)
        if err != nil {
            return c.AuditEnv, err
        }
    }

    // 运行审计命令和审计配置命令（如果存在）
    c.AuditOutput, err = runAudit(c.Audit)
    if err != nil {
		return c.Audit, err
	# 返回 c.Audit 和 err 变量的值
	}

	c.AuditConfigOutput, err = runAudit(c.AuditConfig)
	# 运行 runAudit 函数，将结果赋值给 c.AuditConfigOutput 和 err 变量
	return c.AuditConfig, err
	# 返回 c.AuditConfig 和 err 变量的值
}

func (c *Check) execute() (finalOutput *testOutput, err error) {
	# 定义 execute 方法，返回 finalOutput 和 err 变量
	finalOutput = &testOutput{}

	ts := c.Tests
	# 将 c.Tests 赋值给 ts 变量
	res := make([]testOutput, len(ts.TestItems))
	# 创建一个长度为 ts.TestItems 的 testOutput 类型的切片
	expectedResultArr := make([]string, len(res))
	# 创建一个长度为 res 的字符串切片

	glog.V(3).Infof("%d tests", len(ts.TestItems))
	# 打印日志，显示测试数量

	for i, t := range ts.TestItems {
		# 遍历测试项目
		t.isMultipleOutput = c.IsMultiple
		# 设置 t.isMultipleOutput 为 c.IsMultiple 的值

		// Try with the auditOutput first, and if that's not found, try the auditConfigOutput
		# 首先尝试使用 auditOutput，如果找不到，则尝试使用 auditConfigOutput
		// 设置 t.auditUsed 为 AuditCommand
		t.auditUsed = AuditCommand
		// 执行 AuditOutput 命令，并将结果赋值给 result
		result := *(t.execute(c.AuditOutput))
		// 如果结果未找到
		if !result.found {
			// 设置 t.auditUsed 为 AuditConfig
			t.auditUsed = AuditConfig
			// 执行 AuditConfigOutput 命令，并将结果赋值给 result
			result = *(t.execute(c.AuditConfigOutput))
			// 如果结果仍未找到并且 t.Env 不为空
			if !result.found && t.Env != "" {
				// 设置 t.auditUsed 为 AuditEnv
				t.auditUsed = AuditEnv
				// 执行 AuditEnvOutput 命令，并将结果赋值给 result
				result = *(t.execute(c.AuditEnvOutput))
			}
		}
		// 将结果存入 res 数组
		res[i] = result
		// 将结果的期望值存入 expectedResultArr 数组
		expectedResultArr[i] = res[i].ExpectedResult
	}

	var result bool
	// 如果未指定二进制操作，则默认为 AND
	switch ts.BinOp {
	default:
		// 输出日志信息
		glog.V(2).Info(fmt.Sprintf("unknown binary operator for tests %s\n", ts.BinOp))
		// 设置 finalOutput.actualResult 为未知二进制操作
		finalOutput.actualResult = fmt.Sprintf("unknown binary operator for tests %s\n", ts.BinOp)
		// 返回最终输出和错误信息，错误信息包含未知的二元操作符
		return finalOutput, fmt.Errorf("unknown binary operator for tests %s", ts.BinOp)
	// 如果操作符是 and 或者为空
	case and, "":
		// 设置结果为 true
		result = true
		// 遍历结果数组
		for i := range res {
			// 对结果进行逻辑与操作
			result = result && res[i].testResult
		}
		// 生成一个逻辑与的预期结果
		finalOutput.ExpectedResult = strings.Join(expectedResultArr, " AND ")

	// 如果操作符是 or
	case or:
		// 设置结果为 false
		result = false
		// 遍历结果数组
		for i := range res {
			// 对结果进行逻辑或操作
			result = result || res[i].testResult
		}
		// 生成一个逻辑或的预期结果
		finalOutput.ExpectedResult = strings.Join(expectedResultArr, " OR ")
	}

	// 设置最终输出的测试结果为计算得到的结果
	finalOutput.testResult = result
	// 设置最终输出的实际结果为结果数组的第一个元素的实际结果
	finalOutput.actualResult = res[0].actualResult
# 使用 glog 打印信息级别为 3 的日志，记录执行测试后的最终输出
glog.V(3).Infof("Returning from execute on tests: finalOutput %#v", finalOutput)
# 返回最终输出和空错误
return finalOutput, nil
}

# 运行审核程序
func runAudit(audit string) (output string, err error) {
    # 创建一个字节缓冲区
    var out bytes.Buffer

    # 去除字符串两端的空白字符
    audit = strings.TrimSpace(audit)
    # 如果审核字符串长度为 0，则返回空输出和错误
    if len(audit) == 0 {
        return output, err
    }

    # 创建一个执行命令对象，执行 /bin/sh 命令
    cmd := exec.Command("/bin/sh")
    # 将审核字符串作为标准输入
    cmd.Stdin = strings.NewReader(audit)
    # 将命令的标准输出重定向到 out 缓冲区
    cmd.Stdout = &out
    # 将命令的标准错误重定向到 out 缓冲区
    cmd.Stderr = &out
    # 运行命令，并将错误赋值给 err，输出赋值给 output
    err = cmd.Run()
    output = out.String()
# 如果发生错误，创建一个新的错误信息，包括运行失败的命令、输出和错误信息
if err != nil:
    err = fmt.Errorf("failed to run: %q, output: %q, error: %s", audit, output, err)
# 如果没有发生错误，记录命令和输出的详细信息
else:
    glog.V(3).Infof("Command %q\n - Output:\n %q", audit, output)
# 返回输出和错误信息
return output, err
```