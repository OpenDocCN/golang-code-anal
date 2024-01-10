# `grype\grype\match\details.go`

```
package match

import (
    "fmt"

    "github.com/mitchellh/hashstructure/v2"
)

// Details 是 Detail 的切片类型
type Details []Detail

// Detail 结构体定义了匹配的详细信息
type Detail struct {
    Type       Type        // 匹配的类型（精确匹配、模糊匹配、间接匹配等）
    SearchedBy interface{} // 用于搜索的特定属性（除了包名称和版本）--表示匹配的“方式”
    Found      interface{} // 与漏洞对象匹配的特定属性--表示匹配的“内容”
    Matcher    MatcherType // 发现匹配的匹配器对象
    Confidence float64     // 匹配的确定性作为比率（目前未使用，保留以备将来使用）
}

// String 返回匹配字段的字符串表示形式
func (m Detail) String() string {
    return fmt.Sprintf("Detail(searchedBy=%q found=%q matcher=%q)", m.SearchedBy, m.Found, m.Matcher)
}

// Matchers 返回匹配的匹配器类型切片
func (m Details) Matchers() (tys []MatcherType) {
    if len(m) == 0 {
        return nil
    }
    for _, d := range m {
        tys = append(tys, d.Matcher)
    }
    return tys
}

// Types 返回匹配的类型切片
func (m Details) Types() (tys []Type) {
    if len(m) == 0 {
        return nil
    }
    for _, d := range m {
        tys = append(tys, d.Type)
    }
    return tys
}

// ID 返回匹配的唯一标识符
func (m Detail) ID() string {
    f, err := hashstructure.Hash(&m, hashstructure.FormatV2, &hashstructure.HashOptions{
        ZeroNil:      true,
        SlicesAsSets: true,
    })
    if err != nil {
        return ""
    }

    return fmt.Sprintf("%x", f)
}
```