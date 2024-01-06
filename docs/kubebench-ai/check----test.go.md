# `kubebench-aquasecurity\check\test.go`

```
// 版权声明，版权归 Aqua Security Software Ltd. 所有
// 根据 Apache 许可证 2.0 版本授权
// 除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"按原样"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言下的权限和限制

// 导入所需的包
package check

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "encoding/json"  // 导入 encoding/json 包，用于 JSON 编解码
    "fmt"  // 导入 fmt 包，用于格式化输入输出
// 导入操作系统相关的包
"os"
// 导入正则表达式相关的包
"regexp"
// 导入字符串转换相关的包
"strconv"
// 导入字符串处理相关的包
"strings"

// 导入日志记录相关的包
"github.com/golang/glog"
// 导入 YAML 解析相关的包
yaml "gopkg.in/yaml.v2"
// 导入 JSONPath 解析相关的包
"k8s.io/client-go/util/jsonpath"
)

// 定义测试条件的结构体
// test:
// flag: OPTION
// set: (true|false)
// compare:
//   op: (eq|gt|gte|lt|lte|has)
//   value: val
type binOp string

// 定义比较操作的常量
const (
# 定义常量 binOp 为字符串 "and"
binOp = "and"
# 定义常量为字符串 "or"
or = "or"
# 定义常量 defaultArraySeparator 为字符串 ","
defaultArraySeparator = ","

# 定义结构体 tests，包含 TestItems 和 BinOp 两个字段
type tests struct {
    TestItems []*testItem `yaml:"test_items"`  # 使用 yaml 标签指定在 YAML 文件中的字段名
    BinOp     binOp       `yaml:"bin_op"`      # 使用 yaml 标签指定在 YAML 文件中的字段名
}

# 定义常量 AuditUsed 为字符串类型
type AuditUsed string

# 定义常量 AuditCommand 为字符串 "auditCommand"
const (
    AuditCommand AuditUsed = "auditCommand"
    # 定义常量 AuditConfig 为字符串 "auditConfig"
    AuditConfig AuditUsed = "auditConfig"
    # 定义常量 AuditEnv 为字符串 "auditEnv"
    AuditEnv AuditUsed = "auditEnv"
)

# 定义结构体 testItem，包含字段 Flag
type testItem struct {
    Flag             string  # 字段 Flag 为字符串类型
# 定义了一个结构体，包含了环境、路径、输出、数值、设置、比较、是否多重输出和审计使用等字段
Env              string  # 环境字段，类型为字符串
Path             string  # 路径字段，类型为字符串
Output           string  # 输出字段，类型为字符串
Value            string  # 数值字段，类型为字符串
Set              bool    # 设置字段，类型为布尔值
Compare          compare  # 比较字段，类型为compare结构体
isMultipleOutput bool    # 是否多重输出字段，类型为布尔值
auditUsed  AuditUsed     # 审计使用字段，类型为AuditUsed结构体

# 定义了envTestItem、pathTestItem和flagTestItem结构体，它们都是testItem结构体的别名
type envTestItem testItem
type pathTestItem testItem
type flagTestItem testItem

# 定义了compare结构体，包含了操作和数值两个字段
type compare struct {
    Op    string  # 操作字段，类型为字符串
    Value string  # 数值字段，类型为字符串
}

# 定义了testOutput结构体
type testOutput struct {
// 定义测试结果的布尔值
testResult     bool
// 定义是否找到的布尔值
found      bool
// 定义实际结果的字符串
actualResult   string
// 定义期望结果的字符串
ExpectedResult string
}

// 定义一个失败的测试项，并返回包含实际结果的测试输出
func failTestItem(s string) *testOutput {
	return &testOutput{testResult: false, actualResult: s}
}

// 定义测试项的值方法
func (t testItem) value() string {
	// 如果使用了审计配置，则返回路径
	if t.auditUsed == AuditConfig {
		return t.Path
	}

	// 如果使用了审计环境，则返回环境
	if t.auditUsed == AuditEnv {
		return t.Env
	}

	// 否则返回标志
	return t.Flag
// 定义 testItem 结构体的方法 findValue，用于在字符串中查找匹配的值
func (t testItem) findValue(s string) (match bool, value string, err error) {
	// 如果 t.auditUsed 等于 AuditEnv，则创建 envTestItem 对象并调用其 findValue 方法
	if t.auditUsed == AuditEnv {
		et := envTestItem(t)
		return et.findValue(s)
	}
	// 如果 t.auditUsed 等于 AuditConfig，则创建 pathTestItem 对象并调用其 findValue 方法
	if t.auditUsed == AuditConfig {
		pt := pathTestItem(t)
		return pt.findValue(s)
	}
	// 否则创建 flagTestItem 对象并调用其 findValue 方法
	ft := flagTestItem(t)
	return ft.findValue(s)
}

// 定义 flagTestItem 结构体的方法 findValue，用于在字符串中查找匹配的值
func (t flagTestItem) findValue(s string) (match bool, value string, err error) {
	// 如果 s 为空字符串或者 t.Flag 为空，则直接返回
	if s == "" || t.Flag == "" {
		return
	}
	// 检查字符串 s 中是否包含标志 t.Flag
	match = strings.Contains(s, t.Flag)
	if match {
		// 期望标志的形式为;
		// --flag=somevalue
		// flag: somevalue
		// --flag
		// somevalue
		// 创建匹配标志的正则表达式
		pttn := `(` + t.Flag + `)(=|: *)*([^\s]*) *`
		flagRe := regexp.MustCompile(pttn)
		// 在字符串 s 中查找匹配正则表达式的子串
		vals := flagRe.FindStringSubmatch(s)

		if len(vals) > 0 {
			if vals[3] != "" {
				// 如果找到值，则将其赋给变量 value
				value = vals[3]
			} else {
				// --bool-flag
				// 如果标志以 "--" 开头，则将值设为 "true"
				if strings.HasPrefix(t.Flag, "--") {
					value = "true"
				} else {
// 定义一个函数，用于在测试项中查找值
func (t pathTestItem) findValue(s string) (match bool, value string, err error) {
    // 定义一个接口变量，用于存储 JSON 数据
    var jsonInterface interface{}
    // 将输入的字符串解析为 JSON 数据，并存储到接口变量中
    err = unmarshal(s, &jsonInterface)
    // 如果解析出错，则返回错误信息
    if err != nil {
        return false, "", fmt.Errorf("failed to load YAML or JSON from input \"%s\": %v", s, err)
    }
// 使用给定的 JSON 路径执行 JSONPath 表达式，返回结果和可能的错误
value, err = executeJSONPath(t.Path, &jsonInterface)
// 如果执行 JSONPath 表达式出现错误，返回错误信息
if err != nil {
    return false, "", fmt.Errorf("unable to parse path expression \"%s\": %v", t.Path, err)
}

// 输出日志信息，记录路径测试项中的值
glog.V(3).Infof("In pathTestItem.findValue %s", value)
// 判断值是否为空，返回匹配结果
match = value != ""
// 返回匹配结果、值和可能的错误
return match, value, err
}

// 在环境测试项中查找值
func (t envTestItem) findValue(s string) (match bool, value string, err error) {
    // 如果输入字符串不为空且环境变量不为空
    if s != "" && t.Env != "" {
        // 编译正则表达式，匹配环境变量的值
        r, _ := regexp.Compile(fmt.Sprintf("%s=.*(?:$|\\n)", t.Env))
        // 查找匹配的字符串
        out := r.FindString(s)
        // 替换换行符和环境变量名，得到环境变量的值
        out = strings.Replace(out, "\n", "", 1)
        out = strings.Replace(out, fmt.Sprintf("%s=", t.Env), "", 1)

        // 如果找到环境变量的值
        if len(out) > 0 {
            // 设置匹配结果为 true
            match = true
            // 设置值为找到的环境变量的值
            value = out
		}else{
			// 如果条件不满足，则将 match 设为 false
			match = false
			// 将 value 设为空字符串
			value = ""
		}
	}
	// 返回匹配结果、值和空指针
	return match, value, nil
}

func (t testItem) execute(s string) *testOutput {
	// 创建一个 testOutput 结构体的指针
	result := &testOutput{}
	// 去除字符串末尾的空格和换行符
	s = strings.TrimRight(s, " \n")

	// 如果测试有多个输出需要评估
	var output []string
	if t.isMultipleOutput {
		// 使用换行符分割字符串
		output = strings.Split(s, "\n")
	} else {
		// 否则将整个字符串作为输出
		output = []string{s}
	}
# 遍历输出列表中的每个元素
for _, op := range output:
    # 对当前元素进行评估，并将结果赋给result
    result = t.evaluate(op)
    # 如果当前行的测试失败，则无需继续测试该输出
    if !result.testResult:
        break

# 将实际结果赋给result
result.actualResult = s
# 返回result
return result
```

