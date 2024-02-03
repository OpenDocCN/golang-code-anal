# `kubebench-aquasecurity\cmd\util.go`

```go
package cmd

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化包
    "os"  // 导入操作系统功能包
    "os/exec"  // 导入执行外部命令包
    "path/filepath"  // 导入文件路径包
    "regexp"  // 导入正则表达式包
    "strconv"  // 导入字符串转换包
    "strings"  // 导入字符串处理包

    "github.com/aquasecurity/kube-bench/check"  // 导入自定义包
    "github.com/fatih/color"  // 导入颜色包
    "github.com/golang/glog"  // 导入日志包
    "github.com/spf13/viper"  // 导入配置管理包
)

var (
    // Print colors
    colors = map[check.State]*color.Color{  // 定义颜色映射
        check.PASS: color.New(color.FgGreen),  // PASS 状态对应绿色
        check.FAIL: color.New(color.FgRed),  // FAIL 状态对应红色
        check.WARN: color.New(color.FgYellow),  // WARN 状态对应黄色
        check.INFO: color.New(color.FgBlue),  // INFO 状态对应蓝色
    }
)

var psFunc func(string) string  // 定义函数类型变量
var statFunc func(string) (os.FileInfo, error)  // 定义函数类型变量
var getBinariesFunc func(*viper.Viper, check.NodeType) (map[string]string, error)  // 定义函数类型变量
var TypeMap = map[string][]string{  // 定义字符串数组映射
    "ca":         []string{"cafile", "defaultcafile"},  // ca 对应的文件名数组
    "kubeconfig": []string{"kubeconfig", "defaultkubeconfig"},  // kubeconfig 对应的文件名数组
    "service":    []string{"svc", "defaultsvc"},  // service 对应的文件名数组
    "config":     []string{"confs", "defaultconf"},  // config 对应的文件名数组
}

func init() {
    psFunc = ps  // 初始化函数类型变量
    statFunc = os.Stat  // 初始化函数类型变量
    getBinariesFunc = getBinaries  // 初始化函数类型变量
}

func exitWithError(err error) {
    fmt.Fprintf(os.Stderr, "\n%v\n", err)  // 输出错误信息到标准错误
    // flush before exit non-zero
    glog.Flush()  // 刷新日志
    os.Exit(1)  // 退出程序，返回非零状态
}

func cleanIDs(list string) map[string]bool {
    list = strings.Trim(list, ",")  // 去除字符串两端的逗号
    ids := strings.Split(list, ",")  // 以逗号分割字符串为数组

    set := make(map[string]bool)  // 创建字符串到布尔值的映射

    for _, id := range ids {  // 遍历数组
        id = strings.Trim(id, " ")  // 去除字符串两端的空格
        set[id] = true  // 设置映射值为 true
    }

    return set  // 返回映射
}

// ps execs out to the ps command; it's separated into a function so we can write tests
func ps(proc string) string {
    // TODO: truncate proc to 15 chars  // 截断 proc 字符串为 15 个字符
    // See https://github.com/aquasecurity/kube-bench/issues/328#issuecomment-506813344
    glog.V(2).Info(fmt.Sprintf("ps - proc: %q", proc))  // 记录日志信息
    cmd := exec.Command("/bin/ps", "-C", proc, "-o", "cmd", "--no-headers")  // 创建执行 ps 命令的命令对象
    out, err := cmd.Output()  // 执行命令并获取输出
    if err != nil {  // 如果执行出错
        glog.V(2).Info(fmt.Errorf("%s: %s", cmd.Args, err))  // 记录错误信息到日志
    }
}
    # 使用 glog.V(2) 设置日志级别为2，然后使用 Info 方法记录日志，日志内容为格式化后的字符串 "ps - returning: %q"，其中 %q 会被替换为 out 的字符串形式
    glog.V(2).Info(fmt.Sprintf("ps - returning: %q", string(out)))
    # 返回 out 的字符串形式
    return string(out)
// getBinaries函数用于查找一组候选可执行文件中正在运行的文件，如果有必需的可执行文件没有在运行，则返回错误。
func getBinaries(v *viper.Viper, nodetype check.NodeType) (map[string]string, error) {
    // 创建一个空的可执行文件映射
    binmap := make(map[string]string)

    // 遍历配置文件中的组件列表
    for _, component := range v.GetStringSlice("components") {
        // 获取组件的子配置
        s := v.Sub(component)
        if s == nil {
            continue
        }

        // 检查组件是否是可选的
        optional := s.GetBool("optional")
        // 获取组件的可执行文件列表
        bins := s.GetStringSlice("bins")
        if len(bins) > 0 {
            // 查找正在运行的可执行文件
            bin, err := findExecutable(bins)
            if err != nil && !optional {
                // 如果找不到必需的可执行文件，则返回错误
                glog.V(1).Info(buildComponentMissingErrorMessage(nodetype, component, bins))
                return nil, fmt.Errorf("unable to detect running programs for component %q", component)
            }

            // 如果找不到正在运行的可执行文件，则使用组件的名称作为默认值
            if bin == "" {
                bin = component
                glog.V(2).Info(fmt.Sprintf("Component %s not running", component))
            } else {
                glog.V(2).Info(fmt.Sprintf("Component %s uses running binary %s", component, bin))
            }
            // 将组件和对应的可执行文件添加到映射中
            binmap[component] = bin
        }
    }

    return binmap, nil
}

// getConfigFilePath函数用于定位应该用于CIS版本的配置文件
func getConfigFilePath(benchmarkVersion string, filename string) (path string, err error) {
    glog.V(2).Info(fmt.Sprintf("Looking for config specific CIS version %q", benchmarkVersion))

    // 拼接配置文件路径
    path = filepath.Join(cfgDir, benchmarkVersion)
    file := filepath.Join(path, string(filename))
    glog.V(2).Info(fmt.Sprintf("Looking for file: %s", file))

    // 检查配置文件是否存在
    if _, err := os.Stat(file); err != nil {
        glog.V(2).Infof("error accessing config file: %q error: %v\n", file, err)
        return "", fmt.Errorf("no test files found <= benchmark version: %s", benchmarkVersion)
    }

    return path, nil
}
// getYamlFilesFromDir 从指定目录返回一个 yaml 文件列表，忽略 config.yaml
func getYamlFilesFromDir(path string) (names []string, err error) {
    // 遍历指定目录下的文件和子目录
    err = filepath.Walk(path, func(path string, info os.FileInfo, err error) error {
        // 如果出现错误，返回错误信息
        if err != nil {
            return err
        }
        // 获取文件名
        _, name := filepath.Split(path)
        // 如果文件名不为空且不是 config.yaml 且文件扩展名为 .yaml，则将文件路径添加到列表中
        if name != "" && name != "config.yaml" && filepath.Ext(name) == ".yaml" {
            names = append(names, path)
        }
        // 返回 nil 表示遍历继续
        return nil
    })
    // 返回文件列表和可能出现的错误
    return names, err
}

// decrementVersion 减少版本号
// 即使在我们不提供测试文件的版本中，我们也希望逐个递减版本号，以防有人想为该版本指定自己的测试文件
func decrementVersion(version string) string {
    // 按 . 分割版本号
    split := strings.Split(version, ".")
    // 如果分割后的长度小于 2，返回空字符串
    if len(split) < 2 {
        return ""
    }
    // 将次版本号转换为整数
    minor, err := strconv.Atoi(split[1])
    // 如果转换出错，返回空字符串
    if err != nil {
        return ""
    }
    // 如果次版本号小于等于 1，返回空字符串
    if minor <= 1 {
        return ""
    }
    // 将次版本号减一
    split[1] = strconv.Itoa(minor - 1)
    // 将分割后的版本号重新拼接成字符串并返回
    return strings.Join(split, ".")
}

// getFiles 找出候选文件集中存在的文件
func getFiles(v *viper.Viper, fileType string) map[string]string {
    // 创建一个文件路径到文件内容的映射
    filemap := make(map[string]string)
    // 获取文件类型的主选项和默认选项
    mainOpt := TypeMap[fileType][0]
    defaultOpt := TypeMap[fileType][1]
    # 遍历配置文件中的组件列表
    for _, component := range v.GetStringSlice("components") {
        # 获取组件对应的子配置
        s := v.Sub(component)
        # 如果子配置为空，跳过当前组件
        if s == nil {
            continue
        }

        # 查找候选文件是否存在
        file := findConfigFile(s.GetStringSlice(mainOpt))
        # 如果文件不存在
        if file == "" {
            # 如果设置了默认文件名选项
            if s.IsSet(defaultOpt) {
                # 使用默认文件名作为文件
                file = s.GetString(defaultOpt)
                glog.V(2).Info(fmt.Sprintf("Using default %s file name '%s' for component %s", fileType, file, component))
            } else {
                # 使用组件名作为文件名
                glog.V(2).Info(fmt.Sprintf("Missing %s file for %s", fileType, component))
                file = component
            }
        } else {
            # 使用找到的文件名
            glog.V(2).Info(fmt.Sprintf("Component %s uses %s file '%s'", component, fileType, file))
        }

        # 将组件名和文件名添加到文件映射中
        filemap[component] = file
    }

    # 返回文件映射
    return filemap
// verifyBin检查指定的二进制文件是否正在运行
func verifyBin(bin string) bool {

    // 去除任何引号
    bin = strings.Trim(bin, "'\"")

    // bin可能由多个单词组成
    // 我们将使用第一个单词搜索运行中的进程，然后检查整个进程是否包含在结果中
    proc := strings.Fields(bin)[0]
    out := psFunc(proc)

    // ps输出可能包含多行
    // 二进制文件需要是ps输出的第一个单词，除非它可能被路径所包围
    // 例如，/usr/bin/kubelet是kubelet的匹配
    // 但apiserver不是kube-apiserver的匹配
    reFirstWord := regexp.MustCompile(`^(\S*\/)*` + bin)
    lines := strings.Split(out, "\n")
    for _, l := range lines {
        glog.V(3).Info(fmt.Sprintf("reFirstWord.Match(%s)", l))
        if reFirstWord.Match([]byte(l)) {
            return true
        }
    }

    return false
}

// findConfigFile遍历可能的配置文件列表，并找到第一个存在的文件
func findConfigFile(candidates []string) string {
    for _, c := range candidates {
        _, err := statFunc(c)
        if err == nil {
            return c
        }
        if !os.IsNotExist(err) {
            exitWithError(fmt.Errorf("error looking for file %s: %v", c, err))
        }
    }

    return ""
}

// findExecutable遍历可能的可执行文件名称列表，并找到第一个正在运行的文件
func findExecutable(candidates []string) (string, error) {
    for _, c := range candidates {
        if verifyBin(c) {
            return c, nil
        }
        glog.V(1).Info(fmt.Sprintf("executable '%s' not running", c))
    }

    return "", fmt.Errorf("no candidates running")
}

// multiWordReplace用sub替换s中的subname，如果sub包含多个单词，则在sub周围添加引号
func multiWordReplace(s string, subname string, sub string) string {
    f := strings.Fields(sub)
    if len(f) > 1 {
        sub = "'" + sub + "'"
    }

    return strings.Replace(s, subname, sub, -1)
}
// 定义一个包含错误信息的常量字符串，用于提示找不到 kubectl 或 kubelet 程序的错误信息
const missingKubectlKubeletMessage = `
Unable to find the programs kubectl or kubelet in the PATH.
These programs are used to determine which version of Kubernetes is running.
Make sure the /usr/local/mount-from-host/bin directory is mapped to the container,
either in the job.yaml file, or Docker command.

