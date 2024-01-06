# `kubebench-aquasecurity\check\controls_test.go`

```
// 版权声明，版权归Aqua Security Software Ltd.所有
// 根据Apache许可证2.0进行许可
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

// 导入所需的包
package check

import (
    "bytes"  // 导入bytes包，用于操作字节
    "encoding/json"  // 导入json包，用于JSON编解码
    "encoding/xml"  // 导入xml包，用于XML编解码
	"fmt"  // 导入 fmt 包，用于格式化输入输出
	"io/ioutil"  // 导入 ioutil 包，用于读取文件内容
	"os"  // 导入 os 包，用于操作系统功能
	"path/filepath"  // 导入 filepath 包，用于处理文件路径
	"reflect"  // 导入 reflect 包，用于反射操作
	"testing"  // 导入 testing 包，用于编写测试函数
	"github.com/aws/aws-sdk-go/aws"  // 导入 AWS SDK 包，用于访问 AWS 服务
	"github.com/aws/aws-sdk-go/service/securityhub"  // 导入 AWS Security Hub 包，用于访问安全中心服务
	"github.com/onsi/ginkgo/reporters"  // 导入 ginkgo 报告包，用于生成测试报告
	"github.com/spf13/viper"  // 导入 viper 包，用于处理配置文件
	"github.com/stretchr/testify/assert"  // 导入 testify 断言包，用于编写测试断言
	"github.com/stretchr/testify/mock"  // 导入 testify mock 包，用于编写测试模拟
	"gopkg.in/yaml.v2"  // 导入 yaml 包，用于处理 YAML 格式数据
)

const cfgDir = "../cfg/"  // 定义配置文件目录路径

type mockRunner struct {  // 定义 mockRunner 结构体
	mock.Mock  // 使用 testify mock 包中的 Mock 结构体
}

// 定义一个名为 mockRunner 的结构体的方法 Run，接收一个 Check 类型的参数，返回一个 State 类型的值
func (m *mockRunner) Run(c *Check) State {
    // 调用 m 的 Called 方法，传入参数 c，返回结果赋值给 args
    args := m.Called(c)
    // 返回 args 中索引为 0 的值，并断言为 State 类型
    return args.Get(0).(State)
}

// 测试 YamlFiles 函数
func TestYamlFiles(t *testing.T) {
    // 遍历 cfgDir 目录下的文件和子目录
    err := filepath.Walk(cfgDir, func(path string, info os.FileInfo, err error) error {
        // 如果遍历过程中出现错误，输出错误信息并终止测试
        if err != nil {
            t.Fatalf("failure accessing path %q: %v\n", path, err)
        }
        // 如果当前遍历到的是文件而不是目录，输出正在读取的文件路径
        if !info.IsDir() {
            t.Logf("reading file: %s", path)
            // 读取文件内容并赋值给 in，如果出现错误则输出错误信息并终止测试
            in, err := ioutil.ReadFile(path)
            if err != nil {
                t.Fatalf("error opening file %s: %v", path, err)
            }
// 创建一个新的 Controls 结构体实例
c := new(Controls)
// 将输入的 YAML 数据解析为 Controls 结构体实例
err = yaml.Unmarshal(in, c)
// 如果解析过程中没有出现错误，则打印成功信息，否则打印失败信息
if err == nil {
    t.Logf("YAML file successfully unmarshalled: %s", path)
} else {
    t.Fatalf("failed to load YAML from %s: %v", path, err)
}
// 返回 nil，表示函数执行成功
return nil
// 如果遍历 cfg 目录的过程中出现错误，则打印失败信息
if err != nil {
    t.Fatalf("failure walking cfg dir: %v\n", err)
}

// 测试函数，测试当节点类型未指定时是否返回错误
func TestNewControls(t *testing.T) {
    t.Run("Should return error when node type is not specified", func(t *testing.T) {
        // 给定输入的 YAML 数据
        in := []byte(`
