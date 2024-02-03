# `kubebench-aquasecurity\cmd\common_test.go`

```go
// 版权声明，版权归 Aqua Security Software Ltd. 所有
//
// 根据 Apache 许可证 2.0 版本授权
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// 除非适用法律要求或书面同意，否则按"原样"分发软件
// 没有任何明示或暗示的担保或条件
// 请参阅许可证以了解特定语言下的权限和限制

package cmd

import (
    "encoding/json"
    "errors"
    "fmt"
    "io/ioutil"
    "os"
    "path"
    "path/filepath"
    "testing"
    "time"

    "github.com/aquasecurity/kube-bench/check"
    "github.com/spf13/viper"
    "github.com/stretchr/testify/assert"
)

// 定义 JSON 输出格式的结构体
type JsonOutputFormat struct {
    Controls     []*check.Controls `json:"Controls"`
    TotalSummary map[string]int    `json:"Totals"`
}

// 定义不包含 Totals 的 JSON 输出格式的结构体
type JsonOutputFormatNoTotals struct {
    Controls []*check.Controls `json:"Controls"`
}

// 测试解析跳过的 ID
func TestParseSkipIds(t *testing.T) {
    // 解析跳过的 ID，返回一个 map
    skipMap := parseSkipIds("4.12,4.13,5")
    // 检查是否存在特定的 ID
    _, fourTwelveExists := skipMap["4.12"]
    _, fourThirteenExists := skipMap["4.13"]
    _, fiveExists := skipMap["5"]
    _, other := skipMap["G1"]
    // 断言特定的 ID 是否存在
    assert.True(t, fourThirteenExists)
    assert.True(t, fourTwelveExists)
    assert.True(t, fiveExists)
    assert.False(t, other)
}

// 测试创建运行过滤器
func TestNewRunFilter(t *testing.T) {
    // 定义测试用例结构体
    type TestCase struct {
        Name       string
        FilterOpts FilterOpts
        Group      *check.Group
        Check      *check.Check
        Expected   bool
    }

    // 定义测试用例
    testCases := []TestCase{
        {
            Name:       "test1",
            FilterOpts: FilterOpts{},
            Group:      &check.Group{},
            Check:      &check.Check{},
            Expected:   true,
        },
    }

    // 遍历测试用例并执行测试
    for _, testCase := range testCases {
        t.Run(testCase.Name, func(t *testing.T) {
            filter, _ := NewRunFilter(testCase.FilterOpts)
            assert.Equal(t, testCase.Expected, filter(testCase.Group, testCase.Check))
        })
    }
}
    t.Run("Should return error when both group and check flags are used", func(t *testing.T) {
        // 定义测试用例，验证当同时使用组和检查标志时是否会返回错误
        // given
        opts := FilterOpts{GroupList: "G1", CheckList: "C1"}
        // 初始化过滤器选项，设置组列表为"G1"，检查列表为"C1"
        // when
        _, err := NewRunFilter(opts)
        // 调用NewRunFilter函数，传入过滤器选项，获取返回结果和错误
        // then
        assert.EqualError(t, err, "group option and check option can't be used together")
        // 使用断言验证返回的错误信息是否符合预期
    })
func TestIsMaster(t *testing.T) {
    // 定义测试用例
    testCases := []struct {
        name            string
        cfgFile         string
        getBinariesFunc func(*viper.Viper, check.NodeType) (map[string]string, error)
        isMaster        bool
    }{
        {
            // 第一个测试用例：有效配置，是主节点，所有组件都在运行
            name:    "valid config, is master and all components are running",
            cfgFile: "../cfg/config.yaml",
            getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                return map[string]string{"apiserver": "kube-apiserver"}, nil
            },
            isMaster: true,
        },
        {
            // 第二个测试用例：有效配置，是主节点，但并非所有组件都在运行
            name:    "valid config, is master and but not all components are running",
            cfgFile: "../cfg/config.yaml",
            getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                return map[string]string{}, nil
            },
            isMaster: false,
        },
        {
            // 第三个测试用例：有效配置，是主节点，但并非所有组件都在运行，并且无法找到所有二进制文件
            name:    "valid config, is master, not all components are running and fails to find all binaries",
            cfgFile: "../cfg/config.yaml",
            getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                return map[string]string{}, errors.New("failed to find binaries")
            },
            isMaster: false,
        },
        {
            // 第四个测试用例：有效配置，不包括主节点
            name:     "valid config, does not include master",
            cfgFile:  "../hack/node_only.yaml",
            isMaster: false,
        },
    }
    // 保存旧的配置目录，并设置新的配置目录
    cfgDirOld := cfgDir
    cfgDir = "../cfg"
    // 在函数返回前恢复旧的配置目录
    defer func() {
        cfgDir = cfgDirOld
    }()

    // 执行代码
    execCode := `#!/bin/sh
    echo "Server Version: v1.13.10"
    `
    // 在 PATH 中创建一个假的可执行文件
    restore, err := fakeExecutableInPath("kubectl", execCode)
    if err != nil {
        t.Fatal("Failed when calling fakeExecutableInPath ", err)
    }
    // 在函数返回前恢复 PATH
    defer restore()
}
    # 遍历测试用例数组
    for _, tc := range testCases:
        # 将配置文件设置为当前测试用例的配置文件
        cfgFile = tc.cfgFile
        # 初始化配置
        initConfig()

        # 保存旧的获取二进制文件函数
        oldGetBinariesFunc := getBinariesFunc
        # 将获取二进制文件函数设置为当前测试用例的获取二进制文件函数
        getBinariesFunc = tc.getBinariesFunc
        # 在函数返回时，恢复旧的获取二进制文件函数，并清空配置文件
        defer func() {
            getBinariesFunc = oldGetBinariesFunc
            cfgFile = ""
        }()

        # 断言当前测试用例的是否为主节点与实际是否为主节点相等
        assert.Equal(t, tc.isMaster, isMaster(), tc.name)
    }
func TestMapToCISVersion(t *testing.T) {
    // 加载测试配置文件
    viperWithData, err := loadConfigForTest()
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
    }
    // 加载版本映射
    kubeToBenchmarkMap, err := loadVersionMapping(viperWithData)
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
    }

    // 测试用例
    cases := []struct {
        kubeVersion string  // Kubernetes 版本
        succeed     bool    // 是否成功
        exp         string  // 期望值
        expErr      string  // 期望错误信息
    }{
        {kubeVersion: "1.9", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.9"},
        {kubeVersion: "1.11", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.11"},
        {kubeVersion: "1.12", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.12"},
        {kubeVersion: "1.13", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.13"},
        {kubeVersion: "1.14", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: 1.14"},
        {kubeVersion: "1.15", succeed: true, exp: "cis-1.5"},
        {kubeVersion: "1.16", succeed: true, exp: "cis-1.6"},
        {kubeVersion: "1.17", succeed: true, exp: "cis-1.6"},
        {kubeVersion: "1.18", succeed: true, exp: "cis-1.6"},
        {kubeVersion: "1.19", succeed: true, exp: "cis-1.6"},
        {kubeVersion: "gke-1.0", succeed: true, exp: "gke-1.0"},
        {kubeVersion: "ocp-3.10", succeed: true, exp: "rh-0.7"},
        {kubeVersion: "ocp-3.11", succeed: true, exp: "rh-0.7"},
        {kubeVersion: "unknown", succeed: false, exp: "", expErr: "unable to find a matching Benchmark Version match for kubernetes version: unknown"},
    }
    for _, c := range cases {
        // 遍历测试用例集合
        rv, err := mapToBenchmarkVersion(kubeToBenchmarkMap, c.kubeVersion)
        // 调用函数，将 Kubernetes 版本映射到基准版本，返回结果和可能的错误
        if c.succeed {
            // 如果测试用例预期成功
            if err != nil {
                // 如果出现了意外错误，输出错误信息
                t.Errorf("[%q]-Unexpected error: %v", c.kubeVersion, err)
            }

            if len(rv) == 0 {
                // 如果返回值为空，输出错误信息
                t.Errorf("[%q]-missing return value", c.kubeVersion)
            }

            if c.exp != rv {
                // 如果返回值与预期不符，输出错误信息
                t.Errorf("[%q]- expected %q but Got %q", c.kubeVersion, c.exp, rv)
            }

        } else {
            // 如果测试用例预期失败
            if c.exp != rv {
                // 如果返回值与预期不符，输出错误信息
                t.Errorf("[%q]-mapToBenchmarkVersion kubeversion: %q Got %q expected %s", c.kubeVersion, c.kubeVersion, rv, c.exp)
            }

            if c.expErr != err.Error() {
                // 如果返回的错误信息与预期不符，输出错误信息
                t.Errorf("[%q]-mapToBenchmarkVersion expected Error: %q instead Got %q", c.kubeVersion, c.expErr, err.Error())
            }
        }
    }
func TestLoadVersionMapping(t *testing.T) {
    // 定义一个函数，用于设置默认配置并返回 viper.Viper 对象
    setDefault := func(v *viper.Viper, key string, value interface{}) *viper.Viper {
        v.SetDefault(key, value)
        return v
    }

    // 加载测试用的配置文件
    viperWithData, err := loadConfigForTest()
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
    }

    // 定义测试用例
    cases := []struct {
        n       string
        v       *viper.Viper
        succeed bool
    }{
        {n: "empty", v: viper.New(), succeed: false},
        {
            n:       "novals",
            v:       setDefault(viper.New(), "version_mapping", "novals"),
            succeed: false,
        },
        {
            n:       "good",
            v:       viperWithData,
            succeed: true,
        },
    }
    // 遍历测试用例
    for _, c := range cases {
        // 加载版本映射
        rv, err := loadVersionMapping(c.v)
        if c.succeed {
            // 如果预期成功，检查是否有错误
            if err != nil {
                t.Errorf("[%q]-Unexpected error: %v", c.n, err)
            }
            // 检查返回值是否为空
            if len(rv) == 0 {
                t.Errorf("[%q]-missing mapping value", c.n)
            }
        } else {
            // 如果预期失败，检查是否有错误
            if err == nil {
                t.Errorf("[%q]-Expected error but got none", c.n)
            }
        }
    }
}

func TestGetBenchmarkVersion(t *testing.T) {
    // 加载测试用的配置文件
    viperWithData, err := loadConfigForTest()
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
    }

    // 定义一个函数类型，用于测试获取基准版本
    type getBenchmarkVersionFnToTest func(kubeVersion, benchmarkVersion string, v *viper.Viper) (string, error)

    // 使用假的 kubectl 执行函数进行测试
    withFakeKubectl := func(kubeVersion, benchmarkVersion string, v *viper.Viper, fn getBenchmarkVersionFnToTest) (string, error) {
        // 定义假的 kubectl 执行代码
        execCode := `#!/bin/sh
        echo '{"serverVersion": {"major": "1", "minor": "15", "gitVersion": "v1.15.10"}}'
        `
        // 调用 fakeExecutableInPath 函数来模拟可执行文件
        restore, err := fakeExecutableInPath("kubectl", execCode)
        if err != nil {
            t.Fatal("Failed when calling fakeExecutableInPath ", err)
        }
        defer restore()

        // 调用传入的函数进行测试
        return fn(kubeVersion, benchmarkVersion, v)
    // 定义一个匿名函数，用于在没有路径的情况下获取基准版本
    withNoPath := func(kubeVersion, benchmarkVersion string, v *viper.Viper, fn getBenchmarkVersionFnToTest) (string, error) {
        // 调用 prunePath 函数，获取恢复函数和可能的错误
        restore, err := prunePath()
        // 如果有错误，则输出错误信息并终止测试
        if err != nil {
            t.Fatal("Failed when calling prunePath ", err)
        }
        // 延迟执行恢复函数
        defer restore()

        // 调用传入的函数 fn，并返回结果
        return fn(kubeVersion, benchmarkVersion, v)
    }

    // 定义一个函数类型 getBenchmarkVersionFn
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
        // 测试用例4
        {n: "kubeVersion", kubeVersion: "1.15", benchmarkVersion: "", v: viperWithData, exp: "cis-1.5", callFn: withNoPath, succeed: true},
        // 测试用例5
        {n: "ocpVersion310", kubeVersion: "ocp-3.10", benchmarkVersion: "", v: viperWithData, exp: "rh-0.7", callFn: withNoPath, succeed: true},
        // 测试用例6
        {n: "ocpVersion311", kubeVersion: "ocp-3.11", benchmarkVersion: "", v: viperWithData, exp: "rh-0.7", callFn: withNoPath, succeed: true},
        // 测试用例7
        {n: "gke10", kubeVersion: "gke-1.0", benchmarkVersion: "", v: viperWithData, exp: "gke-1.0", callFn: withNoPath, succeed: true},
    }
    # 遍历测试用例集合
    for _, c := range cases:
        # 调用测试函数，获取返回值和错误信息
        rv, err := c.callFn(c.kubeVersion, c.benchmarkVersion, c.v, getBenchmarkVersion)
        # 如果期望成功
        if c.succeed:
            # 如果有错误，输出错误信息
            if err != nil:
                t.Errorf("[%q]-Unexpected error: %v", c.n, err)
            # 如果返回值长度为0，输出缺少返回值的错误信息
            if len(rv) == 0:
                t.Errorf("[%q]-missing return value", c.n)
            # 如果期望值和返回值不相等，输出期望值和实际值不匹配的错误信息
            if c.exp != rv:
                t.Errorf("[%q]- expected %q but Got %q", c.n, c.exp, rv)
        # 如果期望失败
        else:
            # 如果没有错误，输出期望有错误但实际没有错误的信息
            if err == nil:
                t.Errorf("[%q]-Expected error but got none", c.n)
func TestValidTargets(t *testing.T) {
    // 加载测试配置文件
    viperWithData, err := loadConfigForTest()
    // 如果加载配置文件出错，打印错误信息并终止测试
    if err != nil {
        t.Fatalf("Unable to load config file %v", err)
    }
    // 定义测试用例
    cases := []struct {
        name      string
        benchmark string
        targets   []string
        expected  bool
    }{
        // 第一个测试用例
        {
            name:      "cis-1.5 no dummy",
            benchmark: "cis-1.5",
            targets:   []string{"master", "node", "controlplane", "etcd", "dummy"},
            expected:  false,
        },
        // 第二个测试用例
        {
            name:      "cis-1.5 valid",
            benchmark: "cis-1.5",
            targets:   []string{"master", "node", "controlplane", "etcd", "policies"},
            expected:  true,
        },
        // 第三个测试用例
        {
            name:      "cis-1.6 no Pikachu",
            benchmark: "cis-1.6",
            targets:   []string{"master", "node", "controlplane", "etcd", "Pikachu"},
            expected:  false,
        },
        // 第四个测试用例
        {
            name:      "cis-1.6 valid",
            benchmark: "cis-1.6",
            targets:   []string{"master", "node", "controlplane", "etcd", "policies"},
            expected:  true,
        },
        // 第五个测试用例
        {
            name:      "gke-1.0 valid",
            benchmark: "gke-1.0",
            targets:   []string{"master", "node", "controlplane", "etcd", "policies", "managedservices"},
            expected:  true,
        },
        // 第六个测试用例
        {
            name:      "aks-1.0 valid",
            benchmark: "aks-1.0",
            targets:   []string{"node", "policies", "controlplane", "managedservices"},
            expected:  true,
        },
        // 第七个测试用例
        {
            name:      "eks-1.0 valid",
            benchmark: "eks-1.0",
            targets:   []string{"node", "policies", "controlplane", "managedservices"},
            expected:  true,
        },
    }
}
    # 遍历测试用例集合
    for _, c := range cases:
        # 使用测试用例的名称创建子测试
        t.Run(c.name, func(t *testing.T) {
            # 调用 validTargets 函数，传入测试用例的参数和 viperWithData
            ret, err := validTargets(c.benchmark, c.targets, viperWithData)
            # 如果返回错误不为空，输出错误信息
            if err != nil:
                t.Fatalf("Expected nil error, got: %v", err)
            # 如果返回结果与预期不符，输出错误信息
            if ret != c.expected:
                t.Fatalf("Expected %t, got %t", c.expected, ret)
        })
    }
func TestIsEtcd(t *testing.T) {
    // 定义测试用例
    testCases := []struct {
        name            string
        cfgFile         string
        getBinariesFunc func(*viper.Viper, check.NodeType) (map[string]string, error)
        isEtcd          bool
    }{
        // 第一个测试用例
        {
            name:    "valid config, is etcd and all components are running",
            cfgFile: "../cfg/config.yaml",
            // 定义获取二进制文件的函数
            getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                return map[string]string{"etcd": "etcd"}, nil
            },
            isEtcd: true,
        },
        // 第二个测试用例
        {
            name:    "valid config, is etcd and but not all components are running",
            cfgFile: "../cfg/config.yaml",
            // 定义获取二进制文件的函数
            getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                return map[string]string{}, nil
            },
            isEtcd: false,
        },
        // 第三个测试用例
        {
            name:    "valid config, is etcd, not all components are running and fails to find all binaries",
            cfgFile: "../cfg/config.yaml",
            // 定义获取二进制文件的函数
            getBinariesFunc: func(viper *viper.Viper, nt check.NodeType) (strings map[string]string, i error) {
                return map[string]string{}, errors.New("failed to find binaries")
            },
            isEtcd: false,
        },
        // 第四个测试用例
        {
            name:    "valid config, does not include etcd",
            cfgFile: "../hack/node_only.yaml",
            isEtcd:  false,
        },
    }
    // 保存旧的配置目录，并设置新的配置目录
    cfgDirOld := cfgDir
    cfgDir = "../cfg"
    // 在函数返回前恢复原始配置目录
    defer func() {
        cfgDir = cfgDirOld
    }()

    // 定义要执行的代码
    execCode := `#!/bin/sh
    echo "Server Version: v1.15.03"
    `
    // 在 PATH 中创建一个假的可执行文件
    restore, err := fakeExecutableInPath("kubectl", execCode)
    if err != nil {
        t.Fatal("Failed when calling fakeExecutableInPath ", err)
    }
    // 在函数返回前恢复 PATH
    defer restore()
}
    # 遍历测试用例数组
    for _, tc := range testCases:
        # 将配置文件设置为当前测试用例的配置文件
        cfgFile = tc.cfgFile
        # 初始化配置
        initConfig()

        # 保存旧的获取二进制文件函数
        oldGetBinariesFunc := getBinariesFunc
        # 将获取二进制文件函数设置为当前测试用例的获取二进制文件函数
        getBinariesFunc = tc.getBinariesFunc
        # 在函数返回时，恢复旧的获取二进制文件函数，并清空配置文件
        defer func() {
            getBinariesFunc = oldGetBinariesFunc
            cfgFile = ""
        }()

        # 断言当前测试用例的 isEtcd() 函数的返回值是否与预期相等
        assert.Equal(t, tc.isEtcd, isEtcd(), tc.name)
    }
# 测试写入结果到 JSON 文件的函数
func TestWriteResultToJsonFile(t *testing.T) {
    # 在函数结束时重置全局变量
    defer func() {
        controlsCollection = []*check.Controls{}
        jsonFmt = false
        outputFile = ""
    }()
    var err error
    # 设置 JSON 格式为 true
    jsonFmt = true
    # 设置输出文件路径为临时目录下的时间戳
    outputFile = path.Join(os.TempDir(), fmt.Sprintf("%d", time.Now().UnixNano()))

    # 解析控制集合的 JSON 文件
    controlsCollection, err = parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }
    # 写入输出
    writeOutput(controlsCollection)

    var expect JsonOutputFormat
    var result JsonOutputFormat
    # 解析输出文件的 JSON 结果
    result, err = parseResultJsonFile(outputFile)
    if err != nil {
        t.Error(err)
    }
    # 解析预期结果的 JSON 文件
    expect, err = parseResultJsonFile("./testdata/result.json")
    if err != nil {
        t.Error(err)
    }
    # 断言预期结果和实际结果是否相等
    assert.Equal(t, expect, result)
}

# 测试写入无总数结果到 JSON 文件的函数
func TestWriteResultNoTotalsToJsonFile(t *testing.T) {
    # 在函数结束时重置全局变量
    defer func() {
        controlsCollection = []*check.Controls{}
        jsonFmt = false
        outputFile = ""
    }()
    var err error
    # 设置 JSON 格式为 true
    jsonFmt = true
    # 设置输出文件路径为临时目录下的时间戳
    outputFile = path.Join(os.TempDir(), fmt.Sprintf("%d", time.Now().UnixNano()))
    # 设置无总数标志为 true
    noTotals = true

    # 解析控制集合的 JSON 文件
    controlsCollection, err = parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }
    # 写入输出
    writeOutput(controlsCollection)

    var expect []*check.Controls
    var result []*check.Controls
    # 解析无总数结果的 JSON 文件
    result, err = parseResultNoTotalsJsonFile(outputFile)
    if err != nil {
        t.Error(err)
    }
    # 解析预期结果的无总数 JSON 文件
    expect, err = parseResultNoTotalsJsonFile("./testdata/result_no_totals.json")
    if err != nil {
        t.Error(err)
    }
    # 断言预期结果和实际结果是否相等
    assert.Equal(t, expect, result)
}

# 测试退出码选择函数
func TestExitCodeSelection(t *testing.T) {
    # 设置退出码为 10
    exitCode = 10
    # 解析所有控制都通过的 JSON 文件
    controlsCollectionAllPassed, errPassed := parseControlsJsonFile("./testdata/passedControlsCollection.json")
    if errPassed != nil {
        t.Error(errPassed)
    }
    # 解析带有失败的控制集合的 JSON 文件
    controlsCollectionWithFailures, errFailure := parseControlsJsonFile("./testdata/controlsCollection.json")
    # 如果错误不为空，输出错误信息
    if errFailure != nil:
        t.Error(errFailure)

    # 获取所有通过的控制集的退出代码
    exitCodePassed := exitCodeSelection(controlsCollectionAllPassed)
    # 断言退出代码为0
    assert.Equal(t, 0, exitCodePassed)

    # 获取包含失败的控制集的退出代码
    exitCodeFailure := exitCodeSelection(controlsCollectionWithFailures)
    # 断言退出代码为10
    assert.Equal(t, 10, exitCodeFailure)
}


func TestGenerationDefaultEnvAudit(t *testing.T) {
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
          op: has
          value: "true"
        set: true
    remediation: |
      Edit the config file /this/is/a/file/path and set SomeSampleFlag to true.
    scored: true
`)
    controls, err := check.NewControls(check.MASTER, input)
    assert.NoError(t, err)

    binSubs := []string{"TestBinPath"}
    generateDefaultEnvAudit(controls, binSubs)

    expectedAuditEnv := fmt.Sprintf("cat \"/proc/$(/bin/ps -C %s -o pid= | tr -d ' ')/environ\" | tr '\\0' '\\n'", binSubs[0])
    assert.Equal(t, expectedAuditEnv, controls.Groups[1].Checks[0].AuditEnv)
}


