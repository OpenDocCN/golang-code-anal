# `grype\grype\match\sort.go`

```
package match

import "sort"

var _ sort.Interface = (*ByElements)(nil)

type ByElements []Match

// ByElements 类型实现了 sort.Interface 接口

// Len 返回集合中的元素数量
func (m ByElements) Len() int {
	return len(m)
}

// Less 报告索引为 i 的元素是否应该在索引为 j 的元素之前排序
func (m ByElements) Less(i, j int) bool {
	if m[i].Vulnerability.ID == m[j].Vulnerability.ID {
		if m[i].Package.Name == m[j].Package.Name {
			if m[i].Package.Version == m[j].Package.Version {
				return m[i].Package.Type < m[j].Package.Type
			}
```
在这个示例中，我们看到了一个自定义类型 ByElements，它实现了 sort.Interface 接口的方法 Len 和 Less。这个类型用于对 Match 结构体的切片进行排序。
// 按照包的版本号进行排序，如果版本号相同则按照包的名称进行排序，如果包的名称也相同则按照漏洞的ID进行排序
func (m ByElements) Less(i, j int) bool {
    if m[i].Vulnerability.ID == m[j].Vulnerability.ID {
        if m[i].Package.Name == m[j].Package.Name {
            return m[i].Package.Version < m[j].Package.Version
        }
        return m[i].Package.Name < m[j].Package.Name
    }
    return m[i].Vulnerability.ID < m[j].Vulnerability.ID
}

// 交换索引为i和j的元素
func (m ByElements) Swap(i, j int) {
    m[i], m[j] = m[j], m[i]
}
```