For job.yaml:
...
- name: usr-bin
  mountPath: /usr/local/mount-from-host/bin
...

For docker command:
   docker -v $(which kubectl):/usr/local/mount-from-host/bin/kubectl ....

Alternatively, you can specify the version with --version
   kube-bench --version <VERSION> ...
`

// 定义一个函数用于获取 Kubernetes 的版本信息
func getKubeVersion() (*KubeVersion, error) {

    // 通过 REST API 获取 Kubernetes 的版本信息
    if k8sVer, err := getKubeVersionFromRESTAPI(); err == nil {
        glog.V(2).Info(fmt.Sprintf("Kubernetes REST API Reported version: %s", k8sVer))
        return k8sVer, nil
    }

    // 检查是否能找到 kubectl 程序
    _, err := exec.LookPath("kubectl")

    if err != nil {
        // 如果找不到 kubectl 程序，则检查是否能找到 kubelet 程序
        _, err = exec.LookPath("kubelet")
        if err != nil {
            // 如果找不到 kubelet 程序，则在文件系统中搜索 kubelet 可执行文件并运行第一个匹配项以获取 Kubernetes 版本
            cmd := exec.Command("/bin/sh", "-c", "`find / -type f -executable -name kubelet 2>/dev/null | grep -m1 .` --version")
            out, err := cmd.CombinedOutput()
            if err == nil {
                return getVersionFromKubeletOutput(string(out)), nil
            }

            // 输出找不到 kubectl 或 kubelet 程序的错误信息
            glog.Warning(missingKubectlKubeletMessage)
            return nil, fmt.Errorf("unable to find the programs kubectl or kubelet in the PATH")
        }
        // 从 kubelet 获取 Kubernetes 版本信息
        return getKubeVersionFromKubelet(), nil
    }

    // 从 kubectl 获取 Kubernetes 版本信息
    return getKubeVersionFromKubectl(), nil
}

// 从 kubectl 获取 Kubernetes 版本信息
func getKubeVersionFromKubectl() *KubeVersion {
    cmd := exec.Command("kubectl", "version", "-o", "json")
    out, err := cmd.CombinedOutput()
    if err != nil {
        glog.V(2).Info(err)
    }

    return getVersionFromKubectlOutput(string(out))
}

// 从 kubelet 获取 Kubernetes 版本信息的函数
func getKubeVersionFromKubelet() *KubeVersion {
    // 创建一个执行命令的对象，命令为 "kubelet --version"
    cmd := exec.Command("kubelet", "--version")
    // 执行命令并获取输出和错误信息
    out, err := cmd.CombinedOutput()

    // 如果有错误发生，则记录错误信息
    if err != nil {
        glog.V(2).Info(err)
    }

    // 从 kubelet 输出中获取版本信息并返回
    return getVersionFromKubeletOutput(string(out))
# 从 kubectl 输出中获取版本信息并返回 KubeVersion 对象
func getVersionFromKubectlOutput(s string) *KubeVersion {
    # 打印详细日志信息
    glog.V(2).Info(s)
    # 定义版本结果结构体
    type versionResult struct {
        ServerVersion VersionResponse
    }
    # 创建版本结果结构体对象
    vrObj := &versionResult{}
    # 将字符串 s 反序列化为版本结果结构体对象
    if err := json.Unmarshal([]byte(s), vrObj); err != nil {
        # 打印错误信息
        glog.V(2).Info(err)
        # 如果字符串 s 包含特定信息，则输出警告信息
        if strings.Contains(s, "The connection to the server") {
            msg := fmt.Sprintf(`Warning: Kubernetes version was not auto-detected because kubectl could not connect to the Kubernetes server. This may be because the kubeconfig information is missing or has credentials that do not match the server. Assuming default version %s`, defaultKubeVersion)
            fmt.Fprintln(os.Stderr, msg)
        }
        # 打印警告信息并返回默认版本的 KubeVersion 对象
        glog.V(1).Info(fmt.Sprintf("Unable to get Kubernetes version from kubectl, using default version: %s", defaultKubeVersion))
        return &KubeVersion{baseVersion: defaultKubeVersion}
    }
    # 获取服务器版本信息
    sv := vrObj.ServerVersion
    # 返回包含服务器版本信息的 KubeVersion 对象
    return &KubeVersion{
        Major:      sv.Major,
        Minor:      sv.Minor,
        GitVersion: sv.GitVersion,
    }
}

# 从 kubelet 输出中获取版本信息并返回 KubeVersion 对象
func getVersionFromKubeletOutput(s string) *KubeVersion {
    # 打印详细日志信息
    glog.V(2).Info(s)
    # 定义正则表达式来匹配服务器版本信息
    serverVersionRe := regexp.MustCompile(`Kubernetes v(\d+.\d+)`)
    # 在字符串 s 中查找匹配正则表达式的子串
    subs := serverVersionRe.FindStringSubmatch(s)
    # 如果未找到匹配的子串，则打印警告信息并返回默认版本的 KubeVersion 对象
    if len(subs) < 2 {
        glog.V(1).Info(fmt.Sprintf("Unable to get Kubernetes version from kubelet, using default version: %s", defaultKubeVersion))
        return &KubeVersion{baseVersion: defaultKubeVersion}
    }
    # 返回包含服务器版本信息的 KubeVersion 对象
    return &KubeVersion{baseVersion: subs[1]}
}

# 对字符串进行替换，并返回替换后的字符串和替换的内容列表
func makeSubstitutions(s string, ext string, m map[string]string) (string, []string) {
    # 创建空的替换内容列表
    substitutions := make([]string, 0)
    # 遍历字典 m 中的键值对
    for k, v := range m:
        # 构造替换字符串，格式为 "$" + 键 + 扩展名
        subst := "$" + k + ext
        # 如果值为空字符串，则记录日志并继续下一次循环
        if v == "":
            glog.V(2).Info(fmt.Sprintf("No substitution for '%s'\n", subst))
            continue
        # 记录替换日志
        glog.V(2).Info(fmt.Sprintf("Substituting %s with '%s'\n", subst, v))
        # 保存替换前的字符串
        beforeS := s
        # 调用 multiWordReplace 函数进行多词替换
        s = multiWordReplace(s, subst, v)
        # 如果替换前后字符串不相等，则将 v 添加到 substitutions 列表中
        if beforeS != s:
            substitutions = append(substitutions, v)
    # 返回替换后的字符串和替换过的值列表
    return s, substitutions
}


func isEmpty(str string) bool {
    // 使用 strings.TrimSpace 函数去除字符串两端的空白字符，然后判断是否为空
    return strings.TrimSpace(str) == ""
}


func buildComponentMissingErrorMessage(nodetype check.NodeType, component string, bins []string) string {
    // 定义错误消息模板
    errMessageTemplate := `
