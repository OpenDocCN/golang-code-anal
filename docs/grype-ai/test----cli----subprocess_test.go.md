# `grype\test\cli\subprocess_test.go`

```
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "path"  // 导入 path 包，用于处理文件路径
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，用于处理时间

    "github.com/anchore/stereoscope/pkg/imagetest"  // 导入 imagetest 包，用于图像测试
)

func TestSubprocessStdin(t *testing.T) {
    binDir := path.Dir(getGrypeSnapshotLocation(t, "linux"))  // 获取 Grype 快照位置的目录路径
    tests := []struct {
        name       string  // 测试用例名称
        args       []string  // 命令行参数
        env        map[string]string  // 环境变量
        assertions []traitAssertion  // 断言
    }{
        {
            // regression
            name: "ensure can be used by node subprocess (without hanging)",  // 测试用例名称
            args: []string{"-v", fmt.Sprintf("%s:%s:ro", binDir, "/app/bin"), imagetest.LoadFixtureImageIntoDocker(t, "image-node-subprocess"), "node", "/app.js"},  // 命令行参数
            env: map[string]string{  // 环境变量
                "GRYPE_CHECK_FOR_APP_UPDATE": "false",  // 设置 GRYPE_CHECK_FOR_APP_UPDATE 环境变量为 false
            },
            assertions: []traitAssertion{  // 断言
                assertSucceedingReturnCode,  // 断言成功返回码
            },
        },
        {
            // regression: https://github.com/anchore/grype/issues/267
            name: "ensure can be used by java subprocess (without hanging)",  // 测试用例名称
            args: []string{"-v", fmt.Sprintf("%s:%s:ro", binDir, "/app/bin"), imagetest.LoadFixtureImageIntoDocker(t, "image-java-subprocess"), "java", "/app.java"},  // 命令行参数
            env: map[string]string{  // 环境变量
                "GRYPE_CHECK_FOR_APP_UPDATE": "false",  // 设置 GRYPE_CHECK_FOR_APP_UPDATE 环境变量为 false
            },
            assertions: []traitAssertion{  // 断言
                assertSucceedingReturnCode,  // 断言成功返回码
            },
        },
    }
    # 遍历测试用例切片，每个测试用例包含参数、环境变量和断言函数
    for _, test := range tests {
        # 定义测试函数，参数为 t *testing.T
        testFn := func(t *testing.T) {
            # 获取 Docker 运行命令
            cmd := getDockerRunCommand(t, test.args...)
            # 运行命令并获取标准输出和标准错误
            stdout, stderr := runCommand(cmd, test.env)
            # 遍历断言函数切片，对标准输出、标准错误和进程退出码进行断言
            for _, traitAssertionFn := range test.assertions {
                traitAssertionFn(t, stdout, stderr, cmd.ProcessState.ExitCode())
            }
            # 如果测试失败，记录标准输出、标准错误和命令
            if t.Failed() {
                t.Log("STDOUT:\n", stdout)
                t.Log("STDERR:\n", stderr)
                t.Log("COMMAND:", strings.Join(cmd.Args, " "))
            }
        }

        # 使用超时时间进行测试
        testWithTimeout(t, test.name, 60*time.Second, testFn)
    }
# 闭合前面的函数定义
```