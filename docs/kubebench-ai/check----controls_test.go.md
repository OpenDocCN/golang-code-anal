# `kubebench-aquasecurity\check\controls_test.go`

```go
// 版权声明，声明代码版权和许可证信息
// 该代码遵循 Apache 许可证 2.0 版本
// 可以在符合许可证的情况下使用该文件
// 可以在上述链接获取许可证的副本
// 除非适用法律要求或书面同意，否则不得使用该文件
// 根据许可证分发的软件是基于"按原样"的基础分发的
// 没有任何明示或暗示的担保或条件
// 请查看许可证以获取特定语言的权限和限制

// 导入所需的包
package check

import (
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "encoding/json"  // 导入 json 包，用于 JSON 数据的编解码
    "encoding/xml"  // 导入 xml 包，用于 XML 数据的编解码
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io/ioutil"  // 导入 ioutil 包，用于读取文件内容
    "os"  // 导入 os 包，提供对操作系统功能的访问
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "reflect"  // 导入 reflect 包，用于反射操作
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/aws/aws-sdk-go/aws"  // 导入 AWS SDK 的 aws 包
    "github.com/aws/aws-sdk-go/service/securityhub"  // 导入 AWS SDK 的 securityhub 包
    "github.com/onsi/ginkgo/reporters"  // 导入 ginkgo 的 reporters 包
    "github.com/spf13/viper"  // 导入 viper 包，用于处理配置文件
    "github.com/stretchr/testify/assert"  // 导入 testify 的 assert 包，用于编写断言
    "github.com/stretchr/testify/mock"  // 导入 testify 的 mock 包，用于编写模拟对象
    "gopkg.in/yaml.v2"  // 导入 yaml.v2 包，用于 YAML 数据的编解码
)

// 定义常量 cfgDir，指定配置文件所在的目录
const cfgDir = "../cfg/"

// 定义 mockRunner 结构体，用于模拟运行器
type mockRunner struct {
    mock.Mock
}

// 实现 mockRunner 结构体的 Run 方法，用于运行检查
func (m *mockRunner) Run(c *Check) State {
    args := m.Called(c)
    return args.Get(0).(State)
}

// 测试函数，验证我们要发送的文件是否是有效的 YAML 格式
func TestYamlFiles(t *testing.T) {
    # 使用 filepath.Walk 遍历指定目录下的所有文件和子目录
    err := filepath.Walk(cfgDir, func(path string, info os.FileInfo, err error) error {
        # 如果在遍历过程中出现错误，输出错误信息并终止测试
        if err != nil {
            t.Fatalf("failure accessing path %q: %v\n", path, err)
        }
        # 如果当前路径不是目录
        if !info.IsDir() {
            # 输出正在读取的文件路径
            t.Logf("reading file: %s", path)
            # 读取文件内容
            in, err := ioutil.ReadFile(path)
            # 如果读取文件出现错误，输出错误信息并终止测试
            if err != nil {
                t.Fatalf("error opening file %s: %v", path, err)
            }

            # 创建一个 Controls 结构体实例
            c := new(Controls)
            # 将文件内容解析为 YAML 格式，并存储到 Controls 结构体中
            err = yaml.Unmarshal(in, c)
            # 如果解析成功，输出成功信息；否则输出失败信息并终止测试
            if err == nil {
                t.Logf("YAML file successfully unmarshalled: %s", path)
            } else {
                t.Fatalf("failed to load YAML from %s: %v", path, err)
            }
        }
        # 继续遍历
        return nil
    })
    # 如果遍历过程中出现错误，输出错误信息并终止测试
    if err != nil {
        t.Fatalf("failure walking cfg dir: %v\n", err)
    }
// 测试函数，用于测试NewControls函数
func TestNewControls(t *testing.T) {
    // 子测试1：当节点类型未指定时应返回错误
    t.Run("Should return error when node type is not specified", func(t *testing.T) {
        // 给定输入
        in := []byte(`
