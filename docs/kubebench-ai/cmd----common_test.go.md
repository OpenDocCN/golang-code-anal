# `kubebench-aquasecurity\cmd\common_test.go`

```
// 版权声明和许可证信息
// 该代码受 Apache 许可证版本 2.0 的保护
// 除非符合许可证的规定，否则不得使用该文件
// 可以在 http://www.apache.org/licenses/LICENSE-2.0 获取许可证的副本
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于“按原样”分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

// 导入所需的包
package cmd

import (
	"encoding/json"  // 导入 JSON 编码/解码包
	"errors"  // 导入错误处理包
	"fmt"  // 导入格式化包
# 导入io/ioutil模块，用于读取和写入文件
"io/ioutil"
# 导入os模块，提供了许多与操作系统交互的函数
"os"
# 导入path模块，用于处理文件路径
"path"
# 导入path/filepath模块，用于处理文件路径
"path/filepath"
# 导入testing模块，用于编写测试函数
"testing"
# 导入time模块，用于处理时间
"time"

# 导入kube-bench/check模块中的相关函数和类
"github.com/aquasecurity/kube-bench/check"
# 导入viper模块，用于处理配置文件
"github.com/spf13/viper"
# 导入assert模块，用于编写测试断言
"github.com/stretchr/testify/assert"
)

# 定义JsonOutputFormat结构体，用于存储JSON格式的输出
type JsonOutputFormat struct {
    Controls     []*check.Controls `json:"Controls"`
    TotalSummary map[string]int    `json:"Totals"`
}

# 定义JsonOutputFormatNoTotals结构体，用于存储不包含总结的JSON格式的输出
type JsonOutputFormatNoTotals struct {
    Controls []*check.Controls `json:"Controls"`
}
# 测试函数，用于测试parseSkipIds函数
func TestParseSkipIds(t *testing.T) {
    # 调用parseSkipIds函数，将字符串"4.12,4.13,5"解析成跳过的ID映射
    skipMap := parseSkipIds("4.12,4.13,5")
    # 检查是否存在4.12的键
    _, fourTwelveExists := skipMap["4.12"]
    # 检查是否存在4.13的键
    _, fourThirteenExists := skipMap["4.13"]
    # 检查是否存在5的键
    _, fiveExists := skipMap["5"]
    # 检查是否存在其他键
    _, other := skipMap["G1"]
    # 断言4.13存在
    assert.True(t, fourThirteenExists)
    # 断言4.12存在
    assert.True(t, fourTwelveExists)
    # 断言5存在
    assert.True(t, fiveExists)
    # 断言其他不存在
    assert.False(t, other)
}

# 测试函数，用于测试NewRunFilter函数
func TestNewRunFilter(t *testing.T) {

    # 定义TestCase结构体
    type TestCase struct {
        Name       string
        FilterOpts FilterOpts
        Group      *check.Group
        Check      *check.Check
// 定义一个期望的布尔类型
Expected bool
}

// 定义测试用例的结构体
testCases := []TestCase{
	{
		// 测试用例的名称
		Name:       "Should return true when scored flag is enabled and check is scored",
		// 过滤选项，包括是否得分和未得分
		FilterOpts: FilterOpts{Scored: true, Unscored: false},
		// 检查所属的组
		Group:      &check.Group{},
		// 检查对象
		Check:      &check.Check{Scored: true},
		// 期望的结果为 true
		Expected:   true,
	},
	{
		// 测试用例的名称
		Name:       "Should return false when scored flag is enabled and check is not scored",
		// 过滤选项，包括是否得分和未得分
		FilterOpts: FilterOpts{Scored: true, Unscored: false},
		// 检查所属的组
		Group:      &check.Group{},
		// 检查对象
		Check:      &check.Check{Scored: false},
		// 期望的结果为 false
		Expected:   false,
	},
```
# 第一个测试用例：当未评分标志位启用且检查未评分时应返回true
{
    Name:       "Should return true when unscored flag is enabled and check is not scored",
    FilterOpts: FilterOpts{Scored: false, Unscored: true},  # 设置过滤选项，包括评分为false，未评分为true
    Group:      &check.Group{},  # 创建一个空的检查组
    Check:      &check.Check{Scored: false},  # 创建一个未评分的检查
    Expected:   true,  # 期望结果为true
},

# 第二个测试用例：当未评分标志位启用且检查已评分时应返回false
{
    Name:       "Should return false when unscored flag is enabled and check is scored",
    FilterOpts: FilterOpts{Scored: false, Unscored: true},  # 设置过滤选项，包括评分为false，未评分为true
    Group:      &check.Group{},  # 创建一个空的检查组
    Check:      &check.Check{Scored: true},  # 创建一个已评分的检查
    Expected:   false,  # 期望结果为false
},