```
# 评估测试项
func (t testItem) evaluate(s string) *testOutput:
    # 创建一个测试结果对象
    result := &testOutput{}

    # 在字符串s中查找匹配的值，并将结果赋给match, value, err
    match, value, err := t.findValue(s)
    # 如果出现错误，则将错误信息输出到标准错误，并返回一个失败的测试项
    if err != nil:
        fmt.Fprintf(os.Stderr, err.Error())
        return failTestItem(err.Error())
# 如果条件 t.Set 为真，则执行以下代码块
if t.Set:
    # 如果 match 为真且 t.Compare.Op 不为空，则执行比较操作，并将结果赋给 ExpectedResult 和 testResult
    if match and t.Compare.Op != "":
        result.ExpectedResult, result.testResult = compareOp(t.Compare.Op, value, t.Compare.Value)
    # 否则，将结果赋给 ExpectedResult 和 testResult
    else:
        result.ExpectedResult = fmt.Sprintf("'%s' is present", t.value())
        result.testResult = match
# 如果条件 t.Set 为假，则执行以下代码块
else:
    result.ExpectedResult = fmt.Sprintf("'%s' is not present", t.value())
    result.testResult = !match

# 将 match 的值赋给 result.found
result.found = match
# 记录日志，输出 result.found 的值
glog.V(3).Info(fmt.Sprintf("found %v", result.found))

# 返回 result
return result
# 定义一个名为 compareOp 的函数，接受 tCompareOp、flagVal 和 tCompareValue 三个参数
def compareOp(tCompareOp string, flagVal string, tCompareValue string) (string, bool):
# 初始化预期结果模式和测试结果
expectedResultPattern := ""
testResult := false

# 根据比较操作类型进行不同的处理
switch tCompareOp {
    # 如果是相等比较操作
    case "eq":
        # 设置预期结果模式
        expectedResultPattern = "'%s' is equal to '%s'"
        # 将标志值转换为小写
        value := strings.ToLower(flagVal)
        # 对布尔值进行不区分大小写的比较
        if value == "false" || value == "true" {
            testResult = value == tCompareValue
        } else {
            testResult = flagVal == tCompareValue
        }

    # 如果是不相等比较操作
    case "noteq":
        # 设置预期结果模式
        expectedResultPattern = "'%s' is not equal to '%s'"
        # 将标志值转换为小写
        value := strings.ToLower(flagVal)
        # 对布尔值进行不区分大小写的比较
        if value == "false" || value == "true" {
		// 如果操作符是"eq"，则比较value和tCompareValue是否相等，取反赋值给testResult
		testResult = !(value == tCompareValue)
		// 如果操作符不是"eq"，则比较flagVal和tCompareValue是否相等，取反赋值给testResult
		// 这里是一个else分支
		testResult = !(flagVal == tCompareValue)

	// 如果操作符是"gt"、"gte"、"lt"、"lte"中的一个
	case "gt", "gte", "lt", "lte":
		// 将flagVal和tCompareValue转换为数值类型a和b
		a, b, err := toNumeric(flagVal, tCompareValue)
		// 如果转换过程中出现错误，记录日志并返回错误信息
		if err != nil {
			glog.V(1).Infof(fmt.Sprintf("Not numeric value - flag: %q - compareValue: %q %v\n", flagVal, tCompareValue, err))
			return "Invalid Number(s) used for comparison", false
		}
		// 根据不同的操作符进行比较，并赋值给testResult
		switch tCompareOp {
		case "gt":
			expectedResultPattern = "%s is greater than %s"
			testResult = a > b

		case "gte":
			expectedResultPattern = "%s is greater or equal to %s"
			testResult = a >= b
# 根据不同的操作类型进行不同的比较，并设置预期结果模式和测试结果

# 当操作类型为 "lt" 时，设置预期结果模式为 "%s is lower than %s"，并进行小于比较
case "lt":
    expectedResultPattern = "%s is lower than %s"
    testResult = a < b

# 当操作类型为 "lte" 时，设置预期结果模式为 "%s is lower or equal to %s"，并进行小于等于比较
case "lte":
    expectedResultPattern = "%s is lower or equal to %s"
    testResult = a <= b

# 当操作类型为 "has" 时，设置预期结果模式为 "'%s' has '%s'"，并判断字符串是否包含指定子字符串
case "has":
    expectedResultPattern = "'%s' has '%s'"
    testResult = strings.Contains(flagVal, tCompareValue)

# 当操作类型为 "nothave" 时，设置预期结果模式为 " '%s' not have '%s'"，并判断字符串是否不包含指定子字符串
case "nothave":
    expectedResultPattern = " '%s' not have '%s'"
    testResult = !strings.Contains(flagVal, tCompareValue)

# 当操作类型为 "regex" 时，设置预期结果模式为 " '%s' matched by '%s'"，并使用正则表达式进行匹配
case "regex":
    expectedResultPattern = " '%s' matched by '%s'"
    opRe := regexp.MustCompile(tCompareValue)
		// 使用正则表达式匹配标志值
		testResult = opRe.MatchString(flagVal)

	// 处理 valid_elements 情况
	case "valid_elements":
		// 设置预期结果的格式
		expectedResultPattern = "'%s' contains valid elements from '%s'"
		// 使用默认分隔符分割并移除最后一个分隔符
		s := splitAndRemoveLastSeparator(flagVal, defaultArraySeparator)
		target := splitAndRemoveLastSeparator(tCompareValue, defaultArraySeparator)
		// 检查所有元素是否有效
		testResult = allElementsValid(s, target)

	// 处理 bitmask 情况
	case "bitmask":
		// 设置预期结果的格式
		expectedResultPattern = "bitmask '%s' AND '%s'"
		// 将标志值解析为整数
		requested, err := strconv.ParseInt(flagVal, 8, 64)
		// 如果解析出错，记录日志并返回错误信息
		if err != nil {
			glog.V(1).Infof(fmt.Sprintf("Not numeric value - flag: %q - compareValue: %q %v\n", flagVal, tCompareValue, err))
			return fmt.Sprintf("Not numeric value - flag: %s", flagVal), false
		}
		// 将比较值解析为整数
		max, err := strconv.ParseInt(tCompareValue, 8, 64)
		// 如果解析出错，记录日志并返回错误信息
		if err != nil {
			glog.V(1).Infof(fmt.Sprintf("Not numeric value - flag: %q - compareValue: %q %v\n", flagVal, tCompareValue, err))
			return fmt.Sprintf("Not numeric value - flag: %s", tCompareValue), false
		}
// 检查最大值和请求值的按位与是否等于请求值，将结果赋给testResult
testResult = (max & requested) == requested
}

// 如果预期结果模式为空，则直接返回预期结果模式和testResult
if expectedResultPattern == "" {
	return expectedResultPattern, testResult
}

// 使用预期结果模式格式化字符串，将flagVal和tCompareValue插入格式化字符串中，返回格式化后的字符串和testResult
return fmt.Sprintf(expectedResultPattern, flagVal, tCompareValue), testResult
}

// 将字符串解析为JSON或YAML格式的数据，根据解析结果返回相应的错误
func unmarshal(s string, jsonInterface *interface{}) error {
	// 将字符串转换为字节数组
	data := []byte(s)
	// 尝试将字节数组解析为JSON格式的数据，如果失败则尝试解析为YAML格式的数据
	err := json.Unmarshal(data, jsonInterface)
	if err != nil {
		err := yaml.Unmarshal(data, jsonInterface)
		if err != nil {
			return err
		}
	}
	return nil
}
# 执行 JSON 路径查询，返回查询结果和可能的错误
func executeJSONPath(path string, jsonInterface interface{}) (string, error) {
    # 创建 JSONPath 对象，并允许缺失的键
    j := jsonpath.New("jsonpath")
    j.AllowMissingKeys(true)
    # 解析 JSONPath 查询路径
    err := j.Parse(path)
    if err != nil {
        return "", err
    }

    # 创建一个字节缓冲区
    buf := new(bytes.Buffer)
    # 执行 JSONPath 查询，并将结果写入缓冲区
    err = j.Execute(buf, jsonInterface)
    if err != nil {
        return "", err
    }
    # 将缓冲区中的结果转换为字符串
    jsonpathResult := buf.String()
    return jsonpathResult, nil
}

# 检查两个字符串数组是否都为空
func allElementsValid(s, t []string) bool {
    # 检查源数组是否为空
    sourceEmpty := len(s) == 0
	// 检查目标数组是否为空
	targetEmpty := len(t) == 0

	// 如果源数组和目标数组都为空，则返回 true
	if sourceEmpty && targetEmpty {
		return true
	}

	// 异或比较 -
	//     如果其中一个值为空而另一个不为空，
	//     则不是所有元素都有效
	if (sourceEmpty || targetEmpty) && !(sourceEmpty && targetEmpty) {
		return false
	}

	// 遍历源数组
	for _, sv := range s {
		found := false
		// 遍历目标数组
		for _, tv := range t {
			// 如果找到相同的元素，则将 found 设置为 true 并跳出循环
			if sv == tv {
				found = true
				break
			}
// 检查字符串数组中是否包含特定字符串，如果包含则返回 true，否则返回 false
func containsString(arr []string, str string) bool {
	// 遍历字符串数组
	for _, s := range arr {
		// 如果找到特定字符串，则返回 true
		if s == str {
			return true
		}
	}
	// 如果未找到特定字符串，则返回 false
	return false
}

// 根据指定分隔符分割字符串，并移除末尾的分隔符
func splitAndRemoveLastSeparator(s, sep string) []string {
	// 清除字符串两端的空格，并移除末尾的分隔符
	cleanS := strings.TrimRight(strings.TrimSpace(s), sep)
	// 如果清除后的字符串长度为 0，则返回空字符串数组
	if len(cleanS) == 0 {
		return []string{}
	}

	// 使用指定分隔符分割字符串
	ts := strings.Split(cleanS, sep)
	// 遍历分割后的字符串数组，去除每个字符串两端的空格
	for i := range ts {
		ts[i] = strings.TrimSpace(ts[i])
	}

	// 返回处理后的字符串数组
	return ts
}
// 定义一个函数，将两个字符串转换为整数，并返回转换后的整数和可能出现的错误
func toNumeric(a, b string) (c, d int, err error) {
    // 将字符串a去除首尾空格后转换为整数，并赋值给c
    c, err = strconv.Atoi(strings.TrimSpace(a))
    // 如果转换出错，返回错误信息
    if err != nil {
        return -1, -1, fmt.Errorf("toNumeric - error converting %s: %s", a, err)
    }
    // 将字符串b去除首尾空格后转换为整数，并赋值给d
    d, err = strconv.Atoi(strings.TrimSpace(b))
    // 如果转换出错，返回错误信息
    if err != nil {
        return -1, -1, fmt.Errorf("toNumeric - error converting %s: %s", b, err)
    }

    // 返回转换后的整数和nil，表示没有错误
    return c, d, nil
}

// 定义一个方法，用于将YAML数据解析为testItem对象
func (t *testItem) UnmarshalYAML(unmarshal func(interface{}) error) error {
    // 定义一个新的buildTest对象，将Set参数默认设置为true
    newTestItem := buildTest{Set: true}
// 将数据解析为新的测试项目对象
err := unmarshal(&newTestItem)
// 如果解析过程中出现错误，返回错误信息
if err != nil {
    return err
}
// 将新的测试项目对象赋值给当前对象
*t = testItem(newTestItem)
// 返回空错误，表示解析成功
return nil
```