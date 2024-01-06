# `kubebench-aquasecurity\cmd\root.go`

```
// 代码版权声明和许可证信息
// 该代码受 Apache 许可证版本 2.0 的许可
// 除非符合许可证的规定，否则不得使用此文件
// 可以在 http://www.apache.org/licenses/LICENSE-2.0 获取许可证副本
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于“按原样”分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以获取特定语言的权限和限制

// 导入所需的包
package cmd

import (
	goflag "flag"  // 导入 flag 包并重命名为 goflag
	"fmt"  // 导入 fmt 包
	"os"  // 导入 os 包
# 导入所需的包
"github.com/aquasecurity/kube-bench/check"  # 导入名为 check 的包
"github.com/golang/glog"  # 导入名为 glog 的包
"github.com/spf13/cobra"  # 导入名为 cobra 的包
"github.com/spf13/viper"  # 导入名为 viper 的包

# 定义 FilterOpts 结构体，包含 CheckList、GroupList、Scored 和 Unscored 四个字段
type FilterOpts struct {
    CheckList string  # 检查列表
    GroupList string  # 分组列表
    Scored    bool    # 是否得分
    Unscored  bool    # 是否未得分
}

# 定义全局变量
var (
    envVarsPrefix       = "KUBE_BENCH"  # 环境变量前缀
    defaultKubeVersion  = "1.18"  # 默认的 Kubernetes 版本
    kubeVersion         string    # Kubernetes 版本
    benchmarkVersion    string    # 基准版本
    cfgFile             string    # 配置文件路径
)
# 设置配置文件目录
cfgDir = "./cfg/"
# 设置 JSON 格式标志
jsonFmt bool
# 设置 JUnit 格式标志
junitFmt bool
# 设置 PostgreSQL 格式标志
pgSQL bool
# 设置 aSFF 格式标志
aSFF bool
# 设置 master 文件名
masterFile = "master.yaml"
# 设置 node 文件名
nodeFile = "node.yaml"
# 设置 etcd 文件名
etcdFile = "etcd.yaml"
# 设置 controlplane 文件名
controlplaneFile = "controlplane.yaml"
# 设置 policies 文件名
policiesFile = "policies.yaml"
# 设置 managedservices 文件名
managedservicesFile = "managedservices.yaml"
# 设置退出码
exitCode int
# 设置无结果标志
noResults bool
# 设置无摘要标志
noSummary bool
# 设置无修复标志
noRemediations bool
# 设置跳过的 ID
skipIds string
# 设置无总数标志
noTotals bool
# 设置过滤选项
filterOpts FilterOpts
# 设置包括测试输出标志
includeTestOutput bool
# 设置输出文件名
outputFile string
// 定义一个错误变量 configFileError 和一个 check.Controls 类型的切片 controlsCollection
var (
    configFileError     error
    controlsCollection  []*check.Controls
)

// RootCmd 表示在没有子命令的情况下调用的基本命令
var RootCmd = &cobra.Command{
    Use:   os.Args[0], // 使用当前命令的名称作为命令的使用说明
    Short: "Run CIS Benchmarks checks against a Kubernetes deployment", // 简短的命令描述
    Long:  `This tool runs the CIS Kubernetes Benchmark (https://www.cisecurity.org/benchmark/kubernetes/)`, // 命令的详细描述
    Run: func(cmd *cobra.Command, args []string) { // 当命令被调用时执行的函数
        bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper()) // 获取基准版本
        if err != nil {
            exitWithError(fmt.Errorf("unable to determine benchmark version: %v", err)) // 如果获取基准版本出错，则输出错误信息并退出
        }
        glog.V(1).Infof("Running checks for benchmark %v", bv) // 输出正在运行的基准版本信息

        if isMaster() { // 如果是主节点
            glog.V(1).Info("== Running master checks ==") // 输出正在运行主节点检查的信息
            runChecks(check.MASTER, loadConfig(check.MASTER, bv)) // 运行主节点检查
// 控制平面仅适用于 CIS 1.5 及更高版本，这是对之前版本的门卫
valid, err := validTargets(bv, []string{string(check.CONTROLPLANE)}, viper.GetViper())
// 如果出现错误，则退出并显示错误信息
if err != nil {
    exitWithError(fmt.Errorf("error validating targets: %v", err))
}
// 如果有效，则记录信息并运行控制平面检查
if valid {
    glog.V(1).Info("== Running control plane checks ==")
    runChecks(check.CONTROLPLANE, loadConfig(check.CONTROLPLANE, bv))
} else {
    glog.V(1).Info("== Skipping master checks ==")
}

// Etcd 仅适用于 CIS 1.5 及更高版本，这是对之前版本的门卫
valid, err := validTargets(bv, []string{string(check.ETCD)}, viper.GetViper())
// 如果出现错误，则退出并显示错误信息
if err != nil {
    exitWithError(fmt.Errorf("error validating targets: %v", err))
}
		// 如果 valid 为真并且是 etcd 环境
		if valid && isEtcd() {
			// 输出日志信息，表示正在运行 etcd 检查
			glog.V(1).Info("== Running etcd checks ==")
			// 运行 etcd 检查
			runChecks(check.ETCD, loadConfig(check.ETCD, bv))
		} else {
			// 输出日志信息，表示跳过 etcd 检查
			glog.V(1).Info("== Skipping etcd checks ==")
		}

		// 输出日志信息，表示正在运行节点检查
		glog.V(1).Info("== Running node checks ==")
		// 运行节点检查
		runChecks(check.NODE, loadConfig(check.NODE, bv))

		// Policies 只对 CIS 1.5 及更高版本有效，这是对之前版本的一个门卫。
		// 验证目标是否有效
		valid, err = validTargets(bv, []string{string(check.POLICIES)}, viper.GetViper())
		if err != nil {
			// 如果出现错误，输出错误信息并退出
			exitWithError(fmt.Errorf("error validating targets: %v", err))
		}
		// 如果验证通过
		if valid {
			// 输出日志信息，表示正在运行策略检查
			glog.V(1).Info("== Running policies checks ==")
			// 运行策略检查
			runChecks(check.POLICIES, loadConfig(check.POLICIES, bv))
		} else {
		// 输出日志信息，表示跳过策略检查
		glog.V(1).Info("== Skipping policies checks ==")
		}

		// Managedservices 仅适用于 GKE 1.0 及更高版本，这是对之前版本的一个门卫。
		// 验证目标是否有效
		valid, err = validTargets(bv, []string{string(check.MANAGEDSERVICES)}, viper.GetViper())
		if err != nil {
			// 如果验证出错，输出错误信息并退出程序
			exitWithError(fmt.Errorf("error validating targets: %v", err))
		}
		// 如果验证通过
		if valid {
			// 输出日志信息，表示运行托管服务检查
			glog.V(1).Info("== Running managed services checks ==")
			// 运行托管服务检查
			runChecks(check.MANAGEDSERVICES, loadConfig(check.MANAGEDSERVICES, bv))
		} else {
			// 输出日志信息，表示跳过托管服务检查
			glog.V(1).Info("== Skipping managed services checks ==")
		}

		// 写入输出
		writeOutput(controlsCollection)
		// 选择退出码并退出程序
		exitCode := exitCodeSelection(controlsCollection)
		os.Exit(exitCode)
	},
}

