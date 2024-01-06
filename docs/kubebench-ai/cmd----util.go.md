# `kubebench-aquasecurity\cmd\util.go`

```
package cmd

import (
	"encoding/json"  // 导入 JSON 编码解码包
	"fmt"  // 导入格式化输出包
	"os"  // 导入操作系统功能包
	"os/exec"  // 导入执行外部命令包
	"path/filepath"  // 导入文件路径操作包
	"regexp"  // 导入正则表达式包
	"strconv"  // 导入字符串转换包
	"strings"  // 导入字符串操作包

	"github.com/aquasecurity/kube-bench/check"  // 导入检查包
	"github.com/fatih/color"  // 导入颜色输出包
	"github.com/golang/glog"  // 导入日志包
	"github.com/spf13/viper"  // 导入配置管理包
)

var (
	// Print colors
# 创建一个映射，将check.State映射到color.Color，表示不同状态对应的颜色
colors = map[check.State]*color.Color{
    check.PASS: color.New(color.FgGreen),  # PASS状态对应绿色
    check.FAIL: color.New(color.FgRed),    # FAIL状态对应红色
    check.WARN: color.New(color.FgYellow), # WARN状态对应黄色
    check.INFO: color.New(color.FgBlue),   # INFO状态对应蓝色
}

# 定义一些全局变量
var psFunc func(string) string
var statFunc func(string) (os.FileInfo, error)
var getBinariesFunc func(*viper.Viper, check.NodeType) (map[string]string, error)

# 定义一个映射，将字符串映射到字符串数组
var TypeMap = map[string][]string{
    "ca":         []string{"cafile", "defaultcafile"},
    "kubeconfig": []string{"kubeconfig", "defaultkubeconfig"},
    "service":    []string{"svc", "defaultsvc"},
    "config":     []string{"confs", "defaultconf"},
}

# 初始化函数
func init() {
    psFunc = ps  # 将ps函数赋值给psFunc变量
}
// 定义变量statFunc为os.Stat函数
statFunc = os.Stat
// 定义变量getBinariesFunc为getBinaries函数
getBinariesFunc = getBinaries
}

// 定义函数exitWithError，用于打印错误信息并退出程序
func exitWithError(err error) {
	// 在标准错误输出流中打印错误信息
	fmt.Fprintf(os.Stderr, "\n%v\n", err)
	// 在退出非零状态前刷新日志
	glog.Flush()
	// 退出程序，状态码为1
	os.Exit(1)
}

// 定义函数cleanIDs，用于清理ID列表并返回一个ID的布尔值映射
func cleanIDs(list string) map[string]bool {
	// 去除列表两端的逗号
	list = strings.Trim(list, ",")
	// 将列表按逗号分割成字符串数组
	ids := strings.Split(list, ",")

	// 创建一个空的字符串到布尔值的映射
	set := make(map[string]bool)

	// 遍历ID数组，去除空格并将ID添加到映射中
	for _, id := range ids {
		id = strings.Trim(id, " ")
		set[id] = true
// ps 函数执行 ps 命令；将其分成一个函数，以便我们可以编写测试
func ps(proc string) string {
    // TODO: 截断 proc 为 15 个字符
   // 参考 https://github.com/aquasecurity/kube-bench/issues/328#issuecomment-506813344
    glog.V(2).Info(fmt.Sprintf("ps - proc: %q", proc))
    // 使用 exec 包创建一个 ps 命令
    cmd := exec.Command("/bin/ps", "-C", proc, "-o", "cmd", "--no-headers")
    // 执行命令并获取输出
    out, err := cmd.Output()
    // 如果有错误，记录错误信息
    if err != nil {
        glog.V(2).Info(fmt.Errorf("%s: %s", cmd.Args, err))
    }
    // 记录输出信息
    glog.V(2).Info(fmt.Sprintf("ps - returning: %q", string(out)))
    // 返回命令输出
    return string(out)
}
// getBinaries函数用于查找一组候选可执行文件中正在运行的文件。
// 如果一个必需的可执行文件没有运行，则返回错误。
func getBinaries(v *viper.Viper, nodetype check.NodeType) (map[string]string, error) {
	// 创建一个映射，用于存储可执行文件的名称和路径
	binmap := make(map[string]string)

	// 遍历配置文件中的组件列表
	for _, component := range v.GetStringSlice("components") {
		// 获取组件的子配置
		s := v.Sub(component)
		// 如果子配置为空，则继续下一个组件
		if s == nil {
			continue
		}

		// 获取组件是否为可选
		optional := s.GetBool("optional")
		// 获取组件的可执行文件列表
		bins := s.GetStringSlice("bins")
		// 如果可执行文件列表不为空
		if len(bins) > 0 {
			// 查找正在运行的可执行文件
			bin, err := findExecutable(bins)
			// 如果发生错误且可执行文件不是可选的，则返回错误
			if err != nil && !optional {
				glog.V(1).Info(buildComponentMissingErrorMessage(nodetype, component, bins))
				return nil, fmt.Errorf("unable to detect running programs for component %q", component)
			}
// 如果可执行文件名为空，则将其默认为组件的名称
// 如果不为空，则记录组件使用的可执行文件名
// 将组件和对应的可执行文件名存入映射中
// 返回映射和空错误

// getConfigFilePath 函数用于定位 CIS 版本的配置文件
// 打印日志，记录查找特定 CIS 版本的配置文件
// 将配置文件路径设置为 cfgDir 和 benchmarkVersion 的组合
// 将文件路径设置为路径和文件名的组合
// 使用 glog.V(2) 输出日志信息，表示正在查找文件
glog.V(2).Info(fmt.Sprintf("Looking for file: %s", file))

// 检查文件是否存在，如果不存在则输出错误信息并返回错误
if _, err := os.Stat(file); err != nil {
    glog.V(2).Infof("error accessing config file: %q error: %v\n", file, err)
    return "", fmt.Errorf("no test files found <= benchmark version: %s", benchmarkVersion)
}

// 返回文件路径和空错误
return path, nil
}

// getYamlFilesFromDir 从指定目录返回一个 yaml 文件列表，忽略 config.yaml
func getYamlFilesFromDir(path string) (names []string, err error) {
    // 遍历指定目录下的文件
    err = filepath.Walk(path, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }

        // 获取文件名
        _, name := filepath.Split(path)
        // 如果文件名不为空且不是 config.yaml 且文件扩展名为 .yaml，则将文件路径添加到列表中
        if name != "" && name != "config.yaml" && filepath.Ext(name) == ".yaml" {
            names = append(names, path)
		}

		return nil
	})
	return names, err
}

// decrementVersion 函数用于递减版本号
// 即使在我们没有提供测试文件的版本中，我们也希望逐个递减版本号
// 以防有人想为该版本指定自己的测试文件
func decrementVersion(version string) string {
	// 通过"."分割版本号
	split := strings.Split(version, ".")
	// 如果分割后的数组长度小于2，返回空字符串
	if len(split) < 2 {
		return ""
	}
	// 将次版本号转换为整数
	minor, err := strconv.Atoi(split[1])
	// 如果转换出错，返回空字符串
	if err != nil {
		return ""
	}
	// 如果次版本号小于等于1，返回空字符串
	if minor <= 1 {
		return ""
	}
	// 将版本号拆分为主版本号和次版本号
	split[1] = strconv.Itoa(minor - 1)
	// 将拆分后的版本号重新组合成字符串
	return strings.Join(split, ".")
}

// getFiles finds which of the set of candidate files exist
// getFiles函数用于查找候选文件集中存在的文件
func getFiles(v *viper.Viper, fileType string) map[string]string {
	// 创建一个存储文件路径的map
	filemap := make(map[string]string)
	// 获取文件类型对应的主要选项和默认选项
	mainOpt := TypeMap[fileType][0]
	defaultOpt := TypeMap[fileType][1]

	// 遍历配置文件中的组件
	for _, component := range v.GetStringSlice("components") {
		// 获取组件的子配置
		s := v.Sub(component)
		// 如果子配置为空，则继续下一个组件
		if s == nil {
			continue
		}

		// 查找候选文件中存在的文件
		file := findConfigFile(s.GetStringSlice(mainOpt))
// 如果文件名为空
if file == "" {
    // 如果设置了默认选项，则使用默认文件名
    if s.IsSet(defaultOpt) {
        file = s.GetString(defaultOpt)
        glog.V(2).Info(fmt.Sprintf("Using default %s file name '%s' for component %s", fileType, file, component))
    } else {
        // 否则将文件名默认为组件的名称
        glog.V(2).Info(fmt.Sprintf("Missing %s file for %s", fileType, component))
        file = component
    }
} else {
    // 如果文件名不为空，则输出组件使用的文件名
    glog.V(2).Info(fmt.Sprintf("Component %s uses %s file '%s'", component, fileType, file))
}

// 将组件和文件名映射存入文件映射中
filemap[component] = file

// 返回文件映射
return filemap
}

// verifyBin checks that the binary specified is running
func verifyBin(bin string) bool {
    // 去除任何引号
    bin = strings.Trim(bin, "'\"")

    // bin 可能由多个单词组成
    // 我们将使用第一个单词搜索运行中的进程，然后检查整个进程是否包含所提供的 bin
    proc := strings.Fields(bin)[0]
    out := psFunc(proc)

    // ps 输出可能包含多行
    // 二进制文件需要是 ps 输出中的第一个单词，除非它可能被路径所引导
    // 例如，/usr/bin/kubelet 是 kubelet 的匹配
    // 但是 apiserver 不是 kube-apiserver 的匹配
    reFirstWord := regexp.MustCompile(`^(\S*\/)*` + bin)
    lines := strings.Split(out, "\n")
    for _, l := range lines {
        glog.V(3).Info(fmt.Sprintf("reFirstWord.Match(%s)", l))
        if reFirstWord.Match([]byte(l)) {
// 返回 true
		return true
	}
}

// 返回 false
return false
}

// findConfigFile 遍历可能的配置文件列表，并找到第一个存在的文件
func findConfigFile(candidates []string) string {
	// 遍历候选文件列表
	for _, c := range candidates {
		// 检查文件是否存在
		_, err := statFunc(c)
		// 如果文件存在，返回文件名
		if err == nil {
			return c
		}
		// 如果文件不存在，输出错误信息
		if !os.IsNotExist(err) {
			exitWithError(fmt.Errorf("error looking for file %s: %v", c, err))
		}
	}

	// 如果没有找到存在的文件，返回空字符串
	return ""
// findExecutable函数通过查找可能的可执行文件名列表，找到第一个正在运行的可执行文件
func findExecutable(candidates []string) (string, error) {
	// 遍历候选列表
	for _, c := range candidates {
		// 验证可执行文件是否在运行
		if verifyBin(c) {
			// 如果可执行文件在运行，则返回该文件名
			return c, nil
		}
		// 如果可执行文件不在运行，则记录日志
		glog.V(1).Info(fmt.Sprintf("executable '%s' not running", c))
	}

	// 如果没有找到正在运行的可执行文件，则返回错误
	return "", fmt.Errorf("no candidates running")
}

// multiWordReplace函数用于替换字符串中的子字符串
func multiWordReplace(s string, subname string, sub string) string {
	// 将子字符串分割成单词
	f := strings.Fields(sub)
	// 如果子字符串包含多个单词，则在子字符串两端添加单引号
	if len(f) > 1 {
		sub = "'" + sub + "'"
	}
	// 返回替换后的字符串
}
# 使用 strings 包中的 Replace 方法替换字符串 s 中的 subname 为 sub，-1 表示替换所有匹配项
return strings.Replace(s, subname, sub, -1)
```

