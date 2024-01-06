# `kubebench-aquasecurity\check\check_test.go`

```
// 版权声明，版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0进行许可
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以获取特定语言的权限和限制

// 导入所需的包
package check

import (
	"strings" // 导入字符串操作的包
	"testing" // 导入测试框架的包
)
# 定义测试函数 TestCheck_Run，用于测试 Check 结构体的 Run 方法
func TestCheck_Run(t *testing.T) {
    # 定义测试用例结构体
    type TestCase struct {
        name     string
        check    Check
        Expected State
    }

    # 定义测试用例数组
    testCases := []TestCase{
        {name: "Manual check should WARN", check: Check{Type: MANUAL}, Expected: WARN},
        {name: "Skip check should INFO", check: Check{Type: "skip"}, Expected: INFO},
        {name: "Unscored check (with no type) should WARN on failure", check: Check{Scored: false}, Expected: WARN},
        {
            name: "Unscored check that pass should PASS",
            check: Check{
                Scored: false,
                Audit:  "echo hello",
                Tests: &tests{TestItems: []*testItem{{
                    Flag: "hello",
                    Set:  true,
                    # ...
# 定义测试用例，包括名称、检查对象和期望结果
{name: "Check with no tests should WARN", check: Check{Scored: true}, Expected: WARN},
# 定义一个得分的检查，但是没有测试，期望结果为警告
{name: "Scored check with empty tests should FAIL", check: Check{Scored: true, Tests: &tests{}}, Expected: FAIL},
# 定义一个得分的检查，但是测试为空，期望结果为失败
{
    name: "Scored check that doesn't pass should FAIL",
    check: Check{
        Scored: true,
        Audit:  "echo hello",
        Tests: &tests{TestItems: []*testItem{{
            Flag: "hello",
            Set:  false,
        }}},
    },
    Expected: FAIL,
},
# 定义一个得分的检查，包括命令行命令和测试，但测试未通过，期望结果为失败
# 定义一个测试用例的结构体，包括名称、检查项、期望结果
name: "Scored checks that pass should PASS",
check: Check{
    Scored: true,  # 设置为有分数的检查
    Audit:  "echo hello",  # 执行的审计命令
    Tests: &tests{TestItems: []*testItem{{
        Flag: "hello",  # 测试标志
        Set:  true,  # 设置为真
    }}},
},
Expected: PASS,  # 期望结果为通过

# 遍历测试用例列表
for _, testCase := range testCases {
    t.Run(testCase.name, func(t *testing.T) {
        testCase.check.run()  # 运行测试用例的检查项
        if testCase.check.State != testCase.Expected:  # 检查实际状态是否符合期望
            t.Errorf("expected %s, actual %s", testCase.Expected, testCase.check.State)  # 输出错误信息
    }
}
	}
}

func TestCheckAuditEnv(t *testing.T){
	// 创建通过测试用例的检查项数组
	passingCases := []*Check{
		controls.Groups[2].Checks[0],
		controls.Groups[2].Checks[2],
		controls.Groups[2].Checks[3],
		controls.Groups[2].Checks[4],
	}
	// 创建失败测试用例的检查项数组
	failingCases := []*Check{
		controls.Groups[2].Checks[1],
		controls.Groups[2].Checks[5],
		controls.Groups[2].Checks[6],
	}

	// 遍历通过测试用例的检查项数组
	for _, c := range passingCases {
		// 对每个检查项运行测试
		t.Run(c.Text, func(t *testing.T) {
			c.run()
# 对通过测试用例进行遍历
for _, c := range passingCases:
    # 在测试中运行当前用例
    c.run()
    # 如果当前用例的状态不是"PASS"，则输出错误信息
    if c.State != "PASS":
        t.Errorf("Should PASS, got: %v", c.State)

# 对失败测试用例进行遍历
for _, c := range failingCases:
    # 在测试中运行当前用例
    c.run()
    # 如果当前用例的状态不是"FAIL"，则输出错误信息
    if c.State != "FAIL":
        t.Errorf("Should FAIL, got: %v", c.State)
}

# 定义测试函数TestCheckAuditConfig
func TestCheckAuditConfig(t *testing.T):
    # 创建通过测试用例列表passingCases
    passingCases := []*Check{
        controls.Groups[1].Checks[0],
```
以上是对给定代码的注释。
# 定义通过的测试用例列表
passingCases := []*Check{
    controls.Groups[1].Checks[3],  # 获取第一个分组的第四个检查项
    controls.Groups[1].Checks[5],  # 获取第一个分组的第六个检查项
    controls.Groups[1].Checks[7],  # 获取第一个分组的第八个检查项
    controls.Groups[1].Checks[9],  # 获取第一个分组的第十个检查项
    controls.Groups[1].Checks[15],  # 获取第一个分组的第十六个检查项
}

