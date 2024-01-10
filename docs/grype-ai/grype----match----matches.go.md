# `grype\grype\match\matches.go`

```
package match

import (
    "sort"

    "github.com/anchore/grype/grype/pkg"  // 导入 grype 包中的 pkg 模块
    "github.com/anchore/grype/internal/log"  // 导入 grype 包中的 log 模块
)

type Matches struct {
    byFingerprint map[Fingerprint]Match  // 以指纹为键的匹配结果映射
    byPackage     map[pkg.ID][]Fingerprint  // 以包 ID 为键的指纹列表映射
}

func NewMatches(matches ...Match) Matches {
    m := newMatches()  // 创建新的匹配结果对象
    m.Add(matches...)  // 添加匹配结果
    return m  // 返回匹配结果对象
}

func newMatches() Matches {
    return Matches{
        byFingerprint: make(map[Fingerprint]Match),  // 初始化以指纹为键的匹配结果映射
        byPackage:     make(map[pkg.ID][]Fingerprint),  // 初始化以包 ID 为键的指纹列表映射
    }
}

// GetByPkgID returns a slice of potential matches from an ID
func (r *Matches) GetByPkgID(id pkg.ID) (matches []Match) {
    for _, fingerprint := range r.byPackage[id] {  // 遍历指定包 ID 对应的指纹列表
        matches = append(matches, r.byFingerprint[fingerprint])  // 将匹配结果添加到结果列表中
    }
    return matches  // 返回匹配结果列表
}

// AllByPkgID returns a map of all matches organized by package ID
func (r *Matches) AllByPkgID() map[pkg.ID][]Match {
    matches := make(map[pkg.ID][]Match)  // 创建以包 ID 为键的匹配结果映射
    for id, fingerprints := range r.byPackage {  // 遍历以包 ID 为键的指纹列表映射
        for _, fingerprint := range fingerprints {  // 遍历指纹列表
            matches[id] = append(matches[id], r.byFingerprint[fingerprint])  // 将匹配结果添加到对应包 ID 的结果列表中
        }
    }
    return matches  // 返回匹配结果映射
}

func (r *Matches) Merge(other Matches) {
    for _, fingerprints := range other.byPackage {  // 遍历其他匹配结果对象中的指纹列表
        for _, fingerprint := range fingerprints {  // 遍历指纹列表
            r.Add(other.byFingerprint[fingerprint])  // 将其他匹配结果对象中的匹配结果添加到当前对象中
        }
    }
}

func (r *Matches) Diff(other Matches) *Matches {
    diff := newMatches()  // 创建新的匹配结果对象
    for fingerprint := range r.byFingerprint {  // 遍历当前匹配结果对象中的指纹
        if _, exists := other.byFingerprint[fingerprint]; !exists {  // 如果当前指纹在其他匹配结果对象中不存在
            diff.Add(r.byFingerprint[fingerprint])  // 将当前指纹对应的匹配结果添加到差异匹配结果对象中
        }
    }
    return &diff  // 返回差异匹配结果对象的指针
}

func (r *Matches) Add(matches ...Match) {
    if len(matches) == 0 {  // 如果没有匹配结果
        return  // 直接返回
    }
}
    // 遍历匹配结果数组
    for _, newMatch := range matches {
        // 获取新匹配结果的指纹
        fingerprint := newMatch.Fingerprint()

        // 添加或合并新匹配结果与现有匹配结果
        if existingMatch, exists := r.byFingerprint[fingerprint]; exists {
            // 如果存在相同指纹的匹配结果，则合并新匹配结果
            if err := existingMatch.Merge(newMatch); err != nil {
                // 如果合并失败，则记录警告日志
                log.Warnf("unable to merge matches: original=%q new=%q : %w", existingMatch.String(), newMatch.String(), err)
                // TODO: 在这种情况下丢弃匹配结果，应该找到一种处理方式
            }
            // 更新指纹对应的匹配结果
            r.byFingerprint[fingerprint] = existingMatch
        } else {
            // 如果不存在相同指纹的匹配结果，则直接添加新匹配结果
            r.byFingerprint[fingerprint] = newMatch
        }

        // 记录每个匹配结果对应的包
        r.byPackage[newMatch.Package.ID] = append(r.byPackage[newMatch.Package.ID], fingerprint)
    }
// 枚举Matches对象中的匹配项，返回一个只读的Match通道
func (r *Matches) Enumerate() <-chan Match {
    // 创建一个Match通道
    channel := make(chan Match)
    // 启动一个匿名goroutine
    go func() {
        // 在函数返回时关闭通道
        defer close(channel)
        // 遍历r.byFingerprint中的匹配项，并将它们发送到通道中
        for _, match := range r.byFingerprint {
            channel <- match
        }
    }()
    // 返回通道
    return channel
}

// Sorted方法返回一个已排序的匹配项切片
func (r *Matches) Sorted() []Match {
    // 创建一个空的匹配项切片
    matches := make([]Match, 0)
    // 遍历通过Enumerate方法获取的匹配项
    for m := range r.Enumerate() {
        // 将匹配项追加到切片中
        matches = append(matches, m)
    }

    // 对切片进行排序
    sort.Sort(ByElements(matches))

    // 返回已排序的匹配项切片
    return matches
}

// Count方法返回结果中匹配项的总数
func (r *Matches) Count() int {
    // 返回r.byFingerprint中匹配项的数量
    return len(r.byFingerprint)
}
```