```
# 定义常量 missingKubectlKubeletMessage，包含找不到 kubectl 或 kubelet 程序的错误信息
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
```
# 获取 Kubernetes 版本信息
func getKubeVersion() (*KubeVersion, error) {
    # 从 Kubernetes REST API 获取版本信息
    if k8sVer, err := getKubeVersionFromRESTAPI(); err == nil {
        glog.V(2).Info(fmt.Sprintf("Kubernetes REST API Reported version: %s", k8sVer))
        return k8sVer, nil
    }
    
    # 检查是否存在 kubectl 可执行文件
    _, err := exec.LookPath("kubectl")
    
    if err != nil {
        # 如果不存在 kubectl，则检查是否存在 kubelet 可执行文件
        _, err = exec.LookPath("kubelet")
        if err != nil {
            # 在文件系统中搜索 kubelet 可执行文件并运行第一个匹配项以获取 Kubernetes 版本
            cmd := exec.Command("/bin/sh", "-c", "`find / -type f -executable -name kubelet 2>/dev/null | grep -m1 .` --version")
            out, err := cmd.CombinedOutput()
            if err == nil {
                return getVersionFromKubeletOutput(string(out)), nil
// 如果 kubectl 和 kubelet 都不在 PATH 中，则记录警告并返回错误
if !checkProgramInPath("kubectl") || !checkProgramInPath("kubelet") {
    glog.Warning(missingKubectlKubeletMessage)
    return nil, fmt.Errorf("unable to find the programs kubectl or kubelet in the PATH")
}
// 从 kubelet 获取 Kubernetes 版本信息
return getKubeVersionFromKubelet(), nil

// 从 kubectl 获取 Kubernetes 版本信息
return getKubeVersionFromKubectl(), nil
```
在这段代码中，首先检查 kubectl 和 kubelet 是否在 PATH 中，如果都不在，则记录警告并返回错误。然后根据情况分别从 kubelet 和 kubectl 获取 Kubernetes 版本信息。
# 从 kubelet 获取 Kubernetes 版本信息
func getKubeVersionFromKubelet() *KubeVersion {
    # 创建一个执行命令的对象，执行 kubelet --version 命令
    cmd := exec.Command("kubelet", "--version")
    # 获取命令执行的输出和可能的错误
    out, err := cmd.CombinedOutput()

    # 如果有错误发生，记录错误信息
    if err != nil {
        glog.V(2).Info(err)
    }

    # 将输出的字符串传递给函数，解析并返回 Kubernetes 版本信息
    return getVersionFromKubeletOutput(string(out))
}

# 从 kubectl 输出中获取版本信息
func getVersionFromKubectlOutput(s string) *KubeVersion {
    # 记录输出的字符串
    glog.V(2).Info(s)
    # 定义一个结构来存储版本信息
    type versionResult struct {
        ServerVersion VersionResponse
    }
    vrObj := &versionResult{}
    # 将输出的字符串解析为 JSON 格式，并存储到 vrObj 中
    if err := json.Unmarshal([]byte(s), vrObj); err != nil {
        # 如果解析出错，记录错误信息
        glog.V(2).Info(err)
    }
		// 如果字符串 s 包含 "The connection to the server"，则将 msg 输出到标准错误流
		if strings.Contains(s, "The connection to the server") {
			fmt.Fprintln(os.Stderr, msg)
		}
		// 使用 glog.V(1) 输出日志信息，表示无法从 kubectl 获取 Kubernetes 版本，使用默认版本
		glog.V(1).Info(fmt.Sprintf("Unable to get Kubernetes version from kubectl, using default version: %s", defaultKubeVersion))
		// 返回默认版本的 KubeVersion 对象
		return &KubeVersion{baseVersion: defaultKubeVersion}
	}
	// 从 vrObj.ServerVersion 获取服务器版本信息
	sv := vrObj.ServerVersion
	// 返回包含服务器版本信息的 KubeVersion 对象
	return &KubeVersion{
		Major:      sv.Major,
		Minor:      sv.Minor,
		GitVersion: sv.GitVersion,
	}
}

// 从 kubelet 输出中获取版本信息
func getVersionFromKubeletOutput(s string) *KubeVersion {
	// 使用 glog.V(2) 输出详细信息
	glog.V(2).Info(s)
	// 使用正则表达式匹配 Kubernetes 版本信息
	serverVersionRe := regexp.MustCompile(`Kubernetes v(\d+.\d+)`)
	subs := serverVersionRe.FindStringSubmatch(s)
	// 如果匹配结果小于 2，表示无法获取版本信息，使用默认版本
	if len(subs) < 2 {
		glog.V(1).Info(fmt.Sprintf("Unable to get Kubernetes version from kubelet, using default version: %s", defaultKubeVersion))
// 返回一个指向KubeVersion结构体的指针，其中baseVersion字段的值为defaultKubeVersion
		return &KubeVersion{baseVersion: defaultKubeVersion}
	}
	// 返回一个指向KubeVersion结构体的指针，其中baseVersion字段的值为subs切片的第二个元素
	return &KubeVersion{baseVersion: subs[1]}
}

// makeSubstitutions函数用于替换字符串s中的变量，ext为变量的后缀，m为变量名和值的映射
func makeSubstitutions(s string, ext string, m map[string]string) (string, []string) {
	// 初始化一个空切片substitutions用于存储替换的值
	substitutions := make([]string, 0)
	// 遍历变量名和值的映射m
	for k, v := range m {
		// 构造变量的格式
		subst := "$" + k + ext
		// 如果值为空，则打印日志并继续下一次循环
		if v == "" {
			glog.V(2).Info(fmt.Sprintf("No substitution for '%s'\n", subst))
			continue
		}
		// 打印替换日志
		glog.V(2).Info(fmt.Sprintf("Substituting %s with '%s'\n", subst, v))
		// 保存替换前的字符串
		beforeS := s
		// 调用multiWordReplace函数进行替换
		s = multiWordReplace(s, subst, v)
		// 如果替换前后的字符串不相等，则将替换的值添加到substitutions切片中
		if beforeS != s {
			substitutions = append(substitutions, v)
		}
	}
# 返回两个值 s 和 substitutions
return s, substitutions
}

# 判断字符串是否为空
func isEmpty(str string) bool {
    return strings.TrimSpace(str) == ""
}

# 构建组件缺失的错误消息
func buildComponentMissingErrorMessage(nodetype check.NodeType, component string, bins []string) string {
    # 定义错误消息模板
    errMessageTemplate := `
    Unable to detect running programs for component %q
    The following %q programs have been searched, but none of them have been found:
    %s

    These program names are provided in the config.yaml, section '%s.%s.bins'
    `
    # 定义变量 componentRoleName 和 componentType
    var componentRoleName, componentType string
# 根据节点类型设置组件角色名称和类型
switch nodetype {
    # 如果节点类型为 NODE，则设置组件角色名称为 "worker node"，组件类型为 "node"
    case check.NODE:
        componentRoleName = "worker node"
        componentType = "node"
    # 如果节点类型为 ETCD，则设置组件角色名称为 "etcd node"，组件类型为 "etcd"
    case check.ETCD:
        componentRoleName = "etcd node"
        componentType = "etcd"
    # 如果节点类型为其他情况，则设置组件角色名称为 "master node"，组件类型为 "master"
    default:
        componentRoleName = "master node"
        componentType = "master"
}

# 构建二进制文件列表的字符串
binList := ""
for _, bin := range bins {
    binList = fmt.Sprintf("%s\t- %s\n", binList, bin)
}

# 返回格式化的错误消息模板，包括组件名称、组件角色名称、二进制文件列表、组件类型和组件名称
return fmt.Sprintf(errMessageTemplate, component, componentRoleName, binList, componentType, component)
# 获取平台名称的函数
func getPlatformName() string {
    # 调用 getKubeVersion 函数获取 Kubernetes 版本信息
    kv, err := getKubeVersion()
    # 如果出现错误，记录日志并返回空字符串
    if err != nil {
        glog.V(2).Info(err)
        return ""
    }
    # 调用 getPlatformNameFromVersion 函数获取平台名称
    return getPlatformNameFromVersion(kv.GitVersion)
}

# 从版本信息中获取平台名称的函数
func getPlatformNameFromVersion(s string) string {
    # 使用正则表达式匹配版本信息中的平台名称
    versionRe := regexp.MustCompile(`v\d+\.\d+\.\d+-(\w+)(?:[.\-])\w+`)
    subs := versionRe.FindStringSubmatch(s)
    # 如果匹配结果小于2，返回空字符串
    if len(subs) < 2 {
        return ""
    }
    # 返回匹配到的平台名称
    return subs[1]
}

# 获取平台基准版本的函数
func getPlatformBenchmarkVersion(platform string) string {
# 根据平台名称返回对应的版本号
switch platform:
    # 如果平台是 eks，则返回版本号 eks-1.0
    case "eks":
        return "eks-1.0"
    # 如果平台是 gke，则返回版本号 gke-1.0
    case "gke":
        return "gke-1.0"
# 如果平台不是 eks 或 gke，则返回空字符串
return ""
```