---
controls:
type: # not specified
groups:
`)
		// 定义控件类型和组信息，但未指定具体类型
		// 当
		_, err := NewControls(MASTER, in)
		// 调用 NewControls 函数，传入 MASTER 类型和输入数据
		// 则
		assert.EqualError(t, err, "non-master controls file specified")
		// 断言返回的错误信息为"non-master controls file specified"

	})

	t.Run("Should return error when input YAML is invalid", func(t *testing.T) {
		// 给定
		in := []byte("BOOM")
		// 定义输入数据为"BOOM"
		// 当
		_, err := NewControls(MASTER, in)
		// 调用 NewControls 函数，传入 MASTER 类型和输入数据
		// 则
		assert.EqualError(t, err, "failed to unmarshal YAML: yaml: unmarshal errors:\n  line 1: cannot unmarshal !!str `BOOM` into check.Controls")
		// 断言返回的错误信息为"failed to unmarshal YAML: yaml: unmarshal errors:\n  line 1: cannot unmarshal !!str `BOOM` into check.Controls"
	})

}
```
func TestControls_RunChecks_SkippedCmd(t *testing.T) {
    // 定义测试函数，用于测试跳过指定检查和组的情况
    t.Run("Should skip checks and groups specified by skipMap", func(t *testing.T) {
        // 给定
        normalRunner := &defaultRunner{}
        // 创建一个默认的运行器对象
        // 并
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
        // 定义一个包含 YAML 格式内容的字节切片
		// 使用 NewControls 函数创建控制器对象，类型为 MASTER，输入参数为 in
		controls, err := NewControls(MASTER, in)
		// 断言错误为空
		assert.NoError(t, err)

		// 定义一个名为 allChecks 的谓词函数，始终返回 true
		var allChecks Predicate = func(group *Group, c *Check) bool {
			return true
		}

		// 创建一个名为 skipMap 的空字符串到布尔值的映射
		skipMap := make(map[string]bool, 0)
		// 设置 skipMap 中的键值对
		skipMap["G1"] = true
		skipMap["G2/C1"] = true
		skipMap["G2/C2"] = true
		// 运行控制器对象的检查函数，使用 normalRunner 作为运行器，allChecks 作为谓词函数，skipMap 作为跳过的检查项
		controls.RunChecks(normalRunner, allChecks, skipMap)

		// 获取控制器对象中的第一个组 G1
		G1 := controls.Groups[0]
		// 断言 G1 的摘要信息符合预期
		assertEqualGroupSummary(t, 0, 0, 3, 0, G1)

		// 获取控制器对象中的第二个组 G2
		G2 := controls.Groups[1]
		// 断言 G2 的摘要信息符合预期
		assertEqualGroupSummary(t, 0, 0, 2, 0, G2)
	})
}
func TestControls_RunChecks_Skipped(t *testing.T) {
    // 运行测试用例，检查是否正确跳过标记为跳过的父组的检查
    t.Run("Should skip checks where the parent group is marked as skip", func(t *testing.T) {
        // 给定
        normalRunner := &defaultRunner{}
        // 并且
        in := []byte(`
---
type: "master"
groups:
- id: G1
  skip: true
  checks:
  - id: G1/C1
`)
        // 创建控制对象，并解析输入数据
        controls, err := NewControls(MASTER, in)
        assert.NoError(t, err)

        // 定义一个谓词函数，用于判断是否执行所有检查
        var allChecks Predicate = func(group *Group, c *Check) bool {
            return true
		}
		// 创建一个空的跳过列表，用于存储需要跳过的检查项
		emptySkipList := make(map[string]bool, 0)
		// 运行所有检查项，不跳过任何检查项
		controls.RunChecks(normalRunner, allChecks, emptySkipList)

		// 获取第一个检查组的信息
		G1 := controls.Groups[0]
		// 断言第一个检查组的总结信息是否符合预期
		assertEqualGroupSummary(t, 0, 0, 1, 0, G1)
	})
}

