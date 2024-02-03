# `v2ray-core\common\strmatcher\strmatcher.go`

```go
package strmatcher

import (
    "regexp"
)

// Matcher is the interface to determine a string matches a pattern.
type Matcher interface {
    // Match returns true if the given string matches a predefined pattern.
    Match(string) bool
    String() string
}

// Type is the type of the matcher.
type Type byte

const (
    // Full is the type of matcher that the input string must exactly equal to the pattern.
    Full Type = iota
    // Substr is the type of matcher that the input string must contain the pattern as a sub-string.
    Substr
    // Domain is the type of matcher that the input string must be a sub-domain or itself of the pattern.
    Domain
    // Regex is the type of matcher that the input string must matches the regular-expression pattern.
    Regex
)

// New creates a new Matcher based on the given pattern.
func (t Type) New(pattern string) (Matcher, error) {
    switch t {
    case Full:
        return fullMatcher(pattern), nil
    case Substr:
        return substrMatcher(pattern), nil
    case Domain:
        return domainMatcher(pattern), nil
    case Regex:
        r, err := regexp.Compile(pattern)
        if err != nil {
            return nil, err
        }
        return &regexMatcher{
            pattern: r,
        }, nil
    default:
        panic("Unknown type")
    }
}

// IndexMatcher is the interface for matching with a group of matchers.
type IndexMatcher interface {
    // Match returns the index of a matcher that matches the input. It returns empty array if no such matcher exists.
    Match(input string) []uint32
}

type matcherEntry struct {
    m  Matcher
    id uint32
}

// MatcherGroup is an implementation of IndexMatcher.
// Empty initialization works.
type MatcherGroup struct {
    count         uint32
    fullMatcher   FullMatcherGroup
    domainMatcher DomainMatcherGroup
    otherMatchers []matcherEntry
}

// Add adds a new Matcher into the MatcherGroup, and returns its index. The index will never be 0.
// Add 方法用于向 MatcherGroup 中添加 Matcher，并返回添加的 Matcher 的编号
func (g *MatcherGroup) Add(m Matcher) uint32 {
    // 增加 MatcherGroup 中 Matcher 的数量
    g.count++
    // 将当前 Matcher 的编号赋值给 c
    c := g.count

    // 根据 Matcher 的类型进行不同的处理
    switch tm := m.(type) {
    // 如果是 fullMatcher 类型，则调用 fullMatcher 的 addMatcher 方法
    case fullMatcher:
        g.fullMatcher.addMatcher(tm, c)
    // 如果是 domainMatcher 类型，则调用 domainMatcher 的 addMatcher 方法
    case domainMatcher:
        g.domainMatcher.addMatcher(tm, c)
    // 其他类型的 Matcher 则添加到 otherMatchers 列表中
    default:
        g.otherMatchers = append(g.otherMatchers, matcherEntry{
            m:  m,
            id: c,
        })
    }

    // 返回添加的 Matcher 的编号
    return c
}

// Match 方法实现了 IndexMatcher.Match 接口
func (g *MatcherGroup) Match(pattern string) []uint32 {
    // 初始化结果数组
    result := []uint32{}
    // 将 fullMatcher 匹配结果添加到结果数组中
    result = append(result, g.fullMatcher.Match(pattern)...)
    // 将 domainMatcher 匹配结果添加到结果数组中
    result = append(result, g.domainMatcher.Match(pattern)...)
    // 遍历 otherMatchers，将匹配结果添加到结果数组中
    for _, e := range g.otherMatchers {
        if e.m.Match(pattern) {
            result = append(result, e.id)
        }
    }
    // 返回匹配结果数组
    return result
}

// Size 方法返回 MatcherGroup 中 Matcher 的数量
func (g *MatcherGroup) Size() uint32 {
    // 返回 Matcher 的数量
    return g.count
}
```