Unable to detect running programs for component %q
The following %q programs have been searched, but none of them have been found:
%s

These program names are provided in the config.yaml, section '%s.%s.bins'
`
    // 根据节点类型设置组件角色名称和组件类型
    var componentRoleName, componentType string
    switch nodetype {
    case check.NODE:
        componentRoleName = "worker node"
        componentType = "node"
    case check.ETCD:
        componentRoleName = "etcd node"
        componentType = "etcd"
    default:
        componentRoleName = "master node"
        componentType = "master"
    }
    // 构建程序列表字符串
    binList := ""
    for _, bin := range bins {
        binList = fmt.Sprintf("%s\t- %s\n", binList, bin)
    }
    // 根据错误消息模板和参数构建最终的错误消息
    return fmt.Sprintf(errMessageTemplate, component, componentRoleName, binList, componentType, component)
}


func getPlatformName() string {
    // 调用 getKubeVersion 函数获取版本信息
    kv, err := getKubeVersion()
    if err != nil {
        // 如果出错，记录错误信息并返回空字符串
        glog.V(2).Info(err)
        return ""
    }
    // 从版本信息中获取平台名称
    return getPlatformNameFromVersion(kv.GitVersion)
}


func getPlatformNameFromVersion(s string) string {
    // 使用正则表达式匹配版本字符串，提取平台名称
    versionRe := regexp.MustCompile(`v\d+\.\d+\.\d+-(\w+)(?:[.\-])\w+`)
    subs := versionRe.FindStringSubmatch(s)
    if len(subs) < 2 {
        return ""
    }
    return subs[1]
}


func getPlatformBenchmarkVersion(platform string) string {
    // 根据平台名称返回对应的基准版本
    switch platform {
    case "eks":
        return "eks-1.0"
    case "gke":
        return "gke-1.0"
    }
    return ""
}
```