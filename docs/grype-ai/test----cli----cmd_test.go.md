# `grype\test\cli\cmd_test.go`

```
package cli

import (
    "encoding/json"  // 导入处理 JSON 数据的包
    "path/filepath"   // 导入处理文件路径的包
    "strings"        // 导入处理字符串的包
    "testing"        // 导入测试框架包

    "github.com/stretchr/testify/require"  // 导入断言库
)

func TestCmd(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体
        name       string
        args       []string
        env        map[string]string
        assertions []traitAssertion
    }  // 结构体定义结束

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            cmd, stdout, stderr := runGrype(t, test.env, test.args...)  // 运行命令并获取标准输出和标准错误
            for _, traitFn := range test.assertions {  // 遍历断言函数
                traitFn(t, stdout, stderr, cmd.ProcessState.ExitCode())  // 执行断言函数
            }
            if t.Failed() {  // 如果测试失败
                t.Log("STDOUT:\n", stdout)  // 记录标准输出
                t.Log("STDERR:\n", stderr)  // 记录标准错误
                t.Log("COMMAND:", strings.Join(cmd.Args, " "))  // 记录执行的命令
            }
        })  // 测试用例运行结束
    }  // 遍历测试用例结束
}  // 测试函数结束

func Test_descriptorNameAndVersionSet(t *testing.T) {
    _, output, _ := runGrype(t, nil, "-o", "json", getFixtureImage(t, "image-bare"))  // 运行命令并获取输出

    parsed := map[string]any{}  // 定义空的 map
    err := json.Unmarshal([]byte(output), &parsed)  // 解析 JSON 输出
    require.NoError(t, err)  // 断言解析过程无错误

    desc, _ := parsed["descriptor"].(map[string]any)  // 获取解析后的描述信息
    require.NotNil(t, desc)  // 断言描述信息不为空

    name := desc["name"]  // 获取描述信息中的名称
    require.Equal(t, "grype", name)  // 断言名称为 "grype"

    version := desc["version"]  // 获取描述信息中的版本
    require.NotEmpty(t, version)  // 断言版本不为空
}
```