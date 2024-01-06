# `kubebench-aquasecurity\cmd\common.go`

```
// 该部分代码是版权声明和许可证信息，指明了代码的版权归属和使用许可
// 导入所需的包
package cmd

import (
    "bufio" // 导入用于缓冲读取的包
    "encoding/json" // 导入用于 JSON 编解码的包
    "fmt" // 导入用于格式化输出的包
	"io/ioutil"  // 导入io/ioutil包，用于读取文件内容
	"os"  // 导入os包，用于操作文件和目录
	"path/filepath"  // 导入path/filepath包，用于处理文件路径
	"sort"  // 导入sort包，用于对数据进行排序
	"strconv"  // 导入strconv包，用于字符串和数字之间的转换
	"strings"  // 导入strings包，用于处理字符串

	"github.com/aquasecurity/kube-bench/check"  // 导入check包，用于执行安全检查
	"github.com/golang/glog"  // 导入glog包，用于日志记录
	"github.com/spf13/viper"  // 导入viper包，用于处理配置文件

)

// NewRunFilter constructs a Predicate based on FilterOpts which determines whether tested Checks should be run or not.
// NewRunFilter函数根据FilterOpts构造一个Predicate，用于确定是否应该运行被测试的Checks。
func NewRunFilter(opts FilterOpts) (check.Predicate, error) {
	if opts.CheckList != "" && opts.GroupList != "" {
		return nil, fmt.Errorf("group option and check option can't be used together")  // 如果同时使用了group选项和check选项，则返回错误
	}

	var groupIDs map[string]bool  // 定义一个map，用于存储group的ID
	if opts.GroupList != "" {  // 如果指定了group列表
# 调用 cleanIDs 函数清理 GroupList 中的 ID，并将结果赋值给 groupIDs
groupIDs = cleanIDs(opts.GroupList)

# 声明一个空的 map 类型变量 checkIDs
var checkIDs map[string]bool
# 如果 CheckList 不为空，则调用 cleanIDs 函数清理 CheckList 中的 ID，并将结果赋值给 checkIDs
if opts.CheckList != "" {
    checkIDs = cleanIDs(opts.CheckList)
}

# 返回一个匿名函数，该函数接受一个 check.Group 类型的参数 g 和一个 check.Check 类型的参数 c，并返回一个布尔值
return func(g *check.Group, c *check.Check) bool {
    # 声明一个布尔变量 test，并赋值为 true
    var test = true
    # 如果 groupIDs 中的元素数量大于 0
    if len(groupIDs) > 0 {
        # 检查 groupIDs 中是否存在 g.ID，并将结果赋值给 ok
        _, ok := groupIDs[g.ID]
        # 更新 test 变量，如果 ok 为 true，则 test 为 true，否则为 false
        test = test && ok
    }

    # 如果 checkIDs 中的元素数量大于 0
    if len(checkIDs) > 0 {
        # 检查 checkIDs 中是否存在 c.ID，并将结果赋值给 ok
        _, ok := checkIDs[c.ID]
        # 更新 test 变量，如果 ok 为 true，则 test 为 true，否则为 false
        test = test && ok
    }
		test = test && (opts.Scored && c.Scored || opts.Unscored && !c.Scored)
		// 对测试进行逻辑运算，根据条件判断是否满足要求

		return test
	}, nil
}
// 运行检查
func runChecks(nodetype check.NodeType, testYamlFile string) {
	// 验证配置文件是否在 Cobra 子命令初始化期间加载到 Viper 中
	if configFileError != nil {
		colorPrint(check.FAIL, fmt.Sprintf("Failed to read config file: %v\n", configFileError))
		os.Exit(1)
	}
	// 读取测试 YAML 文件
	in, err := ioutil.ReadFile(testYamlFile)
	if err != nil {
		exitWithError(fmt.Errorf("error opening %s test file: %v", testYamlFile, err))
	}
	// 输出日志信息，使用测试文件
	glog.V(1).Info(fmt.Sprintf("Using test file: %s\n", testYamlFile))
```

