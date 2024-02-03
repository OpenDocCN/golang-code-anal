# `grype\test\cli\registry_auth_test.go`

```go
package cli

import (
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数
)

func TestRegistryAuth(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        name       string  // 测试用例名称
        args       []string  // 测试用例参数
        env        map[string]string  // 测试用例环境变量
        assertions []traitAssertion  // 测试用例断言
    }  // 结构体切片结束

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            cmd, stdout, stderr := runGrype(t, test.env, test.args...)  // 运行测试命令，获取标准输出和标准错误
            for _, traitAssertionFn := range test.assertions {  // 遍历测试用例的断言
                traitAssertionFn(t, stdout, stderr, cmd.ProcessState.ExitCode())  // 执行断言函数
            }
            if t.Failed() {  // 如果测试失败
                t.Log("STDOUT:\n", stdout)  // 记录标准输出
                t.Log("STDERR:\n", stderr)  // 记录标准错误
                t.Log("COMMAND:", strings.Join(cmd.Args, " "))  // 记录执行的命令
            }
        })  // 运行测试用例结束
    }  // 遍历测试用例结束
}
```