# `kubo\test\cli\daemon_test.go`

```
package cli

import (
    "os/exec"  // 导入执行外部命令的包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试辅助包
)

func TestDaemon(t *testing.T) {
    t.Parallel()  // 并行执行测试

    t.Run("daemon starts if api is set to null", func(t *testing.T) {
        t.Parallel()  // 并行执行子测试
        node := harness.NewT(t).NewNode().Init()  // 创建新的测试节点并初始化
        node.SetIPFSConfig("Addresses.API", nil)  // 设置IPFS配置中的API地址为null
        node.Runner.MustRun(harness.RunRequest{
            Path:    node.IPFSBin,  // 设置运行路径为IPFS二进制文件
            Args:    []string{"daemon"},  // 设置运行参数为"daemon"
            RunFunc: (*exec.Cmd).Start,  // 使用Start方法启动命令，不等待完成
        })

        node.StopDaemon()  // 停止守护进程
    })
}
```