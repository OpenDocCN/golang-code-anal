# `kubebench-aquasecurity\cmd\run.go`

```
package cmd

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，提供对操作系统功能的访问
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "strings"  // 导入 strings 包，提供对字符串的操作

    "github.com/aquasecurity/kube-bench/check"  // 导入 kube-bench/check 包
    "github.com/golang/glog"  // 导入 glog 包，用于日志记录
    "github.com/spf13/cobra"  // 导入 cobra 包，用于创建命令行应用
    "github.com/spf13/viper"  // 导入 viper 包，用于处理配置文件
)

func init() {
    RootCmd.AddCommand(runCmd)  // 将 runCmd 命令添加到 RootCmd 中
    runCmd.Flags().StringSliceP("targets", "s", []string{},  // 设置 runCmd 命令的 targets 标志
        `Specify targets of the benchmark to run. These names need to match the filenames in the cfg/<version> directory.
    For example, to run the tests specified in master.yaml and etcd.yaml, specify --targets=master,etcd
    If no targets are specified, run tests from all files in the cfg/<version> directory.
    `)
}

// runCmd represents the run command
var runCmd = &cobra.Command{  // 创建名为 runCmd 的命令
    Use:   "run",  // 设置命令的使用说明
    Short: "Run tests",  // 设置命令的简短描述
    Long:  `Run tests. If no arguments are specified, runs tests from all files`,  // 设置命令的详细描述
    # Run 函数，处理命令行参数并执行相应操作
    Run: func(cmd *cobra.Command, args []string) {
        # 从命令行参数中获取 targets 列表
        targets, err := cmd.Flags().GetStringSlice("targets")
        # 如果获取失败，输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("unable to get `targets` from command line :%v", err))
        }

        # 获取基准版本号
        bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper())
        # 如果获取失败，输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("unable to get benchmark version. error: %v", err))
        }

        # 输出日志信息，检查目标和基准版本
        glog.V(2).Infof("Checking targets %v for %v", targets, bv)
        
        # 加载目标映射
        benchmarkVersionToTargetsMap, err := loadTargetMapping(viper.GetViper())
        # 如果加载失败，输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("error loading targets: %v", err))
        }
        
        # 验证目标是否有效
        valid, err := validTargets(bv, targets, viper.GetViper())
        # 如果验证失败，输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("error validating targets: %v", err))
        }
        
        # 如果指定了目标但不合法，输出错误信息并退出
        if len(targets) > 0 && !valid {
            exitWithError(fmt.Errorf(fmt.Sprintf(`The specified --targets "%s" are not configured for the CIS Benchmark %s\n Valid targets %v`, strings.Join(targets, ","), bv, benchmarkVersionToTargetsMap[bv])))
        }

        # 合并特定版本的配置（如果有）
        path := filepath.Join(cfgDir, bv)
        mergeConfig(path)

        # 执行运行操作
        err = run(targets, bv)
        # 如果运行出错，输出错误信息
        if err != nil {
            fmt.Printf("Error in run: %v\n", err)
        }
    },
// 运行测试
func run(targets []string, benchmarkVersion string) (err error) {
    // 获取测试 YAML 文件
    yamlFiles, err := getTestYamlFiles(targets, benchmarkVersion)
    if err != nil {
        return err
    }

    // 输出正在运行的文件
    glog.V(3).Infof("Running tests from files %v\n", yamlFiles)

    // 遍历 YAML 文件
    for _, yamlFile := range yamlFiles {
        _, name := filepath.Split(yamlFile)
        // 检查节点类型
        testType := check.NodeType(strings.Split(name, ".")[0])
        // 运行检查
        runChecks(testType, yamlFile)
    }

    // 写入输出
    writeOutput(controlsCollection)
    return nil
}

// 获取测试 YAML 文件
func getTestYamlFiles(targets []string, benchmarkVersion string) (yamlFiles []string, err error) {
    // 检查指定的目标是否在配置目录中有对应的 YAML 文件
    configFileDirectory := filepath.Join(cfgDir, benchmarkVersion)
    for _, target := range targets {
        filename := translate(target) + ".yaml"
        file := filepath.Join(configFileDirectory, filename)
        if _, err := os.Stat(file); err != nil {
            return nil, fmt.Errorf("file %s not found for version %s", filename, benchmarkVersion)
        }
        yamlFiles = append(yamlFiles, file)
    }

    // 如果没有指定目标，将从目录中运行所有文件的测试
    if len(yamlFiles) == 0 {
        yamlFiles, err = getYamlFilesFromDir(configFileDirectory)
        if err != nil {
            return nil, err
        }
    }

    return yamlFiles, err
}

// 将目标翻译为字符串
func translate(target string) string {
    return strings.Replace(strings.ToLower(target), "worker", "node", -1)
}
```