# 第三个测试用例：当组标志包含组的ID时应返回true
{
    Name:       "Should return true when group flag contains group's ID",
    FilterOpts: FilterOpts{Scored: true, Unscored: true, GroupList: "G1,G2,G3"},  # 设置过滤选项，包括评分为true，未评分为true，组列表包含G1、G2、G3
    Group:      &check.Group{ID: "G2"},  # 创建一个具有ID为G2的检查组
    Check:      &check.Check{},  # 创建一个空的检查
# 期望返回 true
		},
		{
			# 当组标志不包含组的ID时，应返回false
			Name:       "Should return false when group flag doesn't contain group's ID",
			FilterOpts: FilterOpts{GroupList: "G1,G3"},
			Group:      &check.Group{ID: "G2"},
			Check:      &check.Check{},
			Expected:   false,
		},

		{
			# 当检查标志包含检查的ID时，应返回true
			Name:       "Should return true when check flag contains check's ID",
			FilterOpts: FilterOpts{Scored: true, Unscored: true, CheckList: "C1,C2,C3"},
			Group:      &check.Group{},
			Check:      &check.Check{ID: "C2"},
			Expected:   true,
		},
		{
			# 当检查标志不包含检查的ID时，应返回false
			Name:       "Should return false when check flag doesn't contain check's ID",
			FilterOpts: FilterOpts{CheckList: "C1,C3"},
		Group:      &check.Group{},  // 创建一个空的检查组对象
		Check:      &check.Check{ID: "C2"},  // 创建一个具有指定ID的检查对象
		Expected:   false,  // 设置期望结果为false
	}

	for _, testCase := range testCases {
		t.Run(testCase.Name, func(t *testing.T) {
			filter, _ := NewRunFilter(testCase.FilterOpts)  // 根据测试用例的过滤选项创建过滤器
			assert.Equal(t, testCase.Expected, filter(testCase.Group, testCase.Check))  // 断言过滤器的结果与期望结果相等
		})
	}

	t.Run("Should return error when both group and check flags are used", func(t *testing.T) {
		// given
		opts := FilterOpts{GroupList: "G1", CheckList: "C1"}  // 创建包含组列表和检查列表的过滤选项
		// when
		_, err := NewRunFilter(opts)  // 使用过滤选项创建运行过滤器，捕获可能的错误
		// then
		assert.EqualError(t, err, "group option and check option can't be used together")  // 断言错误信息与预期错误信息相等
	})

}

// TestIsMaster 是一个测试函数，用于测试 isMaster 函数
func TestIsMaster(t *testing.T) {
	// 定义测试用例
	testCases := []struct {
		name            string
		cfgFile         string
		getBinariesFunc func(*viper.Viper, check.NodeType) (map[string]string, error)
		isMaster        bool
	}{
		{
			name:    "valid config, is master and all components are running", // 测试用例名称
			cfgFile: "../cfg/config.yaml", // 配置文件路径
			getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
				return map[string]string{"apiserver": "kube-apiserver"}, nil // 返回二进制文件映射和错误
			},
			isMaster: true, // 期望的结果
		},
		{
# 定义一个测试用例，描述了一个有效的配置，是主节点，但并非所有组件都在运行
name:    "valid config, is master and but not all components are running"
# 配置文件路径
cfgFile: "../cfg/config.yaml"
# 获取二进制文件的函数，返回一个空的字符串映射和空的错误
getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
    return map[string]string{}, nil
},
# 是否为主节点
isMaster: false

# 定义一个测试用例，描述了一个有效的配置，是主节点，但并非所有组件都在运行，并且无法找到所有二进制文件
name:    "valid config, is master, not all components are running and fails to find all binaries"
# 配置文件路径
cfgFile: "../cfg/config.yaml"
# 获取二进制文件的函数，返回一个空的字符串映射和一个找不到二进制文件的错误
getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
    return map[string]string{}, errors.New("failed to find binaries")
},
# 是否为主节点
isMaster: false

# 定义一个测试用例，描述了一个有效的配置，不包括主节点
name:     "valid config, does not include master"
# 配置文件路径
cfgFile:  "../hack/node_only.yaml"
# 是否为主节点
isMaster: false
	}
	// 保存旧的配置目录
	cfgDirOld := cfgDir
	// 更新配置目录
	cfgDir = "../cfg"
	// 在函数返回时恢复原始配置目录
	defer func() {
		cfgDir = cfgDirOld
	}()

	// 执行的代码内容
	execCode := `#!/bin/sh
	echo "Server Version: v1.13.10"
	`
	// 在 PATH 中创建一个假的可执行文件
	restore, err := fakeExecutableInPath("kubectl", execCode)
	// 如果创建失败，输出错误信息并终止测试
	if err != nil {
		t.Fatal("Failed when calling fakeExecutableInPath ", err)
	}
	// 在函数返回时恢复原始的环境
	defer restore()

	// 遍历测试用例
	for _, tc := range testCases {
		// 设置配置文件
		cfgFile = tc.cfgFile
		// 初始化配置
		initConfig()
		// 保存原始的getBinariesFunc函数，以便在测试结束后恢复
		oldGetBinariesFunc := getBinariesFunc
		// 将getBinariesFunc函数替换为tc.getBinariesFunc
		getBinariesFunc = tc.getBinariesFunc
		// 在测试结束后恢复原始的getBinariesFunc函数，并清空cfgFile变量
		defer func() {
			getBinariesFunc = oldGetBinariesFunc
			cfgFile = ""
		}()

		// 断言当前是否为主节点，用于测试
		assert.Equal(t, tc.isMaster, isMaster(), tc.name)
	}
}

