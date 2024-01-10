# `grype\grype\db\v4\store\diff.go`

```
package store

import (
    "github.com/wagoodman/go-partybus"  // 导入第三方库 go-partybus
    "github.com/wagoodman/go-progress"  // 导入第三方库 go-progress

    v4 "github.com/anchore/grype/grype/db/v4"  // 导入 anchore/grype/grype/db/v4 包并重命名为 v4
    "github.com/anchore/grype/grype/event"  // 导入 anchore/grype/grype/event 包
    "github.com/anchore/grype/grype/event/monitor"  // 导入 anchore/grype/grype/event/monitor 包
    "github.com/anchore/grype/internal/bus"  // 导入 anchore/grype/internal/bus 包
)

type storeKey struct {
    id          string  // 定义 storeKey 结构体，包含 id 字段
    namespace   string  // 定义 storeKey 结构体，包含 namespace 字段
    packageName string  // 定义 storeKey 结构体，包含 packageName 字段
}

type PkgMap = map[storeKey][]string  // 定义 PkgMap 类型为 storeKey 到字符串切片的映射

type storeVulnerabilityList struct {
    items map[storeKey][]storeVulnerability  // 定义 storeVulnerabilityList 结构体，包含 items 字段，类型为 storeKey 到 storeVulnerability 切片的映射
    seen  bool  // 定义 storeVulnerabilityList 结构体，包含 seen 字段，类型为布尔值
}
type storeVulnerability struct {
    item *v4.Vulnerability  // 定义 storeVulnerability 结构体，包含 item 字段，类型为指向 v4.Vulnerability 结构体的指针
    seen bool  // 定义 storeVulnerability 结构体，包含 seen 字段，类型为布尔值
}
type storeMetadata struct {
    item *v4.VulnerabilityMetadata  // 定义 storeMetadata 结构体，包含 item 字段，类型为指向 v4.VulnerabilityMetadata 结构体的指针
    seen bool  // 定义 storeMetadata 结构体，包含 seen 字段，类型为布尔值
}

// 创建用于跟踪数据库差异进度的手动进度条
func trackDiff(total int64) (*progress.Manual, *progress.Manual, *progress.Stage) {
    stageProgress := &progress.Manual{}  // 创建 progress.Manual 类型的指针 stageProgress
    stageProgress.SetTotal(total)  // 设置 stageProgress 的总数为 total
    differencesDiscovered := &progress.Manual{}  // 创建 progress.Manual 类型的指针 differencesDiscovered
    stager := &progress.Stage{}  // 创建 progress.Stage 类型的指针 stager

    bus.Publish(partybus.Event{  // 发布 partybus.Event 事件
        Type: event.DatabaseDiffingStarted,  // 设置事件类型为 DatabaseDiffingStarted
        Value: monitor.DBDiff{  // 设置事件值为 monitor.DBDiff 结构体
            Stager:                stager,  // 设置 Stager 字段为 stager
            StageProgress:         progress.Progressable(stageProgress),  // 设置 StageProgress 字段为 stageProgress
            DifferencesDiscovered: progress.Monitorable(differencesDiscovered),  // 设置 DifferencesDiscovered 字段为 differencesDiscovered
        },
    })
    return stageProgress, differencesDiscovered, stager  // 返回 stageProgress、differencesDiscovered 和 stager
}

// 根据模型构建漏洞包映射的函数
func buildVulnerabilityPkgsMap(models *[]v4.Vulnerability) *map[storeKey][]string {
    storeMap := make(map[storeKey][]string)  // 创建 storeKey 到字符串切片的映射 storeMap
    for _, m := range *models {  // 遍历 models 切片
        model := m  // 将 m 赋值给 model
        k := getVulnerabilityParentKey(model)  // 调用 getVulnerabilityParentKey 函数获取漏洞父键
        if storeVuln, exists := storeMap[k]; exists {  // 判断 storeMap 中是否存在 k
            storeMap[k] = append(storeVuln, model.PackageName)  // 如果存在，则将 model.PackageName 追加到 storeVuln 中
        } else {
            storeMap[k] = []string{model.PackageName}  // 如果不存在，则将 model.PackageName 存入 storeMap[k] 中
        }
    }
    return &storeMap  // 返回 storeMap 的指针
}

// 使用包映射信息创建给定键的差异
// 创建一个 Diff 对象，用于表示两个包映射之间的差异
func createDiff(baseStore, targetStore *PkgMap, key storeKey, reason v4.DiffReason) *v4.Diff {
    // 创建一个空的包映射
    pkgMap := make(map[string]struct{})

    // 将包名设置为空字符串
    key.packageName = ""
    // 如果 baseStore 不为空，则将其包含的包名添加到包映射中
    if baseStore != nil {
        if basePkgs, exists := (*baseStore)[key]; exists {
            for _, pkg := range basePkgs {
                pkgMap[pkg] = struct{}{}
            }
        }
    }
    // 如果 targetStore 不为空，则将其包含的包名添加到包映射中
    if targetStore != nil {
        if targetPkgs, exists := (*targetStore)[key]; exists {
            for _, pkg := range targetPkgs {
                pkgMap[pkg] = struct{}{}
            }
        }
    }
    // 将包映射中的包名转换为数组
    pkgs := []string{}
    for pkg := range pkgMap {
        pkgs = append(pkgs, pkg)
    }

    // 返回一个 Diff 对象
    return &v4.Diff{
        Reason:    reason,
        ID:        key.id,
        Namespace: key.namespace,
        Packages:  pkgs,
    }
}

// 从漏洞中获取未打包的键
func getVulnerabilityParentKey(vuln v4.Vulnerability) storeKey {
    return storeKey{vuln.ID, vuln.Namespace, ""}
}

// 从漏洞中获取已打包的键
func getVulnerabilityKey(vuln v4.Vulnerability) storeKey {
    return storeKey{vuln.ID, vuln.Namespace, vuln.PackageName}
}

// 定义一个 VulnerabilitySet 结构
type VulnerabilitySet struct {
    data map[storeKey]*storeVulnerabilityList
}

// 创建一个新的 VulnerabilitySet 对象
func NewVulnerabilitySet(models *[]v4.Vulnerability) *VulnerabilitySet {
    // 创建一个包含指定数量元素的映射
    m := make(map[storeKey]*storeVulnerabilityList, len(*models));
    # 遍历模型列表
    for _, mm := range *models {
        # 将模型赋值给新变量
        model := mm
        # 获取漏洞的父键
        parentKey := getVulnerabilityParentKey(model)
        # 获取漏洞的键
        vulnKey := getVulnerabilityKey(model)
        # 如果父键存在于映射中
        if storeVuln, exists := m[parentKey]; exists {
            # 如果漏洞键存在于父键对应的项目中
            if kk, exists := storeVuln.items[vulnKey]; exists {
                # 将漏洞添加到项目中
                storeVuln.items[vulnKey] = append(kk, storeVulnerability{
                    item: &model,
                    seen: false,
                })
            } else {
                # 如果漏洞键不存在，创建新的项目列表
                storeVuln.items[vulnKey] = []storeVulnerability{{&model, false}}
            }
        } else {
            # 如果父键不存在，创建新的漏洞列表
            vuln := storeVulnerabilityList{
                items: make(map[storeKey][]storeVulnerability),
                seen:  false,
            }
            vuln.items[vulnKey] = []storeVulnerability{{&model, false}}
            m[parentKey] = &vuln
        }
    }
    # 返回漏洞集合
    return &VulnerabilitySet{
        data: m,
    }
# 检查给定的漏洞是否在漏洞集合中
func (v *VulnerabilitySet) in(item v4.Vulnerability) bool {
    # 检查漏洞是否存在于漏洞集合中，如果存在则返回 true，否则返回 false
    _, exists := v.data[getVulnerabilityParentKey(item)]
    return exists
}

# 匹配给定的漏洞是否存在于漏洞集合中
func (v *VulnerabilitySet) match(item v4.Vulnerability) bool {
    # 检查漏洞是否存在于漏洞集合中
    if parent, exists := v.data[getVulnerabilityParentKey(item)]; exists {
        # 如果存在，则将其标记为已见
        parent.seen = true
        # 获取漏洞的键
        key := getVulnerabilityKey(item)
        # 检查漏洞的子项是否存在
        if children, exists := parent.items[key]; exists {
            # 遍历子项，如果找到匹配的漏洞，则将其标记为已见并返回 true
            for idx, child := range children {
                if item.Equal(*child.item) {
                    children[idx].seen = true
                    return true
                }
            }
        }
    }
    # 如果未找到匹配的漏洞，则返回 false
    return false
}

# 获取未匹配的漏洞
func (v *VulnerabilitySet) getUnmatched() ([]storeKey, []storeKey) {
    # 初始化未见的漏洞和部分未见的漏洞列表
    notSeen := []storeKey{}
    notEntirelySeen := []storeKey{}
    # 遍历漏洞集合
    for k, item := range v.data {
        # 如果漏洞未被标记为已见，则将其添加到未见的漏洞列表中
        if !item.seen {
            notSeen = append(notSeen, k)
            continue
        }
        # 遍历漏洞的子项
        componentLoop:
        for _, components := range item.items {
            for _, component := range components {
                # 如果子项中有未被标记为已见的漏洞，则将其添加到部分未见的漏洞列表中
                if !component.seen {
                    notEntirelySeen = append(notEntirelySeen, k)
                    break componentLoop
                }
            }
        }
    }
    # 返回未见的漏洞和部分未见的漏洞列表
    return notSeen, notEntirelySeen
}

# 比较漏洞
func diffVulnerabilities(baseModels, targetModels *[]v4.Vulnerability, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v4.Diff {
    # 创建漏洞差异映射
    diffs := make(map[string]*v4.Diff)
    # 初始化漏洞集合
    m := NewVulnerabilitySet(baseModels)
    # 遍历目标模型列表
    for _, tModel := range *targetModels {
        # 复制目标模型
        targetModel := tModel
        # 获取漏洞键
        k := getVulnerabilityKey(targetModel)
        # 如果目标模型在匹配模型中
        if m.in(targetModel) {
            # 进行匹配
            matched := m.match(targetModel)
            # 如果未匹配
            if !matched {
                # 如果差异中存在相同的键
                if _, exists := diffs[k.id+k.namespace]; exists {
                    # 继续下一次循环
                    continue
                }
                # 创建差异并添加到差异列表中
                diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v4.DiffChanged)
                # 不同项计数加一
                differentItems.Increment()
            }
        } else {
            # 如果差异中存在相同的键
            if _, exists := diffs[k.id+k.namespace]; exists {
                # 继续下一次循环
                continue
            }
            # 创建差异并添加到差异列表中
            diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v4.DiffAdded)
            # 不同项计数加一
            differentItems.Increment()
        }
    }
    # 获取未匹配和部分匹配的键列表
    notSeen, partialSeen := m.getUnmatched()
    # 遍历部分匹配的键列表
    for _, k := range partialSeen {
        # 如果差异中存在相同的键
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 继续下一次循环
            continue
        }
        # 创建差异并添加到差异列表中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v4.DiffChanged)
        # 不同项计数加一
        differentItems.Increment()
    }
    # 遍历未匹配的键列表
    for _, k := range notSeen {
        # 如果差异中存在相同的键
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 继续下一次循环
            continue
        }
        # 创建差异并添加到差异列表中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v4.DiffRemoved)
        # 不同项计数加一
        differentItems.Increment()
    }

    # 返回差异列表的指针
    return &diffs
// 定义 MetadataSet 结构体，包含一个存储 storeKey 到 storeMetadata 指针的映射
type MetadataSet struct {
    data map[storeKey]*storeMetadata
}

// 创建并返回一个 MetadataSet 对象，根据给定的 VulnerabilityMetadata 模型列表
func NewMetadataSet(models *[]v4.VulnerabilityMetadata) *MetadataSet {
    // 创建一个存储 storeKey 到 storeMetadata 指针的映射
    m := make(map[storeKey]*storeMetadata, len(*models))
    // 遍历给定的模型列表，为每个模型创建 storeMetadata 对象，并添加到映射中
    for _, mm := range *models {
        model := mm
        m[getMetadataKey(model)] = &storeMetadata{
            item: &model,
            seen: false,
        }
    }
    // 返回包含映射的 MetadataSet 对象
    return &MetadataSet{
        data: m,
    }
}

// 判断给定的 VulnerabilityMetadata 是否存在于 MetadataSet 中
func (v *MetadataSet) in(item v4.VulnerabilityMetadata) bool {
    _, exists := v.data[getMetadataKey(item)]
    return exists
}

// 判断给定的 VulnerabilityMetadata 是否匹配 MetadataSet 中的元数据，并标记为已匹配
func (v *MetadataSet) match(item v4.VulnerabilityMetadata) bool {
    if baseModel, exists := v.data[getMetadataKey(item)]; exists {
        baseModel.seen = true
        return baseModel.item.Equal(item)
    }
    return false
}

// 获取未匹配的元数据的 storeKey 列表
func (v *MetadataSet) getUnmatched() []storeKey {
    notSeen := []storeKey{}
    // 遍历 MetadataSet 中的元数据，将未匹配的元数据的 storeKey 添加到列表中
    for k, item := range v.data {
        if !item.seen {
            notSeen = append(notSeen, k)
        }
    }
    // 返回未匹配的元数据的 storeKey 列表
    return notSeen
}

// 比较两个 VulnerabilityMetadata 模型列表的差异，并返回差异的映射
func diffVulnerabilityMetadata(baseModels, targetModels *[]v4.VulnerabilityMetadata, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v4.Diff {
    // 创建一个存储字符串到 v4.Diff 指针的映射
    diffs := make(map[string]*v4.Diff)
    // 创建一个包含 baseModels 元数据的 MetadataSet 对象
    m := NewMetadataSet(baseModels)

    // 遍历目标模型列表，比较差异并更新 diffs 映射
    for _, tModel := range *targetModels {
        targetModel := tModel
        k := getMetadataKey(targetModel)
        if m.in(targetModel) {
            if !m.match(targetModel) {
                if _, exists := diffs[k.id+k.namespace]; exists {
                    continue
                }
                diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v4.DiffChanged)
                differentItems.Increment()
            }
        } else {
            if _, exists := diffs[k.id+k.namespace]; exists {
                continue
            }
            diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v4.DiffAdded)
            differentItems.Increment()
        }
    }
}
    # 遍历 m.getUnmatched() 返回的未匹配项列表
    for _, k := range m.getUnmatched() {
        # 检查 diffs 中是否已存在以 k.id+k.namespace 为键的项
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 如果存在，则跳过当前循环，继续下一次循环
            continue
        }
        # 以 k.id+k.namespace 为键，创建一个新的差异项，并添加到 diffs 中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v4.DiffRemoved)
        # 不同项计数器加一
        differentItems.Increment()
    }

    # 返回 diffs 的指针
    return &diffs
# 定义一个函数，用于获取漏洞元数据的键
func getMetadataKey(metadata v4.VulnerabilityMetadata) storeKey {
    # 返回一个包含漏洞ID、命名空间和空字符串的存储键对象
    return storeKey{metadata.ID, metadata.Namespace, ""}
}
```