// 获取此测试部分的 viper 配置
typeConf := viper.Sub(string(nodetype))
if typeConf == nil {
    colorPrint(check.FAIL, fmt.Sprintf("No config settings for %s\n", string(nodetype)))
    os.Exit(1)
}

// 获取我们在这一部分测试中需要的可执行文件集合
binmap, err := getBinaries(typeConf, nodetype)

// 检查我们在这一部分测试中需要的可执行文件是否正在运行
if err != nil {
    glog.V(1).Info(fmt.Sprintf("failed to get a set of executables needed for tests: %v", err))
}

// 获取配置文件的映射
confmap := getFiles(typeConf, "config")
// 获取服务文件的映射
svcmap := getFiles(typeConf, "service")
// 获取 kubeconfig 文件的映射
kubeconfmap := getFiles(typeConf, "kubeconfig")
// 获取 ca 文件的映射
cafilemap := getFiles(typeConf, "ca")
// 变量替换。替换控制文件中所有变量的出现。
s := string(in) // 将输入转换为字符串
s, binSubs := makeSubstitutions(s, "bin", binmap) // 替换控制文件中的"bin"变量，并返回替换后的字符串和替换的映射
s, _ = makeSubstitutions(s, "conf", confmap) // 替换控制文件中的"conf"变量，并返回替换后的字符串
s, _ = makeSubstitutions(s, "svc", svcmap) // 替换控制文件中的"svc"变量，并返回替换后的字符串
s, _ = makeSubstitutions(s, "kubeconfig", kubeconfmap) // 替换控制文件中的"kubeconfig"变量，并返回替换后的字符串
s, _ = makeSubstitutions(s, "cafile", cafilemap) // 替换控制文件中的"cafile"变量，并返回替换后的字符串

// 使用控制文件创建新的控制对象
controls, err := check.NewControls(nodetype, []byte(s))
if err != nil {
    exitWithError(fmt.Errorf("error setting up %s controls: %v", nodetype, err))
}

// 创建新的运行器对象
runner := check.NewRunner()

// 创建新的运行过滤器对象
filter, err := NewRunFilter(filterOpts)
if err != nil {
    exitWithError(fmt.Errorf("error setting up run filter: %v", err))
}

// 生成默认的环境审计
generateDefaultEnvAudit(controls, binSubs)
// 运行检查，传入运行器、过滤器和解析跳过 ID
controls.RunChecks(runner, filter, parseSkipIds(skipIds))
// 将控制集合附加到控制集合中
controlsCollection = append(controlsCollection, controls)
}

