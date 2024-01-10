# `v2ray-core\common\strmatcher\strmatcher_test.go`

```
package strmatcher_test

import (
    "reflect"  // 导入 reflect 包，用于实现运行时反射
    "testing"  // 导入 testing 包，用于编写测试函数

    "v2ray.com/core/common"  // 导入 common 包
    . "v2ray.com/core/common/strmatcher"  // 导入 strmatcher 包，并将其所有公开的成员引入当前命名空间
)

// See https://github.com/v2fly/v2ray-core/issues/92#issuecomment-673238489
func TestMatcherGroup(t *testing.T) {
    rules := []struct {  // 定义结构体切片 rules
        Type   Type  // 定义 Type 类型的字段 Type
        Domain string  // 定义字符串类型的字段 Domain
    }{
        {
            Type:   Regex,  // 设置 Type 字段为 Regex
            Domain: "apis\\.us$",  // 设置 Domain 字段为 "apis\\.us$"
        },
        {
            Type:   Substr,  // 设置 Type 字段为 Substr
            Domain: "apis",  // 设置 Domain 字段为 "apis"
        },
        // ... 其余规则同上

    }
    cases := []struct {  // 定义结构体切片 cases
        Input  string  // 定义字符串类型的字段 Input
        Output []uint32  // 定义 uint32 类型的切片字段 Output
    }{
        {
            Input:  "www.baidu.com",  // 设置 Input 字段为 "www.baidu.com"
            Output: []uint32{5, 9, 4},  // 设置 Output 字段为 []uint32{5, 9, 4}
        },
        // ... 其余测试用例同上
    }
    matcherGroup := &MatcherGroup{}  // 创建 MatcherGroup 对象的指针 matcherGroup
    for _, rule := range rules {  // 遍历 rules 切片
        matcher, err := rule.Type.New(rule.Domain)  // 使用 rule.Type.New 方法创建匹配器
        common.Must(err)  // 检查错误并处理
        matcherGroup.Add(matcher)  // 将创建的匹配器添加到 matcherGroup 中
    }
    # 遍历测试用例集合
    for _, test := range cases:
        # 使用正则表达式匹配器匹配测试输入，并将结果赋值给变量m
        if m := matcherGroup.Match(test.Input); !reflect.DeepEqual(m, test.Output):
            # 如果匹配结果与预期输出不相等，则输出错误信息
            t.Error("unexpected output: ", m, " for test case ", test)
# 闭合前面的函数定义
```