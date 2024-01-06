# `grype\grype\db\v3\store\diff.go`

```
// 导入所需的包
package store

import (
	"github.com/wagoodman/go-partybus" // 导入 go-partybus 包
	"github.com/wagoodman/go-progress" // 导入 go-progress 包

	v3 "github.com/anchore/grype/grype/db/v3" // 导入 v3 版本的 grype 数据库包
	"github.com/anchore/grype/grype/event" // 导入 grype 事件包
	"github.com/anchore/grype/grype/event/monitor" // 导入 grype 监控事件包
	"github.com/anchore/grype/internal/bus" // 导入内部总线包
)

// 定义存储键的结构
type storeKey struct {
	id          string // 存储键的 ID
	namespace   string // 存储键的命名空间
	packageName string // 存储键的包名
}

// 定义存储键到字符串切片的映射类型
type PkgMap = map[storeKey][]string
// 定义存储漏洞列表的结构体，包含一个映射类型的 items 字段和一个布尔类型的 seen 字段
type storeVulnerabilityList struct {
	items map[storeKey][]storeVulnerability
	seen  bool
}

// 定义存储漏洞的结构体，包含一个指向 v3.Vulnerability 类型的 item 字段和一个布尔类型的 seen 字段
type storeVulnerability struct {
	item *v3.Vulnerability
	seen bool
}

// 定义存储元数据的结构体，包含一个指向 v3.VulnerabilityMetadata 类型的 item 字段和一个布尔类型的 seen 字段
type storeMetadata struct {
	item *v3.VulnerabilityMetadata
	seen bool
}

// 创建用于跟踪数据库差异进度的手动进度条
func trackDiff(total int64) (*progress.Manual, *progress.Manual, *progress.Stage) {
	// 创建一个手动进度条，设置总数为 total
	stageProgress := &progress.Manual{}
	stageProgress.SetTotal(total)
	// 创建一个手动进度条，用于记录发现的差异
	differencesDiscovered := &progress.Manual{}
	// 创建一个阶段进度条
	stager := &progress.Stage{}
	// 发布一个数据库差异事件，包含差异类型和监控数据
	bus.Publish(partybus.Event{
		Type: event.DatabaseDiffingStarted,
		Value: monitor.DBDiff{
			Stager:                stager,  // 设置差异监控的阶段
			StageProgress:         progress.Progressable(stageProgress),  // 设置差异监控的阶段进度
			DifferencesDiscovered: progress.Monitorable(differencesDiscovered),  // 设置差异监控的发现差异
		},
	})
	// 返回阶段进度、发现的差异和差异监控器
	return stageProgress, differencesDiscovered, stager
}

// 创建一个从未打包的键到与之关联的所有软件包列表的映射
func buildVulnerabilityPkgsMap(models *[]v3.Vulnerability) *map[storeKey][]string {
	// 创建一个空的映射
	storeMap := make(map[storeKey][]string)
	// 遍历所有漏洞模型
	for _, m := range *models {
		model := m
		// 获取漏洞的父键
		k := getVulnerabilityParentKey(model)
		// 如果映射中已存在该键，则将软件包名称添加到列表中
		if storeVuln, exists := storeMap[k]; exists {
			storeMap[k] = append(storeVuln, model.PackageName)
		} else {
// 创建一个空的字符串数组，并将model.PackageName添加到其中，然后将其存储在storeMap中的键k下
storeMap[k] = []string{model.PackageName}

// 返回storeMap的指针
return &storeMap
}

// 使用包映射信息创建一个与给定键相关的差异，以填充受更新影响的相关包
func createDiff(baseStore, targetStore *PkgMap, key storeKey, reason v3.DiffReason) *v3.Diff {
	// 创建一个包映射
	pkgMap := make(map[string]struct{})

	// 将key.packageName设置为空字符串
	key.packageName = ""
	
	// 如果baseStore不为空，则遍历baseStore中与key相关的包，并将其添加到pkgMap中
	if baseStore != nil {
		if basePkgs, exists := (*baseStore)[key]; exists {
			for _, pkg := range basePkgs {
				pkgMap[pkg] = struct{}{}
			}
		}
	}
	
	// 如果targetStore不为空，则...
# 如果目标包存在于目标存储中，则将目标包添加到包映射中
if targetPkgs, exists := (*targetStore)[key]; exists {
    for _, pkg := range targetPkgs {
        pkgMap[pkg] = struct{}{}
    }
}

# 创建一个空的包列表
pkgs := []string{}

# 遍历包映射，将包添加到包列表中
for pkg := range pkgMap {
    pkgs = append(pkgs, pkg)
}

# 返回一个包含差异信息的结构体
return &v3.Diff{
    Reason:    reason,
    ID:        key.id,
    Namespace: key.namespace,
    Packages:  pkgs,
}
```

