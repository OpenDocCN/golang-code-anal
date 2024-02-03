# `kubo\test\cli\tracing_test.go`

```go
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，提供对操作系统功能的访问
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "path/filepath"  // 导入 path/filepath 包，用于处理文件路径
    "strings"  // 导入 strings 包，提供对字符串的操作
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，提供时间相关的功能

    "github.com/ipfs/kubo/test/cli/harness"  // 导入自定义包 harness
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入自定义包 testutils
    "github.com/stretchr/testify/assert"  // 导入 testify 包中的 assert 模块
    "github.com/stretchr/testify/require"  // 导入 testify 包中的 require 模块
)

var otelCollectorConfigYAML = `
receivers:
  otlp:
    protocols:
      grpc:

processors:
  batch:

exporters:
  file:
    path: /traces/traces.json

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [file]
`

func TestTracing(t *testing.T) {
    testutils.RequiresDocker(t)  // 调用 testutils 包中的 RequiresDocker 函数
    t.Parallel()  // 标记测试函数可以并行执行
    node := harness.NewT(t).NewNode().Init()  // 创建测试节点

    node.WriteBytes("collector-config.yaml", []byte(otelCollectorConfigYAML))  // 写入配置文件

    // touch traces.json and give it 777 perms in case Docker runs as a different user
    node.WriteBytes("traces.json", nil)  // 创建空的 traces.json 文件
    err := os.Chmod(filepath.Join(node.Dir, "traces.json"), 0o777)  // 修改文件权限为 777
    require.NoError(t, err)  // 断言操作没有错误发生

    dockerBin, err := exec.LookPath("docker")  // 查找 docker 可执行文件的路径
    require.NoError(t, err)  // 断言操作没有错误发生
    node.Runner.MustRun(harness.RunRequest{  // 使用 harness 包中的 RunRequest 结构体执行命令
        Path: dockerBin,  // 设置命令路径
        Args: []string{  // 设置命令参数
            "run",
            "--rm",
            "--detach",
            "--volume", fmt.Sprintf("%s:/config.yaml", filepath.Join(node.Dir, "collector-config.yaml")),
            "--volume", fmt.Sprintf("%s:/traces", node.Dir),
            "--net", "host",
            "--name", "ipfs-test-otel-collector",
            "otel/opentelemetry-collector-contrib:0.52.0",
            "--config", "/config.yaml",
        },
    })

    t.Cleanup(func() {  // 在测试函数结束时执行清理操作
        node.Runner.MustRun(harness.RunRequest{  // 使用 harness 包中的 RunRequest 结构体执行命令
            Path: dockerBin,  // 设置命令路径
            Args: []string{"stop", "ipfs-test-otel-collector"},  // 设置命令参数
        })
    })

    node.Runner.Env["OTEL_TRACES_EXPORTER"] = "otlp"  // 设置环境变量
    node.Runner.Env["OTEL_EXPORTER_OTLP_PROTOCOL"] = "grpc"  // 设置环境变量
    node.Runner.Env["OTEL_EXPORTER_OTLP_ENDPOINT"] = "http://localhost:4317"  // 设置环境变量
    node.StartDaemon()  // 启动守护进程
}
    # 使用 assert.Eventually 函数来断言某个条件最终会成立
    assert.Eventually(t,
        # 匿名函数，返回一个布尔值
        func() bool {
            # 读取文件内容到 b，同时检查是否有错误发生
            b, err := os.ReadFile(filepath.Join(node.Dir, "traces.json"))
            # 断言没有错误发生
            require.NoError(t, err)
            # 返回文件内容是否包含特定字符串的布尔值
            return strings.Contains(string(b), "go-ipfs")
        },
        # 最大等待时间
        5*time.Minute,
        # 检查间隔时间
        10*time.Millisecond,
    )
# 闭合前面的函数定义
```