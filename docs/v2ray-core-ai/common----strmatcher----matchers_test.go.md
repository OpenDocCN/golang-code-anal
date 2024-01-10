# `v2ray-core\common\strmatcher\matchers_test.go`

```
package strmatcher_test

import (
    "testing"

    "v2ray.com/core/common"
    . "v2ray.com/core/common/strmatcher"
)

func TestMatcher(t *testing.T) {
    cases := []struct {
        pattern string  // 匹配模式
        mType   Type    // 匹配类型
        input   string  // 输入字符串
        output  bool    // 期望输出结果
    }{
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Domain,        // 设置匹配类型为域名
            input:   "www.v2ray.com",  // 输入字符串
            output:  true,           // 期望输出结果为 true
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Domain,        // 设置匹配类型为域名
            input:   "v2ray.com",   // 输入字符串
            output:  true,           // 期望输出结果为 true
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Domain,        // 设置匹配类型为域名
            input:   "www.v3ray.com",  // 输入字符串
            output:  false,          // 期望输出结果为 false
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Domain,        // 设置匹配类型为域名
            input:   "2ray.com",    // 输入字符串
            output:  false,          // 期望输出结果为 false
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Domain,        // 设置匹配类型为域名
            input:   "xv2ray.com",  // 输入字符串
            output:  false,          // 期望输出结果为 false
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Full,          // 设置匹配类型为完全匹配
            input:   "v2ray.com",   // 输入字符串
            output:  true,           // 期望输出结果为 true
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Full,          // 设置匹配类型为完全匹配
            input:   "xv2ray.com",  // 输入字符串
            output:  false,          // 期望输出结果为 false
        },
        {
            pattern: "v2ray.com",   // 设置匹配模式
            mType:   Regex,         // 设置匹配类型为正则表达式
            input:   "v2rayxcom",   // 输入字符串
            output:  true,           // 期望输出结果为 true
        },
    }
    for _, test := range cases {
        matcher, err := test.mType.New(test.pattern)  // 根据匹配类型和模式创建匹配器
        common.Must(err)  // 检查错误
        if m := matcher.Match(test.input); m != test.output {  // 使用匹配器进行匹配，并检查结果是否符合期望
            t.Error("unexpected output: ", m, " for test case ", test)  // 输出错误信息
        }
    }
}
```