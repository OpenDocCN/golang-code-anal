# `grype\grype\match\matches.go`

```
// 导入所需的包
package match

import (
	"sort"  // 导入排序包

	"github.com/anchore/grype/grype/pkg"  // 导入外部包
	"github.com/anchore/grype/internal/log"  // 导入内部日志包
)

// 定义 Matches 结构体
type Matches struct {
	byFingerprint map[Fingerprint]Match  // 以指纹为键的匹配映射
	byPackage     map[pkg.ID][]Fingerprint  // 以包 ID 为键的指纹列表映射
}

// 创建新的 Matches 实例
func NewMatches(matches ...Match) Matches {
	// 创建新的 Matches 实例
	m := newMatches()
	// 添加匹配到 Matches 实例中
	m.Add(matches...)
	// 返回 Matches 实例
	return m
}
// 创建一个新的 Matches 对象，包含两个空的映射
func newMatches() Matches {
	return Matches{
		byFingerprint: make(map[Fingerprint]Match),  // 通过指纹来匹配的映射
		byPackage:     make(map[pkg.ID][]Fingerprint),  // 通过包ID来匹配的映射
	}
}

// 通过包ID返回潜在匹配的切片
func (r *Matches) GetByPkgID(id pkg.ID) (matches []Match) {
	for _, fingerprint := range r.byPackage[id] {
		matches = append(matches, r.byFingerprint[fingerprint])  // 将匹配结果添加到切片中
	}
	return matches
}

// 返回按包ID组织的所有匹配的映射
func (r *Matches) AllByPkgID() map[pkg.ID][]Match {
	matches := make(map[pkg.ID][]Match)
	for id, fingerprints := range r.byPackage {
		for _, fingerprint := range fingerprints {
		# 将指纹对应的匹配结果添加到匹配字典中
		matches[id] = append(matches[id], r.byFingerprint[fingerprint])
		# 结束当前循环
		}
	}
	# 返回匹配结果字典
	return matches
}

# 合并另一个匹配结果对象到当前对象
func (r *Matches) Merge(other Matches) {
	# 遍历另一个匹配结果对象的包列表
	for _, fingerprints := range other.byPackage {
		# 遍历另一个匹配结果对象的指纹列表
		for _, fingerprint := range fingerprints {
			# 将另一个匹配结果对象中指纹对应的匹配结果添加到当前对象中
			r.Add(other.byFingerprint[fingerprint])
		}
	}
}

# 比较两个匹配结果对象的差异
func (r *Matches) Diff(other Matches) *Matches {
	# 创建一个新的匹配结果对象用于存储差异
	diff := newMatches()
	# 遍历当前匹配结果对象的指纹列表
	for fingerprint := range r.byFingerprint:
		# 如果当前指纹在另一个匹配结果对象中不存在，则将其对应的匹配结果添加到差异对象中
		if _, exists := other.byFingerprint[fingerprint]; !exists {
			diff.Add(r.byFingerprint[fingerprint])
		}
// 添加匹配项到Matches对象中
func (r *Matches) Add(matches ...Match) {
	// 如果没有匹配项，则直接返回
	if len(matches) == 0 {
		return
	}
	// 遍历传入的匹配项
	for _, newMatch := range matches {
		// 获取匹配项的指纹
		fingerprint := newMatch.Fingerprint()

		// 添加或合并新的匹配项到现有的匹配项中
		if existingMatch, exists := r.byFingerprint[fingerprint]; exists {
			// 如果存在相同指纹的匹配项，则尝试合并
			if err := existingMatch.Merge(newMatch); err != nil {
				// 如果合并失败，则记录警告信息
				log.Warnf("unable to merge matches: original=%q new=%q : %w", existingMatch.String(), newMatch.String(), err)
				// TODO: 在这种情况下丢弃匹配项，应该找到一种处理方式
			}
			// 更新匹配项
			r.byFingerprint[fingerprint] = existingMatch
		} else {
			// 如果不存在相同指纹的匹配项，则直接添加新的匹配项
			r.byFingerprint[fingerprint] = newMatch
		}
	}
}
		// 跟踪匹配项对应的包
		r.byPackage[newMatch.Package.ID] = append(r.byPackage[newMatch.Package.ID], fingerprint)
	}
}

func (r *Matches) Enumerate() <-chan Match {
	// 创建一个通道用于枚举匹配项
	channel := make(chan Match)
	// 启动一个 goroutine 来枚举匹配项
	go func() {
		// 在函数结束时关闭通道
		defer close(channel)
		// 遍历按指纹分组的匹配项，并将它们发送到通道中
		for _, match := range r.byFingerprint {
			channel <- match
		}
	}()
	// 返回通道
	return channel
}

func (r *Matches) Sorted() []Match {
	// 创建一个空的匹配项切片
	matches := make([]Match, 0)
// 遍历结果集合中的元素，并将每个元素添加到matches切片中
for m := range r.Enumerate() {
    matches = append(matches, m)
}

// 对matches切片中的元素进行排序
sort.Sort(ByElements(matches))

// 返回排序后的matches切片
return matches
}

// Count方法返回结果集合中匹配项的总数
func (r *Matches) Count() int {
    // 返回结果集合中按指纹分类的数量
    return len(r.byFingerprint)
}
```