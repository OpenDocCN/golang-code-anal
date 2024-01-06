# `kubebench-aquasecurity\cmd\util_test.go`

```
// 该代码段是版权声明和许可证信息，指明了代码的版权归属和使用许可
// 导入所需的包
package cmd

import (
    "github.com/magiconair/properties/assert"  // 导入断言包
    "io/ioutil"  // 导入读取文件内容的包
    "os"  // 导入操作系统功能的包
# 导入路径处理、反射、字符串转换、测试等包
"path/filepath"
"reflect"
"strconv"
"testing"

# 导入 kube-bench 和 viper 包
"github.com/aquasecurity/kube-bench/check"
"github.com/spf13/viper"
)

# 声明全局变量 g，用于存储字符串
var g string
# 声明全局变量 e，用于存储错误列表
var e []error
# 声明全局变量 eIndex，用于记录错误列表的索引
var eIndex int

# 定义函数 fakeps，接收进程名参数，返回字符串
func fakeps(proc string) string {
    return g
}

# 定义函数 fakestat，接收文件名参数，返回文件信息和错误
func fakestat(file string) (os.FileInfo, error) {
    # 获取错误列表中的错误，并递增索引
    err := e[eIndex]
    eIndex++
// 返回空值和错误
return nil, err
}

// 测试验证二进制文件
func TestVerifyBin(t *testing.T) {
    // 定义测试用例
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
    }
// 定义测试用例，包括可执行文件的候选列表、假的 ps 输出和期望在（假的）ps 输出中找到的可执行文件
func TestFindExecutable(t *testing.T) {
    cases := []struct {
        candidates []string // 可执行文件的候选列表
        psOut      string   // ps 的假输出
        exp        string   // 我们期望在（假的）ps 输出中找到的可执行文件
    }{
        {proc: "cmd", psOut: "/usr/bin/kube-cmd", exp: false}, // 设置测试用例的参数
    }

    psFunc = fakeps // 设置 psFunc 为 fakeps
    for id, c := range cases { // 遍历测试用例
        t.Run(strconv.Itoa(id), func(t *testing.T) { // 运行测试
            g = c.psOut // 设置 g 为测试用例的 ps 输出
            v := verifyBin(c.proc) // 调用 verifyBin 函数，传入测试用例的可执行文件名
            if v != c.exp { // 检查实际结果是否与期望结果一致
                t.Fatalf("Expected %v got %v", c.exp, v) // 如果不一致，输出错误信息
            }
        })
    }
}
		expErr     bool
	}{
		// 定义测试用例，包括候选项、预期输出、预期错误等信息
		{candidates: []string{"one", "two", "three"}, psOut: "two", exp: "two"},
		{candidates: []string{"one", "two", "three"}, psOut: "two three", exp: "two"},
		{candidates: []string{"one double", "two double", "three double"}, psOut: "two double is running", exp: "two double"},
		{candidates: []string{"one", "two", "three"}, psOut: "blah", expErr: true},
		{candidates: []string{"one double", "two double", "three double"}, psOut: "two", expErr: true},
		{candidates: []string{"apiserver", "kube-apiserver"}, psOut: "kube-apiserver", exp: "kube-apiserver"},
		{candidates: []string{"apiserver", "kube-apiserver", "hyperkube-apiserver"}, psOut: "kube-apiserver", exp: "kube-apiserver"},
	}

	// 设置 psFunc 为 fakeps
	psFunc = fakeps
	// 遍历测试用例
	for id, c := range cases {
		// 运行子测试
		t.Run(strconv.Itoa(id), func(t *testing.T) {
			// 设置 ps 输出为测试用例中的预期输出
			g = c.psOut
			// 调用 findExecutable 函数，获取实际输出和错误信息
			e, err := findExecutable(c.candidates)
			// 检查实际输出是否符合预期输出
			if e != c.exp {
				t.Fatalf("Expected %v got %v", c.exp, e)
			}
// 如果没有错误并且期望有错误发生，则输出错误信息
if err == nil && c.expErr {
    t.Fatalf("Expected error")
}

// 如果有错误发生并且不期望有错误发生，则输出错误信息
if err != nil && !c.expErr {
    t.Fatalf("Didn't expect error: %v", err)
}
```