// 生成默认环境审核，传入控制和二进制替换
func generateDefaultEnvAudit(controls *check.Controls, binSubs []string){
	// 遍历控制组
	for _, group := range controls.Groups {
		// 遍历检查项
		for _, checkItem := range group.Checks {
			// 如果测试项不为空且未禁用环境测试
			if checkItem.Tests != nil && !checkItem.DisableEnvTesting {
				// 遍历测试项
				for _, test := range checkItem.Tests.TestItems {
					// 如果测试环境不为空且审核环境为空
					if test.Env != "" && checkItem.AuditEnv == "" {
						// 二进制路径为空
						binPath := ""

						// 如果二进制替换长度为1
						if len(binSubs) == 1 {
							// 设置二进制路径为第一个替换值
							binPath = binSubs[0]
						} else {
							// 打印未明确指定审核环境的检查项和无法确定二进制路径的消息
							fmt.Printf("AuditEnv not explicit for check (%s), where bin path cannot be determined\n", checkItem.ID)
						}

						// 如果测试环境不为空且审核环境为空
						if test.Env != "" && checkItem.AuditEnv == "" {
// 设置checkItem的AuditEnv字段为根据binPath获取进程环境变量的命令
// 使用fmt.Sprintf将命令格式化为字符串
// 如果skipIds不为空，将其按逗号分隔后存入skipIdMap中
// 返回skipIdMap
// colorPrint函数用于以特定颜色输出状态和消息字符串
// colorPrint函数用于根据状态打印带有颜色的信息
func colorPrint(state check.State, s string) {
    // 使用状态对应的颜色打印状态信息
    colors[state].Printf("[%s] ", state)
    // 打印字符串信息
    fmt.Printf("%s", s)
}

// prettyPrint函数用于以人类可读的格式将结果输出到标准输出
func prettyPrint(r *check.Controls, summary check.Summary) {
    // 如果不是无结果模式，则打印检查结果
    if !noResults {
        // 打印检查结果的ID和文本信息
        colorPrint(check.INFO, fmt.Sprintf("%s %s\n", r.ID, r.Text))
        // 遍历每个检查组
        for _, g := range r.Groups {
            // 打印检查组的ID和文本信息
            colorPrint(check.INFO, fmt.Sprintf("%s %s\n", g.ID, g.Text))
            // 遍历每个检查
            for _, c := range g.Checks {
                // 根据状态打印检查的ID和文本信息
                colorPrint(c.State, fmt.Sprintf("%s %s\n", c.ID, c.Text)

                // 如果包括测试输出并且状态为失败且实际值长度大于0，则打印原始输出
                if includeTestOutput && c.State == check.FAIL && len(c.ActualValue) > 0 {
                    printRawOutput(c.ActualValue)
                }
            }
        }
    }
}
		fmt.Println()
	}

	// 打印修复措施。
	if !noRemediations {
		// 如果有失败或警告，则打印修复措施的标题
		if summary.Fail > 0 || summary.Warn > 0 {
			colors[check.WARN].Printf("== Remediations %s ==\n", r.Type)
			// 遍历每个分组
			for _, g := range r.Groups {
				// 遍历每个检查
				for _, c := range g.Checks {
					// 如果检查状态为失败，则打印检查ID和修复措施
					if c.State == check.FAIL {
						fmt.Printf("%s %s\n", c.ID, c.Remediation)
					}
					// 如果检查状态为警告
					if c.State == check.WARN {
						// 如果出现问题导致审核命令未能运行，则打印错误信息
						if c.Reason != "" && c.Type != "manual" {
							fmt.Printf("%s audit test did not run: %s\n", c.ID, c.Reason)
						} else {
							// 否则打印检查ID和修复措施
							fmt.Printf("%s %s\n", c.ID, c.Remediation)
						}
		}
	}
	// 打印空行
	fmt.Println()
}

// 打印摘要，将输出颜色设置为最高严重性
if !noSummary {
	// 调用printSummary函数，传入summary和r.Type参数
	printSummary(summary, string(r.Type))
}

// 定义printSummary函数，传入summary和sectionName参数
func printSummary(summary check.Summary, sectionName string) {
	// 定义变量res为check.State类型
	var res check.State
	// 如果summary中的Fail数量大于0，则res为FAIL
	if summary.Fail > 0 {
		res = check.FAIL
	} 
	// 如果summary中的Warn数量大于0，则res为WARN
	else if summary.Warn > 0 {
		res = check.WARN
	} 
	// 否则
	else {
		res = check.PASS
	}

	colors[res].Printf("== Summary %s ==\n", sectionName)
	fmt.Printf("%d checks PASS\n%d checks FAIL\n%d checks WARN\n%d checks INFO\n\n",
		summary.Pass, summary.Fail, summary.Warn, summary.Info,
	)
}

// loadConfig函数用于加载配置文件，根据kubernetes版本找到正确的配置目录，
// 合并找到的任何特定的config.yaml文件与主配置文件，并返回要使用的基准文件。
func loadConfig(nodetype check.NodeType, benchmarkVersion string) string {
	var file string
	var err error

	switch nodetype {
	case check.MASTER:
		file = masterFile
	case check.NODE:
```
在这个示例中，我们为每个语句添加了注释，解释了它们的作用和功能。
	// 根据节点类型选择相应的文件
	file = nodeFile
	case check.CONTROLPLANE:
		file = controlplaneFile
	case check.ETCD:
		file = etcdFile
	case check.POLICIES:
		file = policiesFile
	case check.MANAGEDSERVICES:
		file = managedservicesFile
	}

	// 获取配置文件的路径
	path, err := getConfigFilePath(benchmarkVersion, file)
	if err != nil {
		// 如果找不到配置文件，则输出错误信息并退出
		exitWithError(fmt.Errorf("can't find %s controls file in %s: %v", nodetype, cfgDir, err))
	}

	// 如果存在版本特定的配置，则合并配置
	mergeConfig(path)

	// 返回配置文件的完整路径
	return filepath.Join(path, file)
// 合并配置文件，根据给定的路径读取配置文件并合并
func mergeConfig(path string) error {
    // 设置配置文件路径
    viper.SetConfigFile(path + "/config.yaml")
    // 合并配置文件
    err := viper.MergeInConfig()
    // 如果合并配置文件出错
    if err != nil {
        // 如果文件不存在
        if os.IsNotExist(err) {
            // 输出日志，提示指定路径下没有特定版本的config.yaml文件
            glog.V(2).Info(fmt.Sprintf("No version-specific config.yaml file in %s", path))
        } else {
            // 返回错误信息，指出无法读取配置文件
            return fmt.Errorf("couldn't read config file %s: %v", path+"/config.yaml", err)
        }
    }
    // 输出日志，提示使用的配置文件路径
    glog.V(1).Info(fmt.Sprintf("Using config file: %s\n", viper.ConfigFileUsed()))
    // 返回空值
    return nil
}

// 将映射转换为基准版本
func mapToBenchmarkVersion(kubeToBenchmarkMap map[string]string, kv string) (string, error) {
    // 保存原始的映射值
    kvOriginal := kv
// 从 kubeToBenchmarkMap 中获取对应版本号的 CIS 版本号，如果找到则返回 true，否则返回 false
cisVersion, found := kubeToBenchmarkMap[kv]
// 输出日志信息，记录映射关系的查询结果
glog.V(2).Info(fmt.Sprintf("mapToBenchmarkVersion for k8sVersion: %q cisVersion: %q found: %t\n", kv, cisVersion, found))
// 当未找到匹配的 CIS 版本号且当前版本号不是默认版本且不为空时，循环向前查找匹配的版本号
for !found && (kv != defaultKubeVersion && !isEmpty(kv)) {
    kv = decrementVersion(kv) // 将版本号递减
    cisVersion, found = kubeToBenchmarkMap[kv] // 重新查询匹配的 CIS 版本号
    glog.V(2).Info(fmt.Sprintf("mapToBenchmarkVersion for k8sVersion: %q cisVersion: %q found: %t\n", kv, cisVersion, found)) // 输出日志信息
}

// 如果最终未找到匹配的 CIS 版本号，则输出警告日志并返回错误
if !found {
    glog.V(1).Info(fmt.Sprintf("mapToBenchmarkVersion unable to find a match for: %q", kvOriginal)) // 输出警告日志
    glog.V(3).Info(fmt.Sprintf("mapToBenchmarkVersion kubeToBenchmarkMap: %#v", kubeToBenchmarkMap)) // 输出详细信息日志
    return "", fmt.Errorf("unable to find a matching Benchmark Version match for kubernetes version: %s", kvOriginal) // 返回错误信息
}

// 返回匹配的 CIS 版本号和 nil 错误
return cisVersion, nil
}

// 加载版本映射关系
func loadVersionMapping(v *viper.Viper) (map[string]string, error) {
    kubeToBenchmarkMap := v.GetStringMapString("version_mapping") // 从配置中获取版本映射关系
    // 如果映射关系为空或者长度为 0，则返回空映射和错误
    if kubeToBenchmarkMap == nil || (len(kubeToBenchmarkMap) == 0) {
		return nil, fmt.Errorf("config file is missing 'version_mapping' section")
	}

	return kubeToBenchmarkMap, nil
}

func loadTargetMapping(v *viper.Viper) (map[string][]string, error) {
	benchmarkVersionToTargetsMap := v.GetStringMapStringSlice("target_mapping")
	if len(benchmarkVersionToTargetsMap) == 0 {
		return nil, fmt.Errorf("config file is missing 'target_mapping' section")
	}

	return benchmarkVersionToTargetsMap, nil
}

func getBenchmarkVersion(kubeVersion, benchmarkVersion string, v *viper.Viper) (bv string, err error) {
	if !isEmpty(kubeVersion) && !isEmpty(benchmarkVersion) {
		return "", fmt.Errorf("It is an error to specify both --version and --benchmark flags")
	}
	if isEmpty(benchmarkVersion) && isEmpty(kubeVersion) {
```

注释：
- 第一个函数返回一个错误，指示配置文件缺少'version_mapping'部分，如果没有错误，则返回kubeToBenchmarkMap。
- 第二个函数加载目标映射，如果配置文件缺少'target_mapping'部分，则返回一个错误，否则返回benchmarkVersionToTargetsMap。
- 第三个函数获取基准版本，如果kubeVersion和benchmarkVersion都不为空，则返回一个错误，指示同时指定--version和--benchmark标志是错误的。如果benchmarkVersion和kubeVersion都为空，则...（代码未完整，无法提供完整的注释）。
// 获取平台名称并根据平台名称获取基准版本
benchmarkVersion = getPlatformBenchmarkVersion(getPlatformName())

// 如果基准版本为空
if isEmpty(benchmarkVersion) {
    // 如果 kubeVersion 为空
    if isEmpty(kubeVersion) {
        // 获取 kubeVersion
        kv, err := getKubeVersion()
        // 如果获取失败，返回错误信息
        if err != nil {
            return "", fmt.Errorf("Version check failed: %s\nAlternatively, you can specify the version with --version", err)
        }
        // 获取基本版本
        kubeVersion = kv.BaseVersion()
    }

    // 加载版本映射
    kubeToBenchmarkMap, err := loadVersionMapping(v)
    // 如果加载失败，返回错误信息
    if err != nil {
        return "", err
    }

    // 将 kubeVersion 映射为基准版本
    benchmarkVersion, err = mapToBenchmarkVersion(kubeToBenchmarkMap, kubeVersion)
    // 如果映射失败，返回错误信息
    if err != nil {
        return "", err
    }
}
// glog.V(2).Info(fmt.Sprintf("Mapped Kubernetes version: %s to Benchmark version: %s", kubeVersion, benchmarkVersion))
// 打印日志，记录映射的Kubernetes版本和Benchmark版本的信息

// glog.V(1).Info(fmt.Sprintf("Kubernetes version: %q to Benchmark version: %q", kubeVersion, benchmarkVersion))
// 打印日志，记录Kubernetes版本和Benchmark版本的信息

// isMaster verify if master components are running on the node.
// 检查当前节点是否运行着主节点组件

// isEtcd verify if etcd components are running on the node.
// 检查当前节点是否运行着etcd组件

// isThisNodeRunning(nodeType check.NodeType) bool
// 检查当前节点是否运行着指定类型的组件，返回布尔值
# 使用 glog.V(3) 输出日志信息，检查当前节点是否运行指定类型的组件
glog.V(3).Infof("Checking if the current node is running %s components", nodeType)
# 从 viper 配置中获取指定节点类型的子配置
nodeTypeConf := viper.Sub(string(nodeType))
# 如果未找到指定节点类型的配置，则输出日志信息并返回 false
if nodeTypeConf == nil {
    glog.V(2).Infof("No config for %s components found", nodeType)
    return false
}
# 调用 getBinariesFunc 函数获取指定节点类型的组件信息
components, err := getBinariesFunc(nodeTypeConf, nodeType)
# 如果获取组件信息失败，则输出日志信息并返回 false
if err != nil {
    glog.V(2).Infof("Failed to find %s binaries: %v", nodeType, err)
    return false
}
# 如果未找到指定节点类型的组件，则输出日志信息并返回 false
if len(components) == 0 {
    glog.V(2).Infof("No %s binaries specified", nodeType)
    return false
}
# 输出日志信息，指示节点正在运行指定类型的组件，并返回 true
glog.V(2).Infof("Node is running %s components", nodeType)
return true
# 选择退出代码
# 遍历控件集合，如果有控件失败，则返回退出代码
func exitCodeSelection(controlsCollection []*check.Controls) int {
	for _, control := range controlsCollection {
		if control.Fail > 0 {
			return exitCode
		}
	}
	# 如果没有控件失败，则返回 0
	return 0
}

# 写入输出
# 对控件集合进行排序，然后根据输出格式写入输出
func writeOutput(controlsCollection []*check.Controls) {
	# 对控件集合进行排序，按照控件ID进行升序排序
	sort.Slice(controlsCollection, func(i, j int) bool {
		iid, _ := strconv.Atoi(controlsCollection[i].ID)
		jid, _ := strconv.Atoi(controlsCollection[j].ID)
		return iid < jid
	})
	# 如果输出格式为 junit，则写入 junit 格式的输出并返回
	if junitFmt {
		writeJunitOutput(controlsCollection)
		return
	}
}
# 如果 jsonFmt 为真，则调用 writeJSONOutput 函数输出结果并返回
	}
	if jsonFmt {
		writeJSONOutput(controlsCollection)
		return
	}
# 如果 pgSQL 为真，则调用 writePgsqlOutput 函数输出结果并返回
	if pgSQL {
		writePgsqlOutput(controlsCollection)
		return
	}
# 如果 aSFF 为真，则调用 writeASFFOutput 函数输出结果并返回
	if aSFF {
		writeASFFOutput(controlsCollection)
		return
	}
# 否则调用 writeStdoutOutput 函数输出结果
	writeStdoutOutput(controlsCollection)
}

# 定义函数 writeJSONOutput，用于将 controlsCollection 转换为 JSON 格式输出
func writeJSONOutput(controlsCollection []*check.Controls) {
	# 定义变量 out 和 err
	var out []byte
	var err error
	# 如果 noTotals 为假
	if !noTotals {
		// 声明一个名为totals的OverallControls类型变量
		var totals check.OverallControls
		// 将controlsCollection赋值给totals的Controls字段
		totals.Controls = controlsCollection
		// 调用getSummaryTotals函数计算总结数据，并赋值给totals的Totals字段
		totals.Totals = getSummaryTotals(controlsCollection)
		// 将totals转换为JSON格式，并赋值给out和err变量
		out, err = json.Marshal(totals)
	} else {
		// 如果controlsCollection为空，则将controlsCollection转换为JSON格式，并赋值给out和err变量
		out, err = json.Marshal(controlsCollection)
	}
	// 如果转换过程中出现错误，则输出错误信息并退出程序
	if err != nil {
		exitWithError(fmt.Errorf("failed to output in JSON format: %v", err))
	}
	// 将out输出到指定的outputFile中
	printOutput(string(out), outputFile)
}

func writeJunitOutput(controlsCollection []*check.Controls) {
	// 遍历controlsCollection，对每个controls调用JUnit方法，并将结果赋值给out和err变量
	for _, controls := range controlsCollection {
		out, err := controls.JUnit()
		// 如果转换过程中出现错误，则输出错误信息并退出程序
		if err != nil {
			exitWithError(fmt.Errorf("failed to output in JUnit format: %v", err))
		}
		// 将out输出到指定的outputFile中
		printOutput(string(out), outputFile)
	}
}

// 将控制集合以Postgresql格式输出
func writePgsqlOutput(controlsCollection []*check.Controls) {
	// 遍历控制集合
	for _, controls := range controlsCollection {
		// 将控制转换为JSON格式
		out, err := controls.JSON()
		// 如果转换出错，则输出错误信息并退出
		if err != nil {
			exitWithError(fmt.Errorf("failed to output in Postgresql format: %v", err))
		}
		// 保存JSON格式的数据到Postgresql
		savePgsql(string(out))
	}
}

// 将控制集合以ASFF格式输出
func writeASFFOutput(controlsCollection []*check.Controls) {
	// 遍历控制集合
	for _, controls := range controlsCollection {
		// 将控制转换为ASFF格式
		out, err := controls.ASFF()
		// 如果转换出错，则输出错误信息并退出
		if err != nil {
			exitWithError(fmt.Errorf("failed to format findings as ASFF: %v", err))
		}
		// 将ASFF格式的数据写入
		if err := writeFinding(out); err != nil {
# 如果输出到 ASFF 失败，则输出错误信息并退出程序
exitWithError(fmt.Errorf("failed to output to ASFF: %v", err))

# 将控制集合输出到标准输出
func writeStdoutOutput(controlsCollection []*check.Controls):
    # 遍历控制集合
    for _, controls := range controlsCollection:
        # 获取控制的摘要信息
        summary := controls.Summary
        # 格式化输出控制和摘要信息
        prettyPrint(controls, summary)
    # 如果不需要输出总计信息，则跳过
    if !noTotals:
        # 输出控制集合的总计信息
        printSummary(getSummaryTotals(controlsCollection), "total")

# 获取控制集合的总计摘要信息
func getSummaryTotals(controlsCollection []*check.Controls) check.Summary:
    # 初始化总计摘要信息
    var totalSummary check.Summary
    # 遍历控制集合
    for _, controls := range controlsCollection:
        # 获取控制的摘要信息
        summary := controls.Summary
        # 计算总计摘要信息中的失败数量
        totalSummary.Fail = totalSummary.Fail + summary.Fail
		# 将总结对象的警告数累加到总体总结对象中
		totalSummary.Warn = totalSummary.Warn + summary.Warn
		# 将总结对象的通过数累加到总体总结对象中
		totalSummary.Pass = totalSummary.Pass + summary.Pass
		# 将总结对象的信息数累加到总体总结对象中
		totalSummary.Info = totalSummary.Info + summary.Info
	}
	# 返回总体总结对象
	return totalSummary
}

# 打印原始输出内容
func printRawOutput(output string) {
	# 遍历输出内容的每一行，并打印出来
	for _, row := range strings.Split(output, "\n") {
		fmt.Println(fmt.Sprintf("\t %s", row))
	}
}

# 将输出内容写入文件
func writeOutputToFile(output string, outputFile string) error {
	# 创建一个文件，并将输出内容写入其中
	file, err := os.Create(outputFile)
	# 如果创建文件过程中出现错误，则返回错误
	if err != nil {
		return err
	}
	# 延迟关闭文件，确保函数执行完毕后关闭文件
	defer file.Close()
// 创建一个新的写入器，用于向文件中写入数据
w := bufio.NewWriter(file)
// 将输出内容写入到文件中
fmt.Fprintln(w, output)
// 刷新缓冲区，将数据写入到文件中
return w.Flush()
}

// 打印输出内容到控制台或者指定的输出文件
func printOutput(output string, outputFile string) {
// 如果没有指定输出文件，则直接打印到控制台
if outputFile == "" {
fmt.Println(output)
} else {
// 否则将输出内容写入到指定的输出文件中
err := writeOutputToFile(output, outputFile)
// 如果写入出现错误，则输出错误信息
if err != nil {
exitWithError(fmt.Errorf("Failed to write to output file %s: %v", outputFile, err))
}
}
}

// validTargets 函数用于确定目标是否适用于给定的基准版本
func validTargets(benchmarkVersion string, targets []string, v *viper.Viper) (bool, error) {
// 加载基准版本到目标的映射关系，并返回映射关系和可能的错误
benchmarkVersionToTargetsMap, err := loadTargetMapping(v)
	// 如果发生错误，返回错误信息
	if err != nil {
		return false, err
	}
	// 根据基准版本查找对应的目标，如果找不到，返回错误信息
	providedTargets, found := benchmarkVersionToTargetsMap[benchmarkVersion]
	if !found {
		return false, fmt.Errorf("No targets configured for %s", benchmarkVersion)
	}

	// 遍历传入的目标列表，检查是否都在配置的目标列表中
	for _, pt := range targets {
		f := false
		// 遍历配置的目标列表，检查传入的目标是否在其中
		for _, t := range providedTargets {
			if pt == strings.ToLower(t) {
				f = true
				break
			}
		}

		// 如果有目标不在配置列表中，返回错误信息
		if !f {
			return false, nil
		}
	}
# 结束 if 语句的代码块
}

# 返回 true 和 nil，表示函数执行成功并且没有错误
return true, nil
```