# 定义未通过的测试用例列表
failingCases := []*Check{
    controls.Groups[1].Checks[1],  # 获取第一个分组的第二个检查项
    controls.Groups[1].Checks[2],  # 获取第一个分组的第三个检查项
    controls.Groups[1].Checks[4],  # 获取第一个分组的第五个检查项
    controls.Groups[1].Checks[6],  # 获取第一个分组的第七个检查项
    controls.Groups[1].Checks[8],  # 获取第一个分组的第九个检查项
    controls.Groups[1].Checks[10],  # 获取第一个分组的第十一个检查项
    controls.Groups[1].Checks[11],  # 获取第一个分组的第十二个检查项
    controls.Groups[1].Checks[12],  # 获取第一个分组的第十三个检查项
    controls.Groups[1].Checks[13],  # 获取第一个分组的第十四个检查项
    controls.Groups[1].Checks[14],  # 获取第一个分组的第十五个检查项
    controls.Groups[1].Checks[16],  # 获取第一个分组的第十七个检查项
}
# 遍历通过测试用例列表
for _, c := range passingCases:
    # 对每个测试用例运行测试，并检查状态是否为"PASS"
    t.Run(c.Text, func(t *testing.T):
        c.run()
        if c.State != "PASS":
            t.Errorf("Should PASS, got: %v", c.State)

# 遍历失败测试用例列表
for _, c := range failingCases:
    # 对每个测试用例运行测试，并检查状态是否为"FAIL"
    t.Run(c.Text, func(t *testing.T):
        c.run()
        if c.State != "FAIL":
            t.Errorf("Should FAIL, got: %v", c.State)
# 定义一个测试函数，用于测试 runAudit 函数
func Test_runAudit(t *testing.T) {
    # 定义测试参数结构体
    type args struct {
        audit  string  # 审计参数
        output string  # 输出参数
    }
    # 定义测试用例
    tests := []struct {
        name   string  # 测试用例名称
        args   args    # 测试参数
        errMsg string  # 错误信息
        output string  # 期望输出
    }{
        {
            name: "run success",  # 测试用例名称
            args: args{  # 测试参数
                audit: "echo 'hello world'",  # 审计参数
            },
            errMsg: "",  # 错误信息
            output: "hello world\n",  # 期望输出
        },
        {
# 定义测试用例名称为“运行多行脚本”
name: "run multiple lines script",
# 定义测试参数
args: {
    # 定义要运行的脚本内容
    audit: `
    hello() {
      echo "hello world"
    }

    hello
    `,
},
# 期望的错误消息
errMsg: "",
# 期望的输出结果
output: "hello world\n",
},
# 定义另一个测试用例名称为“运行失败”
name: "run failed",
# 定义测试参数
args: {
    # 定义要运行的脚本内容
    audit: "unknown_command",
},
# 期望的错误消息
errMsg: "failed to run: \"unknown_command\", output: \"/bin/sh: ",
# 期望的输出结果
output: "not found\n",
# 遍历测试用例列表
for _, tt := range tests {
    # 使用测试名称创建子测试
    t.Run(tt.name, func(t *testing.T) {
        # 初始化错误消息字符串
        var errMsg string
        # 运行审计函数，获取输出和错误信息
        output, err := runAudit(tt.args.audit)
        if err != nil {
            errMsg = err.Error()
        }
        # 如果有错误消息并且错误消息不包含预期的错误消息
        if errMsg != "" && !strings.Contains(errMsg, tt.errMsg) {
            # 输出错误消息
            t.Errorf("name %s errMsg = %q, want %q", tt.name, errMsg, tt.errMsg)
        }
        # 如果没有错误消息并且输出不等于预期输出
        if errMsg == "" && output != tt.output {
            # 输出错误消息
            t.Errorf("name %s output = %q, want %q", tt.name, output, tt.output)
        }
        # 如果有错误消息并且输出不包含预期输出
        if errMsg != "" && !strings.Contains(output, tt.output) {
            # 输出错误消息
            t.Errorf("name %s output = %q, want %q", tt.name, output, tt.output)
        }
    })
}
这是一个代码块的结束标记，表示前面的函数或者循环的结束。
```