func TestControls_RunChecks(t *testing.T) {
	t.Run("Should run checks matching the filter and update summaries", func(t *testing.T) {
		// 给定
		runner := new(mockRunner)
		// 并且
		in := []byte(`
---
type: "master"
groups:
- id: G1
  checks:
# 定义了一个名为 G1/C1 的代码段
# 定义了一个名为 G2 的代码段
# 验证 SomeSampleFlag 参数是否设置为 true
# 使用 grep 命令在文件路径 /this/is/a/file/path 中查找 SomeSampleFlag=true，并显示前一行内容
# 测试项：检查 SomeSampleFlag 是否包含值 true，如果是则设置为 true
# 修复措施：编辑配置文件 /this/is/a/file/path 并将 SomeSampleFlag 设置为 true
# 该测试项计入评分
# 创建一个名为 controls 的变量，用于存储从 MASTER 中读取的控制信息
# 如果没有错误发生，则断言 controls 变量的值为 nil
		// 设置对 controls.Groups[0].Checks[0] 的运行结果为 PASS
		runner.On("Run", controls.Groups[0].Checks[0]).Return(PASS)
		// 设置对 controls.Groups[1].Checks[0] 的运行结果为 FAIL
		runner.On("Run", controls.Groups[1].Checks[0]).Return(FAIL)
		// 定义一个 Predicate 函数类型变量 runAll，始终返回 true
		var runAll Predicate = func(group *Group, c *Check) bool {
			return true
		}
		// 创建一个空的跳过列表
		var emptySkipList = make(map[string]bool, 0)
		// 运行 controls 的所有检查
		controls.RunChecks(runner, runAll, emptySkipList)
		// 断言 controls.Groups 的长度为 2
		assert.Equal(t, 2, len(controls.Groups))
		// 断言 controls.Groups[0] 的 ID 为 "G1"
		G1 := controls.Groups[0]
		assert.Equal(t, "G1", G1.ID)
		// 断言 controls.Groups[0] 的第一个检查的 ID 为 "G1/C1"
		assert.Equal(t, "G1/C1", G1.Checks[0].ID)
		// 断言 controls.Groups[0] 的汇总结果符合预期
		assertEqualGroupSummary(t, 1, 0, 0, 0, G1)
		// 断言 controls.Groups[1] 的 ID 为 "G2"
		G2 := controls.Groups[1]
		assert.Equal(t, "G2", G2.ID)
		// 断言检查 G2 对象的属性值是否符合预期
		assert.Equal(t, "G2/C1", G2.Checks[0].ID)
		assert.Equal(t, "has", G2.Checks[0].Tests.TestItems[0].Compare.Op)
		assert.Equal(t, "true", G2.Checks[0].Tests.TestItems[0].Compare.Value)
		assert.Equal(t, true, G2.Checks[0].Tests.TestItems[0].Set)
		assert.Equal(t, "SomeSampleFlag=true", G2.Checks[0].Tests.TestItems[0].Flag)
		assert.Equal(t, "Edit the config file /this/is/a/file/path and set SomeSampleFlag to true.\n", G2.Checks[0].Remediation)
		assert.Equal(t, true, G2.Checks[0].Scored)
		assertEqualGroupSummary(t, 0, 1, 0, 0, G2)
		// and
		// 断言检查 controls.Summary 对象的属性值是否符合预期
		assert.Equal(t, 1, controls.Summary.Pass)
		assert.Equal(t, 1, controls.Summary.Fail)
		assert.Equal(t, 0, controls.Summary.Info)
		assert.Equal(t, 0, controls.Summary.Warn)
		// and
		// 断言检查 runner 对象的期望行为是否符合预期
		runner.AssertExpectations(t)
	})
}

func TestControls_JUnitIncludesJSON(t *testing.T) {
	// 定义测试用例
	testCases := []struct {
		desc   string  // 描述测试用例的字符串
		input  *Controls  // 控制参数的指针
		expect []byte  // 期望的字节数组
	}{
		{
			desc: "Serializes to junit",  // 测试用例描述
			input: &Controls{  // 控制参数的实例
				Groups: []*Group{  // 控制参数中的组列表
					{
						ID: "g1",  // 组的ID
						Checks: []*Check{  // 组中的检查列表
							{ID: "check1id", Text: "check1text", State: PASS},  // 检查的ID、文本和状态
						},
					},
				},
			},
			expect: []byte(`<testsuite name="" tests="0" failures="0" errors="0" time="0">  // 期望的字节数组，表示一个测试套件
    <testcase name="check1id check1text" classname="" time="0">  // 测试用例的名称、类名和时间
    </testcase>
</testsuite>`),  // 测试套件的结束标记
		}, {
			// 描述：汇总值来自于汇总而不是检查
			desc: "Summary values come from summary not checks",
			// 输入：控制结构体
			input: &Controls{
				// 汇总：包括失败、通过、警告、信息的数量
				Summary: Summary{
					Fail: 99,
					Pass: 100,
					Warn: 101,
					Info: 102,
				},
				// 分组：包括分组ID和检查列表
				Groups: []*Group{
					{
						ID: "g1",
						Checks: []*Check{
							{ID: "check1id", Text: "check1text", State: PASS},
						},
					},
				},
			},
			// 期望：期望的输出结果
			expect: []byte(`<testsuite name="" tests="402" failures="99" errors="0" time="0">
    <testcase name="check1id check1text" classname="" time="0">
// 定义一个结构体 Controls，包含了一组检查项的信息
input: &Controls{
    // 定义一个组，包含了一组检查项
    Groups: []*Group{
        {
            // 组的唯一标识符
            ID: "g1",
            // 检查项列表
            Checks: []*Check{
                // 检查项1
                {ID: "check1id", Text: "check1text", State: PASS},
                // 检查项2
                {ID: "check2id", Text: "check2text", State: INFO},
                // 检查项3
                {ID: "check3id", Text: "check3text", State: WARN},
                // 检查项4
                {ID: "check4id", Text: "check4text", State: FAIL},
            },
        },
    },
},
// 期望的输出结果
expect: []byte(`<testsuite name="" tests="0" failures="0" errors="0" time="0">
    // 检查项1的测试结果
    <testcase name="check1id check1text" classname="" time="0">
    </testcase>
```
# 创建一个名为check2id check2text的测试用例，设置类名为空，执行时间为0
<testcase name="check2id check2text" classname="" time="0">
    # 跳过该测试用例
    <skipped></skipped>
</testcase>

# 创建一个名为check3id check3text的测试用例，设置类名为空，执行时间为0
<testcase name="check3id check3text" classname="" time="0">
    # 跳过该测试用例
    <skipped></skipped>
</testcase>

# 创建一个名为check4id check4text的测试用例，设置类名为空，执行时间为0
<testcase name="check4id check4text" classname="" time="0">
    # 测试用例执行失败，没有指定失败类型
    <failure type=""></failure>
</testcase>
</testsuite>`),
		},
	}
# 遍历测试用例集合
for _, tc := range testCases {
    # 对每个测试用例执行以下操作
    t.Run(tc.desc, func(t *testing.T) {
        # 将测试用例转换为JUnit格式的字节流
        junitBytes, err := tc.input.JUnit()
        # 如果转换过程中出现错误，输出错误信息
        if err != nil {
            t.Fatalf("Failed to serialize to JUnit: %v", err)
        }
        # 创建JUnit测试套件对象
        var out reporters.JUnitTestSuite
// 使用xml.Unmarshal将junitBytes反序列化为out变量，如果出现错误则输出错误信息
if err := xml.Unmarshal(junitBytes, &out); err != nil {
    t.Fatalf("Unable to deserialize from resulting JUnit: %v", err)
}

// 检查每个检查是否被序列化为json并存储为systemOut
for iGroup, group := range tc.input.Groups {
    for iCheck, check := range group.Checks {
        // 将check序列化为json格式的字节流
        jsonBytes, err := json.Marshal(check)
        if err != nil {
            t.Fatalf("Failed to serialize to JUnit: %v", err)
        }

        // 检查out.TestCases中的SystemOut是否与jsonBytes相等，如果不相等则输出错误信息
        if out.TestCases[iGroup*iCheck+iCheck].SystemOut != string(jsonBytes) {
            t.Errorf("Expected\n\t%v\n\tbut got\n\t%v",
                out.TestCases[iGroup*iCheck+iCheck].SystemOut,
                string(jsonBytes),
            )
        }
    }
}
# 如果 JUnit 字节数据与预期不相等，则输出错误信息
if !bytes.Equal(junitBytes, tc.expect) {
    t.Errorf("Expected\n\t%v\n\tbut got\n\t%v",
        string(tc.expect),
        string(junitBytes),
    )
}

# 辅助函数，用于比较实际值和预期值是否相等
func assertEqualGroupSummary(t *testing.T, pass, fail, info, warn int, actual *Group) {
    t.Helper()
    assert.Equal(t, pass, actual.Pass)
    assert.Equal(t, fail, actual.Fail)
    assert.Equal(t, info, actual.Info)
    assert.Equal(t, warn, actual.Warn)
}

# 测试函数，用于测试 Controls_ASFF 功能
func TestControls_ASFF(t *testing.T) {
# 定义结构体 fields，包含 ID、Version、Text、Groups 和 Summary 字段
type fields struct {
    ID      string
    Version string
    Text    string
    Groups  []*Group
    Summary Summary
}
# 定义测试用例数组 tests，包含 name、fields、want 和 wantErr 字段
tests := []struct {
    name    string
    fields  fields
    want    []*securityhub.AwsSecurityFinding
    wantErr bool
}{
    # 第一个测试用例
    {
        name: "Test simple conversion",
        fields: fields{
            ID:      "test1",
            Version: "1",
            Text:    "test runnner",
            Summary: Summary{
# 定义不同状态的常量值
Fail: 99,
Pass: 100,
Warn: 101,
Info: 102,
# 定义一个包含多个组的数组
Groups: []*Group{
    # 定义一个组对象
    {
        ID:   "g1",
        Text: "Group text",
        # 定义一个包含多个检查的数组
        Checks: []*Check{
            # 定义一个检查对象
            {
                ID:             "check1id",
                Text:           "check1text",
                State:          FAIL,  # 设置检查状态为失败
                Remediation:    "fix me",
                Reason:         "failed",
                ExpectedResult: "failed",
                ActualValue:    "failed",
            },
        },
    },
},
// 该部分代码是一个测试用例的期望输出，包含了一些安全发现的详细信息
// 设置期望的安全发现信息
want: []*securityhub.AwsSecurityFinding{
    // 设置安全发现的 AWS 账户 ID
    {
        AwsAccountId:  aws.String("foo account"),
        // 设置安全发现的置信度
        Confidence:    aws.Int64(100),
        // 设置安全发现的生成器 ID
        GeneratorId:   aws.String(fmt.Sprintf("%s/cis-kubernetes-benchmark/%s/%s", fmt.Sprintf(ARN, "somewhere"), "1", "check1id")),
        // 设置安全发现的描述
        Description:   aws.String("check1text"),
        // 设置安全发现的产品 ARN
        ProductArn:    aws.String(fmt.Sprintf(ARN, "somewhere")),
        // 设置安全发现的模式版本
        SchemaVersion: aws.String(SCHEMA),
        // 设置安全发现的标题
        Title:         aws.String(fmt.Sprintf("%s %s", "check1id", "check1text")),
        // 设置安全发现的类型
        Types:         []*string{aws.String(TYPE)},
        // 设置安全发现的严重程度
        Severity: &securityhub.Severity{
            Label: aws.String(securityhub.SeverityLabelHigh),
        },
        // 设置安全发现的修复建议
        Remediation: &securityhub.Remediation{
            Recommendation: &securityhub.Recommendation{
                Text: aws.String("fix me"),
            },
        },
        // 设置安全发现的产品字段
        ProductFields: map[string]*string{
            // ...
        },
    },
    // ...
},
# 设置 Reason 字段为 "failed"
"Reason":          aws.String("failed"),
# 设置 Actual result 字段为 "failed"
"Actual result":   aws.String("failed"),
# 设置 Expected result 字段为 "failed"
"Expected result": aws.String("failed"),
# 设置 Section 字段为 "test1 test runnner"
"Section":         aws.String(fmt.Sprintf("%s %s", "test1", "test runnner")),
# 设置 Subsection 字段为 "g1 Group text"
"Subsection":      aws.String(fmt.Sprintf("%s %s", "g1", "Group text")),
# 设置 Resources 字段为包含一个资源对象的数组
Resources: []*securityhub.Resource{
    {
        # 设置资源对象的 Id 字段为 "foo Cluster"
        Id:   aws.String("foo Cluster"),
        # 设置资源对象的 Type 字段为 TYPE 变量的值
        Type: aws.String(TYPE),
    },
},
# 遍历测试用例数组
for _, tt := range tests {
    # 对每个测试用例运行测试
    t.Run(tt.name, func(t *testing.T) {
        # 设置 viper 库的 AWS_ACCOUNT 变量为 "foo account"
        viper.Set("AWS_ACCOUNT", "foo account")
# 设置配置参数 CLUSTER_ARN 为 "foo Cluster"
viper.Set("CLUSTER_ARN", "foo Cluster")
# 设置配置参数 AWS_REGION 为 "somewhere"
viper.Set("AWS_REGION", "somewhere")
# 创建 Controls 对象，初始化其属性值
controls := &Controls{
    ID:      tt.fields.ID,
    Version: tt.fields.Version,
    Text:    tt.fields.Text,
    Groups:  tt.fields.Groups,
    Summary: tt.fields.Summary,
}
# 调用 Controls 对象的 ASFF 方法，获取返回结果
got, _ := controls.ASFF()
# 将获取到的结果中的时间属性赋值给期望结果
tt.want[0].CreatedAt = got[0].CreatedAt
tt.want[0].UpdatedAt = got[0].UpdatedAt
tt.want[0].Id = got[0].Id
# 检查获取到的结果和期望结果是否相等，如果不相等则输出错误信息
if !reflect.DeepEqual(got, tt.want) {
    t.Errorf("Controls.ASFF() = %v, want %v", got, tt.want)
}
```