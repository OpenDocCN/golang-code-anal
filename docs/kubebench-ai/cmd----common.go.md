# `kubebench-aquasecurity\cmd\common.go`

```go
// 版权声明和许可证信息
// 该代码受 Apache 许可证版本 2.0 的保护
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的
// 没有任何形式的担保或条件，无论是明示的还是暗示的
// 请查看许可证以获取特定语言的权限和限制

package cmd

import (
    "bufio"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "os"
    "path/filepath"
    "sort"
    "strconv"
    "strings"

    "github.com/aquasecurity/kube-bench/check"
    "github.com/golang/glog"
    "github.com/spf13/viper"
)

// 根据 FilterOpts 构造一个基于 Predicate 的 NewRunFilter 函数，用于确定是否应该运行测试检查
func NewRunFilter(opts FilterOpts) (check.Predicate, error) {
    // 如果同时使用了 group 选项和 check 选项，则返回错误
    if opts.CheckList != "" && opts.GroupList != "" {
        return nil, fmt.Errorf("group option and check option can't be used together")
    }

    var groupIDs map[string]bool
    // 如果使用了 group 选项，则清理 groupIDs
    if opts.GroupList != "" {
        groupIDs = cleanIDs(opts.GroupList)
    }

    var checkIDs map[string]bool
    // 如果使用了 check 选项，则清理 checkIDs
    if opts.CheckList != "" {
        checkIDs = cleanIDs(opts.CheckList)
    }

    return func(g *check.Group, c *check.Check) bool {
        var test = true
        // 如果 groupIDs 的长度大于 0，则检查是否包含当前 group 的 ID
        if len(groupIDs) > 0 {
            _, ok := groupIDs[g.ID]
            test = test && ok
        }

        // 如果 checkIDs 的长度大于 0，则检查是否包含当前 check 的 ID
        if len(checkIDs) > 0 {
            _, ok := checkIDs[c.ID]
            test = test && ok
        }

        // 检查是否符合得分要求
        test = test && (opts.Scored && c.Scored || opts.Unscored && !c.Scored)

        return test
    }, nil
}

func runChecks(nodetype check.NodeType, testYamlFile string) {
    // 验证在 Cobra 子命令初始化期间是否将配置文件加载到 Viper 中
    if configFileError != nil {
        colorPrint(check.FAIL, fmt.Sprintf("Failed to read config file: %v\n", configFileError))
        os.Exit(1)
    }

    // 读取测试 YAML 文件的内容
    in, err := ioutil.ReadFile(testYamlFile)
    if err != nil {
        exitWithError(fmt.Errorf("error opening %s test file: %v", testYamlFile, err))
    }

    // 输出使用的测试文件信息
    glog.V(1).Info(fmt.Sprintf("Using test file: %s\n", testYamlFile))

    // 获取此部分测试的 viper 配置
    typeConf := viper.Sub(string(nodetype))
    if typeConf == nil {
        colorPrint(check.FAIL, fmt.Sprintf("No config settings for %s\n", string(nodetype)))
        os.Exit(1)
    }

    // 获取此部分测试所需的可执行文件集合
    binmap, err := getBinaries(typeConf, nodetype)

    // 检查此部分所需的可执行文件是否正在运行
    if err != nil {
        glog.V(1).Info(fmt.Sprintf("failed to get a set of executables needed for tests: %v", err))
    }

    // 获取配置文件、服务文件、kubeconfig 文件和 ca 文件的映射
    confmap := getFiles(typeConf, "config")
    svcmap := getFiles(typeConf, "service")
    kubeconfmap := getFiles(typeConf, "kubeconfig")
    cafilemap := getFiles(typeConf, "ca")

    // 变量替换。替换控制文件中所有变量的出现
    s := string(in)
    s, binSubs := makeSubstitutions(s, "bin", binmap)
    s, _ = makeSubstitutions(s, "conf", confmap)
    s, _ = makeSubstitutions(s, "svc", svcmap)
    s, _ = makeSubstitutions(s, "kubeconfig", kubeconfmap)
    s, _ = makeSubstitutions(s, "cafile", cafilemap)

    // 创建新的控制对象
    controls, err := check.NewControls(nodetype, []byte(s))
    if err != nil {
        exitWithError(fmt.Errorf("error setting up %s controls: %v", nodetype, err))
    }

    // 创建新的运行器
    runner := check.NewRunner()
    // 创建新的运行过滤器
    filter, err := NewRunFilter(filterOpts)
    if err != nil {
        exitWithError(fmt.Errorf("error setting up run filter: %v", err))
    }

    // 生成默认的环境审计
    generateDefaultEnvAudit(controls, binSubs)
    # 运行检查函数，传入运行器、过滤器和解析后的跳过ID列表
    controls.RunChecks(runner, filter, parseSkipIds(skipIds))
    # 将运行检查后的结果添加到控制集合中
    controlsCollection = append(controlsCollection, controls)
// 生成默认的环境审计，根据控制组和二进制子集
func generateDefaultEnvAudit(controls *check.Controls, binSubs []string){
    // 遍历控制组
    for _, group := range controls.Groups {
        // 遍历每个检查项
        for _, checkItem := range group.Checks {
            // 如果检查项有测试并且未禁用环境测试
            if checkItem.Tests != nil && !checkItem.DisableEnvTesting {
                // 遍历每个测试项
                for _, test := range checkItem.Tests.TestItems {
                    // 如果测试项有环境变量并且未设置审计环境
                    if test.Env != "" && checkItem.AuditEnv == "" {
                        // 初始化二进制路径为空
                        binPath := ""
                        // 如果二进制子集长度为1，则设置二进制路径为第一个元素
                        if len(binSubs) == 1 {
                            binPath = binSubs[0]
                        } else {
                            // 否则打印警告信息
                            fmt.Printf("AuditEnv not explicit for check (%s), where bin path cannot be determined\n", checkItem.ID)
                        }
                        // 如果测试项有环境变量并且未设置审计环境
                        if test.Env != "" && checkItem.AuditEnv == "" {
                            // 设置审计环境为特定命令
                            checkItem.AuditEnv = fmt.Sprintf("cat \"/proc/$(/bin/ps -C %s -o pid= | tr -d ' ')/environ\" | tr '\\0' '\\n'", binPath)
                        }
                    }
                }
            }
        }
    }
}

// 解析跳过的 ID，返回 ID 到布尔值的映射
func parseSkipIds(skipIds string) map[string]bool {
    // 初始化空的跳过 ID 映射
    var skipIdMap = make(map[string]bool, 0)
    // 如果跳过 ID 非空
    if skipIds != "" {
        // 遍历逗号分隔的跳过 ID
        for _, id := range strings.Split(skipIds, ",") {
            // 去除空格并设置为 true
            skipIdMap[strings.Trim(id, " ")] = true
        }
    }
    // 返回跳过 ID 映射
    return skipIdMap
}

// colorPrint 以特定颜色输出状态和消息字符串
func colorPrint(state check.State, s string) {
    // 使用特定状态的颜色输出消息
    colors[state].Printf("[%s] ", state)
    fmt.Printf("%s", s)
}

// prettyPrint 以人类可读的格式输出结果到标准输出
func prettyPrint(r *check.Controls, summary check.Summary) {
    // 输出检查结果
    // 如果没有结果，则不打印任何内容
    if !noResults {
        // 打印检查结果的 ID 和文本
        colorPrint(check.INFO, fmt.Sprintf("%s %s\n", r.ID, r.Text))
        // 遍历结果的分组
        for _, g := range r.Groups {
            // 打印分组的 ID 和文本
            colorPrint(check.INFO, fmt.Sprintf("%s %s\n", g.ID, g.Text))
            // 遍历分组的检查项
            for _, c := range g.Checks {
                // 打印检查项的状态、ID 和文本
                colorPrint(c.State, fmt.Sprintf("%s %s\n", c.ID, c.Text)

                // 如果包括测试输出，并且检查状态为失败且实际值长度大于0，则打印原始输出
                if includeTestOutput && c.State == check.FAIL && len(c.ActualValue) > 0 {
                    printRawOutput(c.ActualValue)
                }
            }
        }

        // 打印空行
        fmt.Println()
    }

    // 打印修复措施
    if !noRemediations {
        // 如果总体失败数大于0或警告数大于0，则打印修复措施
        if summary.Fail > 0 || summary.Warn > 0 {
            colors[check.WARN].Printf("== Remediations %s ==\n", r.Type)
            // 遍历分组和检查项，打印失败状态的修复措施
            for _, g := range r.Groups {
                for _, c := range g.Checks {
                    if c.State == check.FAIL {
                        fmt.Printf("%s %s\n", c.ID, c.Remediation)
                    }
                    // 如果状态为警告，则根据条件打印错误或修复措施
                    if c.State == check.WARN {
                        if c.Reason != "" && c.Type != "manual" {
                            fmt.Printf("%s audit test did not run: %s\n", c.ID, c.Reason)
                        } else {
                            fmt.Printf("%s %s\n", c.ID, c.Remediation)
                        }
                    }
                }
            }
            // 打印空行
            fmt.Println()
        }
    }

    // 打印总结，根据最高严重性设置输出颜色
    if !noSummary {
        printSummary(summary, string(r.Type))
    }
// 打印检查摘要信息
func printSummary(summary check.Summary, sectionName string) {
    var res check.State
    // 根据检查摘要的结果数量确定最终状态
    if summary.Fail > 0 {
        res = check.FAIL
    } else if summary.Warn > 0 {
        res = check.WARN
    } else {
        res = check.PASS
    }

    // 使用对应状态的颜色打印摘要信息
    colors[res].Printf("== Summary %s ==\n", sectionName)
    // 打印通过、失败、警告和信息检查的数量
    fmt.Printf("%d checks PASS\n%d checks FAIL\n%d checks WARN\n%d checks INFO\n\n",
        summary.Pass, summary.Fail, summary.Warn, summary.Info,
    )
}

// loadConfig 根据 Kubernetes 版本找到正确的配置目录，
// 将找到的任何特定的 config.yaml 文件与主配置文件合并，
// 并返回要使用的基准文件。
func loadConfig(nodetype check.NodeType, benchmarkVersion string) string {
    var file string
    var err error

    switch nodetype {
    case check.MASTER:
        file = masterFile
    case check.NODE:
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
        // 如果找不到配置文件，则输出错误并退出程序
        exitWithError(fmt.Errorf("can't find %s controls file in %s: %v", nodetype, cfgDir, err))
    }

    // 合并特定版本的配置文件（如果有）
    mergeConfig(path)

    return filepath.Join(path, file)
}

// mergeConfig 合并指定路径下的配置文件
func mergeConfig(path string) error {
    // 设置要读取的配置文件路径
    viper.SetConfigFile(path + "/config.yaml")
    // 合并配置文件
    err := viper.MergeInConfig()
    if err != nil {
        if os.IsNotExist(err) {
            // 如果文件不存在，则输出信息
            glog.V(2).Info(fmt.Sprintf("No version-specific config.yaml file in %s", path))
        } else {
            // 如果出现其他错误，则返回错误信息
            return fmt.Errorf("couldn't read config file %s: %v", path+"/config.yaml", err)
        }
    }

    // 输出使用的配置文件路径
    glog.V(1).Info(fmt.Sprintf("Using config file: %s\n", viper.ConfigFileUsed()))

    return nil
}

// mapToBenchmarkVersion 将 Kubernetes 版本映射到基准版本
func mapToBenchmarkVersion(kubeToBenchmarkMap map[string]string, kv string) (string, error) {
    # 将原始的版本号保存到变量 kvOriginal 中
    kvOriginal := kv
    # 在 kubeToBenchmarkMap 中查找当前版本号对应的 CIS 版本号
    cisVersion, found := kubeToBenchmarkMap[kv]
    # 输出日志，记录查找结果
    glog.V(2).Info(fmt.Sprintf("mapToBenchmarkVersion for k8sVersion: %q cisVersion: %q found: %t\n", kv, cisVersion, found))
    # 当未找到匹配的 CIS 版本号且当前版本号不是默认版本且不为空时，循环执行以下操作
    for !found && (kv != defaultKubeVersion && !isEmpty(kv)) {
        # 将当前版本号递减
        kv = decrementVersion(kv)
        # 再次在 kubeToBenchmarkMap 中查找当前版本号对应的 CIS 版本号
        cisVersion, found = kubeToBenchmarkMap[kv]
        # 输出日志，记录查找结果
        glog.V(2).Info(fmt.Sprintf("mapToBenchmarkVersion for k8sVersion: %q cisVersion: %q found: %t\n", kv, cisVersion, found))
    }

    # 如果最终未找到匹配的 CIS 版本号
    if !found {
        # 输出日志，记录未找到匹配的版本号
        glog.V(1).Info(fmt.Sprintf("mapToBenchmarkVersion unable to find a match for: %q", kvOriginal))
        # 输出日志，记录当前 kubeToBenchmarkMap 的内容
        glog.V(3).Info(fmt.Sprintf("mapToBenchmarkVersion kubeToBenchmarkMap: %#v", kubeToBenchmarkMap))
        # 返回错误信息，说明未找到匹配的 CIS 版本号
        return "", fmt.Errorf("unable to find a matching Benchmark Version match for kubernetes version: %s", kvOriginal)
    }

    # 返回找到的 CIS 版本号
    return cisVersion, nil
// 从配置文件中加载版本映射关系，返回映射关系的map和可能的错误
func loadVersionMapping(v *viper.Viper) (map[string]string, error) {
    // 从配置文件中获取版本映射关系
    kubeToBenchmarkMap := v.GetStringMapString("version_mapping")
    // 如果版本映射关系为空或长度为0，则返回错误
    if kubeToBenchmarkMap == nil || (len(kubeToBenchmarkMap) == 0) {
        return nil, fmt.Errorf("config file is missing 'version_mapping' section")
    }

    return kubeToBenchmarkMap, nil
}

// 从配置文件中加载目标映射关系，返回映射关系的map和可能的错误
func loadTargetMapping(v *viper.Viper) (map[string][]string, error) {
    // 从配置文件中获取目标映射关系
    benchmarkVersionToTargetsMap := v.GetStringMapStringSlice("target_mapping")
    // 如果目标映射关系长度为0，则返回错误
    if len(benchmarkVersionToTargetsMap) == 0 {
        return nil, fmt.Errorf("config file is missing 'target_mapping' section")
    }

    return benchmarkVersionToTargetsMap, nil
}

// 获取基准版本，根据输入的kubeVersion和benchmarkVersion，以及配置文件，返回基准版本和可能的错误
func getBenchmarkVersion(kubeVersion, benchmarkVersion string, v *viper.Viper) (bv string, err error) {
    // 如果kubeVersion和benchmarkVersion都不为空，则返回错误
    if !isEmpty(kubeVersion) && !isEmpty(benchmarkVersion) {
        return "", fmt.Errorf("It is an error to specify both --version and --benchmark flags")
    }
    // 如果benchmarkVersion为空且kubeVersion为空，则根据平台获取基准版本
    if isEmpty(benchmarkVersion) && isEmpty(kubeVersion) {
        benchmarkVersion = getPlatformBenchmarkVersion(getPlatformName())
    }

    // 如果benchmarkVersion为空
    if isEmpty(benchmarkVersion) {
        // 如果kubeVersion为空，则获取kubeVersion
        if isEmpty(kubeVersion) {
            kv, err := getKubeVersion()
            if err != nil {
                return "", fmt.Errorf("Version check failed: %s\nAlternatively, you can specify the version with --version", err)
            }
            kubeVersion = kv.BaseVersion()
        }

        // 从配置文件中加载版本映射关系
        kubeToBenchmarkMap, err := loadVersionMapping(v)
        if err != nil {
            return "", err
        }

        // 根据kubeVersion映射到benchmarkVersion
        benchmarkVersion, err = mapToBenchmarkVersion(kubeToBenchmarkMap, kubeVersion)
        if err != nil {
            return "", err
        }

        // 记录日志
        glog.V(2).Info(fmt.Sprintf("Mapped Kubernetes version: %s to Benchmark version: %s", kubeVersion, benchmarkVersion))
    }

    // 记录日志
    glog.V(1).Info(fmt.Sprintf("Kubernetes version: %q to Benchmark version: %q", kubeVersion, benchmarkVersion))
    return benchmarkVersion, nil
}
// isMaster 验证节点上是否运行主节点组件。
func isMaster() bool {
    return isThisNodeRunning(check.MASTER)
}

// isEtcd 验证节点上是否运行 etcd 组件。
func isEtcd() bool {
    return isThisNodeRunning(check.ETCD)
}

// isThisNodeRunning 验证节点上是否运行指定类型的组件。
func isThisNodeRunning(nodeType check.NodeType) bool {
    glog.V(3).Infof("Checking if the current node is running %s components", nodeType)
    // 获取指定类型组件的配置
    nodeTypeConf := viper.Sub(string(nodeType))
    if nodeTypeConf == nil {
        glog.V(2).Infof("No config for %s components found", nodeType)
        return false
    }

    // 获取指定类型组件的二进制文件
    components, err := getBinariesFunc(nodeTypeConf, nodeType)
    if err != nil {
        glog.V(2).Infof("Failed to find %s binaries: %v", nodeType, err)
        return false
    }
    if len(components) == 0 {
        glog.V(2).Infof("No %s binaries specified", nodeType)
        return false
    }

    glog.V(2).Infof("Node is running %s components", nodeType)
    return true
}

// exitCodeSelection 选择退出代码
func exitCodeSelection(controlsCollection []*check.Controls) int {
    for _, control := range controlsCollection {
        if control.Fail > 0 {
            return exitCode
        }
    }

    return 0
}

// writeOutput 写入输出
func writeOutput(controlsCollection []*check.Controls) {
    // 按照控件 ID 对控件集合进行排序
    sort.Slice(controlsCollection, func(i, j int) bool {
        iid, _ := strconv.Atoi(controlsCollection[i].ID)
        jid, _ := strconv.Atoi(controlsCollection[j].ID)
        return iid < jid
    })
    // 根据输出格式写入不同的输出
    if junitFmt {
        writeJunitOutput(controlsCollection)
        return
    }
    if jsonFmt {
        writeJSONOutput(controlsCollection)
        return
    }
    if pgSQL {
        writePgsqlOutput(controlsCollection)
        return
    }
    if aSFF {
        writeASFFOutput(controlsCollection)
        return
    }
    writeStdoutOutput(controlsCollection)
}

// writeJSONOutput 写入 JSON 格式的输出
func writeJSONOutput(controlsCollection []*check.Controls) {
    var out []byte
    var err error
    // ...
}
    # 如果不是禁用总计功能
    if !noTotals:
        # 创建一个名为totals的OverallControls结构体变量
        var totals check.OverallControls
        # 将controlsCollection赋值给totals的Controls字段
        totals.Controls = controlsCollection
        # 调用getSummaryTotals函数计算总计并赋值给totals的Totals字段
        totals.Totals = getSummaryTotals(controlsCollection)
        # 将totals转换为JSON格式并赋值给out，err
        out, err = json.Marshal(totals)
    # 如果禁用了总计功能
    else:
        # 将controlsCollection转换为JSON格式并赋值给out，err
        out, err = json.Marshal(controlsCollection)
    # 如果出现错误
    if err != nil:
        # 输出错误信息并退出程序
        exitWithError(fmt.Errorf("failed to output in JSON format: %v", err))
    # 打印输出到指定的输出文件
    printOutput(string(out), outputFile)
# 将控件集合以 JUnit 格式输出
func writeJunitOutput(controlsCollection []*check.Controls) {
    # 遍历控件集合
    for _, controls := range controlsCollection {
        # 调用控件的 JUnit 方法，获取输出和可能的错误
        out, err := controls.JUnit()
        # 如果有错误，则输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("failed to output in JUnit format: %v", err))
        }
        # 打印输出到指定的输出文件
        printOutput(string(out), outputFile)
    }
}

# 将控件集合以 Postgresql 格式输出
func writePgsqlOutput(controlsCollection []*check.Controls) {
    # 遍历控件集合
    for _, controls := range controlsCollection {
        # 调用控件的 JSON 方法，获取输出和可能的错误
        out, err := controls.JSON()
        # 如果有错误，则输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("failed to output in Postgresql format: %v", err))
        }
        # 保存输出到 Postgresql 数据库
        savePgsql(string(out))
    }
}

# 将控件集合格式化为 ASFF 输出
func writeASFFOutput(controlsCollection []*check.Controls) {
    # 遍历控件集合
    for _, controls := range controlsCollection {
        # 调用控件的 ASFF 方法，获取输出和可能的错误
        out, err := controls.ASFF()
        # 如果有错误，则输出错误信息并退出
        if err != nil {
            exitWithError(fmt.Errorf("failed to format findings as ASFF: %v", err))
        }
        # 将输出写入 ASFF
        if err := writeFinding(out); err != nil {
            exitWithError(fmt.Errorf("failed to output to ASFF: %v", err))
        }
    }
}

# 将控件集合以标准输出格式输出
func writeStdoutOutput(controlsCollection []*check.Controls) {
    # 遍历控件集合
    for _, controls := range controlsCollection {
        # 获取控件的摘要信息
        summary := controls.Summary
        # 以漂亮的格式打印控件和摘要信息
        prettyPrint(controls, summary)
    }
    # 如果不是无总计模式，则打印总计摘要信息
    if !noTotals {
        printSummary(getSummaryTotals(controlsCollection), "total")
    }
}

# 获取控件集合的总计摘要信息
func getSummaryTotals(controlsCollection []*check.Controls) check.Summary {
    # 初始化总计摘要信息
    var totalSummary check.Summary
    # 遍历控件集合
    for _, controls := range controlsCollection {
        # 获取控件的摘要信息
        summary := controls.Summary
        # 累加每个控件的失败、警告、通过和信息数量到总计摘要信息中
        totalSummary.Fail = totalSummary.Fail + summary.Fail
        totalSummary.Warn = totalSummary.Warn + summary.Warn
        totalSummary.Pass = totalSummary.Pass + summary.Pass
        totalSummary.Info = totalSummary.Info + summary.Info
    }
    # 返回总计摘要信息
    return totalSummary
}

# 打印原始输出
func printRawOutput(output string) {
    # 遍历输出的每一行
    for _, row := range strings.Split(output, "\n") {
        # 格式化并打印每一行
        fmt.Println(fmt.Sprintf("\t %s", row))
    }
}
// 将输出写入到指定的文件中
func writeOutputToFile(output string, outputFile string) error {
    // 创建或截断指定文件，并返回一个文件对象
    file, err := os.Create(outputFile)
    if err != nil {
        return err
    }
    // 延迟关闭文件
    defer file.Close()

    // 创建文件的缓冲写入器
    w := bufio.NewWriter(file)
    // 将输出写入到文件中
    fmt.Fprintln(w, output)
    // 刷新缓冲写入器
    return w.Flush()
}

// 打印输出到控制台或写入到文件
func printOutput(output string, outputFile string) {
    // 如果未指定输出文件，则打印到控制台
    if outputFile == "" {
        fmt.Println(output)
    } else {
        // 否则将输出写入到指定的文件中
        err := writeOutputToFile(output, outputFile)
        if err != nil {
            // 如果写入文件失败，则输出错误信息并退出程序
            exitWithError(fmt.Errorf("Failed to write to output file %s: %v", outputFile, err))
        }
    }
}

// 验证目标是否合法
// 根据 benchmarkVersion 和 targets 判断目标是否合法
func validTargets(benchmarkVersion string, targets []string, v *viper.Viper) (bool, error) {
    // 加载目标映射
    benchmarkVersionToTargetsMap, err := loadTargetMapping(v)
    if err != nil {
        return false, err
    }
    // 获取指定 benchmarkVersion 对应的目标列表
    providedTargets, found := benchmarkVersionToTargetsMap[benchmarkVersion]
    if !found {
        return false, fmt.Errorf("No targets configured for %s", benchmarkVersion)
    }

    // 遍历传入的 targets，判断是否在指定的目标列表中
    for _, pt := range targets {
        f := false
        for _, t := range providedTargets {
            if pt == strings.ToLower(t) {
                f = true
                break
            }
        }

        if !f {
            return false, nil
        }
    }

    return true, nil
}
```