# `kubebench-aquasecurity\cmd\master.go`

```go
// 版权声明，声明代码版权归 Aqua Security Software Ltd. 所有
// 根据 Apache 许可证 2.0 版本授权，除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则按“原样”分发软件，不附带任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

package cmd

import (
    "fmt"

    "github.com/aquasecurity/kube-bench/check"  // 导入 kube-bench/check 包
    "github.com/spf13/cobra"  // 导入 spf13/cobra 包
    "github.com/spf13/viper"  // 导入 spf13/viper 包
)

// masterCmd 表示 master 命令
var masterCmd = &cobra.Command{  // 创建名为 masterCmd 的命令
    Use:   "master",  // 使用说明
    Short: "Run Kubernetes benchmark checks from the master.yaml file.",  // 简短说明
    Long:  `Run Kubernetes benchmark checks from the master.yaml file in cfg/<version>.`,  // 详细说明
    Run: func(cmd *cobra.Command, args []string) {  // 运行命令的函数
        bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper())  // 获取基准版本
        if err != nil {  // 如果出错
            exitWithError(fmt.Errorf("unable to determine benchmark version: %v", err))  // 输出错误信息
        }

        filename := loadConfig(check.MASTER, bv)  // 加载配置文件
        runChecks(check.MASTER, filename)  // 运行检查
        writeOutput(controlsCollection)  // 写入输出结果
    },
}

func init() {
    masterCmd.PersistentFlags().StringVarP(&masterFile,  // 初始化 masterCmd 的持久标志
        "file",  // 标志名称
        "f",  // 简称
        "/master.yaml",  // 默认值
        "Alternative YAML file for master checks",  // 标志说明
    )

    RootCmd.AddCommand(masterCmd)  // 将 masterCmd 添加到 RootCmd
}
```