```
// 从漏洞中获取一个未打包的键
// 根据漏洞获取其父级键值
func getVulnerabilityParentKey(vuln v3.Vulnerability) storeKey {
	return storeKey{vuln.ID, vuln.Namespace, ""}
}

// 从漏洞获取打包的键
func getVulnerabilityKey(vuln v3.Vulnerability) storeKey {
	return storeKey{vuln.ID, vuln.Namespace, vuln.PackageName}
}

// 定义漏洞集合结构
type VulnerabilitySet struct {
	data map[storeKey]*storeVulnerabilityList
}

// 创建一个新的漏洞集合
func NewVulnerabilitySet(models *[]v3.Vulnerability) *VulnerabilitySet {
	// 创建一个包含漏洞键值对的映射
	m := make(map[storeKey]*storeVulnerabilityList, len(*models))
	// 遍历漏洞模型列表
	for _, mm := range *models {
		model := mm
		// 获取漏洞的父级键值
		parentKey := getVulnerabilityParentKey(model)
		// 获取漏洞的键值
		vulnKey := getVulnerabilityKey(model)
		// 检查父级键值是否存在于映射中
		if storeVuln, exists := m[parentKey]; exists {
# 如果存储漏洞的键存在于存储漏洞项中，则将模型追加到现有的漏洞项列表中
if kk, exists := storeVuln.items[vulnKey]; exists {
    storeVuln.items[vulnKey] = append(kk, storeVulnerability{
        item: &model,
        seen: false,
    })
# 如果存储漏洞的键不存在于存储漏洞项中，则创建一个新的漏洞项列表并将模型添加到其中
} else {
    storeVuln.items[vulnKey] = []storeVulnerability{{&model, false}}
}

# 如果存储漏洞列表中不存在指定的父键，则创建一个新的存储漏洞列表，并将模型添加到其中
else {
    vuln := storeVulnerabilityList{
        items: make(map[storeKey][]storeVulnerability),
        seen: false,
    }
    vuln.items[vulnKey] = []storeVulnerability{{&model, false}}
    m[parentKey] = &vuln
}

# 返回包含数据的漏洞集合
return &VulnerabilitySet{
    data: m,
}
// 检查给定的漏洞是否在漏洞集合中
func (v *VulnerabilitySet) in(item v3.Vulnerability) bool {
	// 检查漏洞是否存在于漏洞集合中
	_, exists := v.data[getVulnerabilityParentKey(item)]
	return exists
}

// 检查给定的漏洞是否与漏洞集合中的某个漏洞匹配
func (v *VulnerabilitySet) match(item v3.Vulnerability) bool {
	// 检查漏洞的父级是否存在于漏洞集合中
	if parent, exists := v.data[getVulnerabilityParentKey(item)]; exists {
		// 将父级漏洞标记为已查看
		parent.seen = true
		// 获取漏洞的键
		key := getVulnerabilityKey(item)
		// 检查是否存在与给定漏洞匹配的子漏洞
		if children, exists := parent.items[key]; exists {
			// 遍历子漏洞列表
			for idx, child := range children {
				// 如果找到匹配的子漏洞，则将其标记为已查看并返回 true
				if item.Equal(*child.item) {
					children[idx].seen = true
					return true
				}
			}
		}
	}
}
// 返回 false
	return false
}

// 获取未匹配的漏洞
func (v *VulnerabilitySet) getUnmatched() ([]storeKey, []storeKey) {
	// 未完全匹配的漏洞列表
	notSeen := []storeKey{}
	// 完全未匹配的漏洞列表
	notEntirelySeen := []storeKey{}
	// 遍历漏洞数据
	for k, item := range v.data {
		// 如果漏洞未被匹配，则添加到未完全匹配的列表中
		if !item.seen {
			notSeen = append(notSeen, k)
			continue
		}
		// 遍历漏洞的组件
	componentLoop:
		for _, components := range item.items {
			for _, component := range components {
				// 如果组件未被匹配，则添加到未完全匹配的列表中
				if !component.seen {
					notEntirelySeen = append(notEntirelySeen, k)
					break componentLoop
				}
			}
		}
	}
	}
	return notSeen, notEntirelySeen
}

func diffVulnerabilities(baseModels, targetModels *[]v3.Vulnerability, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v3.Diff {
	// 创建一个空的漏洞差异映射
	diffs := make(map[string]*v3.Diff)
	// 创建一个基础漏洞模型的集合
	m := NewVulnerabilitySet(baseModels)

	// 遍历目标漏洞模型
	for _, tModel := range *targetModels {
		// 复制目标漏洞模型
		targetModel := tModel
		// 获取漏洞模型的键
		k := getVulnerabilityKey(targetModel)
		// 如果基础漏洞模型集合中存在目标漏洞模型
		if m.in(targetModel) {
			// 在基础漏洞模型集合中查找匹配的漏洞模型
			matched := m.match(targetModel)
			// 如果没有找到匹配的漏洞模型
			if !matched {
				// 如果差异映射中已经存在相同键的差异
				if _, exists := diffs[k.id+k.namespace]; exists {
					// 继续下一次循环
					continue
				}
				// 创建基础和目标包映射的差异，并添加到差异映射中
				diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v3.DiffChanged)
				// 增加不同项的计数
				differentItems.Increment()
			}
		} else {
			// 如果 diffs 中已经存在相同的键值对，则跳过当前循环
			if _, exists := diffs[k.id+k.namespace]; exists {
				continue
			}
			// 创建一个新的差异对象，并将其添加到 diffs 中
			diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v3.DiffAdded)
			// 增加不同项的计数
			differentItems.Increment()
		}
	}
	// 获取未匹配和部分匹配的项
	notSeen, partialSeen := m.getUnmatched()
	// 遍历部分匹配的项
	for _, k := range partialSeen {
		// 如果 diffs 中已经存在相同的键值对，则跳过当前循环
		if _, exists := diffs[k.id+k.namespace]; exists {
			continue
		}
		// 创建一个新的差异对象，并将其添加到 diffs 中
		diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v3.DiffChanged)
		// 增加不同项的计数
		differentItems.Increment()
	}
	// 遍历未匹配的项
	for _, k := range notSeen {
		// 如果 diffs 中已经存在相同的键值对，则跳过当前循环
		if _, exists := diffs[k.id+k.namespace]; exists {
			continue
		}
# 创建一个名为diffs的字典，键为k.id+k.namespace，值为使用createDiff函数创建的差异对象
diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v3.DiffRemoved)
# 增加不同项的计数
differentItems.Increment()
# 返回diffs字典的指针
return &diffs
}

# 定义一个名为MetadataSet的结构体
type MetadataSet struct {
    data map[storeKey]*storeMetadata
}

# 创建一个新的MetadataSet对象
func NewMetadataSet(models *[]v3.VulnerabilityMetadata) *MetadataSet {
    # 创建一个名为m的map，键为storeKey类型，值为storeMetadata类型的指针
    m := make(map[storeKey]*storeMetadata, len(*models))
    # 遍历models切片中的每个元素
    for _, mm := range *models {
        # 将mm的值复制给model
        model := mm
        # 将getMetadataKey(model)作为键，创建一个storeMetadata对象作为值，存入m中
        m[getMetadataKey(model)] = &storeMetadata{
            item: &model,
            seen: false,
        }
    }
}
// 返回一个包含给定数据的 MetadataSet 结构体指针
return &MetadataSet{
    data: m,
}

// 检查给定的 VulnerabilityMetadata 是否存在于 MetadataSet 中
func (v *MetadataSet) in(item v3.VulnerabilityMetadata) bool {
    // 使用 getMetadataKey 函数获取元数据的键，检查该键是否存在于 MetadataSet 中
    _, exists := v.data[getMetadataKey(item)]
    return exists
}

// 检查给定的 VulnerabilityMetadata 是否与 MetadataSet 中的元数据匹配
func (v *MetadataSet) match(item v3.VulnerabilityMetadata) bool {
    // 如果基础模型存在于 MetadataSet 中，则将其标记为已见，并检查其是否与给定的元数据相等
    if baseModel, exists := v.data[getMetadataKey(item)]; exists {
        baseModel.seen = true
        return baseModel.item.Equal(item)
    }
    return false
}

// 获取未匹配的元数据的存储键列表
func (v *MetadataSet) getUnmatched() []storeKey {
    notSeen := []storeKey{}
	// 遍历数据集合中的每个键值对
	for k, item := range v.data {
		// 如果项目未被标记为已见，则将其键添加到未见列表中
		if !item.seen {
			notSeen = append(notSeen, k)
		}
	}
	// 返回未见列表
	return notSeen
}

// 比较漏洞元数据的差异并返回差异结果
func diffVulnerabilityMetadata(baseModels, targetModels *[]v3.VulnerabilityMetadata, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v3.Diff {
	// 创建一个空的差异结果映射
	diffs := make(map[string]*v3.Diff)
	// 创建一个基础模型的元数据集合
	m := NewMetadataSet(baseModels)

	// 遍历目标模型的每个元数据
	for _, tModel := range *targetModels {
		// 复制目标模型以便在闭包中使用
		targetModel := tModel
		// 获取目标模型的元数据键
		k := getMetadataKey(targetModel)
		// 如果基础模型集合中存在目标模型
		if m.in(targetModel) {
			// 如果目标模型与基础模型不匹配
			if !m.match(targetModel) {
				// 如果差异结果映射中已存在相同键的差异结果，则跳过当前循环
				if _, exists := diffs[k.id+k.namespace]; exists {
					continue
				}
# 遍历一个数据结构，根据条件创建不同类型的差异对象，并将其存储在diffs字典中
for _, k := range m.getUnmatched() {
    # 如果差异对象已存在，则跳过
    if _, exists := diffs[k.id+k.namespace]; exists {
        continue
    }
    # 创建一个差异对象，表示被移除的内容，并存储在diffs字典中
    diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v3.DiffRemoved)
    # 增加不同项的计数
    differentItems.Increment()
}

# 返回存储差异对象的字典
return &diffs
# 定义一个函数，根据给定的漏洞元数据返回一个存储键
func getMetadataKey(metadata v3.VulnerabilityMetadata) storeKey {
    # 返回一个存储键对象，包含漏洞ID、命名空间和空字符串
    return storeKey{metadata.ID, metadata.Namespace, ""}
}
```