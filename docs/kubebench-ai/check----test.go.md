# `kubebench-aquasecurity\check\test.go`

```go
// 版权声明
// 该代码版权归 Aqua Security Software Ltd. 所有
// 根据 Apache 许可证 2.0 版本授权
// 除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本
// http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则按"原样"分发软件
// 没有任何形式的担保或条件，无论是明示的还是暗示的
// 请查看许可证以了解特定语言的权限和限制

package check

import (
    "bytes"  // 导入 bytes 包
    "encoding/json"  // 导入 json 包
    "fmt"  // 导入 fmt 包
    "os"  // 导入 os 包
    "regexp"  // 导入 regexp 包
    "strconv"  // 导入 strconv 包
    "strings"  // 导入 strings 包

    "github.com/golang/glog"  // 导入 glog 包
    yaml "gopkg.in/yaml.v2"  // 导入 yaml.v2 包
    "k8s.io/client-go/util/jsonpath"  // 导入 jsonpath 包
)

// test 结构体定义
// flag: OPTION
// set: (true|false)
// compare:
//   op: (eq|gt|gte|lt|lte|has)
//   value: val
type binOp string  // 定义 binOp 类型为字符串

const (
    and                   binOp = "and"  // 定义 and 常量为 binOp 类型的字符串 "and"
    or                          = "or"  // 定义 or 常量为 binOp 类型的字符串 "or"
    defaultArraySeparator       = ","  // 定义 defaultArraySeparator 常量为逗号
)

// tests 结构体定义
type tests struct {
    TestItems []*testItem `yaml:"test_items"`  // 定义 TestItems 字段为 testItem 类型的切片
    BinOp     binOp       `yaml:"bin_op"`  // 定义 BinOp 字段为 binOp 类型
}

// AuditUsed 类型定义
const (
    AuditCommand AuditUsed = "auditCommand"  // 定义 AuditCommand 常量为 AuditUsed 类型的字符串 "auditCommand"
    AuditConfig AuditUsed = "auditConfig"  // 定义 AuditConfig 常量为 AuditUsed 类型的字符串 "auditConfig"
    AuditEnv AuditUsed = "auditEnv"  // 定义 AuditEnv 常量为 AuditUsed 类型的字符串 "auditEnv"
)

// testItem 结构体定义
type testItem struct {
    Flag             string  // 定义 Flag 字段为字符串类型
    Env              string  // 定义 Env 字段为字符串类型
    Path             string  // 定义 Path 字段为字符串类型
    Output           string  // 定义 Output 字段为字符串类型
    Value            string  // 定义 Value 字段为字符串类型
    Set              bool    // 定义 Set 字段为布尔类型
    Compare          compare  // 定义 Compare 字段为 compare 类型
    isMultipleOutput bool    // 定义 isMultipleOutput 字段为布尔类型
    auditUsed  AuditUsed  // 定义 auditUsed 字段为 AuditUsed 类型
}

// envTestItem、pathTestItem、flagTestItem 类型定义
type envTestItem testItem  // 定义 envTestItem 类型为 testItem 类型
type pathTestItem testItem  // 定义 pathTestItem 类型为 testItem 类型
type flagTestItem testItem  // 定义 flagTestItem 类型为 testItem 类型

// compare 结构体定义
type compare struct {
    Op    string  // 定义 Op 字段为字符串类型
    Value string  // 定义 Value 字段为字符串类型
}

// testOutput 结构体定义
type testOutput struct {
    testResult     bool    // 定义 testResult 字段为布尔类型
    found      bool    // 定义 found 字段为布尔类型
    actualResult   string  // 定义 actualResult 字段为字符串类型
    ExpectedResult string  // 定义 ExpectedResult 字段为字符串类型
}

// failTestItem 函数定义
func failTestItem(s string) *testOutput {
    return &testOutput{testResult: false, actualResult: s}  // 返回 testOutput 类型的指针
}
# 定义一个方法，返回 testItem 结构体的值
func (t testItem) value() string {
    # 如果 auditUsed 字段的取值为 AuditConfig，则返回 Path 字段的值
    if t.auditUsed == AuditConfig {
        return t.Path
    }
    # 如果 auditUsed 字段的取值为 AuditEnv，则返回 Env 字段的值
    if t.auditUsed == AuditEnv {
        return t.Env
    }
    # 否则返回 Flag 字段的值
    return t.Flag
}

# 定义一个方法，查找 testItem 结构体中的值
func (t testItem) findValue(s string) (match bool, value string, err error) {
    # 如果 auditUsed 字段的取值为 AuditEnv，则创建 envTestItem 结构体并调用其 findValue 方法
    if t.auditUsed == AuditEnv {
        et := envTestItem(t)
        return et.findValue(s)
    }
    # 如果 auditUsed 字段的取值为 AuditConfig，则创建 pathTestItem 结构体并调用其 findValue 方法
    if t.auditUsed == AuditConfig {
        pt := pathTestItem(t)
        return pt.findValue(s)
    }
    # 否则创建 flagTestItem 结构体并调用其 findValue 方法
    ft := flagTestItem(t)
    return ft.findValue(s)
}

# 定义一个方法，查找 flagTestItem 结构体中的值
func (t flagTestItem) findValue(s string) (match bool, value string, err error) {
    # 如果输入字符串为空或者 Flag 字段为空，则直接返回
    if s == "" || t.Flag == "" {
        return
    }
    # 判断输入字符串中是否包含 Flag 字段的值
    match = strings.Contains(s, t.Flag)
    if match:
        # 定义匹配 Flag 的正则表达式
        pttn := `(` + t.Flag + `)(=|: *)*([^\s]*) *`
        flagRe := regexp.MustCompile(pttn)
        # 使用正则表达式匹配输入字符串中的值
        vals := flagRe.FindStringSubmatch(s)
        # 如果匹配到值
        if len(vals) > 0 {
            # 如果匹配到的值不为空，则赋给 value
            if vals[3] != "" {
                value = vals[3]
            } else {
                # 如果 Flag 以 "--" 开头，则赋值为 "true"，否则赋值为 vals[1]
                if strings.HasPrefix(t.Flag, "--") {
                    value = "true"
                } else {
                    value = vals[1]
                }
            }
        } else {
            # 如果未匹配到值，则返回错误信息
            err = fmt.Errorf("invalid flag in testItem definition: %s", s)
        }
    }
    # 输出日志信息
    glog.V(3).Infof("In flagTestItem.findValue %s, match %v, s %s, t.Flag %s", value, match, s, t.Flag)

    return match, value, err
}

# 定义一个方法，查找 pathTestItem 结构体中的值
func (t pathTestItem) findValue(s string) (match bool, value string, err error) {
    # 定义一个空接口变量
    var jsonInterface interface{}
    # 解析输入字符串为 JSON 格式，并赋值给 jsonInterface
    err = unmarshal(s, &jsonInterface)
    if err != nil {
        # 如果解析失败，则返回错误信息
        return false, "", fmt.Errorf("failed to load YAML or JSON from input \"%s\": %v", s, err)
    }
    # 执行 JSON 路径查询，并返回结果
    value, err = executeJSONPath(t.Path, &jsonInterface)
}
    # 如果错误不为空，则返回错误信息
    if err != nil:
        return false, "", fmt.Errorf("unable to parse path expression \"%s\": %v", t.Path, err)
    
    # 输出日志信息，记录路径测试项中的数值
    glog.V(3).Infof("In pathTestItem.findValue %s", value)
    
    # 判断数值是否为空，如果不为空则匹配成功
    match = value != ""
    
    # 返回匹配结果、数值和错误信息
    return match, value, err
// 在 envTestItem 结构体上定义一个方法，用于在字符串中查找指定环境变量的值
func (t envTestItem) findValue(s string) (match bool, value string, err error) {
    // 如果输入字符串不为空且环境变量不为空
    if s != "" && t.Env != "" {
        // 编译正则表达式，用于匹配环境变量的键值对
        r, _ := regexp.Compile(fmt.Sprintf("%s=.*(?:$|\\n)", t.Env))
        // 在输入字符串中查找匹配的内容
        out := r.FindString(s)
        // 替换换行符和环境变量键，得到环境变量的值
        out = strings.Replace(out, "\n", "", 1)
        out = strings.Replace(out, fmt.Sprintf("%s=", t.Env), "", 1)

        // 如果找到了环境变量的值
        if len(out) > 0 {
            match = true
            value = out
        } else {
            match = false
            value = ""
        }
    }
    // 返回匹配结果、值和错误
    return match, value, nil
}

// 在 testItem 结构体上定义一个方法，用于执行测试并返回测试结果
func (t testItem) execute(s string) *testOutput {
    // 创建测试结果对象
    result := &testOutput{}
    // 去除字符串末尾的空格和换行符
    s = strings.TrimRight(s, " \n")

    // 如果测试有多个输出需要评估
    var output []string
    if t.isMultipleOutput {
        // 根据换行符分割字符串为多个输出
        output = strings.Split(s, "\n")
    } else {
        output = []string{s}
    }

    // 遍历每个输出并进行评估
    for _, op := range output {
        result = t.evaluate(op)
        // 如果当前输出的测试失败，则无需继续测试其他输出
        if !result.testResult {
            break
        }
    }

    // 设置实际测试结果
    result.actualResult = s
    return result
}

// 在 testItem 结构体上定义一个方法，用于评估测试输出并返回测试结果
func (t testItem) evaluate(s string) *testOutput {
    // 创建测试结果对象
    result := &testOutput{}

    // 查找环境变量的值
    match, value, err := t.findValue(s)
    if err != nil {
        // 如果查找出错，则输出错误信息并返回失败的测试结果
        fmt.Fprintf(os.Stderr, err.Error())
        return failTestItem(err.Error())
    }

    // 如果设置了环境变量，并且有比较操作符
    if t.Set {
        if match && t.Compare.Op != "" {
            // 比较环境变量的值和给定值
            result.ExpectedResult, result.testResult = compareOp(t.Compare.Op, value, t.Compare.Value)
        } else {
            // 输出环境变量的值是否存在
            result.ExpectedResult = fmt.Sprintf("'%s' is present", t.value())
            result.testResult = match
        }
    } else {
        // 输出环境变量的值是否不存在
        result.ExpectedResult = fmt.Sprintf("'%s' is not present", t.value())
        result.testResult = !match
    }

    // 设置测试结果的匹配状态
    result.found = match
    // 输出匹配状态的日志信息
    glog.V(3).Info(fmt.Sprintf("found %v", result.found))

    return result
}

// 定义一个比较操作的函数，用于比较给定值和环境变量的值
func compareOp(tCompareOp string, flagVal string, tCompareValue string) (string, bool) {
    # 初始化预期结果模式和测试结果
    expectedResultPattern := ""
    testResult := false

    # 根据比较操作类型进行不同的处理
    switch tCompareOp:
        # 如果是相等比较
        case "eq":
            # 设置预期结果模式
            expectedResultPattern = "'%s' is equal to '%s'"
            # 将标志值转换为小写
            value := strings.ToLower(flagVal)
            # 对布尔值进行不区分大小写的比较
            if value == "false" || value == "true":
                testResult = value == tCompareValue
            else:
                testResult = flagVal == tCompareValue

        # 如果是不相等比较
        case "noteq":
            # 设置预期结果模式
            expectedResultPattern = "'%s' is not equal to '%s'"
            # 将标志值转换为小写
            value := strings.ToLower(flagVal)
            # 对布尔值进行不区分大小写的比较
            if value == "false" || value == "true":
                testResult = !(value == tCompareValue)
            else:
                testResult = !(flagVal == tCompareValue)

        # 如果是大于、大于等于、小于、小于等于比较
        case "gt", "gte", "lt", "lte":
            # 将标志值和比较值转换为数值类型
            a, b, err := toNumeric(flagVal, tCompareValue)
            # 如果转换出错，返回错误信息
            if err != nil:
                glog.V(1).Infof(fmt.Sprintf("Not numeric value - flag: %q - compareValue: %q %v\n", flagVal, tCompareValue, err))
                return "Invalid Number(s) used for comparison", false
            # 根据比较操作类型进行不同的处理
            switch tCompareOp:
                # 如果是大于比较
                case "gt":
                    expectedResultPattern = "%s is greater than %s"
                    testResult = a > b

                # 如果是大于等于比较
                case "gte":
                    expectedResultPattern = "%s is greater or equal to %s"
                    testResult = a >= b

                # 如果是小于比较
                case "lt":
                    expectedResultPattern = "%s is lower than %s"
                    testResult = a < b

                # 如果是小于等于比较
                case "lte":
                    expectedResultPattern = "%s is lower or equal to %s"
                    testResult = a <= b

        # 如果是包含比较
        case "has":
            # 设置预期结果模式
            expectedResultPattern = "'%s' has '%s'"
            # 判断标志值是否包含比较值
            testResult = strings.Contains(flagVal, tCompareValue)

        # 如果是不包含比较
        case "nothave":
            # 设置预期结果模式
            expectedResultPattern = " '%s' not have '%s'"
            # 判断标志值是否不包含比较值
            testResult = !strings.Contains(flagVal, tCompareValue)
    # 如果匹配类型是正则表达式
    case "regex":
        # 设置预期结果的格式
        expectedResultPattern = " '%s' matched by '%s'"
        # 编译正则表达式
        opRe := regexp.MustCompile(tCompareValue)
        # 测试匹配结果
        testResult = opRe.MatchString(flagVal)

    # 如果匹配类型是有效元素
    case "valid_elements":
        # 设置预期结果的格式
        expectedResultPattern = "'%s' contains valid elements from '%s'"
        # 分割并移除最后一个分隔符
        s := splitAndRemoveLastSeparator(flagVal, defaultArraySeparator)
        target := splitAndRemoveLastSeparator(tCompareValue, defaultArraySeparator)
        # 测试所有元素是否有效
        testResult = allElementsValid(s, target)

    # 如果匹配类型是位掩码
    case "bitmask":
        # 设置预期结果的格式
        expectedResultPattern = "bitmask '%s' AND '%s'"
        # 将标志值解析为整数
        requested, err := strconv.ParseInt(flagVal, 8, 64)
        if err != nil:
            glog.V(1).Infof(fmt.Sprintf("Not numeric value - flag: %q - compareValue: %q %v\n", flagVal, tCompareValue, err))
            return fmt.Sprintf("Not numeric value - flag: %s", flagVal), false
        # 将比较值解析为整数
        max, err := strconv.ParseInt(tCompareValue, 8, 64)
        if err != nil:
            glog.V(1).Infof(fmt.Sprintf("Not numeric value - flag: %q - compareValue: %q %v\n", flagVal, tCompareValue, err))
            return fmt.Sprintf("Not numeric value - flag: %s", tCompareValue), false
        # 测试位掩码
        testResult = (max & requested) == requested
    }
    # 如果预期结果格式为空
    if expectedResultPattern == "":
        return expectedResultPattern, testResult

    # 返回格式化后的预期结果和测试结果
    return fmt.Sprintf(expectedResultPattern, flagVal, tCompareValue), testResult
}

// 将字符串解析为 JSON 或 YAML 对象
func unmarshal(s string, jsonInterface *interface{}) error {
    // 将字符串转换为字节数组
    data := []byte(s)
    // 尝试将数据解析为 JSON 对象
    err := json.Unmarshal(data, jsonInterface)
    if err != nil {
        // 如果解析为 JSON 失败，则尝试解析为 YAML 对象
        err := yaml.Unmarshal(data, jsonInterface)
        if err != nil {
            return err
        }
    }
    return nil
}

// 执行 JSONPath 查询并返回结果
func executeJSONPath(path string, jsonInterface interface{}) (string, error) {
    // 创建 JSONPath 对象
    j := jsonpath.New("jsonpath")
    // 允许缺失的键
    j.AllowMissingKeys(true)
    // 解析 JSONPath 查询
    err := j.Parse(path)
    if err != nil {
        return "", err
    }

    // 创建缓冲区
    buf := new(bytes.Buffer)
    // 执行 JSONPath 查询
    err = j.Execute(buf, jsonInterface)
    if err != nil {
        return "", err
    }
    // 获取 JSONPath 查询结果
    jsonpathResult := buf.String()
    return jsonpathResult, nil
}

// 检查两个字符串数组中的元素是否有效
func allElementsValid(s, t []string) bool {
    // 检查源数组和目标数组是否为空
    sourceEmpty := len(s) == 0
    targetEmpty := len(t) == 0

    if sourceEmpty && targetEmpty {
        return true
    }

    // 异或比较 -
    //     如果其中一个数组为空，另一个不为空，则不是所有元素都有效
    if (sourceEmpty || targetEmpty) && !(sourceEmpty && targetEmpty) {
        return false
    }

    // 遍历源数组，检查是否所有元素都在目标数组中
    for _, sv := range s {
        found := false
        for _, tv := range t {
            if sv == tv {
                found = true
                break
            }
        }
        if !found {
            return false
        }
    }
    return true
}

// 分割字符串并移除末尾的分隔符
func splitAndRemoveLastSeparator(s, sep string) []string {
    // 清除字符串两端的空格和指定的分隔符
    cleanS := strings.TrimRight(strings.TrimSpace(s), sep)
    if len(cleanS) == 0 {
        return []string{}
    }

    // 使用指定的分隔符分割字符串，并去除每个部分两端的空格
    ts := strings.Split(cleanS, sep)
    for i := range ts {
        ts[i] = strings.TrimSpace(ts[i])
    }

    return ts
}

// 将两个字符串转换为整数
func toNumeric(a, b string) (c, d int, err error) {
    // 将字符串转换为整数，并去除两端的空格
    c, err = strconv.Atoi(strings.TrimSpace(a))
    if err != nil {
        return -1, -1, fmt.Errorf("toNumeric - error converting %s: %s", a, err)
    }
    // 将字符串转换为整数，并去除两端的空格
    d, err = strconv.Atoi(strings.TrimSpace(b))
    // 返回转换结果
    return c, d, err
}
    # 如果 err 不为空，表示发生了错误
    if err != nil:
        # 返回错误信息，包括转换失败的值和具体的错误信息
        return -1, -1, fmt.Errorf("toNumeric - error converting %s: %s", b, err)
    
    # 如果没有错误，返回转换后的数值和 nil 表示没有错误
    return c, d, nil
// UnmarshalYAML 方法用于将 YAML 数据解析为 testItem 结构体
func (t *testItem) UnmarshalYAML(unmarshal func(interface{}) error) error {
    // 定义一个内部类型 buildTest，用于解析 YAML 数据
    type buildTest testItem

    // 将 Set 参数默认设置为 true
    newTestItem := buildTest{Set: true}
    // 调用 unmarshal 函数解析 YAML 数据，并将结果存储到 newTestItem 中
    err := unmarshal(&newTestItem)
    // 如果解析出错，则返回错误
    if err != nil {
        return err
    }
    // 将 newTestItem 转换为 testItem 类型，并赋值给 t
    *t = testItem(newTestItem)
    // 返回空错误，表示解析成功
    return nil
}
```