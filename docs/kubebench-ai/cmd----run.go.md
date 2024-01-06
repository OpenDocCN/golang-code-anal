# `kubebench-aquasecurity\cmd\run.go`

```
package cmd

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"os"  // 导入 os 包，用于操作系统功能
	"path/filepath"  // 导入 filepath 包，用于处理文件路径
	"strings"  // 导入 strings 包，用于字符串操作

	"github.com/aquasecurity/kube-bench/check"  // 导入 check 包
	"github.com/golang/glog"  // 导入 glog 包，用于日志记录
	"github.com/spf13/cobra"  // 导入 cobra 包，用于创建命令行应用
	"github.com/spf13/viper"  // 导入 viper 包，用于处理配置文件
)

func init() {
	RootCmd.AddCommand(runCmd)  // 将 runCmd 命令添加到 RootCmd 中
	runCmd.Flags().StringSliceP("targets", "s", []string{},  // 为 runCmd 命令添加名为 "targets" 的参数
		`Specify targets of the benchmark to run. These names need to match the filenames in the cfg/<version> directory.
	For example, to run the tests specified in master.yaml and etcd.yaml, specify --targets=master,etcd
	If no targets are specified, run tests from all files in the cfg/<version> directory.
// runCmd 表示运行命令
var runCmd = &cobra.Command{
    Use:   "run", // 命令名称
    Short: "Run tests", // 简短描述
    Long:  `Run tests. If no arguments are specified, runs tests from all files`, // 长描述
    Run: func(cmd *cobra.Command, args []string) { // 运行命令的函数
        targets, err := cmd.Flags().GetStringSlice("targets") // 获取命令行参数中的 targets
        if err != nil {
            exitWithError(fmt.Errorf("unable to get `targets` from command line :%v", err)) // 如果获取失败，输出错误信息
        }

        bv, err := getBenchmarkVersion(kubeVersion, benchmarkVersion, viper.GetViper()) // 获取基准版本
        if err != nil {
            exitWithError(fmt.Errorf("unable to get benchmark version. error: %v", err)) // 如果获取失败，输出错误信息
        }

        glog.V(2).Infof("Checking targets %v for %v", targets, bv) // 输出日志信息
		// 从配置文件中加载版本到目标的映射关系
		benchmarkVersionToTargetsMap, err := loadTargetMapping(viper.GetViper())
		// 如果加载出错，则打印错误信息并退出程序
		if err != nil {
			exitWithError(fmt.Errorf("error loading targets: %v", err))
		}
		// 验证目标是否有效
		valid, err := validTargets(bv, targets, viper.GetViper())
		// 如果验证出错，则打印错误信息并退出程序
		if err != nil {
			exitWithError(fmt.Errorf("error validating targets: %v", err))
		}
		// 如果目标不为空且验证不通过
		if len(targets) > 0 && !valid {
		}

		// 合并特定版本的配置（如果有的话）
		path := filepath.Join(cfgDir, bv)
		mergeConfig(path)

		// 运行目标
		err = run(targets, bv)
		// 如果运行出错，则打印错误信息
		if err != nil {
			fmt.Printf("Error in run: %v\n", err)
		}
	},
// run 函数接收目标列表和基准版本作为参数，返回一个错误
func run(targets []string, benchmarkVersion string) (err error) {
    // 获取测试 YAML 文件列表
    yamlFiles, err := getTestYamlFiles(targets, benchmarkVersion)
    if err != nil {
        return err
    }

    // 输出日志，显示运行测试的文件列表
    glog.V(3).Infof("Running tests from files %v\n", yamlFiles)

    // 遍历测试 YAML 文件列表
    for _, yamlFile := range yamlFiles {
        // 获取文件名
        _, name := filepath.Split(yamlFile)
        // 解析文件名，获取测试类型
        testType := check.NodeType(strings.Split(name, ".")[0])
        // 运行检查
        runChecks(testType, yamlFile)
    }

    // 写入输出
    writeOutput(controlsCollection)
    return nil
}
// 获取测试 YAML 文件列表
func getTestYamlFiles(targets []string, benchmarkVersion string) (yamlFiles []string, err error) {
	// 检查指定的目标是否在配置目录中有对应的 YAML 文件
	configFileDirectory := filepath.Join(cfgDir, benchmarkVersion)
	for _, target := range targets {
		// 根据目标名称翻译成文件名
		filename := translate(target) + ".yaml"
		// 拼接文件路径
		file := filepath.Join(configFileDirectory, filename)
		// 检查文件是否存在
		if _, err := os.Stat(file); err != nil {
			// 如果文件不存在，返回错误
			return nil, fmt.Errorf("file %s not found for version %s", filename, benchmarkVersion)
		}
		// 将文件路径添加到 YAML 文件列表中
		yamlFiles = append(yamlFiles, file)
	}

	// 如果没有指定目标，将从目录中的所有文件运行测试
	if len(yamlFiles) == 0 {
		// 从目录中获取所有 YAML 文件
		yamlFiles, err = getYamlFilesFromDir(configFileDirectory)
		if err != nil {
			// 如果获取失败，返回错误
			return nil, err
		}
	}
# 返回 yamlFiles 和 err 两个变量
return yamlFiles, err
# 将目标字符串中的 "worker" 替换为 "node"，并转换为小写
func translate(target string) string:
    return strings.Replace(strings.ToLower(target), "worker", "node", -1)
```