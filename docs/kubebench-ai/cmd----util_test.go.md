# `kubebench-aquasecurity\cmd\util_test.go`

```go
// 版权声明
// 2017年版权归Aqua Security Software Ltd. <info@aquasec.com>所有
// 根据Apache许可证2.0版（“许可证”）获得许可；
// 除非符合许可证的规定，否则您不得使用此文件。
// 您可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则不得根据许可证分发软件
// 根据许可证，以“原样”分发，不附带任何担保或条件，无论是明示的还是暗示的。
// 请参阅许可证以了解特定语言的权限和限制

package cmd

import (
    "github.com/magiconair/properties/assert"
    "io/ioutil"
    "os"
    "path/filepath"
    "reflect"
    "strconv"
    "testing"

    "github.com/aquasecurity/kube-bench/check"
    "github.com/spf13/viper"
)

var g string
var e []error
var eIndex int

// 模拟ps命令的输出
func fakeps(proc string) string {
    return g
}

// 模拟stat命令的输出
func fakestat(file string) (os.FileInfo, error) {
    err := e[eIndex]
    eIndex++
    return nil, err
}

// 测试VerifyBin函数
func TestVerifyBin(t *testing.T) {
    // 测试用例
    cases := []struct {
        proc  string
        psOut string
        exp   bool
    }{
        {proc: "single", psOut: "single", exp: true},
        {proc: "single", psOut: "", exp: false},
        {proc: "two words", psOut: "two words", exp: true},
        {proc: "two words", psOut: "", exp: false},
        {proc: "cmd", psOut: "cmd param1 param2", exp: true},
        {proc: "cmd param", psOut: "cmd param1 param2", exp: true},
        {proc: "cmd param", psOut: "cmd", exp: false},
        {proc: "cmd", psOut: "cmd x \ncmd y", exp: true},
        {proc: "cmd y", psOut: "cmd x \ncmd y", exp: true},
        {proc: "cmd", psOut: "/usr/bin/cmd", exp: true},
        {proc: "cmd", psOut: "kube-cmd", exp: false},
        {proc: "cmd", psOut: "/usr/bin/kube-cmd", exp: false},
    }

    // 设置psFunc为fakeps函数
    psFunc = fakeps
}
    # 遍历 cases 列表，获取索引 id 和元素 c
    for id, c := range cases {
        # 使用测试框架运行子测试，并使用索引 id 作为子测试的名称
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 设置全局变量 g 为 c.psOut
            g = c.psOut
            # 调用 verifyBin 函数，传入 c.proc 作为参数，获取返回值并赋给变量 v
            v := verifyBin(c.proc)
            # 如果返回值 v 不等于 c.exp，则输出错误信息并终止测试
            if v != c.exp {
                t.Fatalf("Expected %v got %v", c.exp, v)
            }
        })
    }
// 测试查找可执行文件的函数
func TestFindExecutable(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        candidates []string // 我们考虑的可执行文件列表
        psOut      string   // ps 的假输出
        exp        string   // 我们期望在 (假的) ps 输出中找到的可执行文件
        expErr     bool
    }{
        {candidates: []string{"one", "two", "three"}, psOut: "two", exp: "two"},
        {candidates: []string{"one", "two", "three"}, psOut: "two three", exp: "two"},
        {candidates: []string{"one double", "two double", "three double"}, psOut: "two double is running", exp: "two double"},
        {candidates: []string{"one", "two", "three"}, psOut: "blah", expErr: true},
        {candidates: []string{"one double", "two double", "three double"}, psOut: "two", expErr: true},
        {candidates: []string{"apiserver", "kube-apiserver"}, psOut: "kube-apiserver", exp: "kube-apiserver"},
        {candidates: []string{"apiserver", "kube-apiserver", "hyperkube-apiserver"}, psOut: "kube-apiserver", exp: "kube-apiserver"},
    }

    // 使用 fakeps 作为 psFunc
    psFunc = fakeps
    // 遍历测试用例
    for id, c := range cases {
        // 运行子测试
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            // 设置 ps 输出
            g = c.psOut
            // 调用 findExecutable 函数
            e, err := findExecutable(c.candidates)
            // 检查结果是否符合预期
            if e != c.exp {
                t.Fatalf("Expected %v got %v", c.exp, e)
            }
            // 检查是否符合预期的错误情况
            if err == nil && c.expErr {
                t.Fatalf("Expected error")
            }
            // 检查是否出现了不符合预期的错误
            if err != nil && !c.expErr {
                t.Fatalf("Didn't expect error: %v", err)
            }
        })
    }
}

// 测试获取二进制文件的函数
func TestGetBinaries(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        config    map[string]interface{}
        psOut     string
        exp       map[string]string
        expectErr bool
    }

    // 创建新的 viper 实例
    v := viper.New()
    // 使用 fakeps 作为 psFunc
    psFunc = fakeps
}
    # 遍历 cases 列表，获取索引 id 和元素 c
    for id, c := range cases {
        # 使用测试框架运行子测试，子测试名称为 id 的字符串形式
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 设置全局变量 g 为 c.psOut
            g = c.psOut
            # 遍历 c.config 中的键值对，将值设置到 v 中
            for k, val := range c.config {
                v.Set(k, val)
            }
            # 调用 getBinaries 函数获取二进制数据，传入参数 v 和 check.MASTER
            m, err := getBinaries(v, check.MASTER)
            # 如果 c.expectErr 为 true，则判断 err 是否为 nil，如果是则输出错误信息
            if c.expectErr {
                if err == nil {
                    t.Fatal("Got nil Expected error")
                }
            } 
            # 如果 c.expectErr 为 false，且 m 与 c.exp 不相等，则输出错误信息
            else if !reflect.DeepEqual(m, c.exp) {
                t.Fatalf("Got %v\nExpected %v", m, c.exp)
            }
        })
    }
}
// 测试多词替换函数
func TestMultiWordReplace(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        input   string
        sub     string
        subname string
        output  string
    }{
        {input: "Here's a file with no substitutions", sub: "blah", subname: "blah", output: "Here's a file with no substitutions"},
        {input: "Here's a file with a substitution", sub: "blah", subname: "substitution", output: "Here's a file with a blah"},
        {input: "Here's a file with multi-word substitutions", sub: "multi word", subname: "multi-word", output: "Here's a file with 'multi word' substitutions"},
        {input: "Here's a file with several several substitutions several", sub: "blah", subname: "several", output: "Here's a file with blah blah substitutions blah"},
    }
    // 遍历测试用例
    for id, c := range cases {
        // 运行子测试
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            // 调用多词替换函数
            s := multiWordReplace(c.input, c.subname, c.sub)
            // 检查结果是否符合预期
            if s != c.output {
                t.Fatalf("Expected %s got %s", c.output, s)
            }
        })
    }
}

// 测试从 kubectl 输出中获取版本号
func Test_getVersionFromKubectlOutput(t *testing.T) {
    // 调用获取版本号函数，传入模拟的 kubectl 输出
    ver := getVersionFromKubectlOutput(`{
  "serverVersion": {
    "major": "1",
    "minor": "8",
    "gitVersion": "v1.8.0"
  }
}`)
    // 检查获取的版本号是否符合预期
    if ver.BaseVersion() != "1.8" {
        t.Fatalf("Expected 1.8 got %s", ver.BaseVersion())
    }

    // 测试处理完全不同的输入
    ver = getVersionFromKubectlOutput("Something completely different")
    // 检查处理结果是否符合预期
    if ver.BaseVersion() != defaultKubeVersion {
        t.Fatalf("Expected %s got %s", defaultKubeVersion, ver.BaseVersion())
    }
}

// 测试查找配置文件函数
func TestFindConfigFile(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        input       []string
        statResults []error
        exp         string
    }{
        {input: []string{"myfile"}, statResults: []error{nil}, exp: "myfile"},
        {input: []string{"thisfile", "thatfile"}, statResults: []error{os.ErrNotExist, nil}, exp: "thatfile"},
        {input: []string{"thisfile", "thatfile"}, statResults: []error{os.ErrNotExist, os.ErrNotExist}, exp: ""},
    }
    # 将 statFunc 设置为 fakestat 函数
    statFunc = fakestat
    # 遍历 cases 列表，id 为索引，c 为值
    for id, c := range cases {
        # 使用测试框架运行子测试
        t.Run(strconv.Itoa(id), func(t *testing.T) {
            # 将 e 设置为 c.statResults
            e = c.statResults
            # 将 eIndex 设置为 0
            eIndex = 0
            # 查找配置文件并将结果赋值给 conf
            conf := findConfigFile(c.input)
            # 如果 conf 不等于 c.exp，则输出错误信息
            if conf != c.exp {
                t.Fatalf("Got %s expected %s", conf, c.exp)
            }
        })
    }
}

# 测试获取配置文件
func TestGetConfigFiles(t *testing.T):
    # 定义测试用例
    cases := []struct {
        config      map[string]interface{}
        exp         map[string]string
        statResults []error
    }

    # 创建 viper 对象
    v := viper.New()
    # 设置 statFunc 为 fakestat

    statFunc = fakestat

    # 遍历测试用例
    for id, c := range cases:
        t.Run(strconv.Itoa(id), func(t *testing.T):
            # 设置配置项
            for k, val := range c.config:
                v.Set(k, val)
            # 设置 e 为 c.statResults
            e = c.statResults
            # 设置 eIndex 为 0

            eIndex = 0

            # 调用 getFiles 函数获取文件
            m := getFiles(v, "config")
            # 检查结果是否符合预期
            if !reflect.DeepEqual(m, c.exp):
                t.Fatalf("Got %v\nExpected %v", m, c.exp)
        })
    }
}

# 测试获取服务文件
func TestGetServiceFiles(t *testing.T):
    # 定义测试用例
    cases := []struct {
        config      map[string]interface{}
        exp         map[string]string
        statResults []error
    }

    # 创建 viper 对象
    v := viper.New()
    # 设置 statFunc 为 fakestat

    statFunc = fakestat

    # 遍历测试用例
    for id, c := range cases:
        t.Run(strconv.Itoa(id), func(t *testing.T):
            # 设置配置项
            for k, val := range c.config:
                v.Set(k, val)
            # 设置 e 为 c.statResults
            e = c.statResults
            # 设置 eIndex 为 0

            eIndex = 0

            # 调用 getFiles 函数获取文件
            m := getFiles(v, "service")
            # 检查结果是否符合预期
            if !reflect.DeepEqual(m, c.exp):
                t.Fatalf("Got %v\nExpected %v", m, c.exp)
        })
    }
}

# 测试进行替换
func TestMakeSubsitutions(t *testing.T):
    # 定义测试用例
    cases := []struct {
        input string
        subst map[string]string
        exp   string
        expectedSubs []string
    }{
        {input: "Replace $thisbin", subst: map[string]string{"this": "that"}, exp: "Replace that", expectedSubs: []string{"that"}},
        {input: "Replace $thisbin", subst: map[string]string{"this": "that", "here": "there"}, exp: "Replace that", expectedSubs: []string{"that"}},
        {input: "Replace $thisbin and $herebin", subst: map[string]string{"this": "that", "here": "there"}, exp: "Replace that and there", expectedSubs: []string{"that", "there"}},
    }
    # 遍历测试用例集合
    for _, c := range cases:
        # 使用测试用例的输入作为子测试的名称，并执行测试
        t.Run(c.input, func(t *testing.T):
            # 调用 makeSubstitutions 函数，对输入进行替换操作，返回替换后的字符串和替换的次数
            s, subs := makeSubstitutions(c.input, "bin", c.subst)
            # 检查替换后的字符串是否符合预期，如果不符合则输出错误信息
            if s != c.exp:
                t.Fatalf("Got %s expected %s", s, c.exp)
            # 使用 assert.Equal 函数检查替换的次数是否符合预期
            assert.Equal(t, c.expectedSubs, subs)
        )
    )
}
// 测试获取配置文件路径的函数
func TestGetConfigFilePath(t *testing.T) {
    var err error
    // 创建临时目录
    cfgDir, err = ioutil.TempDir("", "kube-bench-test")
    if err != nil {
        t.Fatalf("Failed to create temp directory")
    }
    // 在函数返回时删除临时目录
    defer os.RemoveAll(cfgDir)
    d := filepath.Join(cfgDir, "cis-1.4")
    // 创建临时目录
    err = os.Mkdir(d, 0766)
    if err != nil {
        t.Fatalf("Failed to create temp dir")
    }
    // 创建临时文件
    err = ioutil.WriteFile(filepath.Join(d, "master.yaml"), []byte("hello world"), 0666)
    if err != nil {
        t.Logf("Failed to create temp file")
    }

    cases := []struct {
        benchmarkVersion string
        succeed          bool
        exp              string
    }{
        {benchmarkVersion: "cis-1.4", succeed: true, exp: d},
        {benchmarkVersion: "cis-1.5", succeed: false, exp: ""},
        {benchmarkVersion: "1.1", succeed: false, exp: ""},
    }

    // 遍历测试用例
    for _, c := range cases {
        t.Run(c.benchmarkVersion, func(t *testing.T) {
            // 获取配置文件路径
            path, err := getConfigFilePath(c.benchmarkVersion, "/master.yaml")
            if c.succeed {
                if err != nil {
                    t.Fatalf("Error %v", err)
                }
                if path != c.exp {
                    t.Fatalf("Got %s expected %s", path, c.exp)
                }
            } else {
                if err == nil {
                    t.Fatalf("Expected Error, but none")
                }
            }
        })
    }
}

// 测试版本递减的函数
func TestDecrementVersion(t *testing.T) {

    cases := []struct {
        kubeVersion string
        succeed     bool
        exp         string
    }{
        {kubeVersion: "1.13", succeed: true, exp: "1.12"},
        {kubeVersion: "1.15", succeed: true, exp: "1.14"},
        {kubeVersion: "1.11", succeed: true, exp: "1.10"},
        {kubeVersion: "1.1", succeed: true, exp: ""},
        {kubeVersion: "invalid", succeed: false, exp: ""},
    }
}
    # 遍历测试用例集合
    for _, c := range cases:
        # 调用decrementVersion函数，得到返回值
        rv := decrementVersion(c.kubeVersion)
        # 如果测试用例预期成功
        if c.succeed:
            # 检查返回值是否与预期值相等
            if c.exp != rv:
                t.Fatalf("decrementVersion(%q) - Got %q expected %s", c.kubeVersion, rv, c.exp)
        # 如果测试用例预期失败
        else:
            # 检查返回值是否为空字符串
            if len(rv) > 0:
                t.Fatalf("decrementVersion(%q) - Expected empty string but Got %s", c.kubeVersion, rv)
// 测试函数，用于测试从目录中获取 YAML 文件
func TestGetYamlFilesFromDir(t *testing.T) {
    // 创建临时目录
    cfgDir, err := ioutil.TempDir("", "kube-bench-test")
    if err != nil {
        t.Fatalf("Failed to create temp directory")
    }
    // 延迟删除临时目录
    defer os.RemoveAll(cfgDir)

    // 创建子目录 cis-1.4
    d := filepath.Join(cfgDir, "cis-1.4")
    err = os.Mkdir(d, 0766)
    if err != nil {
        t.Fatalf("Failed to create temp dir")
    }

    // 写入文件 something.yaml
    err = ioutil.WriteFile(filepath.Join(d, "something.yaml"), []byte("hello world"), 0666)
    if err != nil {
        t.Fatalf("error writing file %v", err)
    }
    // 写入文件 config.yaml
    err = ioutil.WriteFile(filepath.Join(d, "config.yaml"), []byte("hello world"), 0666)
    if err != nil {
        t.Fatalf("error writing file %v", err)
    }

    // 从目录中获取 YAML 文件
    files, err := getYamlFilesFromDir(d)
    if err != nil {
        t.Fatalf("Unexpected error: %v", err)
    }
    // 检查获取的文件数量是否符合预期
    if len(files) != 1 {
        t.Fatalf("Expected to find one file, found %d", len(files))
    }

    // 检查获取的文件名是否符合预期
    if files[0] != filepath.Join(d, "something.yaml") {
        t.Fatalf("Expected to find something.yaml, found %s", files[0])
    }
}

// 测试函数，用于测试从 kubectl 输出中获取平台名称
func Test_getPlatformNameFromKubectlOutput(t *testing.T) {
    // 定义测试参数和预期结果
    type args struct {
        s string
    }
    tests := []struct {
        name string
        args args
        want string
    }{
        {
            name: "eks",
            args: args{s: "v1.17.9-eks-4c6976"},
            want: "eks",
        },
        {
            name: "gke",
            args: args{s: "v1.17.6-gke.1"},
            want: "gke",
        },
        {
            name: "unknown",
            args: args{s: "v1.17.6"},
            want: "",
        },
        {
            name: "empty string",
            args: args{s: ""},
            want: "",
        },
    }
    // 遍历测试用例
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // 调用函数并检查结果是否符合预期
            if got := getPlatformNameFromVersion(tt.args.s); got != tt.want {
                t.Errorf("getPlatformNameFromKubectlOutput() = %v, want %v", got, tt.want)
            }
        })
    }
}
func Test_getPlatformBenchmarkVersion(t *testing.T) {
    // 定义测试函数的参数结构
    type args struct {
        platform string
    }
    // 定义测试用例
    tests := []struct {
        name string
        args args
        want string
    }{
        {
            name: "eks",
            args: args{
                platform: "eks",
            },
            want: "eks-1.0",
        },
        {
            name: "gke",
            args: args{
                platform: "gke",
            },
            want: "gke-1.0",
        },
        {
            name: "unknown",
            args: args{
                platform: "rh",
            },
            want: "",
        },
        {
            name: "empty",
            args: args{
                platform: "",
            },
            want: "",
        },
    }
    // 遍历测试用例
    for _, tt := range tests {
        // 运行单个测试用例
        t.Run(tt.name, func(t *testing.T) {
            // 检查函数返回值是否符合预期
            if got := getPlatformBenchmarkVersion(tt.args.platform); got != tt.want {
                t.Errorf("getPlatformBenchmarkVersion() = %v, want %v", got, tt.want)
            }
        })
    }
}
```