---
controls:
type: # not specified
groups:
`)
        // 当
        _, err := NewControls(MASTER, in)
        // 则
        assert.EqualError(t, err, "non-master controls file specified")
    })

    // 子测试2：当输入的YAML无效时应返回错误
    t.Run("Should return error when input YAML is invalid", func(t *testing.T) {
        // 给定输入
        in := []byte("BOOM")
        // 当
        _, err := NewControls(MASTER, in)
        // 则
        assert.EqualError(t, err, "failed to unmarshal YAML: yaml: unmarshal errors:\n  line 1: cannot unmarshal !!str `BOOM` into check.Controls")
    })

}

// 测试函数，用于测试Controls的RunChecks方法
func TestControls_RunChecks_SkippedCmd(t *testing.T) {
    // 子测试：当skipMap指定时应跳过相应的检查和组
    t.Run("Should skip checks and groups specified by skipMap", func(t *testing.T) {
        // 给定普通的运行器
        normalRunner := &defaultRunner{}
        // 和输入
        in := []byte(`
---
type: "master"
groups:
- id: G1
  checks:
  - id: G1/C1
  - id: G1/C2
  - id: G1/C3
- id: G2
  checks:
  - id: G2/C1
  - id: G2/C2
`)
        controls, err := NewControls(MASTER, in)
        assert.NoError(t, err)

        var allChecks Predicate = func(group *Group, c *Check) bool {
            return true
        }

        // 创建跳过映射
        skipMap := make(map[string]bool, 0)
        skipMap["G1"] = true
        skipMap["G2/C1"] = true
        skipMap["G2/C2"] = true
        // 运行RunChecks方法
        controls.RunChecks(normalRunner, allChecks, skipMap)

        // 断言检查结果
        G1 := controls.Groups[0]
        assertEqualGroupSummary(t, 0, 0, 3, 0, G1)

        G2 := controls.Groups[1]
        assertEqualGroupSummary(t, 0, 0, 2, 0, G2)
    })
}

// 测试函数，用于测试Controls的RunChecks方法
func TestControls_RunChecks_Skipped(t *testing.T) {
    // 子测试：当父组标记为跳过时应跳过相应的检查
    t.Run("Should skip checks where the parent group is marked as skip", func(t *testing.T) {
        // 给定普通的运行器
        normalRunner := &defaultRunner{}
        // 和输入
        in := []byte(`
---
type: "master"
groups:
- id: G1
  skip: true
  checks:
  - id: G1/C1
        // 创建一个新的控制器对象，类型为 MASTER，输入为 in
        controls, err := NewControls(MASTER, in)
        // 断言没有错误发生
        assert.NoError(t, err)

        // 定义一个 Predicate 函数，始终返回 true
        var allChecks Predicate = func(group *Group, c *Check) bool {
            return true
        }
        // 创建一个空的跳过列表
        emptySkipList := make(map[string]bool, 0)
        // 运行所有检查
        controls.RunChecks(normalRunner, allChecks, emptySkipList)

        // 获取第一个组 G1
        G1 := controls.Groups[0]
        // 断言组 G1 的摘要与预期值相等
        assertEqualGroupSummary(t, 0, 0, 1, 0, G1)
    })
}

