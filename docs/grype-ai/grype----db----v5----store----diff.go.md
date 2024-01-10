# `grype\grype\db\v5\store\diff.go`

```
package store

import (
    "github.com/wagoodman/go-partybus"  // 导入事件总线库
    "github.com/wagoodman/go-progress"  // 导入进度条库

    v5 "github.com/anchore/grype/grype/db/v5"  // 导入版本5的漏洞数据库
    "github.com/anchore/grype/grype/event"  // 导入事件相关库
    "github.com/anchore/grype/grype/event/monitor"  // 导入监控事件相关库
    "github.com/anchore/grype/internal/bus"  // 导入内部总线库
)

type storeKey struct {
    id          string  // 存储键的ID
    namespace   string  // 存储键的命名空间
    packageName string  // 存储键的包名
}

type PkgMap = map[storeKey][]string  // 定义存储键到包名列表的映射类型

type storeVulnerabilityList struct {
    items map[storeKey][]storeVulnerability  // 存储键到漏洞列表的映射
    seen  bool  // 是否已经查看过
}
type storeVulnerability struct {
    item *v5.Vulnerability  // 漏洞对象
    seen bool  // 是否已经查看过
}
type storeMetadata struct {
    item *v5.VulnerabilityMetadata  // 漏洞元数据对象
    seen bool  // 是否已经查看过
}

// 创建用于跟踪数据库差异进度的手动进度条
func trackDiff(total int64) (*progress.Manual, *progress.Manual, *progress.Stage) {
    stageProgress := &progress.Manual{}  // 创建阶段进度的手动进度条
    stageProgress.SetTotal(total)  // 设置阶段进度的总数
    differencesDiscovered := &progress.Manual{}  // 创建发现差异的手动进度条
    stager := &progress.Stage{}  // 创建阶段对象

    bus.Publish(partybus.Event{  // 发布事件，通知数据库差异开始
        Type: event.DatabaseDiffingStarted,
        Value: monitor.DBDiff{
            Stager:                stager,
            StageProgress:         progress.Progressable(stageProgress),
            DifferencesDiscovered: progress.Monitorable(differencesDiscovered),
        },
    })
    return stageProgress, differencesDiscovered, stager  // 返回阶段进度、发现差异的进度和阶段对象
}

// 根据漏洞模型构建漏洞包映射
func buildVulnerabilityPkgsMap(models *[]v5.Vulnerability) *map[storeKey][]string {
    storeMap := make(map[storeKey][]string)  // 创建存储键到包名列表的映射
    for _, m := range *models {  // 遍历漏洞模型列表
        model := m
        k := getVulnerabilityParentKey(model)  // 获取漏洞的父级键
        if storeVuln, exists := storeMap[k]; exists {  // 如果存储键已存在
            storeMap[k] = append(storeVuln, model.PackageName)  // 则将包名追加到列表中
        } else {
            storeMap[k] = []string{model.PackageName}  // 否则创建新的包名列表
        }
    }
    return &storeMap  // 返回存储键到包名列表的映射
}

// 使用包映射信息创建给定键的差异
// 根据更新影响的相关包创建差异
func createDiff(baseStore, targetStore *PkgMap, key storeKey, reason v5.DiffReason) *v5.Diff {
    // 创建一个包名的映射，用于存储不同版本的包
    pkgMap := make(map[string]struct{})

    // 将包名设置为空字符串
    key.packageName = ""
    // 如果基础存储不为空
    if baseStore != nil {
        // 如果基础存储中存在指定键
        if basePkgs, exists := (*baseStore)[key]; exists {
            // 遍历基础存储中的包名，将其添加到包名映射中
            for _, pkg := range basePkgs {
                pkgMap[pkg] = struct{}{}
            }
        }
    }
    // 如果目标存储不为空
    if targetStore != nil {
        // 如果目标存储中存在指定键
        if targetPkgs, exists := (*targetStore)[key]; exists {
            // 遍历目标存储中的包名，将其添加到包名映射中
            for _, pkg := range targetPkgs {
                pkgMap[pkg] = struct{}{}
            }
        }
    }
    // 创建一个包名切片
    pkgs := []string{}
    // 遍历包名映射，将包名添加到切片中
    for pkg := range pkgMap {
        pkgs = append(pkgs, pkg)
    }

    // 返回差异对象
    return &v5.Diff{
        Reason:    reason,
        ID:        key.id,
        Namespace: key.namespace,
        Packages:  pkgs,
    }
}

// 从漏洞中获取未打包的键
func getVulnerabilityParentKey(vuln v5.Vulnerability) storeKey {
    return storeKey{vuln.ID, vuln.Namespace, ""}
}

// 从漏洞中获取已打包的键
func getVulnerabilityKey(vuln v5.Vulnerability) storeKey {
    return storeKey{vuln.ID, vuln.Namespace, vuln.PackageName}
}

// 漏洞集合结构
type VulnerabilitySet struct {
    data map[storeKey]*storeVulnerabilityList
}

// 创建新的漏洞集合
func NewVulnerabilitySet(models *[]v5.Vulnerability) *VulnerabilitySet {
    // 创建一个存储键和存储漏洞列表的映射
    m := make(map[storeKey]*storeVulnerabilityList, len(*models));
    # 遍历模型列表
    for _, mm := range *models {
        # 复制模型对象
        model := mm
        # 获取漏洞的父键
        parentKey := getVulnerabilityParentKey(model)
        # 获取漏洞的键
        vulnKey := getVulnerabilityKey(model)
        # 如果父键存在于映射中
        if storeVuln, exists := m[parentKey]; exists {
            # 如果漏洞键存在于父键对应的项目中
            if kk, exists := storeVuln.items[vulnKey]; exists {
                # 将漏洞添加到现有项目中
                storeVuln.items[vulnKey] = append(kk, storeVulnerability{
                    item: &model,
                    seen: false,
                })
            } else {
                # 创建新的项目并将漏洞添加到其中
                storeVuln.items[vulnKey] = []storeVulnerability{{&model, false}}
            }
        } else {
            # 创建新的漏洞列表
            vuln := storeVulnerabilityList{
                items: make(map[storeKey][]storeVulnerability),
                seen:  false,
            }
            vuln.items[vulnKey] = []storeVulnerability{{&model, false}}
            # 将新的漏洞列表添加到映射中
            m[parentKey] = &vuln
        }
    }
    # 返回包含映射的漏洞集合
    return &VulnerabilitySet{
        data: m,
    }
# 检查给定的漏洞是否在漏洞集合中
func (v *VulnerabilitySet) in(item v5.Vulnerability) bool:
    # 检查漏洞是否存在于漏洞集合中，如果存在则返回 true
    _, exists := v.data[getVulnerabilityParentKey(item)]
    return exists

# 检查给定的漏洞是否与漏洞集合中的某个漏洞匹配
func (v *VulnerabilitySet) match(item v5.Vulnerability) bool:
    # 检查漏洞是否与漏洞集合中的某个漏洞匹配
    if parent, exists := v.data[getVulnerabilityParentKey(item)]; exists:
        # 如果父漏洞存在于漏洞集合中，则将其标记为已查看
        parent.seen = true
        # 获取漏洞的键值
        key := getVulnerabilityKey(item)
        # 检查漏洞的子漏洞是否存在于漏洞集合中
        if children, exists := parent.items[key]; exists:
            # 遍历子漏洞列表，如果找到匹配的子漏洞，则将其标记为已查看并返回 true
            for idx, child := range children:
                if item.Equal(*child.item):
                    children[idx].seen = true
                    return true
    # 如果没有找到匹配的漏洞，则返回 false
    return false

# 获取未匹配的漏洞
func (v *VulnerabilitySet) getUnmatched() ([]storeKey, []storeKey):
    # 初始化未查看的漏洞列表和部分查看的漏洞列表
    notSeen := []storeKey{}
    notEntirelySeen := []storeKey{}
    # 遍历漏洞集合中的漏洞
    for k, item := range v.data:
        # 如果漏洞未被查看，则将其添加到未查看的漏洞列表中
        if !item.seen:
            notSeen = append(notSeen, k)
            continue
        # 否则，遍历漏洞的子漏洞列表
        componentLoop:
        for _, components := range item.items:
            for _, component := range components:
                # 如果子漏洞未被完全查看，则将其添加到部分查看的漏洞列表中并跳出循环
                if !component.seen:
                    notEntirelySeen = append(notEntirelySeen, k)
                    break componentLoop
    # 返回未查看的漏洞列表和部分查看的漏洞列表
    return notSeen, notEntirelySeen

# 比较漏洞
func diffVulnerabilities(baseModels, targetModels *[]v5.Vulnerability, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v5.Diff:
    # 初始化漏洞差异的映射
    diffs := make(map[string]*v5.Diff)
    # 创建基础漏洞集合
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
                # 如果差异中已存在相同键
                if _, exists := diffs[k.id+k.namespace]; exists {
                    # 继续下一次循环
                    continue
                }
                # 创建差异并添加到差异列表中
                diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v5.DiffChanged)
                # 不同项计数加一
                differentItems.Increment()
            }
        } else {
            # 如果差异中已存在相同键
            if _, exists := diffs[k.id+k.namespace]; exists {
                # 继续下一次循环
                continue
            }
            # 创建差异并添加到差异列表中
            diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v5.DiffAdded)
            # 不同项计数加一
            differentItems.Increment()
        }
    }
    # 获取未匹配和部分匹配的键列表
    notSeen, partialSeen := m.getUnmatched()
    # 遍历部分匹配的键列表
    for _, k := range partialSeen {
        # 如果差异中已存在相同键
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 继续下一次循环
            continue
        }
        # 创建差异并添加到差异列表中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v5.DiffChanged)
        # 不同项计数加一
        differentItems.Increment()
    }
    # 遍历未匹配的键列表
    for _, k := range notSeen {
        # 如果差异中已存在相同键
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 继续下一次循环
            continue
        }
        # 创建差异并添加到差异列表中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v5.DiffRemoved)
        # 不同项计数加一
        differentItems.Increment()
    }

    # 返回差异列表的指针
    return &diffs
// 定义 MetadataSet 结构体，包含一个存储 storeMetadata 的 map
type MetadataSet struct {
    data map[storeKey]*storeMetadata
}

// 创建并返回一个 MetadataSet 对象，根据给定的 VulnerabilityMetadata 模型列表
func NewMetadataSet(models *[]v5.VulnerabilityMetadata) *MetadataSet {
    // 创建一个存储 storeMetadata 的 map，长度为给定模型列表的长度
    m := make(map[storeKey]*storeMetadata, len(*models))
    // 遍历给定模型列表，为每个模型创建 storeMetadata 对象，并添加到 map 中
    for _, mm := range *models {
        model := mm
        m[getMetadataKey(model)] = &storeMetadata{
            item: &model,
            seen: false,
        }
    }
    // 返回包含创建的 map 的 MetadataSet 对象
    return &MetadataSet{
        data: m,
    }
}

// 判断给定的 VulnerabilityMetadata 是否存在于 MetadataSet 中
func (v *MetadataSet) in(item v5.VulnerabilityMetadata) bool {
    _, exists := v.data[getMetadataKey(item)]
    return exists
}

// 判断给定的 VulnerabilityMetadata 是否匹配 MetadataSet 中的数据
func (v *MetadataSet) match(item v5.VulnerabilityMetadata) bool {
    if baseModel, exists := v.data[getMetadataKey(item)]; exists {
        baseModel.seen = true
        return baseModel.item.Equal(item)
    }
    return false
}

// 获取 MetadataSet 中未匹配的数据的键列表
func (v *MetadataSet) getUnmatched() []storeKey {
    notSeen := []storeKey{}
    for k, item := range v.data {
        if !item.seen {
            notSeen = append(notSeen, k)
        }
    }
    return notSeen
}

// 比较两个 VulnerabilityMetadata 模型列表的差异，并返回差异的 map
func diffVulnerabilityMetadata(baseModels, targetModels *[]v5.VulnerabilityMetadata, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v5.Diff {
    // 创建一个存储 v5.Diff 对象的 map
    diffs := make(map[string]*v5.Diff)
    // 创建一个 MetadataSet 对象，根据给定的 baseModels
    m := NewMetadataSet(baseModels)

    // 遍历目标模型列表，比较与基础模型列表的差异
    for _, tModel := range *targetModels {
        targetModel := tModel
        k := getMetadataKey(targetModel)
        // 如果目标模型存在于基础模型列表中
        if m.in(targetModel) {
            // 如果目标模型与基础模型不匹配
            if !m.match(targetModel) {
                // 如果差异 map 中已存在相同键的差异对象，则跳过
                if _, exists := diffs[k.id+k.namespace]; exists {
                    continue
                }
                // 创建一个差异对象，并添加到差异 map 中
                diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v5.DiffChanged)
                // 增加差异项的计数
                differentItems.Increment()
            }
        } else {
            // 如果差异 map 中已存在相同键的差异对象，则跳过
            if _, exists := diffs[k.id+k.namespace]; exists {
                continue
            }
            // 创建一个差异对象，并添加到差异 map 中
            diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v5.DiffAdded)
            // 增加差异项的计数
            differentItems.Increment()
        }
    }
}
    # 遍历 m.getUnmatched() 返回的未匹配项列表
    for _, k := range m.getUnmatched() {
        # 检查 diffs 中是否存在与当前未匹配项对应的键
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 如果存在，则跳过当前未匹配项，继续下一次循环
            continue
        }
        # 在 diffs 中创建以未匹配项 id 和 namespace 组合作为键的新条目，值为使用 createDiff 函数创建的差异对象
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v5.DiffRemoved)
        # 不同项计数器增加
        differentItems.Increment()
    }

    # 返回 diffs 的指针
    return &diffs
# 定义一个函数，根据给定的漏洞元数据返回存储键
func getMetadataKey(metadata v5.VulnerabilityMetadata) storeKey {
    # 返回一个存储键对象，包含漏洞ID、命名空间和空字符串
    return storeKey{metadata.ID, metadata.Namespace, ""}
}
```