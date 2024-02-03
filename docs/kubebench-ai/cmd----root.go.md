# `kubebench-aquasecurity\cmd\root.go`

```go
// 版权声明和许可证信息
// 2017年版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0版授权
// 除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则不得分发软件
// 根据许可证的规定分发的软件是基于"AS IS"的基础分发的
// 没有任何形式的保证或条件，无论是明示的还是暗示的
// 请查看许可证以获取特定语言的权限和限制

// 导入所需的包
package cmd

import (
    goflag "flag"  // 导入flag包并重命名为goflag
    "fmt"  // 导入fmt包
    "os"  // 导入os包

    "github.com/aquasecurity/kube-bench/check"  // 导入check包
    "github.com/golang/glog"  // 导入glog包
    "github.com/spf13/cobra"  // 导入cobra包
    "github.com/spf13/viper"  // 导入viper包
)

// 定义FilterOpts结构体
type FilterOpts struct {
    CheckList string  // 检查列表
    GroupList string  // 分组列表
    Scored    bool  // 是否计分
    Unscored  bool  // 是否不计分
}

// 定义一系列变量
var (
    envVarsPrefix       = "KUBE_BENCH"  // 环境变量前缀
    defaultKubeVersion  = "1.18"  // 默认的Kubernetes版本
    kubeVersion         string  // Kubernetes版本
    benchmarkVersion    string  // 基准版本
    cfgFile             string  // 配置文件
    cfgDir              = "./cfg/"  // 配置文件目录
    jsonFmt             bool  // JSON格式
    junitFmt            bool  // JUnit格式
    pgSQL               bool  // 是否使用PostgreSQL
    aSFF                bool  // 是否使用ASFF
    masterFile          = "master.yaml"  // 主节点文件
    nodeFile            = "node.yaml"  // 节点文件
    etcdFile            = "etcd.yaml"  // etcd文件
    controlplaneFile    = "controlplane.yaml"  // 控制平面文件
    policiesFile        = "policies.yaml"  // 策略文件
    managedservicesFile = "managedservices.yaml"  // 管理服务文件
    exitCode            int  // 退出码
    noResults           bool  // 是否无结果
    noSummary           bool  // 是否无摘要
    noRemediations      bool  // 是否无修复
    skipIds             string  // 跳过的ID
    noTotals            bool  // 是否无总数
    filterOpts          FilterOpts  // 过滤选项
    includeTestOutput   bool  // 是否包含测试输出
    outputFile          string  // 输出文件
    configFileError     error  // 配置文件错误
    controlsCollection  []*check.Controls  // 控制集合
)

// RootCmd表示在没有子命令的情况下调用时的基本命令
var RootCmd = &cobra.Command{
    Use:   os.Args[0],  // 使用os.Args[0]作为命令名称
    # 简短描述：对Kubernetes部署运行CIS基准检查
    Short: "Run CIS Benchmarks checks against a Kubernetes deployment",
    # 长描述：该工具运行CIS Kubernetes基准（https://www.cisecurity.org/benchmark/kubernetes/）
    Long:  `This tool runs the CIS Kubernetes Benchmark (https://www.cisecurity.org/benchmark/kubernetes/)`,
    },
// Execute函数将所有子命令添加到根命令，并设置标志位。这个函数由main.main()调用。只需要对根命令执行一次。
func Execute() {
    // 解析命令行参数
    goflag.CommandLine.Parse([]string{})

    // 执行根命令，如果出现错误则打印错误信息
    if err := RootCmd.Execute(); err != nil {
        fmt.Println(err)
        // 在退出非零状态之前刷新日志
        glog.Flush()
        os.Exit(-1)
    }
    // 在退出之前刷新日志
    glog.Flush()
}

