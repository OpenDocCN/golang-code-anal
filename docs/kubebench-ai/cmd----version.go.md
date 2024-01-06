# `kubebench-aquasecurity\cmd\version.go`

```
// 导入必要的包
package cmd

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"github.com/spf13/cobra"  // 导入 cobra 包，用于创建命令行工具
)

var KubeBenchVersion string  // 定义全局变量 KubeBenchVersion，用于存储 kube-bench 的版本信息

// versionCmd 表示版本命令
var versionCmd = &cobra.Command{
	Use:   "version",  // 命令名称为 version
	Short: "Shows the version of kube-bench.",  // 简短描述
	Long:  `Shows the version of kube-bench.`,  // 详细描述
	Run: func(cmd *cobra.Command, args []string) {  // 定义命令执行的函数
		fmt.Println(KubeBenchVersion)  // 打印 KubeBenchVersion 的值
	},
}

func init() {
# 将versionCmd命令添加到RootCmd中
RootCmd.AddCommand(versionCmd)
```