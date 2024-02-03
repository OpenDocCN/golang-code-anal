# `v2ray-core\common\platform\ctlcmd\ctlcmd.go`

```go
package ctlcmd

import (
    "io"
    "os"
    "os/exec"
    "strings"

    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/platform"
)

//go:generate go run v2ray.com/core/common/errors/errorgen

// Run runs the command with the given arguments and input, and returns the output buffer and any error encountered
func Run(args []string, input io.Reader) (buf.MultiBuffer, error) {
    // 获取 v2ctl 工具的位置
    v2ctl := platform.GetToolLocation("v2ctl")
    // 检查 v2ctl 是否存在
    if _, err := os.Stat(v2ctl); err != nil {
        return nil, newError("v2ctl doesn't exist").Base(err)
    }

    // 初始化错误缓冲区和输出缓冲区
    var errBuffer buf.MultiBufferContainer
    var outBuffer buf.MultiBufferContainer

    // 创建执行命令对象
    cmd := exec.Command(v2ctl, args...)
    // 将标准错误输出重定向到错误缓冲区
    cmd.Stderr = &errBuffer
    // 将标准输出重定向到输出缓冲区
    cmd.Stdout = &outBuffer
    // 获取系统进程属性
    cmd.SysProcAttr = getSysProcAttr()
    // 如果有输入，则将输入流重定向到命令的标准输入
    if input != nil {
        cmd.Stdin = input
    }

    // 启动命令
    if err := cmd.Start(); err != nil {
        return nil, newError("failed to start v2ctl").Base(err)
    }

    // 等待命令执行结束
    if err := cmd.Wait(); err != nil {
        // 构造错误消息
        msg := "failed to execute v2ctl"
        if errBuffer.Len() > 0 {
            msg += ": \n" + strings.TrimSpace(errBuffer.MultiBuffer.String())
        }
        return nil, newError(msg).Base(err)
    }

    // 记录标准错误输出的信息
    if !errBuffer.IsEmpty() {
        newError("<v2ctl message> \n", strings.TrimSpace(errBuffer.MultiBuffer.String())).AtInfo().WriteToLog()
    }

    // 返回输出缓冲区和空错误
    return outBuffer.MultiBuffer, nil
}
```