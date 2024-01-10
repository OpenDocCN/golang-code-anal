# `v2ray-core\common\strmatcher\matchers.go`

```
package strmatcher

import (
    "regexp"  // 导入正则表达式包
    "strings"  // 导入字符串处理包
)

type fullMatcher string  // 定义全匹配类型，类型为字符串

func (m fullMatcher) Match(s string) bool {  // 定义全匹配类型的匹配方法
    return string(m) == s  // 返回字符串是否完全匹配
}

func (m fullMatcher) String() string {  // 定义全匹配类型的字符串方法
    return "full:" + string(m)  // 返回全匹配类型的字符串表示
}

type substrMatcher string  // 定义子串匹配类型，类型为字符串

func (m substrMatcher) Match(s string) bool {  // 定义子串匹配类型的匹配方法
    return strings.Contains(s, string(m))  // 返回字符串是否包含子串
}

func (m substrMatcher) String() string {  // 定义子串匹配类型的字符串方法
    return "keyword:" + string(m)  // 返回子串匹配类型的字符串表示
}

type domainMatcher string  // 定义域名匹配类型，类型为字符串

func (m domainMatcher) Match(s string) bool {  // 定义域名匹配类型的匹配方法
    pattern := string(m)  // 获取域名匹配类型的字符串表示
    if !strings.HasSuffix(s, pattern) {  // 如果字符串不以指定的域名结尾
        return false  // 返回 false
    }
    return len(s) == len(pattern) || s[len(s)-len(pattern)-1] == '.'  // 返回字符串长度是否等于域名长度或者字符串中倒数第二个字符是否为'.'
}

func (m domainMatcher) String() string {  // 定义域名匹配类型的字符串方法
    return "domain:" + string(m)  // 返回域名匹配类型的字符串表示
}

type regexMatcher struct {  // 定义正则表达式匹配类型
    pattern *regexp.Regexp  // 定义正则表达式对象
}

func (m *regexMatcher) Match(s string) bool {  // 定义正则表达式匹配类型的匹配方法
    return m.pattern.MatchString(s)  // 返回字符串是否匹配正则表达式
}

func (m *regexMatcher) String() string {  // 定义正则表达式匹配类型的字符串方法
    return "regexp:" + m.pattern.String()  // 返回正则表达式匹配类型的字符串表示
}
```