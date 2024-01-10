# `grype\grype\db\v3\store\diff.go`

```
package store

import (
    "github.com/wagoodman/go-partybus"  // 导入第三方库 go-partybus
    "github.com/wagoodman/go-progress"  // 导入第三方库 go-progress

    v3 "github.com/anchore/grype/grype/db/v3"  // 导入 anchore/grype/grype/db/v3 包，并重命名为 v3
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
    item *v3.Vulnerability  // 定义 storeVulnerability 结构体，包含 item 字段，类型为指向 v3.Vulnerability 的指针
    seen bool  // 定义 storeVulnerability 结构体，包含 seen 字段，类型为布尔值
}
type storeMetadata struct {
    item *v3.VulnerabilityMetadata  // 定义 storeMetadata 结构体，包含 item 字段，类型为指向 v3.VulnerabilityMetadata 的指针
    seen bool  // 定义 storeMetadata 结构体，包含 seen 字段，类型为布尔值
}

// 创建用于跟踪数据库差异进度的手动进度条
func trackDiff(total int64) (*progress.Manual, *progress.Manual, *progress.Stage) {
    stageProgress := &progress.Manual{}  // 创建 progress.Manual 类型的指针 stageProgress
    stageProgress.SetTotal(total)  // 设置 stageProgress 的总数为 total
    differencesDiscovered := &progress.Manual{}  // 创建 progress.Manual 类型的指针 differencesDiscovered
    stager := &progress.Stage{}  // 创建 progress.Stage 类型的指针 stager

    bus.Publish(partybus.Event{  // 发布事件
        Type: event.DatabaseDiffingStarted,  // 设置事件类型为 DatabaseDiffingStarted
        Value: monitor.DBDiff{  // 设置事件值为 monitor.DBDiff 结构体
            Stager:                stager,  // 设置 Stager 字段为 stager
            StageProgress:         progress.Progressable(stageProgress),  // 设置 StageProgress 字段为 stageProgress
            DifferencesDiscovered: progress.Monitorable(differencesDiscovered),  // 设置 DifferencesDiscovered 字段为 differencesDiscovered
        },
    })
    return stageProgress, differencesDiscovered, stager  // 返回 stageProgress、differencesDiscovered 和 stager
}

// 根据漏洞模型构建漏洞包映射
func buildVulnerabilityPkgsMap(models *[]v3.Vulnerability) *map[storeKey][]string {
    storeMap := make(map[storeKey][]string)  // 创建 storeKey 到字符串切片的映射 storeMap
    for _, m := range *models {  // 遍历 models
        model := m  // 将 m 赋值给 model
        k := getVulnerabilityParentKey(model)  // 调用 getVulnerabilityParentKey 函数，获取漏洞父键
        if storeVuln, exists := storeMap[k]; exists {  // 判断 storeMap 中是否存在 k
            storeMap[k] = append(storeVuln, model.PackageName)  // 如果存在，则将 model.PackageName 追加到 storeVuln 中
        } else {
            storeMap[k] = []string{model.PackageName}  // 如果不存在，则将 model.PackageName 放入新的字符串切片中
        }
    }
    return &storeMap  // 返回 storeMap 的指针
}

// 使用包映射信息创建给定键的差异
// 创建一个 Diff 对象，用于表示两个包映射之间的差异
func createDiff(baseStore, targetStore *PkgMap, key storeKey, reason v3.DiffReason) *v3.Diff {
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

    // 返回一个包含差异信息的 Diff 对象
    return &v3.Diff{
        Reason:    reason,
        ID:        key.id,
        Namespace: key.namespace,
        Packages:  pkgs,
    }
}

// 从漏洞中获取未打包的键
func getVulnerabilityParentKey(vuln v3.Vulnerability) storeKey {
    return storeKey{vuln.ID, vuln.Namespace, ""}
}

// 从漏洞中获取已打包的键
func getVulnerabilityKey(vuln v3.Vulnerability) storeKey {
    return storeKey{vuln.ID, vuln.Namespace, vuln.PackageName}
}

// 定义 VulnerabilitySet 结构
type VulnerabilitySet struct {
    data map[storeKey]*storeVulnerabilityList
}

// 创建一个新的 VulnerabilitySet 对象
func NewVulnerabilitySet(models *[]v3.Vulnerability) *VulnerabilitySet {
    // 创建一个包含 storeKey 和 storeVulnerabilityList 的映射
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
                # 如果项目中不存在漏洞键，则创建一个新的项目列表
                storeVuln.items[vulnKey] = []storeVulnerability{{&model, false}}
            }
        } else {
            # 如果父键不存在于映射中，则创建一个新的漏洞列表
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
func (v *VulnerabilitySet) in(item v3.Vulnerability) bool:
    # 检查漏洞是否存在于漏洞集合中，返回布尔值
    _, exists := v.data[getVulnerabilityParentKey(item)]
    return exists

# 检查给定的漏洞是否匹配漏洞集合中的项
func (v *VulnerabilitySet) match(item v3.Vulnerability) bool:
    # 检查漏洞是否匹配漏洞集合中的项
    if parent, exists := v.data[getVulnerabilityParentKey(item)]; exists:
        # 如果父项存在，则将其标记为已见
        parent.seen = true
        # 获取漏洞的键
        key := getVulnerabilityKey(item)
        # 检查子项是否存在，如果存在则遍历并比较
        if children, exists := parent.items[key]; exists:
            for idx, child := range children:
                # 如果漏洞匹配，则将其标记为已见并返回 true
                if item.Equal(*child.item):
                    children[idx].seen = true
                    return true
    # 如果漏洞不匹配，则返回 false
    return false

# 获取未匹配的漏洞
func (v *VulnerabilitySet) getUnmatched() ([]storeKey, []storeKey):
    # 初始化未见漏洞和部分未见漏洞的列表
    notSeen := []storeKey{}
    notEntirelySeen := []storeKey{}
    # 遍历漏洞集合中的项
    for k, item := range v.data:
        # 如果父项未见，则将其添加到未见漏洞列表中
        if !item.seen:
            notSeen = append(notSeen, k)
            continue
        # 否则，遍历子项并检查是否部分未见
        componentLoop:
        for _, components := range item.items:
            for _, component := range components:
                if !component.seen:
                    notEntirelySeen = append(notEntirelySeen, k)
                    break componentLoop
    # 返回未见漏洞和部分未见漏洞的列表
    return notSeen, notEntirelySeen

# 比较漏洞
func diffVulnerabilities(baseModels, targetModels *[]v3.Vulnerability, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v3.Diff:
    # 创建漏洞差异的映射
    diffs := make(map[string]*v3.Diff)
    # 使用基础模型创建漏洞集合
    m := NewVulnerabilitySet(baseModels)
    # 遍历目标模型列表
    for _, tModel := range *targetModels {
        # 复制目标模型
        targetModel := tModel
        # 获取漏洞键
        k := getVulnerabilityKey(targetModel)
        # 如果目标模型在匹配器中
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
                # 创建变更差异并添加到差异列表中
                diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v3.DiffChanged)
                # 增加不同项计数
                differentItems.Increment()
            }
        } else {
            # 如果差异中存在相同的键
            if _, exists := diffs[k.id+k.namespace]; exists {
                # 继续下一次循环
                continue
            }
            # 创建新增差异并添加到差异列表中
            diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v3.DiffAdded)
            # 增加不同项计数
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
        # 创建变更差异并添加到差异列表中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v3.DiffChanged)
        # 增加不同项计数
        differentItems.Increment()
    }
    # 遍历未匹配的键列表
    for _, k := range notSeen {
        # 如果差异中存在相同的键
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 继续下一次循环
            continue
        }
        # 创建移除差异并添加到差异列表中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v3.DiffRemoved)
        # 增加不同项计数
        differentItems.Increment()
    }

    # 返回差异列表的指针
    return &diffs
// 定义 MetadataSet 结构体，包含一个存储 storeMetadata 的 map
type MetadataSet struct {
    data map[storeKey]*storeMetadata
}

// 创建并返回一个 MetadataSet 对象，根据给定的 VulnerabilityMetadata 模型列表
func NewMetadataSet(models *[]v3.VulnerabilityMetadata) *MetadataSet {
    // 创建一个存储 storeMetadata 的 map，长度为模型列表的长度
    m := make(map[storeKey]*storeMetadata, len(*models))
    // 遍历模型列表，为每个模型创建 storeMetadata 对象，并存储到 map 中
    for _, mm := range *models {
        model := mm
        m[getMetadataKey(model)] = &storeMetadata{
            item: &model,
            seen: false,
        }
    }
    // 返回包含 map 的 MetadataSet 对象
    return &MetadataSet{
        data: m,
    }
}

// 判断给定的 VulnerabilityMetadata 是否存在于 MetadataSet 中
func (v *MetadataSet) in(item v3.VulnerabilityMetadata) bool {
    _, exists := v.data[getMetadataKey(item)]
    return exists
}

// 判断给定的 VulnerabilityMetadata 是否匹配 MetadataSet 中的数据，并标记为已匹配
func (v *MetadataSet) match(item v3.VulnerabilityMetadata) bool {
    if baseModel, exists := v.data[getMetadataKey(item)]; exists {
        baseModel.seen = true
        return baseModel.item.Equal(item)
    }
    return false
}

// 获取未匹配的数据的键列表
func (v *MetadataSet) getUnmatched() []storeKey {
    notSeen := []storeKey{}
    // 遍历 MetadataSet 中的数据，将未匹配的键存储到列表中
    for k, item := range v.data {
        if !item.seen {
            notSeen = append(notSeen, k)
        }
    }
    return notSeen
}

// 比较两个 VulnerabilityMetadata 模型列表的差异，并返回差异结果的 map
func diffVulnerabilityMetadata(baseModels, targetModels *[]v3.VulnerabilityMetadata, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v3.Diff {
    // 创建一个存储 Diff 对象的 map
    diffs := make(map[string]*v3.Diff)
    // 创建一个 MetadataSet 对象，用于存储基准模型列表的数据
    m := NewMetadataSet(baseModels)

    // 遍历目标模型列表，比较与基准模型列表的差异
    for _, tModel := range *targetModels {
        targetModel := tModel
        k := getMetadataKey(targetModel)
        // 如果目标模型存在于基准模型列表中
        if m.in(targetModel) {
            // 如果目标模型与基准模型不匹配
            if !m.match(targetModel) {
                // 如果差异结果中已存在相同键的差异对象，则跳过
                if _, exists := diffs[k.id+k.namespace]; exists {
                    continue
                }
                // 创建差异对象，并存储到差异结果中
                diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v3.DiffChanged)
                // 增加差异项计数
                differentItems.Increment()
            }
        } else {
            // 如果差异结果中已存在相同键的差异对象，则跳过
            if _, exists := diffs[k.id+k.namespace]; exists {
                continue
            }
            // 创建差异对象，并存储到差异结果中
            diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v3.DiffAdded)
            // 增加差异项计数
            differentItems.Increment()
        }
    }
}
    # 遍历 m.getUnmatched() 返回的未匹配项列表
    for _, k := range m.getUnmatched() {
        # 检查 diffs 中是否已存在与当前未匹配项对应的差异项
        if _, exists := diffs[k.id+k.namespace]; exists {
            # 如果存在，则跳过当前未匹配项，继续下一项
            continue
        }
        # 创建与当前未匹配项对应的差异项，并添加到 diffs 中
        diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v3.DiffRemoved)
        # 不同项计数器加一
        differentItems.Increment()
    }

    # 返回 diffs 的指针
    return &diffs
# 定义一个函数，用于获取漏洞元数据的键
func getMetadataKey(metadata v3.VulnerabilityMetadata) storeKey {
    # 返回一个包含漏洞ID、命名空间和空字符串的存储键对象
    return storeKey{metadata.ID, metadata.Namespace, ""}
}
```