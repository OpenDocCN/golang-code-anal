# `kubebench-aquasecurity\check\test_test.go`

```go
// 版权声明和许可证信息
// 该代码受 Apache 许可证版本 2.0 的许可
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以获取特定语言的权限和限制

package check

import (
    "fmt"
    "io/ioutil"
    "os"
    "strings"
    "testing"
)

var (
    in       []byte
    controls *Controls
)

func init() {
    var err error
    in, err = ioutil.ReadFile("data")
    if err != nil {
        panic("Failed reading test data: " + err.Error())
    }

    // 在数据文件中替换变量
    user := os.Getenv("USER")
    s := strings.Replace(string(in), "$user", user, -1)

    controls, err = NewControls(MASTER, []byte(s))
    // controls, err = NewControls(MASTER, in)
    if err != nil {
        panic("Failed creating test controls: " + err.Error())
    }
}

func TestTestExecute(t *testing.T) {

    cases := []struct {
        *Check
        str       string
        strConfig string
        strEnv    string
    }

    for _, c := range cases {
        t.Run(c.Text, func(t *testing.T) {
            c.Check.AuditOutput = c.str
            c.Check.AuditConfigOutput = c.strConfig
            c.Check.AuditEnvOutput = c.strEnv
            res, err := c.Check.execute()
            if err != nil {
                t.Errorf(err.Error())
            }
            if !res.testResult {
                t.Errorf("expected:%v, got:%v", true, res)
            }
        })
    }
}

func TestTestExecuteExceptions(t *testing.T) {

    cases := []struct {
        *Check
        str string
    # 定义一个包含测试用例的数组，每个测试用例包含一个检查对象和一个字符串
    {
        # 第一个测试用例，检查对象为 controls.Groups[0].Checks[23]，字符串为 "this is not valid json {} at all"
        {
            controls.Groups[0].Checks[23],
            "this is not valid json {} at all",
        },
        # 第二个测试用例，检查对象为 controls.Groups[0].Checks[24]，字符串为 "{\"key\": \"value\"}"
        {
            controls.Groups[0].Checks[24],
            "{\"key\": \"value\"}",
        },
        # 第三个测试用例，检查对象为 controls.Groups[0].Checks[25]，字符串为 "broken } yaml\nenabled: true"
        {
            controls.Groups[0].Checks[25],
            "broken } yaml\nenabled: true",
        },
        # 第四个测试用例，检查对象为 controls.Groups[0].Checks[26]，字符串为 "currentMasterVersion: 1.11"
        {
            controls.Groups[0].Checks[26],
            "currentMasterVersion: 1.11",
        },
        # 第五个测试用例，检查对象为 controls.Groups[0].Checks[26]，字符串为 "currentMasterVersion: "
        {
            controls.Groups[0].Checks[26],
            "currentMasterVersion: ",
        },
    }

    # 遍历测试用例数组
    for _, c := range cases {
        # 对每个测试用例运行子测试
        t.Run(c.Text, func(t *testing.T) {
            # 设置检查对象的 AuditConfigOutput 为测试用例的字符串
            c.Check.AuditConfigOutput = c.str
            # 执行检查对象的 execute 方法，获取结果和可能的错误
            res, err := c.Check.execute()
            # 如果有错误，输出错误信息
            if err != nil {
                t.Errorf(err.Error())
            }
            # 如果测试结果为真，输出预期值和实际值
            if res.testResult {
                t.Errorf("expected:%v, got:%v", false, res)
            }
        })
    }
# 定义测试函数TestUnmarshal，用于测试unmarshal函数
func TestTestUnmarshal(t *testing.T) {
    # 定义结构体kubeletConfig，包含Kind、ApiVersion和Address三个字段
    type kubeletConfig struct {
        Kind       string
        ApiVersion string
        Address    string
    }
    # 定义测试用例数组cases，包含content、jsonInterface和expectedToFail三个字段
    cases := []struct {
        content        string
        jsonInterface  interface{}
        expectedToFail bool
    }{
        {
            # 第一个测试用例的content、jsonInterface和expectedToFail字段的取值
            `{
            "kind": "KubeletConfiguration",
            "apiVersion": "kubelet.config.k8s.io/v1beta1",
            "address": "0.0.0.0"
            }
            `,
            kubeletConfig{}, # 使用上面定义的kubeletConfig结构体
            false, # 期望不会失败
        }, {
            # 第二个测试用例的content、jsonInterface和expectedToFail字段的取值
            `