func TestGetSummaryTotals(t *testing.T) {
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }

    resultTotals := getSummaryTotals(controlsCollection)
    assert.Equal(t, 12, resultTotals.Fail)
    assert.Equal(t, 14, resultTotals.Warn)
    assert.Equal(t, 0, resultTotals.Info)
    assert.Equal(t, 49, resultTotals.Pass)
}


func TestPrintSummary(t *testing.T) {
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }

    resultTotals := getSummaryTotals(controlsCollection)
    rescueStdout := os.Stdout
    r, w, _ := os.Pipe()
    os.Stdout = w
    printSummary(resultTotals, "totals")
    w.Close()
    out, _ := ioutil.ReadAll(r)
    os.Stdout = rescueStdout

    assert.Contains(t, string(out), "49 checks PASS\n12 checks FAIL\n14 checks WARN\n0 checks INFO\n\n")
}


func TestPrettyPrintNoSummary(t *testing.T) {
    // 从指定路径解析控制集合的 JSON 文件，返回控制集合和可能的错误
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    // 如果有错误，输出错误信息
    if err != nil {
        t.Error(err)
    }

    // 获取控制集合的总结统计信息
    resultTotals := getSummaryTotals(controlsCollection)
    
    // 保存当前的标准输出，并创建一个新的管道
    rescueStdout := os.Stdout
    r, w, _ := os.Pipe()
    os.Stdout = w
    
    // 设置不输出总结信息
    noSummary = true
    
    // 打印控制集合的第一个控制和总结统计信息
    prettyPrint(controlsCollection[0], resultTotals)
    
    // 关闭管道写入端
    w.Close()
    
    // 读取管道中的输出
    out, _ := ioutil.ReadAll(r)
    
    // 恢复原来的标准输出
    os.Stdout = rescueStdout
    
    // 断言输出中不包含特定字符串
    assert.NotContains(t, string(out), "49 checks PASS")
# 测试函数，用于测试输出漂亮的摘要
func TestPrettyPrintSummary(t *testing.T) {
    # 解析控制集合的 JSON 文件，返回控制集合和错误信息
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }

    # 获取摘要总数
    resultTotals := getSummaryTotals(controlsCollection)
    rescueStdout := os.Stdout
    r, w, _ := os.Pipe()
    os.Stdout = w
    noSummary = false
    # 输出漂亮的摘要
    prettyPrint(controlsCollection[0], resultTotals)
    w.Close()
    out, _ := ioutil.ReadAll(r)
    os.Stdout = rescueStdout

    # 断言输出包含特定内容
    assert.Contains(t, string(out), "49 checks PASS")
}