func TestControls_RunChecks(t *testing.T) {
    t.Run("Should run checks matching the filter and update summaries", func(t *testing.T) {
        // 给定一个模拟的运行器对象
        runner := new(mockRunner)
        // 并且
        in := []byte(`
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
        compare:
          op: has
          value: "true"
        set: true
    remediation: |
      Edit the config file /this/is/a/file/path and set SomeSampleFlag to true.
    scored: true
        // 创建一个测试用例切片，包含描述、输入和期望输出
        testCases := []struct {
            desc   string
            input  *Controls
            expect []byte

- 创建一个测试用例切片，包含描述、输入和期望输出


        // 遍历测试用例切片
        for _, tc := range testCases {
            // 使用测试框架的 Run 方法执行测试
            t.Run(tc.desc, func(t *testing.T) {
                // 设置期望输出
                expected := string(tc.expect)
                // 调用 JUnitIncludesJSON 方法，传入输入参数，并获取返回值
                result := tc.input.JUnitIncludesJSON()
                // 断言实际输出与期望输出相等
                assert.Equal(t, expected, string(result))
            })
        }
    }
}

- 遍历测试用例切片
- 使用测试框架的 Run 方法执行测试
- 设置期望输出
- 调用 JUnitIncludesJSON 方法，传入输入参数，并获取返回值
- 断言实际输出与期望输出相等
    }{
        {
            // 描述：序列化为 JUnit 格式
            desc: "Serializes to junit",
            // 输入：控制信息
            input: &Controls{
                // 分组列表
                Groups: []*Group{
                    {
                        // 分组ID和检查列表
                        ID: "g1",
                        Checks: []*Check{
                            {ID: "check1id", Text: "check1text", State: PASS},
                        },
                    },
                },
            },
            // 期望输出：空字节
            expect: []byte(`<testsuite name="" tests="0" failures="0" errors="0" time="0">
    <testcase name="check1id check1text" classname="" time="0">
        <system-out>{&#34;test_number&#34;:&#34;check1id&#34;,&#34;test_desc&#34;:&#34;check1text&#34;,&#34;audit&#34;:&#34;&#34;,&#34;AuditEnv&#34;:&#34;&#34;,&#34;AuditConfig&#34;:&#34;&#34;,&#34;type&#34;:&#34;&#34;,&#34;remediation&#34;:&#34;&#34;,&#34;test_info&#34;:null,&#34;status&#34;:&#34;PASS&#34;,&#34;actual_value&#34;:&#34;&#34;,&#34;scored&#34;:false,&#34;IsMultiple&#34;:false,&#34;expected_result&#34;:&#34;&#34;}</system-out>
    </testcase>
# 创建一个包含测试用例结果的 XML 文档
<testsuite name="" tests="402" failures="99" errors="0" time="0">
    # 创建一个测试用例，名称为 check1id check1text，状态为 PASS
    <testcase name="check1id check1text" classname="" time="0">
        # 在测试用例中输出系统信息，包括测试编号、测试描述、审核信息等
        <system-out>{&#34;test_number&#34;:&#34;check1id&#34;,&#34;test_desc&#34;:&#34;check1text&#34;,&#34;audit&#34;:&#34;&#34;,&#34;AuditEnv&#34;:&#34;&#34;,&#34;AuditConfig&#34;:&#34;&#34;,&#34;type&#34;:&#34;&#34;,&#34;remediation&#34;:&#34;&#34;,&#34;test_info&#34;:null,&#34;status&#34;:&#34;PASS&#34;,&#34;actual_value&#34;:&#34;&#34;,&#34;scored&#34;:false,&#34;IsMultiple&#34;:false,&#34;expected_result&#34;:&#34;&#34;}</system-out>
    </testcase>
</testsuite>
    # 创建一个测试用例，名称为"check1id check1text"，没有指定类名，执行时间为0
    <testcase name="check1id check1text" classname="" time="0">
        # 在系统输出中包含测试信息的 JSON 对象
        <system-out>{&#34;test_number&#34;:&#34;check1id&#34;,&#34;test_desc&#34;:&#34;check1text&#34;,&#34;audit&#34;:&#34;&#34;,&#34;AuditEnv&#34;:&#34;&#34;,&#34;AuditConfig&#34;:&#34;&#34;,&#34;type&#34;:&#34;&#34;,&#34;remediation&#34;:&#34;&#34;,&#34;test_info&#34;:null,&#34;status&#34;:&#34;PASS&#34;,&#34;actual_value&#34;:&#34;&#34;,&#34;scored&#34;:false,&#34;IsMultiple&#34;:false,&#34;expected_result&#34;:&#34;&#34;}</system-out>
    </testcase>
    # 创建另一个测试用例，名称为"check2id check2text"，没有指定类名，执行时间为0
    <testcase name="check2id check2text" classname="" time="0">
        # 标记该测试用例被跳过
        <skipped></skipped>
        # 在系统输出中包含测试信息的 JSON 对象
        <system-out>{&#34;test_number&#34;:&#34;check2id&#34;,&#34;test_desc&#34;:&#34;check2text&#34;,&#34;audit&#34;:&#34;&#34;,&#34;AuditEnv&#34;:&#34;&#34;,&#34;AuditConfig&#34;:&#34;&#34;,&#34;type&#34;:&#34;&#34;,&#34;remediation&#34;:&#34;&#34;,&#34;test_info&#34;:null,&#34;status&#34;:&#34;INFO&#34;,&#34;actual_value&#34;:&#34;&#34;,&#34;scored&#34;:false,&#34;IsMultiple&#34;:false,&#34;expected_result&#34;:&#34;&#34;}</system-out>
    </testcase>
    # 创建另一个测试用例，名称为"check3id check3text"，没有指定类名，执行时间为0
    <testcase name="check3id check3text" classname="" time="0">
        # 标记该测试用例被跳过
        <skipped></skipped>
        # 在系统输出中包含测试信息的 JSON 对象
        <system-out>{&#34;test_number&#34;:&#34;check3id&#34;,&#34;test_desc&#34;:&#34;check3text&#34;,&#34;audit&#34;:&#34;&#34;,&#34;AuditEnv&#34;:&#34;&#34;,&#34;AuditConfig&#34;:&#34;&#34;,&#34;type&#34;:&#34;&#34;,&#34;remediation&#34;:&#34;&#34;,&#34;test_info&#34;:null,&#34;status&#34;:&#34;WARN&#34;,&#34;actual_value&#34;:&#34;&#34;,&#34;scored&#34;:false,&#34;IsMultiple&#34;:false,&#34;expected_result&#34;:&#34;&#34;}</system-out>
    </testcase>
    # 创建一个测试用例，名称为"check4id check4text"，时间为0
    <testcase name="check4id check4text" classname="" time="0">
        # 用于记录测试失败的信息
        <failure type=""></failure>
        # 输出测试结果的详细信息，包括测试编号、描述、审核信息等
        <system-out>{&#34;test_number&#34;:&#34;check4id&#34;,&#34;test_desc&#34;:&#34;check4text&#34;,&#34;audit&#34;:&#34;&#34;,&#34;AuditEnv&#34;:&#34;&#34;,&#34;AuditConfig&#34;:&#34;&#34;,&#34;type&#34;:&#34;&#34;,&#34;remediation&#34;:&#34;&#34;,&#34;test_info&#34;:null,&#34;status&#34;:&#34;FAIL&#34;,&#34;actual_value&#34;:&#34;&#34;,&#34;scored&#34;:false,&#34;IsMultiple&#34;:false,&#34;expected_result&#34;:&#34;&#34;}</system-out>
    </testcase>
# 创建测试用例结构体
testCases := []struct {
    desc   string
    input  reporters.Report
    expect []byte
}{
    {
        desc: "Test case 1",
        input: reporters.Report{
            // 设置测试报告的内容
            Content: "This is a test report",
            // 设置测试报告的组
            Groups: []Group{
                {
                    // 设置组的名称
                    Name: "Group 1",
                    // 设置组的检查项
                    Checks: []Check{
                        {
                            // 设置检查项的名称
                            Name: "Check 1",
                            // 设置检查项的结果
                            Result: "Pass",
                        },
                        {
                            Name: "Check 2",
                            Result: "Fail",
                        },
                    },
                },
            },
        },
        // 设置预期的 JUnit 输出
        expect: []byte(`<testsuite name="Test case 1" tests="2" failures="1" time="0.000">
    <testcase name="Group 1 - Check 1" time="0.000">
        <failure message="Check failed">Check 1 failed</failure>
    </testcase>
    <testcase name="Group 1 - Check 2" time="0.000"/>
</testsuite>`),
    },
}
# 遍历测试用例
for _, tc := range testCases {
    t.Run(tc.desc, func(t *testing.T) {
        # 将测试报告序列化为 JUnit 格式
        junitBytes, err := tc.input.JUnit()
        if err != nil {
            t.Fatalf("Failed to serialize to JUnit: %v", err)
        }

        var out reporters.JUnitTestSuite
        # 将 JUnit 格式的数据反序列化为结构体
        if err := xml.Unmarshal(junitBytes, &out); err != nil {
            t.Fatalf("Unable to deserialize from resulting JUnit: %v", err)
        }

        # 检查每个检查项是否被序列化为 JSON 并存储在 SystemOut 中
        for iGroup, group := range tc.input.Groups {
            for iCheck, check := range group.Checks:
                jsonBytes, err := json.Marshal(check)
                if err != nil {
                    t.Fatalf("Failed to serialize to JUnit: %v", err)
                }

                if out.TestCases[iGroup*iCheck+iCheck].SystemOut != string(jsonBytes) {
                    t.Errorf("Expected\n\t%v\n\tbut got\n\t%v",
                        out.TestCases[iGroup*iCheck+iCheck].SystemOut,
                        string(jsonBytes),
                    )
                }
            }
        }

        # 检查序列化后的 JUnit 数据是否与预期值相等
        if !bytes.Equal(junitBytes, tc.expect) {
            t.Errorf("Expected\n\t%v\n\tbut got\n\t%v",
                string(tc.expect),
                string(junitBytes),
            )
        }
    })
}

# 定义辅助函数，用于断言组的摘要信息是否相等
func assertEqualGroupSummary(t *testing.T, pass, fail, info, warn int, actual *Group) {
    t.Helper()
    assert.Equal(t, pass, actual.Pass)
    assert.Equal(t, fail, actual.Fail)
    assert.Equal(t, info, actual.Info)
    assert.Equal(t, warn, actual.Warn)
}

# 测试 Controls_ASFF 函数
func TestControls_ASFF(t *testing.T) {
    type fields struct {
        ID      string
        Version string
        Text    string
        Groups  []*Group
        Summary Summary
    }
}
    # 定义测试用例结构体数组，包含名称、字段、期望结果和是否出错的标志
    tests := []struct {
        name    string
        fields  fields
        want    []*securityhub.AwsSecurityFinding
        wantErr bool
    }
    # 遍历测试用例数组
    for _, tt := range tests {
        # 运行测试用例
        t.Run(tt.name, func(t *testing.T) {
            # 设置 viper 配置
            viper.Set("AWS_ACCOUNT", "foo account")
            viper.Set("CLUSTER_ARN", "foo Cluster")
            viper.Set("AWS_REGION", "somewhere")
            # 创建 Controls 对象
            controls := &Controls{
                ID:      tt.fields.ID,
                Version: tt.fields.Version,
                Text:    tt.fields.Text,
                Groups:  tt.fields.Groups,
                Summary: tt.fields.Summary,
            }
            # 调用 ASFF 方法获取结果
            got, _ := controls.ASFF()
            # 更新期望结果的时间和 ID
            tt.want[0].CreatedAt = got[0].CreatedAt
            tt.want[0].UpdatedAt = got[0].UpdatedAt
            tt.want[0].Id = got[0].Id
            # 比较实际结果和期望结果
            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("Controls.ASFF() = %v, want %v", got, tt.want)
            }
        })
    }
# 闭合前面的函数定义
```