// 初始化函数
func init() {
    // 在初始化时调用initConfig函数
    cobra.OnInitialize(initConfig)

    // 输出控制
    RootCmd.PersistentFlags().IntVar(&exitCode, "exit-code", 0, "Specify the exit code for when checks fail")
    RootCmd.PersistentFlags().BoolVar(&noResults, "noresults", false, "Disable printing of results section")
    RootCmd.PersistentFlags().BoolVar(&noSummary, "nosummary", false, "Disable printing of summary section")
    RootCmd.PersistentFlags().BoolVar(&noRemediations, "noremediations", false, "Disable printing of remediations section")
    RootCmd.PersistentFlags().BoolVar(&noTotals, "nototals", false, "Disable printing of totals for failed, passed, ... checks across all sections")
    RootCmd.PersistentFlags().BoolVar(&jsonFmt, "json", false, "Prints the results as JSON")
    RootCmd.PersistentFlags().BoolVar(&junitFmt, "junit", false, "Prints the results as JUnit")
    RootCmd.PersistentFlags().BoolVar(&pgSQL, "pgsql", false, "Save the results to PostgreSQL")
    RootCmd.PersistentFlags().BoolVar(&aSFF, "asff", false, "Send the results to AWS Security Hub")
    RootCmd.PersistentFlags().BoolVar(&filterOpts.Scored, "scored", true, "Run the scored CIS checks")
    RootCmd.PersistentFlags().BoolVar(&filterOpts.Unscored, "unscored", true, "Run the unscored CIS checks")
    RootCmd.PersistentFlags().StringVar(&skipIds, "skip", "", "List of comma separated values of checks to be skipped")
    RootCmd.PersistentFlags().BoolVar(&includeTestOutput, "include-test-output", false, "Prints the actual result when test fails")
}
    # 设置RootCmd的持久标志，用于指定输出文件的名称和路径
    RootCmd.PersistentFlags().StringVar(&outputFile, "outputfile", "", "Writes the JSON results to output file")
    
    # 设置RootCmd的持久标志，用于指定要运行的检查列表，以逗号分隔的形式指定在CIS文档中运行的检查。示例 --check="1.1.1,1.1.2"
    RootCmd.PersistentFlags().StringVarP(
        &filterOpts.CheckList,
        "check",
        "c",
        "",
        `A comma-delimited list of checks to run as specified in CIS document. Example --check="1.1.1,1.1.2"`,
    )
    
    # 设置RootCmd的持久标志，用于指定要运行的组列表，以逗号分隔的形式指定要运行的所有检查的组。示例 --group="1.1"
    RootCmd.PersistentFlags().StringVarP(
        &filterOpts.GroupList,
        "group",
        "g",
        "",
        `Run all the checks under this comma-delimited list of groups. Example --group="1.1"`,
    )
    
    # 设置RootCmd的持久标志，用于指定配置文件的名称和路径，默认为./cfg/config.yaml
    RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is ./cfg/config.yaml)")
    
    # 设置RootCmd的持久标志，用于指定配置目录的名称和路径
    RootCmd.PersistentFlags().StringVarP(&cfgDir, "config-dir", "D", cfgDir, "config directory")
    
    # 设置RootCmd的持久标志，用于手动指定Kubernetes版本，如果未设置，则自动检测
    RootCmd.PersistentFlags().StringVar(&kubeVersion, "version", "", "Manually specify Kubernetes version, automatically detected if unset")
    
    # 设置RootCmd的持久标志，用于手动指定CIS基准版本。如果同时指定--version和--benchmark标志，则会出现错误
    RootCmd.PersistentFlags().StringVar(&benchmarkVersion, "benchmark", "", "Manually specify CIS benchmark version. It would be an error to specify both --version and --benchmark flags")
    
    # 设置logtostderr标志为true
    if err := goflag.Set("logtostderr", "true"); err != nil {
        fmt.Printf("unable to set logtostderr: %+v\n", err)
        os.Exit(-1)
    }
    
    # 遍历所有的goflag，并将其添加到RootCmd的持久标志中
    goflag.CommandLine.VisitAll(func(goflag *goflag.Flag) {
        RootCmd.PersistentFlags().AddGoFlag(goflag)
    })
// initConfig 读取配置文件和环境变量（如果设置了的话）
func initConfig() {
    if cfgFile != "" { // 通过标志启用通过文件指定配置文件
        viper.SetConfigFile(cfgFile)
    } else {
        viper.SetConfigName("config") // 配置文件的名称（不包括扩展名）
        viper.AddConfigPath(cfgDir)   // 将 ./cfg 添加为第一个搜索路径
    }

    // 从环境变量中读取标志值
    // 优先级：命令行标志优先于环境变量
    viper.SetEnvPrefix(envVarsPrefix)
    viper.AutomaticEnv()

    if kubeVersion == "" {
        if env := viper.Get("version"); env != nil {
            kubeVersion = env.(string)
        }
    }

    // 如果找到配置文件，则读取它
    if err := viper.ReadInConfig(); err != nil {
        if _, ok := err.(viper.ConfigFileNotFoundError); ok {
            // 未找到配置文件；暂时忽略错误，以防止不需要配置文件的命令退出
            configFileError = err
        } else {
            // 找到配置文件，但产生了另一个错误
            colorPrint(check.FAIL, fmt.Sprintf("Failed to read config file: %v\n", err))
            os.Exit(1)
        }
    }
}
```