# `v2ray-core\common\strmatcher\domain_matcher.go`

```
package strmatcher

import "strings"

// 将域名字符串按照"."分割成字符串数组
func breakDomain(domain string) []string {
    return strings.Split(domain, ".")
}

type node struct {
    values []uint32
    sub    map[string]*node
}

// DomainMatcherGroup 是一个大型域名匹配器的索引匹配器。
// 仅供测试可见。
type DomainMatcherGroup struct {
    root *node
}

// 向域名匹配器组中添加域名和对应的值
func (g *DomainMatcherGroup) Add(domain string, value uint32) {
    if g.root == nil {
        g.root = new(node)
    }

    current := g.root
    parts := breakDomain(domain)
    for i := len(parts) - 1; i >= 0; i-- {
        part := parts[i]
        if current.sub == nil {
            current.sub = make(map[string]*node)
        }
        next := current.sub[part]
        if next == nil {
            next = new(node)
            current.sub[part] = next
        }
        current = next
    }

    current.values = append(current.values, value)
}

// 向域名匹配器组中添加域名匹配器和对应的值
func (g *DomainMatcherGroup) addMatcher(m domainMatcher, value uint32) {
    g.Add(string(m), value)
}

// 匹配给定的域名，返回匹配到的值的数组
func (g *DomainMatcherGroup) Match(domain string) []uint32 {
    if domain == "" {
        return nil
    }

    current := g.root
    if current == nil {
        return nil
    }

    // 获取下一个部分的索引
    nextPart := func(idx int) int {
        for i := idx - 1; i >= 0; i-- {
            if domain[i] == '.' {
                return i
            }
        }
        return -1
    }

    matches := [][]uint32{}
    idx := len(domain)
    for {
        if idx == -1 || current.sub == nil {
            break
        }

        nidx := nextPart(idx)
        part := domain[nidx+1 : idx]
        next := current.sub[part]
        if next == nil {
            break
        }
        current = next
        idx = nidx
        if len(current.values) > 0 {
            matches = append(matches, current.values)
        }
    }
    switch len(matches) {
    case 0:
        return nil
    case 1:
        return matches[0]
    }
}
    // 默认情况下
    default:
        // 初始化一个空的无符号32位整数切片
        result := []uint32{}
        // 遍历匹配结果的索引
        for idx := range matches {
            // 反向插入，匹配得越远的子域名排名越高
            result = append(result, matches[len(matches)-1-idx]...)
        }
        // 返回结果
        return result
    }
# 闭合前面的函数定义
```