# 测试函数，用于测试写入标准输出没有总数
func TestWriteStdoutOutputNoTotal(t *testing.T) {
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }

    rescueStdout := os.Stdout
    r, w, _ := os.Pipe()
    os.Stdout = w
    noTotals = true
    # 写入标准输出
    writeStdoutOutput(controlsCollection)
    w.Close()
    out, _ := ioutil.ReadAll(r)
    os.Stdout = rescueStdout

    # 断言输出不包含特定内容
    assert.NotContains(t, string(out), "49 checks PASS")
}

# 测试函数，用于测试写入标准输出有总数
func TestWriteStdoutOutputTotal(t *testing.T) {
    controlsCollection, err := parseControlsJsonFile("./testdata/controlsCollection.json")
    if err != nil {
        t.Error(err)
    }

    rescueStdout := os.Stdout

    r, w, _ := os.Pipe()

    os.Stdout = w
    noTotals = false
    # 写入标准输出
    writeStdoutOutput(controlsCollection)
    w.Close()
    out, _ := ioutil.ReadAll(r)

    os.Stdout = rescueStdout

    # 断言输出包含特定内容
    assert.Contains(t, string(out), "49 checks PASS")
}

# 解析控制集合的 JSON 文件，返回控制集合和错误信息
func parseControlsJsonFile(filepath string) ([]*check.Controls, error) {
    var result []*check.Controls

    d, err := ioutil.ReadFile(filepath)
    if err != nil {
        return nil, err
    }
    err = json.Unmarshal(d, &result)
    if err != nil {
        return nil, err
    }

    return result, nil
}