kind: KubeletConfiguration
address: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
  enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
tlsCipherSuites:
  - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
  - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
`,
            kubeletConfig{}, # 使用上面定义的kubeletConfig结构体
            false, # 期望不会失败
        },
        {
            # 第三个测试用例的content、jsonInterface和expectedToFail字段的取值
            `
kind: ddress: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta
`,
            kubeletConfig{}, # 使用上面定义的kubeletConfig结构体
            true, # 期望会失败
        },
    }

    # 遍历测试用例数组cases
    for id, c := range cases {
        # 对每个测试用例运行子测试
        t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
            # 调用unmarshal函数，解析content并将结果存入jsonInterface
            err := unmarshal(c.content, &c.jsonInterface)
            # 判断是否有错误发生
            if err != nil {
                # 如果不期望失败，则输出错误信息
                if !c.expectedToFail {
                    t.Errorf("should pass, got error:%v", err)
                }
            } else {
                # 如果期望失败，则输出错误信息
                if c.expectedToFail {
                    t.Errorf("should fail, but passed")
                }
            }
        })
    }
}

# 定义测试函数TestExecuteJSONPath，用于测试executeJSONPath函数
func TestExecuteJSONPath(t *testing.T) {
    # 定义结构体kubeletConfig，包含Kind、ApiVersion和Address三个字段
    type kubeletConfig struct {
        Kind       string
        ApiVersion string
        Address    string
    }
    # 定义测试用例数组cases，包含name、jsonPath、jsonInterface、expectedResult和expectedToFail五个字段
    cases := []struct {
        name           string
        jsonPath       string
        jsonInterface  kubeletConfig
        expectedResult string
        expectedToFail bool
    # 定义测试用例数组，每个元素包含测试名称、JSONPath、JSON接口对象、期望结果和是否期望失败
    cases := []struct {
        name            string
        jsonPath        string
        jsonInterface   interface{}
        expectedResult   string
        expectedToFail  bool
    }{
        # 第一个测试用例：JSONPath 解析成功，但结果不匹配
        {
            "JSONPath parse works, results don't match",
            "{.Kind}",
            kubeletConfig{
                Kind:       "KubeletConfiguration",
                ApiVersion: "kubelet.config.k8s.io/v1beta1",
                Address:    "127.0.0.0",
            },
            "blah",
            true,
        },
        # 第二个测试用例：JSONPath 解析成功，结果匹配
        {
            "JSONPath parse works, results match",
            "{.Kind}",
            kubeletConfig{
                Kind:       "KubeletConfiguration",
                ApiVersion: "kubelet.config.k8s.io/v1beta1",
                Address:    "127.0.0.0",
            },
            "KubeletConfiguration",
            false,
        },
        # 第三个测试用例：JSONPath 解析失败
        {
            "JSONPath parse fails",
            "{.ApiVersion",
            kubeletConfig{
                Kind:       "KubeletConfiguration",
                ApiVersion: "kubelet.config.k8s.io/v1beta1",
                Address:    "127.0.0.0",
            },
            "",
            true,
        },
    }
    # 遍历测试用例数组，执行测试
    for _, c := range cases {
        t.Run(c.name, func(t *testing.T) {
            # 执行 JSONPath 解析，获取结果和错误信息
            result, err := executeJSONPath(c.jsonPath, c.jsonInterface)
            # 如果有错误且不期望失败，则输出错误信息
            if err != nil && !c.expectedToFail {
                t.Fatalf("jsonPath:%q, expectedResult:%q got:%v", c.jsonPath, c.expectedResult, err)
            }
            # 如果结果不符合期望且不期望失败，则输出错误信息
            if c.expectedResult != result && !c.expectedToFail {
                t.Errorf("jsonPath:%q, expectedResult:%q got:%q", c.jsonPath, c.expectedResult, result)
            }
        })
    }
// 测试所有元素是否有效
func TestAllElementsValid(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        source []string
        target []string
        valid  bool
    }
    // 遍历测试用例
    for id, c := range cases {
        // 运行子测试
        t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
            // 检查所有元素是否有效，如果无效则报错
            if !allElementsValid(c.source, c.target) && c.valid {
                t.Errorf("Not All Elements in %q are found in %q", c.source, c.target)
            }
        })
    }
}

// 测试分割并移除最后一个分隔符
func TestSplitAndRemoveLastSeparator(t *testing.T) {
    // 定义测试用例
    cases := []struct {
        source     string
        valid      bool
        elementCnt int
    }{
        {
            source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_AES_128_GCM_SHA256",
            valid:      true,
            elementCnt: 8,
        },
        {
            source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",
            valid:      true,
            elementCnt: 2,
        },
        {
            source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",
            valid:      true,
            elementCnt: 2,
        },
        {
            source:     "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, ",
            valid:      true,
            elementCnt: 2,
        },
        {
            source:     " TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,",
            valid:      true,
            elementCnt: 2,
        },
    }
}
    # 遍历 cases 列表，获取索引 id 和元素 c
    for id, c := range cases {
        # 使用测试框架运行子测试，格式化子测试名称为 id
        t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
            # 调用 splitAndRemoveLastSeparator 函数，将 c.source 按照默认分隔符分割并移除最后一个分隔符
            as := splitAndRemoveLastSeparator(c.source, defaultArraySeparator)
            # 如果分割后的数组长度为 0 且 c.valid 为真，则输出错误信息
            if len(as) == 0 && c.valid {
                t.Errorf("Split did not work with %q", c.source)
            }

            # 如果分割后的数组长度不等于 c.elementCnt，则输出错误信息
            if c.elementCnt != len(as) {
                t.Errorf("Split did not work with %q expected: %d got: %d", c.source, c.elementCnt, len(as))
            }
        })
    }
# 定义一个测试函数，用于比较操作
func TestCompareOp(t *testing.T) {
    # 定义测试用例的结构体
    cases := []struct {
        label                 string
        op                    string
        flagVal               string
        compareValue          string
        expectedResultPattern string
        testResult            bool
    }

    # 遍历测试用例
    for _, c := range cases {
        # 运行测试用例
        t.Run(c.label, func(t *testing.T) {
            # 调用比较操作函数，获取预期结果和测试结果
            expectedResultPattern, testResult := compareOp(c.op, c.flagVal, c.compareValue)
            # 检查预期结果是否与实际结果匹配
            if expectedResultPattern != c.expectedResultPattern {
                t.Errorf("'expectedResultPattern' did not match - op: %q expected:%q  got:%q", c.op, c.expectedResultPattern, expectedResultPattern)
            }
            # 检查测试结果是否与预期结果匹配
            if testResult != c.testResult {
                t.Errorf("'testResult' did not match - lop: %q expected:%t  got:%t", c.op, c.testResult, testResult)
            }
        })
    }
}

# 定义一个测试函数，用于将字符串转换为数字
func TestToNumeric(t *testing.T) {
    # 定义测试用例的结构体
    cases := []struct {
        firstValue     string
        secondValue    string
        expectedToFail bool
    }{
        {
            firstValue:     "a",
            secondValue:    "b",
            expectedToFail: true,
        },
        {
            firstValue:     "5",
            secondValue:    "b",
            expectedToFail: true,
        },
        {
            firstValue:     "5",
            secondValue:    "6",
            expectedToFail: false,
        },
    }

    # 遍历测试用例
    for id, c := range cases {
        # 运行测试用例
        t.Run(fmt.Sprintf("%d", id), func(t *testing.T) {
            # 调用将字符串转换为数字的函数，获取转换结果和可能的错误
            f, s, err := toNumeric(c.firstValue, c.secondValue)
            # 如果预期结果是失败，并且没有出现错误
            if c.expectedToFail && err == nil {
                t.Errorf("Expected error while converting %s and %s", c.firstValue, c.secondValue)
            }
            # 如果预期结果不是失败，并且转换结果不符合预期
            if !c.expectedToFail && (f != 5 || s != 6) {
                t.Errorf("Expected to return %d,%d - got %d,%d", 5, 6, f, s)
            }
        })
    }
}
```