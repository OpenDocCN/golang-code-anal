# `kubebench-aquasecurity\check\check_test.go`

```go
// 版权声明和许可证信息
// 2017-2020年版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0版授权
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请参阅许可证以获取特定语言的权限和限制

// 导入所需的包
package check

import (
    "strings"
    "testing"
)

// 定义测试函数TestCheck_Run
func TestCheck_Run(t *testing.T) {
    // 定义TestCase结构体，包含name、check和Expected字段
    type TestCase struct {
        name     string
        check    Check
        Expected State
    }
    # 定义测试用例的切片
    testCases := []TestCase{
        # 第一个测试用例：手动检查应该警告
        {name: "Manual check should WARN", check: Check{Type: MANUAL}, Expected: WARN},
        # 第二个测试用例：跳过检查应该是信息
        {name: "Skip check should INFO", check: Check{Type: "skip"}, Expected: INFO},
        # 第三个测试用例：未评分的检查（没有类型）在失败时应该警告
        {name: "Unscored check (with no type) should WARN on failure", check: Check{Scored: false}, Expected: WARN},
        # 第四个测试用例：未评分的检查通过应该通过
        {
            name: "Unscored check that pass should PASS",
            check: Check{
                Scored: false,
                Audit:  "echo hello",
                Tests: &tests{TestItems: []*testItem{{
                    Flag: "hello",
                    Set:  true,
                }}},
            },
            Expected: PASS,
        },
        # 第五个测试用例：没有测试的检查应该警告
        {name: "Check with no tests should WARN", check: Check{Scored: true}, Expected: WARN},
        # 第六个测试用例：具有空测试的评分检查应该失败
        {name: "Scored check with empty tests should FAIL", check: Check{Scored: true, Tests: &tests{}}, Expected: FAIL},
        # 第七个测试用例：不通过的评分检查应该失败
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
        # 第八个测试用例：通过的评分检查应该通过
        {
            name: "Scored checks that pass should PASS",
            check: Check{
                Scored: true,
                Audit:  "echo hello",
                Tests: &tests{TestItems: []*testItem{{
                    Flag: "hello",
                    Set:  true,
                }}},
            },
            Expected: PASS,
        },
    }

    # 遍历测试用例切片
    for _, testCase := range testCases {
        # 使用测试用例的名称创建子测试
        t.Run(testCase.name, func(t *testing.T) {
            # 运行测试用例的检查
            testCase.check.run()
            # 检查实际状态是否与预期状态相符，如果不符则输出错误信息
            if testCase.check.State != testCase.Expected {
                t.Errorf("expected %s, actual %s", testCase.Expected, testCase.check.State)
            }
        })
    }
func TestCheckAuditEnv(t *testing.T){
    passingCases := []*Check{
        controls.Groups[2].Checks[0],  # 从控制组中获取通过的检查项
        controls.Groups[2].Checks[2],
        controls.Groups[2].Checks[3],
        controls.Groups[2].Checks[4],
    }

    failingCases := []*Check{
        controls.Groups[2].Checks[1],  # 从控制组中获取失败的检查项
        controls.Groups[2].Checks[5],
        controls.Groups[2].Checks[6],
    }

    for _, c := range passingCases {  # 遍历通过的检查项
        t.Run(c.Text, func(t *testing.T) {  # 运行测试用例
            c.run()  # 执行检查项
            if c.State != "PASS" {  # 检查状态是否为通过
                t.Errorf("Should PASS, got: %v", c.State)  # 输出错误信息
            }
        })
    }

    for _, c := range failingCases {  # 遍历失败的检查项
        t.Run(c.Text, func(t *testing.T) {  # 运行测试用例
            c.run()  # 执行检查项
            if c.State != "FAIL" {  # 检查状态是否为失败
                t.Errorf("Should FAIL, got: %v", c.State)  # 输出错误信息
            }
        })
    }
}

func TestCheckAuditConfig(t *testing.T) {

    passingCases := []*Check{
        controls.Groups[1].Checks[0],  # 从控制组中获取通过的检查项
        controls.Groups[1].Checks[3],
        controls.Groups[1].Checks[5],
        controls.Groups[1].Checks[7],
        controls.Groups[1].Checks[9],
        controls.Groups[1].Checks[15],
    }

    failingCases := []*Check{
        controls.Groups[1].Checks[1],  # 从控制组中获取失败的检查项
        controls.Groups[1].Checks[2],
        controls.Groups[1].Checks[4],
        controls.Groups[1].Checks[6],
        controls.Groups[1].Checks[8],
        controls.Groups[1].Checks[10],
        controls.Groups[1].Checks[11],
        controls.Groups[1].Checks[12],
        controls.Groups[1].Checks[13],
        controls.Groups[1].Checks[14],
        controls.Groups[1].Checks[16],
    }

    for _, c := range passingCases {  # 遍历通过的检查项
        t.Run(c.Text, func(t *testing.T) {  # 运行测试用例
            c.run()  # 执行检查项
            if c.State != "PASS" {  # 检查状态是否为通过
                t.Errorf("Should PASS, got: %v", c.State)  # 输出错误信息
            }
        })
    }
    # 遍历失败案例列表，使用下划线 _ 忽略索引值，c 为当前遍历的失败案例
    for _, c := range failingCases:
        # 使用测试框架的 Run 方法执行当前失败案例的测试
        t.Run(c.Text, func(t *testing.T) {
            # 运行当前失败案例的测试
            c.run()
            # 如果当前失败案例的状态不是 "FAIL"，则输出错误信息
            if c.State != "FAIL" {
                t.Errorf("Should FAIL, got: %v", c.State)
            }
        })
    }
func Test_runAudit(t *testing.T) {
    // 定义测试函数的参数结构
    type args struct {
        audit  string
        output string
    }
    // 定义测试用例
    tests := []struct {
        name   string
        args   args
        errMsg string
        output string
    }{
        {
            name: "run success",
            args: args{
                audit: "echo 'hello world'",
            },
            errMsg: "",
            output: "hello world\n",
        },
        {
            name: "run multiple lines script",
            args: args{
                audit: `
hello() {
  echo "hello world"
}

hello
`,
            },
            errMsg: "",
            output: "hello world\n",
        },
        {
            name: "run failed",
            args: args{
                audit: "unknown_command",
            },
            errMsg: "failed to run: \"unknown_command\", output: \"/bin/sh: ",
            output: "not found\n",
        },
    }
    // 遍历测试用例
    for _, tt := range tests {
        // 运行单个测试用例
        t.Run(tt.name, func(t *testing.T) {
            var errMsg string
            // 调用 runAudit 函数执行测试用例中的命令
            output, err := runAudit(tt.args.audit)
            // 如果有错误，将错误信息赋值给 errMsg
            if err != nil {
                errMsg = err.Error()
            }
            // 如果 errMsg 不为空且不包含预期的错误信息，输出错误信息
            if errMsg != "" && !strings.Contains(errMsg, tt.errMsg) {
                t.Errorf("name %s errMsg = %q, want %q", tt.name, errMsg, tt.errMsg)
            }
            // 如果 errMsg 为空且输出不等于预期输出，输出错误信息
            if errMsg == "" && output != tt.output {
                t.Errorf("name %s output = %q, want %q", tt.name, output, tt.output)
            }
            // 如果 errMsg 不为空且输出不包含预期输出，输出错误信息
            if errMsg != "" && !strings.Contains(output, tt.output) {
                t.Errorf("name %s output = %q, want %q", tt.name, output, tt.output)
            }
        })
    }
}
```