# `v2ray-core\common\strmatcher\domain_matcher_test.go`

```
package strmatcher_test

import (
    "reflect"  // 导入 reflect 包，用于深度比较
    "testing"  // 导入 testing 包，用于编写测试函数

    . "v2ray.com/core/common/strmatcher"  // 导入 strmatcher 包，并使用其中的所有函数
)

func TestDomainMatcherGroup(t *testing.T) {
    g := new(DomainMatcherGroup)  // 创建一个 DomainMatcherGroup 对象
    g.Add("v2ray.com", 1)  // 向 DomainMatcherGroup 对象中添加域名和对应的值
    g.Add("google.com", 2)
    g.Add("x.a.com", 3)
    g.Add("a.b.com", 4)
    g.Add("c.a.b.com", 5)
    g.Add("x.y.com", 4)
    g.Add("x.y.com", 6)

    testCases := []struct {  // 定义测试用例的结构体
        Domain string  // 域名
        Result []uint32  // 期望的匹配结果
    }{
        {
            Domain: "x.v2ray.com",  // 测试用例1：域名为 x.v2ray.com
            Result: []uint32{1},  // 期望的匹配结果为 [1]
        },
        {
            Domain: "y.com",  // 测试用例2：域名为 y.com
            Result: nil,  // 期望的匹配结果为 nil
        },
        {
            Domain: "a.b.com",  // 测试用例3：域名为 a.b.com
            Result: []uint32{4},  // 期望的匹配结果为 [4]
        },
        { // Matches [c.a.b.com, a.b.com]
            Domain: "c.a.b.com",  // 测试用例4：域名为 c.a.b.com
            Result: []uint32{5, 4},  // 期望的匹配结果为 [5, 4]
        },
        {
            Domain: "c.a..b.com",  // 测试用例5：域名为 c.a..b.com
            Result: nil,  // 期望的匹配结果为 nil
        },
        {
            Domain: ".com",  // 测试用例6：域名为 .com
            Result: nil,  // 期望的匹配结果为 nil
        },
        {
            Domain: "com",  // 测试用例7：域名为 com
            Result: nil,  // 期望的匹配结果为 nil
        },
        {
            Domain: "",  // 测试用例8：域名为空
            Result: nil,  // 期望的匹配结果为 nil
        },
        {
            Domain: "x.y.com",  // 测试用例9：域名为 x.y.com
            Result: []uint32{4, 6},  // 期望的匹配结果为 [4, 6]
        },
    }

    for _, testCase := range testCases {  // 遍历测试用例
        r := g.Match(testCase.Domain)  // 调用 Match 方法进行匹配
        if !reflect.DeepEqual(r, testCase.Result) {  // 深度比较实际结果和期望结果
            t.Error("Failed to match domain: ", testCase.Domain, ", expect ", testCase.Result, ", but got ", r)  // 输出匹配失败的信息
        }
    }
}

func TestEmptyDomainMatcherGroup(t *testing.T) {
    g := new(DomainMatcherGroup)  // 创建一个空的 DomainMatcherGroup 对象
    r := g.Match("v2ray.com")  // 调用 Match 方法进行匹配
    if len(r) != 0 {  // 判断实际结果的长度是否为 0
        t.Error("Expect [], but ", r)  // 输出匹配失败的信息
    }
}
```