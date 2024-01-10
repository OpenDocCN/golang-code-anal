# `kubebench-aquasecurity\cmd\version.go`

```
// 导入必要的包
package cmd

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "github.com/spf13/cobra"  // 导入 cobra 包，用于创建命令行应用
)

var KubeBenchVersion string  // 定义全局变量 KubeBenchVersion，用于存储版本信息

// versionCmd 表示版本命令
var versionCmd = &cobra.Command{
    Use:   "version",  // 命令名称
    Short: "Shows the version of kube-bench.",  // 简短描述
    Long:  `Shows the version of kube-bench.`,  // 详细描述
    Run: func(cmd *cobra.Command, args []string) {  // 执行命令时的操作
        fmt.Println(KubeBenchVersion)  // 打印 KubeBenchVersion 变量的值
    },
}

func init() {
    RootCmd.AddCommand(versionCmd)  // 将 versionCmd 命令添加到 RootCmd 中
}
```