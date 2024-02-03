# `v2ray-core\common\strmatcher\full_matcher_test.go`

```go
package strmatcher_test

import (
    "reflect"  // 导入 reflect 包，用于深度比较
    "testing"  // 导入 testing 包，用于编写测试函数

    . "v2ray.com/core/common/strmatcher"  // 导入 strmatcher 包，并使用其中的所有函数
)

func TestFullMatcherGroup(t *testing.T) {
    g := new(FullMatcherGroup)  // 创建 FullMatcherGroup 对象
    g.Add("v2ray.com", 1)  // 向 FullMatcherGroup 对象中添加匹配规则
    g.Add("google.com", 2)  // 向 FullMatcherGroup 对象中添加匹配规则
    g.Add("x.a.com", 3)  // 向 FullMatcherGroup 对象中添加匹配规则
    g.Add("x.y.com", 4)  // 向 FullMatcherGroup 对象中添加匹配规则
    g.Add("x.y.com", 6)  // 向 FullMatcherGroup 对象中添加匹配规则

    testCases := []struct {
        Domain string
        Result []uint32
    }{
        {
            Domain: "v2ray.com",
            Result: []uint32{1},
        },
        {
            Domain: "y.com",
            Result: nil,
        },
        {
            Domain: "x.y.com",
            Result: []uint32{4, 6},
        },
    }

    for _, testCase := range testCases {
        r := g.Match(testCase.Domain)  // 调用 FullMatcherGroup 对象的 Match 方法进行匹配
        if !reflect.DeepEqual(r, testCase.Result) {  // 深度比较匹配结果和预期结果
            t.Error("Failed to match domain: ", testCase.Domain, ", expect ", testCase.Result, ", but got ", r)  // 输出匹配失败的信息
        }
    }
}

func TestEmptyFullMatcherGroup(t *testing.T) {
    g := new(FullMatcherGroup)  // 创建 FullMatcherGroup 对象
    r := g.Match("v2ray.com")  // 调用 FullMatcherGroup 对象的 Match 方法进行匹配
    if len(r) != 0 {  // 判断匹配结果的长度是否为 0
        t.Error("Expect [], but ", r)  // 输出匹配失败的信息
    }
}
```