```
// 测试获取二进制文件
func TestGetBinaries(t *testing.T) {
    // 测试用例
    cases := []struct {
        config    map[string]interface{}
        psOut     string
        exp       map[string]string
        expectErr bool
    }{
        {
            // 测试用例1
            config:    map[string]interface{}{"components": []string{"apiserver"}, "apiserver": map[string]interface{}{"bins": []string{"apiserver", "kube-apiserver"}},
		{
			// 设置 psOut 为 "kube-apiserver"，期望结果为 map{"apiserver": "kube-apiserver"}，不期望出现错误
			psOut:     "kube-apiserver",
			exp:       map[string]string{"apiserver": "kube-apiserver"},
			expectErr: false,
		},
		{
			// "thing" 不在组件列表中
			psOut:     "kube-apiserver thing",
			exp:       map[string]string{"apiserver": "kube-apiserver"},
			expectErr: false,
		},
		{
			// "anotherthing" 在组件列表中，但没有定义
			psOut:     "kube-apiserver thing",
			exp:       map[string]string{"apiserver": "kube-apiserver"},
			expectErr: false,
		},
		{
			// 多于一个组件
			psOut:     "kube-apiserver \nthing",
			exp:       map[string]string{"apiserver": "kube-apiserver", "thing": "thing"},
		{
			// 期望没有错误发生
			expectErr: false,
		},
		{
			// 将默认的二进制文件名映射到组件名称
			psOut:     "kube-apiserver \notherthing some params",
			exp:       map[string]string{"apiserver": "kube-apiserver", "thing": "thing"},
			expectErr: false,
		},
		{
			// 缺少必需的组件
			psOut:     "otherthing some params",
			exp:       map[string]string{"apiserver": "kube-apiserver", "thing": "thing"},
			expectErr: true,
		}

	v := viper.New()  // 创建一个新的 viper 实例
	psFunc = fakeps  // 设置 psFunc 为 fakeps

	for id, c := range cases {  // 遍历 cases 列表
# 使用测试框架运行测试用例
t.Run(strconv.Itoa(id), func(t *testing.T) {
    # 将变量 g 设置为 c.psOut 的值
    g = c.psOut
    # 遍历配置项，将其设置到变量 v 中
    for k, val := range c.config {
        v.Set(k, val)
    }
    # 调用 getBinaries 函数获取二进制数据
    m, err := getBinaries(v, check.MASTER)
    # 如果期望出现错误
    if c.expectErr {
        # 如果 err 为 nil，则输出错误信息
        if err == nil {
            t.Fatal("Got nil Expected error")
        }
    } 
    # 如果不期望出现错误，且获取的二进制数据与期望的不相等
    else if !reflect.DeepEqual(m, c.exp) {
        # 输出获取的数据和期望的数据
        t.Fatalf("Got %v\nExpected %v", m, c.exp)
    }
})
# 结束测试用例
}
# 定义测试函数 TestMultiWordReplace
func TestMultiWordReplace(t *testing.T) {
    # 定义测试用例
    cases := []struct {
        input   string
# 定义结构体，包含输入字符串、替换字符串、替换名称和输出字符串
sub     string
subname string
output  string
}{
# 测试用例，包括输入字符串、替换字符串、替换名称和预期输出字符串
{input: "Here's a file with no substitutions", sub: "blah", subname: "blah", output: "Here's a file with no substitutions"},
{input: "Here's a file with a substitution", sub: "blah", subname: "substitution", output: "Here's a file with a blah"},
{input: "Here's a file with multi-word substitutions", sub: "multi word", subname: "multi-word", output: "Here's a file with 'multi word' substitutions"},
{input: "Here's a file with several several substitutions several", sub: "blah", subname: "several", output: "Here's a file with blah blah substitutions blah"},
}
# 遍历测试用例
for id, c := range cases {
	t.Run(strconv.Itoa(id), func(t *testing.T) {
		# 调用 multiWordReplace 函数进行替换
		s := multiWordReplace(c.input, c.subname, c.sub)
		# 检查实际输出是否与预期输出一致
		if s != c.output {
			t.Fatalf("Expected %s got %s", c.output, s)
		}
	})
}
}

func Test_getVersionFromKubectlOutput(t *testing.T) {
// 从kubectl输出中获取版本信息，并进行版本比较
ver := getVersionFromKubectlOutput(`{
  "serverVersion": {
    "major": "1",
    "minor": "8",
    "gitVersion": "v1.8.0"
  }
}`)
// 如果版本不是1.8，则输出错误信息
if ver.BaseVersion() != "1.8" {
	t.Fatalf("Expected 1.8 got %s", ver.BaseVersion())
}

// 从kubectl输出中获取版本信息，并进行版本比较
ver = getVersionFromKubectlOutput("Something completely different")
// 如果版本不是默认的kube版本，则输出错误信息
if ver.BaseVersion() != defaultKubeVersion {
	t.Fatalf("Expected %s got %s", defaultKubeVersion, ver.BaseVersion())
}

// 测试查找配置文件的函数
func TestFindConfigFile(t *testing.T) {
	// 定义测试用例
	cases := []struct {
		input       []string
		statResults []error  // 定义一个存储错误信息的切片
		exp         string    // 定义一个存储期望结果的字符串
	}{
		{input: []string{"myfile"}, statResults: []error{nil}, exp: "myfile"},  // 设置输入参数、期望的错误信息和期望的结果
		{input: []string{"thisfile", "thatfile"}, statResults: []error{os.ErrNotExist, nil}, exp: "thatfile"},  // 设置输入参数、期望的错误信息和期望的结果
		{input: []string{"thisfile", "thatfile"}, statResults: []error{os.ErrNotExist, os.ErrNotExist}, exp: ""},  // 设置输入参数、期望的错误信息和期望的结果
	}

	statFunc = fakestat  // 设置 statFunc 为 fakestat 函数
	for id, c := range cases {  // 遍历 cases 切片
		t.Run(strconv.Itoa(id), func(t *testing.T) {  // 运行测试用例
			e = c.statResults  // 将错误信息赋值给 e
			eIndex = 0  // 将错误信息索引设置为 0
			conf := findConfigFile(c.input)  // 调用 findConfigFile 函数查找配置文件
			if conf != c.exp {  // 判断实际结果是否与期望结果相符
				t.Fatalf("Got %s expected %s", conf, c.exp)  // 输出错误信息
			}
		})
	}
}
func TestGetConfigFiles(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        config      map[string]interface{}  // 配置信息
        exp         map[string]string       // 期望结果
        statResults []error                 // 状态结果
    }{
        {
            // 第一个测试用例
            config:      map[string]interface{}{"components": []string{"apiserver"}, "apiserver": map[string]interface{}{"confs": []string{"apiserver", "kube-apiserver"}}},
            statResults: []error{os.ErrNotExist, nil},  // 状态结果
            exp:         map[string]string{"apiserver": "kube-apiserver"},  // 期望结果
        },
        {
            // 第二个测试用例
            // 组件 "thing" 不在组件列表中
            config: map[string]interface{}{
                "components": []string{"apiserver"},
                "apiserver":  map[string]interface{}{"confs": []string{"apiserver", "kube-apiserver"}},
                "thing":      map[string]interface{}{"confs": []string{"/my/file/thing"}}},
            statResults: []error{os.ErrNotExist, nil},  // 状态结果
            exp:         map[string]string{"apiserver": "kube-apiserver"},  // 期望结果
		},
		{
			// More than one component
			// 配置包含多个组件
			config: map[string]interface{}{
				"components": []string{"apiserver", "thing"},
				// 每个组件的配置信息
				"apiserver":  map[string]interface{}{"confs": []string{"apiserver", "kube-apiserver"}},
				"thing":      map[string]interface{}{"confs": []string{"/my/file/thing"}}},
			// 模拟每个组件的状态结果
			statResults: []error{os.ErrNotExist, nil, nil},
			// 期望的结果
			exp:         map[string]string{"apiserver": "kube-apiserver", "thing": "/my/file/thing"},
		},
		{
			// Default thing to specified default config
			// 将默认配置指定给指定的默认组件
			config: map[string]interface{}{
				"components": []string{"apiserver", "thing"},
				// 每个组件的配置信息
				"apiserver":  map[string]interface{}{"confs": []string{"apiserver", "kube-apiserver"}},
				"thing":      map[string]interface{}{"confs": []string{"/my/file/thing"}, "defaultconf": "another/thing"}},
			// 模拟每个组件的状态结果
			statResults: []error{os.ErrNotExist, nil, os.ErrNotExist},
			// 期望的结果
			exp:         map[string]string{"apiserver": "kube-apiserver", "thing": "another/thing"},
		},
		{
			// 默认将组件名称映射到默认配置
			config: map[string]interface{}{
				"components": []string{"apiserver", "thing"},  // 组件列表
				"apiserver":  map[string]interface{}{"confs": []string{"apiserver", "kube-apiserver"}},  // apiserver配置
				"thing":      map[string]interface{}{"confs": []string{"/my/file/thing"}},  // thing配置
			},
			statResults: []error{os.ErrNotExist, nil, os.ErrNotExist},  // 状态结果列表
			exp:         map[string]string{"apiserver": "kube-apiserver", "thing": "thing"},  // 期望结果
		},
	}

	v := viper.New()  // 创建新的 viper 实例
	statFunc = fakestat  // 设置 statFunc 为 fakestat 函数

	for id, c := range cases {  // 遍历测试用例
		t.Run(strconv.Itoa(id), func(t *testing.T) {  // 运行测试
			for k, val := range c.config {  // 遍历配置
				v.Set(k, val)  // 设置 viper 实例的配置
			}
			e = c.statResults  // 设置 e 为 statResults
			eIndex = 0  // 设置 eIndex 为 0
// 获取指定类型为"config"的文件列表
m := getFiles(v, "config")
// 如果获取到的文件列表与期望的文件列表不相等，则输出错误信息
if !reflect.DeepEqual(m, c.exp) {
    t.Fatalf("Got %v\nExpected %v", m, c.exp)
}
// 结束当前测试用例
})
}

// 测试获取服务文件列表的函数
func TestGetServiceFiles(t *testing.T) {
    // 测试用例参数
    cases := []struct {
        config      map[string]interface{}
        exp         map[string]string
        statResults []error
    }{
        {
            // 测试用例1
            config: map[string]interface{}{
                "components": []string{"kubelet"},
                "kubelet":    map[string]interface{}{"svc": []string{"kubelet", "10-kubeadm.conf"}},
            },
		{
			// 定义 statResults 数组，包含两个错误对象：os.ErrNotExist 和 nil
			statResults: []error{os.ErrNotExist, nil},
			// 定义 exp 字典，包含一个键值对："kubelet" 对应 "10-kubeadm.conf"
			exp:         map[string]string{"kubelet": "10-kubeadm.conf"},
		},
		{
			// 组件 "thing" 不在组件列表中
			config: map[string]interface{}{
				// 定义 components 列表，包含一个元素 "kubelet"
				"components": []string{"kubelet"},
				// 定义 kubelet 对象，包含一个键值对："svc" 对应 ["kubelet", "10-kubeadm.conf"]
				"kubelet":    map[string]interface{}{"svc": []string{"kubelet", "10-kubeadm.conf"}},
				// 定义 thing 对象，包含一个键值对："svc" 对应 ["/my/file/thing"]
				"thing":      map[string]interface{}{"svc": []string{"/my/file/thing"}},
			},
			// 定义 statResults 数组，包含两个错误对象：os.ErrNotExist 和 nil
			statResults: []error{os.ErrNotExist, nil},
			// 定义 exp 字典，包含一个键值对："kubelet" 对应 "10-kubeadm.conf"
			exp:         map[string]string{"kubelet": "10-kubeadm.conf"},
		},
		{
			// 多于一个组件
			config: map[string]interface{}{
				// 定义 components 列表，包含两个元素 "kubelet" 和 "thing"
				"components": []string{"kubelet", "thing"},
				// 定义 kubelet 对象，包含一个键值对："svc" 对应 ["kubelet", "10-kubeadm.conf"]
				"kubelet":    map[string]interface{}{"svc": []string{"kubelet", "10-kubeadm.conf"}},
				// 定义 thing 对象，包含一个键值对："svc" 对应 ["/my/file/thing"]
				"thing":      map[string]interface{}{"svc": []string{"/my/file/thing"}},
			},
		{
			// 设置 statResults 数组，包含三个错误值
			statResults: []error{os.ErrNotExist, nil, nil},
			// 设置期望的结果，包含两个键值对
			exp:         map[string]string{"kubelet": "10-kubeadm.conf", "thing": "/my/file/thing"},
		},
		{
			// 将默认的 thing 设置为指定的默认服务
			config: map[string]interface{}{
				"components": []string{"kubelet", "thing"},
				"kubelet":    map[string]interface{}{"svc": []string{"kubelet", "10-kubeadm.conf"}},
				"thing":      map[string]interface{}{"svc": []string{"/my/file/thing"}, "defaultsvc": "another/thing"},
			},
			// 设置 statResults 数组，包含三个错误值
			statResults: []error{os.ErrNotExist, nil, os.ErrNotExist},
			// 设置期望的结果，包含两个键值对
			exp:         map[string]string{"kubelet": "10-kubeadm.conf", "thing": "another/thing"},
		},
		{
			// 将默认的 thing 设置为组件名称
			config: map[string]interface{}{
				"components": []string{"kubelet", "thing"},
				"kubelet":    map[string]interface{}{"svc": []string{"kubelet", "10-kubeadm.conf"}},
				"thing":      map[string]interface{}{"svc": []string{"/my/file/thing"}},
			},
```

			// 定义一个包含三个错误的切片
			statResults: []error{os.ErrNotExist, nil, os.ErrNotExist},
			// 定义一个包含两个字符串的映射
			exp:         map[string]string{"kubelet": "10-kubeadm.conf", "thing": "thing"},
		},

	}

	// 创建一个新的 viper 对象
	v := viper.New()
	// 将 statFunc 设置为 fakestat

	statFunc = fakestat

	// 遍历测试用例
	for id, c := range cases {
		// 对每个测试用例运行子测试
		t.Run(strconv.Itoa(id), func(t *testing.T) {
			// 遍历测试用例中的配置项，将其设置到 viper 对象中
			for k, val := range c.config {
				v.Set(k, val)
			}
			// 将 e 设置为当前测试用例的 statResults
			e = c.statResults
			// 将 eIndex 设置为 0

			eIndex = 0

			// 调用 getFiles 函数，获取文件列表
			m := getFiles(v, "service")
			// 检查获取的文件列表是否与期望的映射相等，如果不相等则输出错误信息
			if !reflect.DeepEqual(m, c.exp) {
				t.Fatalf("Got %v\nExpected %v", m, c.exp)
			}
// 定义测试函数TestMakeSubsitutions，用于测试makeSubstitutions函数
func TestMakeSubsitutions(t *testing.T) {
	// 定义测试用例
	cases := []struct {
		input string
		subst map[string]string
		exp   string
		expectedSubs []string
	}{
		// 测试用例1
		{input: "Replace $thisbin", subst: map[string]string{"this": "that"}, exp: "Replace that", expectedSubs: []string{"that"}},
		// 测试用例2
		{input: "Replace $thisbin", subst: map[string]string{"this": "that", "here": "there"}, exp: "Replace that", expectedSubs: []string{"that"}},
		// 测试用例3
		{input: "Replace $thisbin and $herebin", subst: map[string]string{"this": "that", "here": "there"}, exp: "Replace that and there", expectedSubs: []string{"that", "there"}},
	}
	// 遍历测试用例
	for _, c := range cases {
		// 对每个测试用例运行子测试
		t.Run(c.input, func(t *testing.T) {
			// 调用makeSubstitutions函数进行替换
			s, subs := makeSubstitutions(c.input, "bin", c.subst)
			// 检查替换后的结果是否符合预期
			if s != c.exp {
				t.Fatalf("Got %s expected %s", s, c.exp)
// 结束当前测试用例
		}
		// 使用断言检查实际结果和期望结果是否相等
		assert.Equal(t, c.expectedSubs, subs)
	})
}

// 测试获取配置文件路径的函数
func TestGetConfigFilePath(t *testing.T) {
	// 声明变量 err
	var err error
	// 创建临时目录作为配置目录
	cfgDir, err = ioutil.TempDir("", "kube-bench-test")
	if err != nil {
		// 如果创建失败，输出错误信息
		t.Fatalf("Failed to create temp directory")
	}
	// 在函数结束时删除临时目录
	defer os.RemoveAll(cfgDir)
	// 创建子目录 cis-1.4
	d := filepath.Join(cfgDir, "cis-1.4")
	err = os.Mkdir(d, 0766)
	if err != nil {
		// 如果创建失败，输出错误信息
		t.Fatalf("Failed to create temp dir")
	}
	// 在子目录中写入文件 master.yaml
	err = ioutil.WriteFile(filepath.Join(d, "master.yaml"), []byte("hello world"), 0666)
	if err != nil {
		// 输出日志，表示创建临时文件失败
		t.Logf("Failed to create temp file")
	}

	// 定义测试用例
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
		// 在测试中运行每个测试用例
		t.Run(c.benchmarkVersion, func(t *testing.T) {
			// 获取配置文件路径和可能的错误
			path, err := getConfigFilePath(c.benchmarkVersion, "/master.yaml")
			// 如果测试用例成功
			if c.succeed {
				// 如果出现错误，输出错误信息
				if err != nil {
					t.Fatalf("Error %v", err)
				}
// 如果路径不等于预期值，则输出错误信息
if path != c.exp {
    t.Fatalf("Got %s expected %s", path, c.exp)
} else {
    // 如果没有错误，输出预期错误信息
    if err == nil {
        t.Fatalf("Expected Error, but none")
    }
}
// 结束测试
})
}

// 测试版本递减
func TestDecrementVersion(t *testing.T) {
// 测试用例
cases := []struct {
    kubeVersion string
    succeed     bool
    exp         string
}{
    // 设置测试参数
    {kubeVersion: "1.13", succeed: true, exp: "1.12"},
// 定义测试用例数组，每个元素包含 kubeVersion（Kubernetes 版本）、succeed（是否成功）、exp（预期结果）
cases := []struct {
    kubeVersion string
    succeed     bool
    exp         string
}{
    {kubeVersion: "1.15", succeed: true, exp: "1.14"},
    {kubeVersion: "1.11", succeed: true, exp: "1.10"},
    {kubeVersion: "1.1", succeed: true, exp: ""},
    {kubeVersion: "invalid", succeed: false, exp: ""},
}

// 遍历测试用例数组
for _, c := range cases {
    // 调用 decrementVersion 函数，获取返回值
    rv := decrementVersion(c.kubeVersion)
    // 如果测试用例成功
    if c.succeed {
        // 检查返回值是否符合预期
        if c.exp != rv {
            t.Fatalf("decrementVersion(%q) - Got %q expected %s", c.kubeVersion, rv, c.exp)
        }
    } else {
        // 如果测试用例失败，检查返回值是否为空
        if len(rv) > 0 {
            t.Fatalf("decrementVersion(%q) - Expected empty string but Got %s", c.kubeVersion, rv)
        }
    }
}
```

```
func TestGetYamlFilesFromDir(t *testing.T) {
```

注释：
```
// 定义测试函数 TestGetYamlFilesFromDir，用于测试从目录中获取 YAML 文件
# 创建临时目录，用于存放配置文件
cfgDir, err := ioutil.TempDir("", "kube-bench-test")
if err != nil {
    t.Fatalf("Failed to create temp directory")
}
# 在函数返回时删除临时目录
defer os.RemoveAll(cfgDir)

# 创建子目录 cis-1.4
d := filepath.Join(cfgDir, "cis-1.4")
err = os.Mkdir(d, 0766)
if err != nil {
    t.Fatalf("Failed to create temp dir")
}

# 在 cis-1.4 目录下写入文件 something.yaml
err = ioutil.WriteFile(filepath.Join(d, "something.yaml"), []byte("hello world"), 0666)
if err != nil {
    t.Fatalf("error writing file %v", err)
}

# 在 cis-1.4 目录下写入文件 config.yaml
err = ioutil.WriteFile(filepath.Join(d, "config.yaml"), []byte("hello world"), 0666)
if err != nil {
    t.Fatalf("error writing file %v", err)
}
	files, err := getYamlFilesFromDir(d)  // 从指定目录获取 YAML 文件列表
	if err != nil {  // 如果出现错误
		t.Fatalf("Unexpected error: %v", err)  // 报告意外错误
	}
	if len(files) != 1 {  // 如果文件列表长度不为1
		t.Fatalf("Expected to find one file, found %d", len(files))  // 报告找到的文件数不符合预期
	}

	if files[0] != filepath.Join(d, "something.yaml") {  // 如果第一个文件不是指定的文件
		t.Fatalf("Expected to find something.yaml, found %s", files[0])  // 报告找到的文件不符合预期
	}
}

func Test_getPlatformNameFromKubectlOutput(t *testing.T) {  // 测试从 kubectl 输出中获取平台名称
	type args struct {
		s string
	}
	tests := []struct {
		name string  // 测试用例名称
		args args  # 定义测试用例的参数结构
		want string  # 定义期望的返回结果类型
	}{
		{
			name: "eks",  # 测试用例名称
			args: args{s: "v1.17.9-eks-4c6976"},  # 测试用例参数
			want: "eks",  # 期望的返回结果
		},
		{
			name: "gke",  # 测试用例名称
			args: args{s: "v1.17.6-gke.1"},  # 测试用例参数
			want: "gke",  # 期望的返回结果
		},
		{
			name: "unknown",  # 测试用例名称
			args: args{s: "v1.17.6"},  # 测试用例参数
			want: "",  # 期望的返回结果
		},
		{
			name: "empty string",  # 测试用例名称
# 定义测试用例
args: args{s: ""},  # 定义参数 s 为空字符串
want: "",  # 期望返回空字符串
},
}
# 遍历测试用例
for _, tt := range tests:
t.Run(tt.name, func(t *testing.T):  # 运行测试用例
if got := getPlatformNameFromVersion(tt.args.s); got != tt.want:  # 调用函数并检查返回值是否符合预期
t.Errorf("getPlatformNameFromKubectlOutput() = %v, want %v", got, tt.want)  # 输出测试结果
}
}

# 定义测试函数
func Test_getPlatformBenchmarkVersion(t *testing.T):
type args struct:  # 定义参数结构
platform string  # 参数为平台名称
tests := []struct:  # 定义测试用例
name string  # 测试用例名称
args args  # 测试用例参数
		want string
	}{
		{
			name: "eks",  # 测试用例名称为 eks
			args: args{  # 参数为 platform: "eks"
				platform: "eks",
			},
			want: "eks-1.0",  # 期望结果为 "eks-1.0"
		},
		{
			name: "gke",  # 测试用例名称为 gke
			args: args{  # 参数为 platform: "gke"
				platform: "gke",
			},
			want: "gke-1.0",  # 期望结果为 "gke-1.0"
		},
		{
			name: "unknown",  # 测试用例名称为 unknown
			args: args{  # 参数为 platform: "rh"
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
	}
}
# 遍历测试用例
for _, tt := range tests {
	# 在测试中运行每个测试用例
	t.Run(tt.name, func(t *testing.T) {
		# 检查获取的平台基准版本是否与预期相符
		if got := getPlatformBenchmarkVersion(tt.args.platform); got != tt.want {
			t.Errorf("getPlatformBenchmarkVersion() = %v, want %v", got, tt.want)
		}
	})
}
```