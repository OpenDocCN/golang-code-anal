# `grype\internal\stringutil\stringset.go`

```
# 定义一个名为 stringutil 的包

# 定义一个名为 StringSet 的类型，它是一个映射，将字符串映射到空结构体
type StringSet map[string]struct{}

# 创建一个新的 StringSet 实例
func NewStringSet() StringSet {
    return make(StringSet)
}

# 从给定的字符串切片创建一个新的 StringSet 实例
func NewStringSetFromSlice(start []string) StringSet {
    ret := make(StringSet)
    # 遍历给定的字符串切片，将每个字符串添加到新的 StringSet 实例中
    for _, s := range start {
        ret.Add(s)
    }
    return ret
}

# 向 StringSet 实例中添加一个字符串
func (s StringSet) Add(i string) {
    s[i] = struct{}{}
}
# 从字符串集合中删除指定的字符串
func (s StringSet) Remove(i string) {
    delete(s, i)
}

# 检查字符串集合中是否包含指定的字符串，返回布尔值
func (s StringSet) Contains(i string) bool {
    _, ok := s[i]
    return ok
}

# 将字符串集合转换为字符串切片
func (s StringSet) ToSlice() []string {
    # 创建一个与字符串集合大小相同的字符串切片
    ret := make([]string, len(s))
    idx := 0
    # 遍历字符串集合，将元素添加到字符串切片中
    for v := range s:
        ret[idx] = v
        idx++
    }
    return ret
}
```