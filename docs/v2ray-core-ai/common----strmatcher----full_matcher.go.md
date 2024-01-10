# `v2ray-core\common\strmatcher\full_matcher.go`

```
package strmatcher

// 定义 FullMatcherGroup 结构体
type FullMatcherGroup struct {
    matchers map[string][]uint32  // 使用 map 存储字符串到 uint32 切片的映射关系
}

// 添加 domain 和 value 到 matchers 中
func (g *FullMatcherGroup) Add(domain string, value uint32) {
    // 如果 matchers 为空，创建一个新的 map
    if g.matchers == nil {
        g.matchers = make(map[string][]uint32)
    }

    // 将 value 添加到 domain 对应的切片中
    g.matchers[domain] = append(g.matchers[domain], value)
}

// 调用 Add 方法，将 fullMatcher 转换为字符串后添加到 matchers 中
func (g *FullMatcherGroup) addMatcher(m fullMatcher, value uint32) {
    g.Add(string(m), value)
}

// 根据输入的字符串在 matchers 中查找对应的 uint32 切片并返回
func (g *FullMatcherGroup) Match(str string) []uint32 {
    // 如果 matchers 为空，返回空切片
    if g.matchers == nil {
        return nil
    }

    // 根据输入的字符串返回对应的 uint32 切片
    return g.matchers[str]
}
```