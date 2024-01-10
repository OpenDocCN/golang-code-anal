# `kubo\test\cli\completion_test.go`

```
package cli

import (
    "fmt"
    "testing"

    "github.com/ipfs/kubo/test/cli/harness"
    . "github.com/ipfs/kubo/test/cli/testutils"
    "github.com/stretchr/testify/assert"
)

func TestBashCompletion(t *testing.T) {
    t.Parallel()  // 并行执行测试用例
    h := harness.NewT(t)  // 创建测试用例的辅助对象
    node := h.NewNode()  // 创建一个新的节点

    res := node.IPFS("commands", "completion", "bash")  // 调用节点的 IPFS 方法获取 Bash 自动补全信息

    length := len(res.Stdout.String())  // 获取输出的字符串长度
    if length < 100 {  // 如果输出长度小于 100
        t.Fatalf("expected a long Bash completion file, but got one of length %d", length)  // 抛出测试失败的错误信息
    }

    t.Run("completion file can be loaded in bash", func(t *testing.T) {  // 运行子测试
        RequiresLinux(t)  // 要求运行环境为 Linux

        completionFile := h.WriteToTemp(res.Stdout.String())  // 将输出的字符串写入临时文件
        res = h.Sh(fmt.Sprintf("source %s && type -t _ipfs", completionFile))  // 执行 shell 命令，加载自动补全文件并检查是否存在 _ipfs 命令
        assert.NoError(t, res.Err)  // 断言没有错误发生
    })
}

func TestZshCompletion(t *testing.T) {
    t.Parallel()  // 并行执行测试用例
    h := harness.NewT(t)  // 创建测试用例的辅助对象
    node := h.NewNode()  // 创建一个新的节点

    res := node.IPFS("commands", "completion", "zsh")  // 调用节点的 IPFS 方法获取 Zsh 自动补全信息

    length := len(res.Stdout.String())  // 获取输出的字符串长度
    if length < 100 {  // 如果输出长度小于 100
        t.Fatalf("expected a long Bash completion file, but got one of length %d", length)  // 抛出测试失败的错误信息
    }

    t.Run("completion file can be loaded in bash", func(t *testing.T) {  // 运行子测试
        RequiresLinux(t)  // 要求运行环境为 Linux

        completionFile := h.WriteToTemp(res.Stdout.String())  // 将输出的字符串写入临时文件
        res = h.Runner.Run(harness.RunRequest{  // 运行命令
            Path: "zsh",
            Args: []string{"-c", fmt.Sprintf("autoload -Uz compinit && compinit && source %s && echo -E $_comps[ipfs]", completionFile)},
        })

        assert.NoError(t, res.Err)  // 断言没有错误发生
        assert.NotEmpty(t, res.Stdout.String())  // 断言输出的字符串不为空
    })
}
```