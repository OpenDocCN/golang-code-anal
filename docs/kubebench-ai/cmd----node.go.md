# `kubebench-aquasecurity\cmd\node.go`

```go
// 版权声明，声明代码的版权信息
// 根据 Apache 许可证 2.0 版本授权，限制了对该文件的使用条件
// 获取 Apache 许可证 2.0 版本的副本
// 如果符合适用法律要求或经书面同意，可以分发该软件
// 根据许可证的规定，以“原样”分发，不提供任何明示或暗示的担保或条件
// 查看许可证以了解特定语言的权限和限制

package cmd

import (
    "fmt"
    // 导入 kube-bench/check 包
    "github.com/aquasecurity/kube-bench/check"
    // 导入 cobra 包
    "github.com/spf13/cobra"
    // 导入 viper 包
    "github.com/spf13/viper"
)

// nodeCmd 表示 node 命令
var nodeCmd = &cobra.Command{
    // 使用说明
    Use:   "node",
    // 简短说明
    Short: "Run Kubernetes benchmark checks from the node.yaml file.",
    // 长说明
    Long:  `Run Kubernetes benchmark checks from the node.yaml file in cfg/<version>.`,
    // 运行命令
    Run: func(cmd *cobra.Command, args []string) {
        // 获取基准版本
        bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper())
        if err != nil {
            // 如果获取基准版本出错，输出错误信息并退出
            exitWithError(fmt.Errorf("unable to determine benchmark version: %v", err))
        }
        // 加载配置文件
        filename := loadConfig(check.NODE, bv)
        // 运行节点检查
        runChecks(check.NODE, filename)
        // 写入输出
        writeOutput(controlsCollection)
    },
}

func init() {
    // 添加持久标志
    nodeCmd.PersistentFlags().StringVarP(&nodeFile,
        "file",
        "f",
        "/node.yaml",
        "Alternative YAML file for node checks",
    )
    // 添加 nodeCmd 命令
    RootCmd.AddCommand(nodeCmd)
}
```