func TestMapToCISVersion(t *testing.T) {

	// 加载测试用的viper配置文件
	viperWithData, err := loadConfigForTest()
	if err != nil {
		t.Fatalf("Unable to load config file %v", err)
	}
	// 加载版本映射关系
	kubeToBenchmarkMap, err := loadVersionMapping(viperWithData)
	if err != nil {
		t.Fatalf("Unable to load config file %v", err)
	}

	// 定义测试用例的结构体，包括 kubeVersion 字符串、succeed 布尔值、exp 字符串、expErr 字符串
	cases := []struct {
		kubeVersion string
		succeed     bool
		exp         string
		expErr      string
	}{
		// 测试用例1
		{kubeVersion: "1.9", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.9"},
		// 测试用例2
		{kubeVersion: "1.11", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.11"},
		// 测试用例3
		{kubeVersion: "1.12", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.12"},
		// 测试用例4
		{kubeVersion: "1.13", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.13"},
		// 测试用例5
		{kubeVersion: "1.14", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.14"},
		// 测试用例6
		{kubeVersion: "1.15", succeed: true, exp: "cis-1.5"},
		// 测试用例7
		{kubeVersion: "1.16", succeed: true, exp: "cis-1.6"},
		// 测试用例8
		{kubeVersion: "1.17", succeed: true, exp: "cis-1.6"},
		// 测试用例9
		{kubeVersion: "1.18", succeed: true, exp: "cis-1.6"},
		// 测试用例10
		{kubeVersion: "1.19", succeed: true, exp: "cis-1.6"},
		// 测试用例11
		{kubeVersion: "gke-1.0", succeed: true, exp: "gke-1.0"},
		// 测试用例12
		{kubeVersion: "ocp-3.10", succeed: true, exp: "rh-0.7"},
// 定义测试用例数组，包含不同的 kubeVersion、succeed、exp 等字段
cases := []struct {
    kubeVersion string // kubeVersion 字段
    succeed bool // succeed 字段
    exp string // exp 字段
    expErr string // expErr 字段
}{
    {kubeVersion: "ocp-3.11", succeed: true, exp: "rh-0.7"}, // 第一个测试用例
    {kubeVersion: "unknown", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: unknown"}, // 第二个测试用例
}

// 遍历测试用例数组
for _, c := range cases {
    // 调用 mapToBenchmarkVersion 函数，传入 kubeToBenchmarkMap 和当前测试用例的 kubeVersion
    rv, err := mapToBenchmarkVersion(kubeToBenchmarkMap, c.kubeVersion)
    // 如果测试用例中 succeed 为 true
    if c.succeed {
        // 如果返回错误不为空，输出错误信息
        if err != nil {
            t.Errorf("[%q]-Unexpected error: %v", c.kubeVersion, err)
        }
        // 如果返回值长度为 0，输出错误信息
        if len(rv) == 0 {
            t.Errorf("[%q]-missing return value", c.kubeVersion)
        }
        // 如果返回值不等于期望值，输出错误信息
        if c.exp != rv {
            t.Errorf("[%q]- expected %q but Got %q", c.kubeVersion, c.exp, rv)
        }
    } else {
        // 如果测试用例中 succeed 为 false，判断返回值是否等于期望值
        if c.exp != rv {
// 使用错误格式化字符串打印测试失败信息，包括 kubeVersion、rv 和期望值
t.Errorf("[%q]-mapToBenchmarkVersion kubeversion: %q Got %q expected %s", c.kubeVersion, c.kubeVersion, rv, c.exp)

// 检查期望的错误信息是否与实际错误信息相匹配，如果不匹配则打印测试失败信息
t.Errorf("[%q]-mapToBenchmarkVersion expected Error: %q instead Got %q", c.kubeVersion, c.expErr, err.Error())

// 定义一个函数 setDefault，用于设置 viper 对象的默认值并返回该对象
func setDefault(v *viper.Viper, key string, value interface{}) *viper.Viper {
    v.SetDefault(key, value)
    return v
}

// 加载测试用的配置文件，并返回 viper 对象和可能的错误
viperWithData, err := loadConfigForTest()
if err != nil {
    // 如果加载配置文件出错，则打印错误信息并终止测试
    t.Fatalf("Unable to load config file %v", err)
}
	# 定义一个结构体切片，每个结构体包含一个字符串和一个viper.Viper指针，以及一个布尔值
	cases := []struct {
		n       string
		v       *viper.Viper
		succeed bool
	}{
		# 第一个结构体，n为"empty"，v为一个新的viper.Viper对象，succeed为false
		{n: "empty", v: viper.New(), succeed: false},
		# 第二个结构体，n为"novals"，v为设置了默认值的viper.Viper对象，succeed为false
		{
			n:       "novals",
			v:       setDefault(viper.New(), "version_mapping", "novals"),
			succeed: false,
		},
		# 第三个结构体，n为"good"，v为viperWithData对象，succeed为true
		{
			n:       "good",
			v:       viperWithData,
			succeed: true,
		},
	}
	# 遍历结构体切片
	for _, c := range cases {
		# 调用loadVersionMapping函数，传入viper.Viper对象，返回结果和错误
		rv, err := loadVersionMapping(c.v)
// 如果操作成功
if c.succeed {
    // 如果出现错误，输出错误信息
    if err != nil {
        t.Errorf("[%q]-Unexpected error: %v", c.n, err)
    }
    // 如果返回值长度为0，输出缺少映射值的错误信息
    if len(rv) == 0 {
        t.Errorf("[%q]-missing mapping value", c.n)
    }
} else {
    // 如果期望出现错误但未出现，输出错误信息
    if err == nil {
        t.Errorf("[%q]-Expected error but got none", c.n)
    }
}
}

// 测试获取基准版本
func TestGetBenchmarkVersion(t *testing.T) {
    // 加载测试用的配置文件
    viperWithData, err := loadConfigForTest()
    // 如果加载配置文件出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
	}

	// 定义一个函数类型，用于测试获取基准版本
	type getBenchmarkVersionFnToTest func(kubeVersion, benchmarkVersion string, v *viper.Viper) (string, error)

	// 定义一个带有假的 kubectl 的函数，用于测试获取基准版本
	withFakeKubectl := func(kubeVersion, benchmarkVersion string, v *viper.Viper, fn getBenchmarkVersionFnToTest) (string, error) {
		// 定义一个假的可执行代码，模拟 kubectl 命令的输出
		execCode := `#!/bin/sh
		echo '{"serverVersion": {"major": "1", "minor": "15", "gitVersion": "v1.15.10"}}'
		`
		// 将假的可执行代码添加到系统路径中
		restore, err := fakeExecutableInPath("kubectl", execCode)
		if err != nil {
			t.Fatal("Failed when calling fakeExecutableInPath ", err)
		}
		defer restore() // 在函数结束时恢复系统路径

		// 调用传入的函数，获取基准版本
		return fn(kubeVersion, benchmarkVersion, v)
	}

	// 定义一个不包含路径的函数，用于测试获取基准版本
	withNoPath := func(kubeVersion, benchmarkVersion string, v *viper.Viper, fn getBenchmarkVersionFnToTest) (string, error) {
		// 移除系统路径中的所有路径
		restore, err := prunePath()
		if err != nil {
		// 如果调用 prunePath 出错，则输出错误信息并终止测试
		t.Fatal("Failed when calling prunePath ", err)
		// 延迟执行 restore 函数
		defer restore()
		// 调用 fn 函数，并返回结果
		return fn(kubeVersion, benchmarkVersion, v)
	}

	// 定义 getBenchmarkVersionFn 类型的函数
	type getBenchmarkVersionFn func(string, string, *viper.Viper, getBenchmarkVersionFnToTest) (string, error)
	// 定义测试用例
	cases := []struct {
		n                string
		kubeVersion      string
		benchmarkVersion string
		v                *viper.Viper
		callFn           getBenchmarkVersionFn
		exp              string
		succeed          bool
	}{
		// 测试用例1
		{n: "both versions", kubeVersion: "1.11", benchmarkVersion: "cis-1.3", exp: "cis-1.3", callFn: withNoPath, v: viper.New(), succeed: false},
		// 测试用例2
		{n: "no version-missing-kubectl", kubeVersion: "", benchmarkVersion: "", v: viperWithData, exp: "", callFn: withNoPath, succeed: false},
		// 测试用例3
		{n: "no version-fakeKubectl", kubeVersion: "", benchmarkVersion: "", v: viperWithData, exp: "cis-1.5", callFn: withFakeKubectl, succeed: true},
这段代码是一个测试用例，用于测试不同版本的 Kubernetes 和 benchmark 版本的匹配情况。

{n: "kubeVersion", kubeVersion: "1.15", benchmarkVersion: "", v: viperWithData, exp: "cis-1.5", callFn: withNoPath, succeed: true},
上面是一个测试用例，包括了 kubeVersion、benchmarkVersion、v、exp、callFn 和 succeed 等字段。

for _, c := range cases {
遍历测试用例数组 cases。

rv, err := c.callFn(c.kubeVersion, c.benchmarkVersion, c.v, getBenchmarkVersion)
调用测试用例中的 callFn 函数，传入 kubeVersion、benchmarkVersion、v 和 getBenchmarkVersion 参数，获取返回值和错误信息。

if c.succeed {
如果测试用例标记为成功。

if err != nil {
如果有错误发生，输出错误信息。

if len(rv) == 0 {
如果返回值长度为0，输出缺少返回值的错误信息。

if c.exp != rv {
如果期望值和返回值不相等，输出期望值和实际值不匹配的错误信息。
# 定义测试函数，检查是否有错误发生
func TestValidTargets(t *testing.T) {
    # 加载测试用的配置文件
    viperWithData, err := loadConfigForTest()
    # 如果加载配置文件出错，打印错误信息并终止测试
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
    }
    # 定义测试用例
    cases := []struct {
        name      string
        benchmark string
        targets   []string
        expected  bool
    }{
        {
            name:      "cis-1.5 no dummy",
# 定义一个基准测试用例，包括基准版本、目标对象、期望结果
{
    name:      "cis-1.5 invalid",
    benchmark: "cis-1.5",
    targets:   []string{"master", "node", "controlplane", "etcd", "dummy"},
    expected:  false,
},
{
    name:      "cis-1.5 valid",
    benchmark: "cis-1.5",
    targets:   []string{"master", "node", "controlplane", "etcd", "policies"},
    expected:  true,
},
{
    name:      "cis-1.6 no Pikachu",
    benchmark: "cis-1.6",
    targets:   []string{"master", "node", "controlplane", "etcd", "Pikachu"},
    expected:  false,
},
{
    name:      "cis-1.6 valid",
    benchmark: "cis-1.6",
    targets:   []string{"master", "node", "controlplane", "etcd", "policies"},
    expected:  true,
}
# 定义一个测试用例，验证是否为 true
{
    name:      "gke-1.0 valid",  # 测试用例名称
    benchmark: "gke-1.0",        # 测试用例所属的基准版本
    targets:   ["master", "node", "controlplane", "etcd", "policies", "managedservices"],  # 测试用例的目标
    expected:  true,              # 期望的测试结果
},
{
    name:      "aks-1.0 valid",  # 测试用例名称
    benchmark: "aks-1.0",        # 测试用例所属的基准版本
    targets:   ["node", "policies", "controlplane", "managedservices"],  # 测试用例的目标
    expected:  true,              # 期望的测试结果
},
{
    name:      "eks-1.0 valid",  # 测试用例名称
    benchmark: "eks-1.0",        # 测试用例所属的基准版本
    targets:   ["node", "policies", "controlplane", "managedservices"],  # 测试用例的目标
    expected:  true,              # 期望的测试结果
},
	}

	for _, c := range cases {
		// 对每个测试用例运行测试
		t.Run(c.name, func(t *testing.T) {
			// 调用 validTargets 函数，验证目标是否有效
			ret, err := validTargets(c.benchmark, c.targets, viperWithData)
			// 如果有错误，输出错误信息
			if err != nil {
				t.Fatalf("Expected nil error, got: %v", err)
			}
			// 如果返回结果与期望结果不符，输出错误信息
			if ret != c.expected {
				t.Fatalf("Expected %t, got %t", c.expected, ret)
			}
		})
	}
}

func TestIsEtcd(t *testing.T) {
	// 定义测试用例
	testCases := []struct {
		name            string
		cfgFile         string
		getBinariesFunc func(*viper.Viper, check.NodeType) (map[string]string, error)
```
注释：以上代码是针对测试用例的循环和测试函数的定义。对每个测试用例运行测试函数，并验证返回结果是否符合预期。
		isEtcd          bool  // 声明一个布尔类型的变量isEtcd

	}{  // 匿名结构体，包含以下字段和值

		// 第一个测试用例：有效配置，是etcd，并且所有组件都在运行
		{
			name:    "valid config, is etcd and all components are running",  // 测试用例的名称
			cfgFile: "../cfg/config.yaml",  // 配置文件的路径
			getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {  // 获取二进制文件的函数
				return map[string]string{"etcd": "etcd"}, nil  // 返回etcd二进制文件的映射和nil错误
			},
			isEtcd: true,  // 设置isEtcd为true
		},

		// 第二个测试用例：有效配置，是etcd，但并非所有组件都在运行
		{
			name:    "valid config, is etcd and but not all components are running",  // 测试用例的名称
			cfgFile: "../cfg/config.yaml",  // 配置文件的路径
			getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {  // 获取二进制文件的函数
				return map[string]string{}, nil  // 返回空的二进制文件映射和nil错误
			},
			isEtcd: false,  // 设置isEtcd为false
		},

		// 第三个测试用例：有效配置，是etcd，不是所有组件都在运行，并且无法找到所有二进制文件
			cfgFile: "../cfg/config.yaml", 
            # 设置配置文件路径为 "../cfg/config.yaml"
			getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                # 设置获取二进制文件的函数，如果失败则返回空map和错误信息
				return map[string]string{}, errors.New("failed to find binaries")
			},
            # 设置是否为 etcd
			isEtcd: false,
		},
		{
            # 设置名称为 "valid config, does not include etcd"
			name:    "valid config, does not include etcd",
            # 设置配置文件路径为 "../hack/node_only.yaml"
			cfgFile: "../hack/node_only.yaml",
            # 设置是否为 etcd
			isEtcd:  false,
		}
	}
    # 保存当前的配置文件路径
	cfgDirOld := cfgDir
    # 设置配置文件路径为 "../cfg"
	cfgDir = "../cfg"
    # 在函数结束时恢复原来的配置文件路径
	defer func() {
		cfgDir = cfgDirOld
	}()

    # 设置执行的代码
	execCode := `#!/bin/sh
	echo "Server Version: v1.15.03"
# 调用 fakeExecutableInPath 函数，将返回的结果赋值给 restore 变量，将可能出现的错误赋值给 err 变量
restore, err := fakeExecutableInPath("kubectl", execCode)
# 如果出现错误，则输出错误信息并终止程序
if err != nil {
    t.Fatal("Failed when calling fakeExecutableInPath ", err)
}
# 延迟执行 restore 函数，用于在函数返回前执行一些清理工作
defer restore()

# 遍历 testCases 列表
for _, tc := range testCases {
    # 将 tc.cfgFile 赋值给 cfgFile 变量
    cfgFile = tc.cfgFile
    # 调用 initConfig 函数
    initConfig()

    # 将 getBinariesFunc 赋值给 oldGetBinariesFunc 变量
    oldGetBinariesFunc := getBinariesFunc
    # 将 tc.getBinariesFunc 赋值给 getBinariesFunc 变量
    getBinariesFunc = tc.getBinariesFunc
    # 延迟执行匿名函数，用于在函数返回前执行一些清理工作
    defer func() {
        # 将 oldGetBinariesFunc 赋值给 getBinariesFunc 变量
        getBinariesFunc = oldGetBinariesFunc
        # 将空字符串赋值给 cfgFile 变量
        cfgFile = ""
    }()

    # 断言 isEtcd() 函数的返回值与 tc.isEtcd 是否相等，如果不相等则输出 tc.name
    assert.Equal(t, tc.isEtcd, isEtcd(), tc.name)
}
# 定义测试函数，用于将结果写入 JSON 文件
func TestWriteResultToJsonFile(t *testing.T) {
    # 在函数结束时执行清理操作，将 controlsCollection 重置为空数组，jsonFmt 重置为 false，outputFile 重置为空字符串
    defer func() {
        controlsCollection = []*check.Controls{}
        jsonFmt = false
        outputFile = ""
    }()
    # 声明变量 err
    var err error
    # 将 jsonFmt 设置为 true
    jsonFmt = true
    # 将 outputFile 设置为临时目录下的一个文件名，使用当前时间的纳秒级时间戳
    outputFile = path.Join(os.TempDir(), fmt.Sprintf("%d", time.Now().UnixNano()))

    # 解析 controlsCollection.json 文件，将结果存储在 controlsCollection 中
    controlsCollection, err = parseControlsJsonFile("./testdata/controlsCollection.json")
    # 如果解析过程中出现错误，输出错误信息
    if err != nil {
        t.Error(err)
    }
    # 将 controlsCollection 写入输出文件
    writeOutput(controlsCollection)

    # 声明变量 expect 和 result，用于存储 JSON 格式的输出
    var expect JsonOutputFormat
    var result JsonOutputFormat
}
# 调用 parseResultJsonFile 函数解析输出文件，将结果和错误赋值给 result 和 err
result, err = parseResultJsonFile(outputFile)

# 如果有错误发生，使用 t.Error 输出错误信息
if err != nil {
    t.Error(err)
}

# 调用 parseResultJsonFile 函数解析测试数据文件，将结果和错误赋值给 expect 和 err
expect, err = parseResultJsonFile("./testdata/result.json")

# 如果有错误发生，使用 t.Error 输出错误信息
if err != nil {
    t.Error(err)
}

# 使用 assert.Equal 检查 expect 和 result 是否相等
assert.Equal(t, expect, result)
}

# 定义测试函数 TestWriteResultNoTotalsToJsonFile
func TestWriteResultNoTotalsToJsonFile(t *testing.T) {
    # 延迟执行函数，清空 controlsCollection、jsonFmt、outputFile 变量
    defer func() {
        controlsCollection = []*check.Controls{}
        jsonFmt = false
        outputFile = ""
    }()
    # 声明 err 变量
    var err error
    # 将 jsonFmt 变量设置为 true
    jsonFmt = true
# 创建一个临时文件路径，文件名为当前时间的纳秒级 Unix 时间戳
outputFile = path.Join(os.TempDir(), fmt.Sprintf("%d", time.Now().UnixNano()))

# 设置一个标志变量，表示是否需要输出总数
noTotals = true

# 解析控制集合的 JSON 文件，将结果存储在 controlsCollection 变量中
controlsCollection, err = parseControlsJsonFile("./testdata/controlsCollection.json")
if err != nil:
    # 如果解析出错，输出错误信息
    t.Error(err)

# 将控制集合输出到控制台
writeOutput(controlsCollection)

# 声明两个变量，用于存储期望结果和实际结果
var expect []*check.Controls
var result []*check.Controls

# 解析输出文件中的结果，存储在 result 变量中
result, err = parseResultNoTotalsJsonFile(outputFile)
if err != nil:
    # 如果解析出错，输出错误信息
    t.Error(err)

# 解析期望结果文件中的结果，存储在 expect 变量中
expect, err = parseResultNoTotalsJsonFile("./testdata/result_no_totals.json")
if err != nil:
    # 如果解析出错，输出错误信息
    t.Error(err)
# 断言结果是否相等
assert.Equal(t, expect, result)
}

# 测试退出码选择函数
func TestExitCodeSelection(t *testing.T) {
    # 设置退出码为10
    exitCode = 10
    # 解析通过的控制集合的 JSON 文件
    controlsCollectionAllPassed, errPassed := parseControlsJsonFile("./testdata/passedControlsCollection.json")
    if errPassed != nil {
        t.Error(errPassed)
    }
    # 解析带有失败的控制集合的 JSON 文件
    controlsCollectionWithFailures, errFailure := parseControlsJsonFile("./testdata/controlsCollection.json")
    if errFailure != nil {
        t.Error(errFailure)
    }

    # 选择通过的控制集合的退出码
    exitCodePassed := exitCodeSelection(controlsCollectionAllPassed)
    assert.Equal(t, 0, exitCodePassed)

    # 选择带有失败的控制集合的退出码
    exitCodeFailure := exitCodeSelection(controlsCollectionWithFailures)
    assert.Equal(t, 10, exitCodeFailure)
```

# 定义一个测试函数，用于生成默认环境审计
func TestGenerationDefaultEnvAudit(t *testing.T) {
    # 定义输入的字节流
    input := []byte(`
---
type: "master"
groups:
- id: G1
  checks:
  - id: G1/C1
- id: G2
  checks:
  - id: G2/C1
    text: "Verify that the SomeSampleFlag argument is set to true"
    audit: "grep -B1 SomeSampleFlag=true /this/is/a/file/path"
    tests:
      test_items:
      - flag: "SomeSampleFlag=true"
        env: "SOME_SAMPLE_FLAG"
        compare:
```
// 设置操作为 has，值为 "true"
op: has
// 设置为 true
value: "true"
// 修复措施，编辑配置文件 /this/is/a/file/path，并将 SomeSampleFlag 设置为 true
remediation: |
  Edit the config file /this/is/a/file/path and set SomeSampleFlag to true.
// 设置为 true
scored: true
```

```go
// 创建一个新的控制对象集合，类型为 MASTER，使用给定的输入数据
controls, err := check.NewControls(check.MASTER, input)
// 断言错误为 nil
assert.NoError(t, err)

// 设置二进制替代品的列表
binSubs := []string{"TestBinPath"}
// 生成默认的环境审计
generateDefaultEnvAudit(controls, binSubs)

// 期望的审计环境字符串
expectedAuditEnv := fmt.Sprintf("cat \"/proc/$(/bin/ps -C %s -o pid= | tr -d ' ')/environ\" | tr '\\0' '\\n'", binSubs[0])
// 断言相等
assert.Equal(t, expectedAuditEnv, controls.Groups[1].Checks[0].AuditEnv)
```

```go
// 测试获取摘要总数
func TestGetSummaryTotals(t *testing.T) {
	// 解析控制对象集合的 JSON 文件
	controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
	// 如果出现错误，则打印错误信息
	if err != nil {
		// 如果有错误发生，输出错误信息
		t.Error(err)
	}

	// 获取控制集合的总结统计信息
	resultTotals := getSummaryTotals(controlsCollection)
	// 断言总结统计信息中的失败数量为12
	assert.Equal(t, 12, resultTotals.Fail)
	// 断言总结统计信息中的警告数量为14
	assert.Equal(t, 14, resultTotals.Warn)
	// 断言总结统计信息中的信息数量为0
	assert.Equal(t, 0, resultTotals.Info)
	// 断言总结统计信息中的通过数量为49
	assert.Equal(t, 49, resultTotals.Pass)
}

func TestPrintSummary(t *testing.T) {
	// 解析控制集合的 JSON 文件
	controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
	// 如果有错误发生，输出错误信息
	if err != nil {
		t.Error(err)
	}

	// 获取控制集合的总结统计信息
	resultTotals := getSummaryTotals(controlsCollection)
	// 保存标准输出，将标准输出重定向到管道
	rescueStdout := os.Stdout
	r, w, _ := os.Pipe()
	os.Stdout = w
# 打印总结信息
printSummary(resultTotals, "totals")
# 关闭写入端口
w.Close()
# 读取所有的输入
out, _ := ioutil.ReadAll(r)
# 恢复标准输出
os.Stdout = rescueStdout

# 断言输出内容包含指定字符串
assert.Contains(t, string(out), "49 checks PASS\n12 checks FAIL\n14 checks WARN\n0 checks INFO\n\n")
}

# 测试不带总结信息的漂亮打印
func TestPrettyPrintNoSummary(t *testing.T):
    # 解析控制集合的 JSON 文件
	controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
	if err != nil:
		t.Error(err)
	
	# 获取总结信息
	resultTotals := getSummaryTotals(controlsCollection)
	# 保存标准输出
	rescueStdout := os.Stdout
	# 创建管道
	r, w, _ := os.Pipe()
	# 重定向标准输出到管道
	os.Stdout = w
	# 设置不带总结信息的标志为真
	noSummary = true
	# 漂亮打印控制集合的第一个元素和总结信息
	prettyPrint(controlsCollection[0], resultTotals)
// 关闭写入端口
w.Close()
// 读取管道中的所有数据
out, _ := ioutil.ReadAll(r)
// 恢复标准输出
os.Stdout = rescueStdout
// 断言输出中不包含特定字符串
assert.NotContains(t, string(out), "49 checks PASS")
}

func TestPrettyPrintSummary(t *testing.T) {
	// 解析控制集合的 JSON 文件
	controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
	if err != nil {
		t.Error(err)
	}

	// 获取控制集合的总结统计
	resultTotals := getSummaryTotals(controlsCollection)
	// 保存标准输出
	rescueStdout := os.Stdout
	// 创建管道
	r, w, _ := os.Pipe()
	// 重定向标准输出到管道
	os.Stdout = w
	// 设置不显示总结
	noSummary = false
	// 打印漂亮的总结
	prettyPrint(controlsCollection[0], resultTotals)
	// 关闭写入端口
	w.Close()
// 从读取器中读取所有数据，并存储在out变量中
out, _ := ioutil.ReadAll(r)

// 将标准输出重定向到rescueStdout
os.Stdout = rescueStdout

// 断言out中包含"49 checks PASS"字符串
assert.Contains(t, string(out), "49 checks PASS")
}

// 测试函数，测试写入标准输出的输出是否包含总数
func TestWriteStdoutOutputNoTotal(t *testing.T) {
	// 从文件中解析控制集合
	controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
	if err != nil {
		t.Error(err)
	}

	// 保存当前标准输出
	rescueStdout := os.Stdout
	// 创建管道，将标准输出重定向到管道写入端
	r, w, _ := os.Pipe()
	os.Stdout = w
	// 设置noTotals为true
	noTotals = true
	// 将控制集合写入标准输出
	writeStdoutOutput(controlsCollection)
	// 关闭管道写入端
	w.Close()
	// 从读取器中读取所有数据，并存储在out变量中
	out, _ := ioutil.ReadAll(r)
	// 将标准输出重定向回rescueStdout
	os.Stdout = rescueStdout
# 使用断言检查输出中是否不包含特定字符串
assert.NotContains(t, string(out), "49 checks PASS")
# 测试写入标准输出的总数
func TestWriteStdoutOutputTotal(t *testing.T):
    # 解析控制集合的 JSON 文件
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil:
        t.Error(err)
    # 保存标准输出
    rescueStdout := os.Stdout
    # 创建管道
    r, w, _ := os.Pipe()
    # 重定向标准输出到管道
    os.Stdout = w
    # 设置 noTotals 为 false
    noTotals = false
    # 写入标准输出
    writeStdoutOutput(controlsCollection)
    # 关闭管道写端
    w.Close()
    # 读取管道中的输出
    out, _ := ioutil.ReadAll(r)
# 将标准输出重定向到 rescueStdout 变量
os.Stdout = rescueStdout

# 使用断言检查 out 字符串中是否包含 "49 checks PASS"，并将结果传递给测试对象 t
assert.Contains(t, string(out), "49 checks PASS")
}

# 从指定文件中解析控制信息的 JSON 文件，并返回 Controls 对象的切片
func parseControlsJsonFile(filepath string) ([]*check.Controls, error) {
    # 声明一个指向 Controls 对象的切片的变量 result
    var result []*check.Controls

    # 读取指定文件的内容到变量 d 中
    d, err := ioutil.ReadFile(filepath)
    if err != nil {
        return nil, err
    }
    # 将读取的 JSON 数据解析为 Controls 对象的切片，并将结果存储到 result 变量中
    err = json.Unmarshal(d, &result)
    if err != nil {
        return nil, err
    }

    # 返回解析后的 Controls 对象的切片
    return result, nil
}
// 根据文件路径解析结果的 JSON 文件，返回解析后的结果和可能的错误
func parseResultJsonFile(filepath string) (JsonOutputFormat, error) {
	// 声明一个用于存储 JSON 结果的变量
	var result JsonOutputFormat

	// 读取文件内容
	d, err := ioutil.ReadFile(filepath)
	// 如果读取文件出错，则返回空结果和错误
	if err != nil {
		return result, err
	}
	// 解析 JSON 数据到 result 变量中
	err = json.Unmarshal(d, &result)
	// 如果解析 JSON 出错，则返回空结果和错误
	if err != nil {
		return result, err
	}

	// 返回解析后的结果和空错误
	return result, nil
}

// 根据文件路径解析没有总数的结果的 JSON 文件，返回解析后的结果和可能的错误
func parseResultNoTotalsJsonFile(filepath string) ([]*check.Controls, error) {
	// 声明一个用于存储 JSON 结果的变量
	var result []*check.Controls

	// 读取文件内容
	d, err := ioutil.ReadFile(filepath)
	// 如果读取文件出错，则返回空结果和错误
	if err != nil {
		// 如果发生错误，返回空值和错误信息
		return nil, err
	}
	// 将数据解析为 JSON 格式并存储到 result 变量中
	err = json.Unmarshal(d, &result)
	// 如果发生错误，返回空值和错误信息
	if err != nil {
		return nil, err
	}

	// 返回解析后的结果和空值
	return result, nil
}

// 为测试加载配置文件
func loadConfigForTest() (*viper.Viper, error) {
	// 创建一个带有数据的 viper 对象
	viperWithData := viper.New()
	// 设置配置文件路径
	viperWithData.SetConfigFile("../cfg/config.yaml")
	// 读取配置文件
	if err := viperWithData.ReadInConfig(); err != nil {
		// 如果发生错误，返回空值和错误信息
		return nil, err
	}
	// 返回带有数据的 viper 对象和空值
	return viperWithData, nil
}

// 定义一个恢复函数类型
type restoreFn func()
# 创建一个临时目录用于存放假的可执行文件，并返回一个恢复函数和错误信息
func fakeExecutableInPath(execFile, execCode string) (restoreFn, error) {
    # 获取当前环境的 PATH 变量
    pathenv := os.Getenv("PATH")
    # 创建一个临时目录
    tmp, err := ioutil.TempDir("", "TestfakeExecutableInPath")
    if err != nil {
        return nil, err
    }
    # 获取当前工作目录
    wd, err := os.Getwd()
    if err != nil {
        return nil, err
    }
    # 如果传入的可执行文件代码不为空，则将代码写入临时目录下的可执行文件中
    if len(execCode) > 0 {
        ioutil.WriteFile(filepath.Join(tmp, execFile), []byte(execCode), 0700)
    } else {
        # 否则，创建一个新的可执行文件，并设置权限为0700
        f, err := os.OpenFile(execFile, os.O_CREATE|os.O_EXCL, 0700)
        if err != nil {
            return nil, err
        }
		// 关闭文件
		err = f.Close()
		// 如果关闭文件时出现错误，返回空和错误信息
		if err != nil {
			return nil, err
		}
	}

	// 设置环境变量 PATH，将临时目录和原始 PATH 拼接起来
	err = os.Setenv("PATH", fmt.Sprintf("%s:%s", tmp, pathenv))
	// 如果设置环境变量时出现错误，返回空和错误信息
	if err != nil {
		return nil, err
	}

	// 定义恢复环境的函数
	restorePath := func() {
		// 删除临时目录
		os.RemoveAll(tmp)
		// 切换回原始工作目录
		os.Chdir(wd)
		// 恢复原始的 PATH 环境变量
		os.Setenv("PATH", pathenv)
	}

	// 返回恢复环境的函数和空错误信息
	return restorePath, nil
}
# 定义一个函数 prunePath，该函数返回一个恢复函数和一个错误
func prunePath() (restoreFn, error) {
    # 获取当前环境变量中的 PATH 值
    pathenv := os.Getenv("PATH")
    # 将环境变量中的 PATH 设置为空
    err := os.Setenv("PATH", "")
    # 如果设置失败，返回空和错误
    if err != nil {
        return nil, err
    }
    # 定义一个恢复函数，用于恢复 PATH 环境变量的值
    restorePath := func() {
        os.Setenv("PATH", pathenv)
    }
    # 返回恢复函数和空错误
    return restorePath, nil
}
```