# 解析结果的 JSON 文件，返回 JSON 输出格式和错误信息
func parseResultJsonFile(filepath string) (JsonOutputFormat, error) {
    var result JsonOutputFormat

    d, err := ioutil.ReadFile(filepath)
    if err != nil {
        return result, err
    }
    err = json.Unmarshal(d, &result)
    # 如果错误不为空，返回结果和错误
    if err != nil:
        return result, err
    # 否则，返回结果和空错误
    return result, nil
// 解析不包含总数的 JSON 文件，返回检查控制列表和可能的错误
func parseResultNoTotalsJsonFile(filepath string) ([]*check.Controls, error) {
    var result []*check.Controls

    // 读取文件内容
    d, err := ioutil.ReadFile(filepath)
    if err != nil {
        return nil, err
    }
    // 反序列化 JSON 数据到 result 变量
    err = json.Unmarshal(d, &result)
    if err != nil {
        return nil, err
    }

    return result, nil
}

// 为测试加载配置文件
func loadConfigForTest() (*viper.Viper, error) {
    // 创建一个新的 viper 实例
    viperWithData := viper.New()
    // 设置配置文件路径
    viperWithData.SetConfigFile("../cfg/config.yaml")
    // 读取配置文件
    if err := viperWithData.ReadInConfig(); err != nil {
        return nil, err
    }
    return viperWithData, nil
}

// 定义一个函数类型 restoreFn
type restoreFn func()

// 模拟可执行文件在 PATH 中的情况
func fakeExecutableInPath(execFile, execCode string) (restoreFn, error) {
    // 获取当前的 PATH 环境变量
    pathenv := os.Getenv("PATH")
    // 创建临时目录
    tmp, err := ioutil.TempDir("", "TestfakeExecutableInPath")
    if err != nil {
        return nil, err
    }

    // 获取当前工作目录
    wd, err := os.Getwd()
    if err != nil {
        return nil, err
    }

    // 如果 execCode 长度大于 0，则将 execCode 写入临时目录下的 execFile 文件中
    if len(execCode) > 0 {
        ioutil.WriteFile(filepath.Join(tmp, execFile), []byte(execCode), 0700)
    } else {
        // 否则创建一个空的可执行文件
        f, err := os.OpenFile(execFile, os.O_CREATE|os.O_EXCL, 0700)
        if err != nil {
            return nil, err
        }
        err = f.Close()
        if err != nil {
            return nil, err
        }
    }

    // 将临时目录添加到 PATH 环境变量中
    err = os.Setenv("PATH", fmt.Sprintf("%s:%s", tmp, pathenv))
    if err != nil {
        return nil, err
    }

    // 定义一个恢复函数，用于清理临时目录和恢复 PATH 环境变量
    restorePath := func() {
        os.RemoveAll(tmp)
        os.Chdir(wd)
        os.Setenv("PATH", pathenv)
    }

    return restorePath, nil
}

// 清空 PATH 环境变量
func prunePath() (restoreFn, error) {
    // 获取当前的 PATH 环境变量
    pathenv := os.Getenv("PATH")
    // 清空 PATH 环境变量
    err := os.Setenv("PATH", "")
    if err != nil {
        return nil, err
    }
    // 定义一个恢复函数，用于恢复 PATH 环境变量
    restorePath := func() {
        os.Setenv("PATH", pathenv)
    }
    return restorePath, nil
}
```