// Execute函数将所有子命令添加到根命令，并设置标志位。这个函数由main.main()调用。只需要对根命令执行一次。
func Execute() {
    // 解析命令行参数
    goflag.CommandLine.Parse([]string{})

    // 执行根命令，如果出现错误则打印错误信息
    if err := RootCmd.Execute(); err != nil {
        fmt.Println(err)
        // 在退出前刷新日志
        glog.Flush()
        // 以非零状态退出
        os.Exit(-1)
    }
    // 在退出前刷新日志
    glog.Flush()
}

func init() {
    // 在执行命令之前初始化配置
    cobra.OnInitialize(initConfig)
}
	// 设置 RootCmd 的持久标志，指定当检查失败时的退出代码
	RootCmd.PersistentFlags().IntVar(&exitCode, "exit-code", 0, "Specify the exit code for when checks fail")
	// 设置 RootCmd 的持久标志，禁用打印结果部分
	RootCmd.PersistentFlags().BoolVar(&noResults, "noresults", false, "Disable printing of results section")
	// 设置 RootCmd 的持久标志，禁用打印摘要部分
	RootCmd.PersistentFlags().BoolVar(&noSummary, "nosummary", false, "Disable printing of summary section")
	// 设置 RootCmd 的持久标志，禁用打印补救措施部分
	RootCmd.PersistentFlags().BoolVar(&noRemediations, "noremediations", false, "Disable printing of remediations section")
	// 设置 RootCmd 的持久标志，禁用打印所有部分的失败、通过等检查的总数
	RootCmd.PersistentFlags().BoolVar(&noTotals, "nototals", false, "Disable printing of totals for failed, passed, ... checks across all sections")
	// 设置 RootCmd 的持久标志，打印结果为 JSON 格式
	RootCmd.PersistentFlags().BoolVar(&jsonFmt, "json", false, "Prints the results as JSON")
	// 设置 RootCmd 的持久标志，打印结果为 JUnit 格式
	RootCmd.PersistentFlags().BoolVar(&junitFmt, "junit", false, "Prints the results as JUnit")
	// 设置 RootCmd 的持久标志，将结果保存到 PostgreSQL
	RootCmd.PersistentFlags().BoolVar(&pgSQL, "pgsql", false, "Save the results to PostgreSQL")
	// 设置 RootCmd 的持久标志，将结果发送到 AWS Security Hub
	RootCmd.PersistentFlags().BoolVar(&aSFF, "asff", false, "Send the results to AWS Security Hub")
	// 设置 RootCmd 的持久标志，运行得分的 CIS 检查
	RootCmd.PersistentFlags().BoolVar(&filterOpts.Scored, "scored", true, "Run the scored CIS checks")
	// 设置 RootCmd 的持久标志，运行未得分的 CIS 检查
	RootCmd.PersistentFlags().BoolVar(&filterOpts.Unscored, "unscored", true, "Run the unscored CIS checks")
	// 设置 RootCmd 的持久标志，跳过指定的检查
	RootCmd.PersistentFlags().StringVar(&skipIds, "skip", "", "List of comma separated values of checks to be skipped")
	// 设置 RootCmd 的持久标志，当测试失败时打印实际结果
	RootCmd.PersistentFlags().BoolVar(&includeTestOutput, "include-test-output", false, "Prints the actual result when test fails")
	// 设置 RootCmd 的持久标志，将 JSON 结果写入输出文件
	RootCmd.PersistentFlags().StringVar(&outputFile, "outputfile", "", "Writes the JSON results to output file")

	// 设置 RootCmd 的持久标志，指定检查列表
	RootCmd.PersistentFlags().StringVarP(
		&filterOpts.CheckList,
		"check",
		"c",
		"",
		// 设置要在 CIS 文档中指定的要运行的检查的逗号分隔列表。例如 --check="1.1.1,1.1.2"
		`A comma-delimited list of checks to run as specified in CIS document. Example --check="1.1.1,1.1.2"`,
	)
	RootCmd.PersistentFlags().StringVarP(
		&filterOpts.GroupList,
		"group",
		"g",
		"",
		// 运行此逗号分隔的组列表下的所有检查。例如 --group="1.1"
		`Run all the checks under this comma-delimited list of groups. Example --group="1.1"`,
	)
	RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is ./cfg/config.yaml)")
	RootCmd.PersistentFlags().StringVarP(&cfgDir, "config-dir", "D", cfgDir, "config directory")
	RootCmd.PersistentFlags().StringVar(&kubeVersion, "version", "", "Manually specify Kubernetes version, automatically detected if unset")
	RootCmd.PersistentFlags().StringVar(&benchmarkVersion, "benchmark", "", "Manually specify CIS benchmark version. It would be an error to specify both --version and --benchmark flags")

	// 设置日志输出到标准错误流
	if err := goflag.Set("logtostderr", "true"); err != nil {
		fmt.Printf("unable to set logtostderr: %+v\n", err)
		os.Exit(-1)
	}
	// 遍历所有的命令行标志
	goflag.CommandLine.VisitAll(func(goflag *goflag.Flag) {
// 添加一个Go标志到RootCmd的持久标志中
RootCmd.PersistentFlags().AddGoFlag(goflag)
// 初始化配置，读取配置文件和环境变量
func initConfig() {
    if cfgFile != "" { // 如果指定了配置文件，则设置配置文件
        viper.SetConfigFile(cfgFile)
    } else {
        viper.SetConfigName("config") // 设置配置文件的名称（不包括扩展名）
        viper.AddConfigPath(cfgDir)   // 将./cfg添加为第一个搜索路径
    }

    // 从环境变量中读取标志值
    // 优先级：命令行标志优先于环境变量
    viper.SetEnvPrefix(envVarsPrefix)
    viper.AutomaticEnv()

    if kubeVersion == "" {
// 如果viper.Get("version")返回的值不为空，则将其赋值给env，并将env转换为string类型赋值给kubeVersion
if env := viper.Get("version"); env != nil {
    kubeVersion = env.(string)
}

// 如果找到配置文件，则读取配置文件
if err := viper.ReadInConfig(); err != nil {
    // 如果是配置文件未找到的错误，则暂时忽略错误，以防止不需要配置文件的命令退出
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        configFileError = err
    } else {
        // 如果找到配置文件但出现其他错误，则打印错误信息并退出程序
        colorPrint(check.FAIL, fmt.Sprintf("Failed to read config file: %v\n", err))
        os.Exit(1)
    }
}
```