# `kubebench-aquasecurity\cmd\master.go`

```
// 该代码段是版权声明和许可证信息，指明了代码的版权归属和使用许可
// 导入了 fmt 和 kube-bench/check 包，用于格式化输出和执行 kube-bench 的检查
// 导入cobra库，用于创建命令行应用
"github.com/spf13/cobra"

// 导入viper库，用于处理配置文件
"github.com/spf13/viper"
)

// masterCmd 表示 master 命令
var masterCmd = &cobra.Command{
	// 命令名称
	Use:   "master",
	// 命令简短描述
	Short: "Run Kubernetes benchmark checks from the master.yaml file.",
	// 命令详细描述
	Long:  `Run Kubernetes benchmark checks from the master.yaml file in cfg/<version>.`,
	// 命令执行函数
	Run: func(cmd *cobra.Command, args []string) {
		// 获取基准版本
		bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper())
		if err != nil {
			// 如果获取失败，输出错误信息并退出
			exitWithError(fmt.Errorf("unable to determine benchmark version: %v", err))
		}

		// 加载配置文件
		filename := loadConfig(check.MASTER, bv)
		// 运行检查
		runChecks(check.MASTER, filename)
		// 写入输出
		writeOutput(controlsCollection)
	},
}
# 在程序初始化时执行的函数
func init() {
    # 为 masterCmd 命令设置持久标志，用于指定 masterFile 变量的值
    masterCmd.PersistentFlags().StringVarP(&masterFile,
        "file", # 标志的名称
        "f",    # 标志的简称
        "/master.yaml", # 默认值
        "Alternative YAML file for master checks" # 标志的说明
    )

    # 将 masterCmd 命令添加到 RootCmd 命令中
    RootCmd.AddCommand(masterCmd)
}
```