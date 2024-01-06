# `kubebench-aquasecurity\cmd\node.go`

```
// 该代码段是版权声明和许可证信息，指明了代码的版权归属和使用许可
// 导入了 fmt 和 kube-bench/check 包，用于格式化输出和执行 kube-bench 的检查
// 导入cobra库，用于创建命令行工具
"github.com/spf13/cobra"

// 导入viper库，用于处理配置文件
"github.com/spf13/viper"
)

// nodeCmd代表node命令
var nodeCmd = &cobra.Command{
	// 命令名称
	Use:   "node",
	// 命令简短描述
	Short: "Run Kubernetes benchmark checks from the node.yaml file.",
	// 命令详细描述
	Long:  `Run Kubernetes benchmark checks from the node.yaml file in cfg/<version>.`,
	// 命令执行的具体操作
	Run: func(cmd *cobra.Command, args []string) {
		// 获取基准版本
		bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper())
		// 如果获取失败，输出错误信息并退出
		if err != nil {
			exitWithError(fmt.Errorf("unable to determine benchmark version: %v", err))
		}

		// 加载配置文件
		filename := loadConfig(check.NODE, bv)
		// 运行检查
		runChecks(check.NODE, filename)
		// 写入输出
		writeOutput(controlsCollection)
	},
}
# 在程序初始化时执行的函数
func init() {
    # 为 nodeCmd 命令添加持久标志，用于设置 nodeFile 变量的值
    nodeCmd.PersistentFlags().StringVarP(&nodeFile,
        "file", # 标志的名称
        "f",    # 标志的简称
        "/node.yaml",   # 默认值
        "Alternative YAML file for node checks",   # 标志的说明
    )

    # 将 nodeCmd 命令添加到 RootCmd 命令中
    RootCmd.AddCommand(nodeCmd)
}
```