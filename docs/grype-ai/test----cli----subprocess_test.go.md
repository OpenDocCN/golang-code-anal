# `grype\test\cli\subprocess_test.go`

```
package cli  // 声明当前文件所属的包

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"path"  // 导入 path 包，用于处理文件路径
	"strings"  // 导入 strings 包，用于处理字符串
	"testing"  // 导入 testing 包，用于编写测试函数
	"time"  // 导入 time 包，用于处理时间

	"github.com/anchore/stereoscope/pkg/imagetest"  // 导入 imagetest 包
)

func TestSubprocessStdin(t *testing.T) {  // 定义测试函数 TestSubprocessStdin
	binDir := path.Dir(getGrypeSnapshotLocation(t, "linux"))  // 获取 Grype 快照位置的目录路径
	tests := []struct {  // 定义测试用例结构体切片
		name       string  // 测试用例名称
		args       []string  // 测试用例参数
		env        map[string]string  // 测试用例环境变量
		assertions []traitAssertion  // 测试用例断言
	}{
		{
			// 回归测试
			name: "ensure can be used by node subprocess (without hanging)", // 确保可以被 Node 子进程使用（不会挂起）
			args: []string{"-v", fmt.Sprintf("%s:%s:ro", binDir, "/app/bin"), imagetest.LoadFixtureImageIntoDocker(t, "image-node-subprocess"), "node", "/app.js"}, // 参数列表，包括挂载目录、加载镜像和执行命令
			env: map[string]string{
				"GRYPE_CHECK_FOR_APP_UPDATE": "false", // 设置环境变量
			},
			assertions: []traitAssertion{
				assertSucceedingReturnCode, // 断言：确保返回码为成功
			},
		},
		{
			// 回归测试: https://github.com/anchore/grype/issues/267
			name: "ensure can be used by java subprocess (without hanging)", // 确保可以被 Java 子进程使用（不会挂起）
			args: []string{"-v", fmt.Sprintf("%s:%s:ro", binDir, "/app/bin"), imagetest.LoadFixtureImageIntoDocker(t, "image-java-subprocess"), "java", "/app.java"}, // 参数列表，包括挂载目录、加载镜像和执行命令
			env: map[string]string{
				"GRYPE_CHECK_FOR_APP_UPDATE": "false", // 设置环境变量
			},
			assertions: []traitAssertion{
				assertSucceedingReturnCode, // 断言：确保返回码为成功
# 遍历测试用例列表
for _, test := range tests:
    # 定义测试函数
    testFn := func(t *testing.T):
        # 获取 Docker 运行命令
        cmd := getDockerRunCommand(t, test.args...)
        # 运行命令并获取标准输出和标准错误
        stdout, stderr := runCommand(cmd, test.env)
        # 对每个断言函数进行断言
        for _, traitAssertionFn := range test.assertions:
            traitAssertionFn(t, stdout, stderr, cmd.ProcessState.ExitCode())
        # 如果测试失败，记录标准输出、标准错误和命令
        if t.Failed():
            t.Log("STDOUT:\n", stdout)
            t.Log("STDERR:\n", stderr)
            t.Log("COMMAND:", strings.Join(cmd.Args, " "))
    # 对测试函数设置超时时间并执行
    testWithTimeout(t, test.name, 60*time.Second, testFn)
这是一个代码块的结束标志，表示前面的函数或者循环等代码块的结束。
```