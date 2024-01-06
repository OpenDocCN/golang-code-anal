# `grype\grype\db\v4\store\diff.go`

```
// 导入所需的包
package store

import (
	"github.com/wagoodman/go-partybus" // 导入go-partybus包
	"github.com/wagoodman/go-progress" // 导入go-progress包

	v4 "github.com/anchore/grype/grype/db/v4" // 导入v4版本的数据库包
	"github.com/anchore/grype/grype/event" // 导入事件包
	"github.com/anchore/grype/grype/event/monitor" // 导入监控事件包
	"github.com/anchore/grype/internal/bus" // 导入内部总线包
)

// 定义存储键的结构体
type storeKey struct {
	id          string // 存储键的ID
	namespace   string // 存储键的命名空间
	packageName string // 存储键的包名
}

// 定义存储键到字符串切片的映射类型
type PkgMap = map[storeKey][]string
// 定义存储漏洞列表的结构体，包含漏洞项的映射和是否已查看的标志
type storeVulnerabilityList struct {
	items map[storeKey][]storeVulnerability
	seen  bool
}

// 定义存储漏洞的结构体，包含漏洞项和是否已查看的标志
type storeVulnerability struct {
	item *v4.Vulnerability
	seen bool
}

// 定义存储元数据的结构体，包含漏洞元数据项和是否已查看的标志
type storeMetadata struct {
	item *v4.VulnerabilityMetadata
	seen bool
}

// 创建用于跟踪数据库差异进度的手动进度条
func trackDiff(total int64) (*progress.Manual, *progress.Manual, *progress.Stage) {
	// 创建阶段进度条，并设置总数
	stageProgress := &progress.Manual{}
	stageProgress.SetTotal(total)
	// 创建发现差异的进度条
	differencesDiscovered := &progress.Manual{}
	// 创建阶段进度对象
	stager := &progress.Stage{}
}
// 发布一个事件，表示数据库差异比对开始
bus.Publish(partybus.Event{
    Type: event.DatabaseDiffingStarted,
    Value: monitor.DBDiff{
        Stager:                stager,
        StageProgress:         progress.Progressable(stageProgress),
        DifferencesDiscovered: progress.Monitorable(differencesDiscovered),
    },
})
// 返回阶段进度、发现的差异和阶段器
return stageProgress, differencesDiscovered, stager
}

// 创建一个从未打包的键到与之关联的所有软件包列表的映射
func buildVulnerabilityPkgsMap(models *[]v4.Vulnerability) *map[storeKey][]string {
    // 创建一个空的映射
    storeMap := make(map[storeKey][]string)
    // 遍历每个漏洞模型
    for _, m := range *models {
        // 复制漏洞模型
        model := m
        // 获取漏洞父键
        k := getVulnerabilityParentKey(model)
        // 如果映射中已经存在该键，则将软件包名追加到对应值的列表中
        if storeVuln, exists := storeMap[k]; exists {
            storeMap[k] = append(storeVuln, model.PackageName)
        } else {
		// 如果存储映射中不存在给定的键，则创建一个新的切片并将包名添加到其中
		storeMap[k] = []string{model.PackageName}
		// 返回存储映射的指针
	}
}
return &storeMap
}

// 使用包映射信息创建一个差异，以填充受更新影响的相关包
func createDiff(baseStore, targetStore *PkgMap, key storeKey, reason v4.DiffReason) *v4.Diff {
	// 创建一个包映射
	pkgMap := make(map[string]struct{})

	// 将键的包名设置为空
	key.packageName = ""
	// 如果基本存储不为空
	if baseStore != nil {
		// 如果基本包存在于基本存储中，则将其添加到包映射中
		if basePkgs, exists := (*baseStore)[key]; exists {
			for _, pkg := range basePkgs {
				pkgMap[pkg] = struct{}{}
			}
		}
	}
	// 如果目标存储不为空
	if targetStore != nil {
		// 检查目标存储中是否存在指定的键，如果存在则获取对应的包列表
		if targetPkgs, exists := (*targetStore)[key]; exists {
			// 遍历目标包列表，将包名添加到包名映射中
			for _, pkg := range targetPkgs {
				pkgMap[pkg] = struct{}{}
			}
		}
	}
	// 将包名映射转换为包名列表
	pkgs := []string{}
	for pkg := range pkgMap {
		pkgs = append(pkgs, pkg)
	}

	// 返回漏洞的差异信息
	return &v4.Diff{
		Reason:    reason,      // 差异原因
		ID:        key.id,      // 键的ID
		Namespace: key.namespace,  // 键的命名空间
		Packages:  pkgs,         // 包名列表
	}
}

// 从漏洞中获取一个未打包的键
// 根据漏洞获取其父级键值
func getVulnerabilityParentKey(vuln v4.Vulnerability) storeKey {
	return storeKey{vuln.ID, vuln.Namespace, ""}
}

// 从漏洞获取打包的键值
func getVulnerabilityKey(vuln v4.Vulnerability) storeKey {
	return storeKey{vuln.ID, vuln.Namespace, vuln.PackageName}
}

// 定义漏洞集合结构
type VulnerabilitySet struct {
	data map[storeKey]*storeVulnerabilityList
}

// 创建新的漏洞集合
func NewVulnerabilitySet(models *[]v4.Vulnerability) *VulnerabilitySet {
	// 创建一个包含漏洞键值和存储漏洞列表的映射
	m := make(map[storeKey]*storeVulnerabilityList, len(*models))
	// 遍历漏洞模型列表
	for _, mm := range *models {
		// 复制漏洞模型
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

# 如果存储漏洞列表中不存在父键，则创建一个新的存储漏洞列表，并将模型添加到其中
if _, exists := m[parentKey]; !exists {
    vuln := storeVulnerabilityList{
        items: make(map[storeKey][]storeVulnerability),
        seen:  false,
    }
    vuln.items[vulnKey] = []storeVulnerability{{&model, false}}
    m[parentKey] = &vuln
}

# 返回包含数据的漏洞集合
return &VulnerabilitySet{
    data: m,
}
// 检查给定的漏洞是否在漏洞集合中，返回布尔值
func (v *VulnerabilitySet) in(item v4.Vulnerability) bool {
	// 检查漏洞是否存在于漏洞集合中
	_, exists := v.data[getVulnerabilityParentKey(item)]
	return exists
}

// 检查给定的漏洞是否与漏洞集合中的某个漏洞匹配，返回布尔值
func (v *VulnerabilitySet) match(item v4.Vulnerability) bool {
	// 检查漏洞的父级是否存在于漏洞集合中
	if parent, exists := v.data[getVulnerabilityParentKey(item)]; exists {
		// 将父级漏洞标记为已查看
		parent.seen = true
		// 获取漏洞的键
		key := getVulnerabilityKey(item)
		// 检查漏洞的子级是否存在于父级漏洞中
		if children, exists := parent.items[key]; exists {
			// 遍历子级漏洞
			for idx, child := range children {
				// 如果找到匹配的子级漏洞，则将其标记为已查看并返回 true
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

func diffVulnerabilities(baseModels, targetModels *[]v4.Vulnerability, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v4.Diff {
	// 创建一个空的差异映射
	diffs := make(map[string]*v4.Diff)
	// 创建一个基础模型的漏洞集合
	m := NewVulnerabilitySet(baseModels)

	// 遍历目标模型的漏洞
	for _, tModel := range *targetModels {
		targetModel := tModel
		// 获取漏洞的键
		k := getVulnerabilityKey(targetModel)
		// 如果基础模型中存在目标模型的漏洞
		if m.in(targetModel) {
			// 在基础模型中找到匹配的漏洞
			matched := m.match(targetModel)
			// 如果没有找到匹配的漏洞
			if !matched {
				// 如果差异映射中已经存在相同的键
				if _, exists := diffs[k.id+k.namespace]; exists {
					// 继续下一次循环
					continue
				}
				// 创建一个差异对象，并添加到差异映射中
				diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v4.DiffChanged)
				// 增加不同项的计数
				differentItems.Increment()
			}
		} else {
			// 如果 diffs 中已经存在了相同的键值对，则跳过当前循环
			if _, exists := diffs[k.id+k.namespace]; exists {
				continue
			}
			// 创建一个新的差异对象，并将其添加到 diffs 中
			diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v4.DiffAdded)
			// 增加不同项的计数
			differentItems.Increment()
		}
	}
	// 获取未匹配和部分匹配的项
	notSeen, partialSeen := m.getUnmatched()
	// 遍历部分匹配的项
	for _, k := range partialSeen {
		// 如果 diffs 中已经存在了相同的键值对，则跳过当前循环
		if _, exists := diffs[k.id+k.namespace]; exists {
			continue
		}
		// 创建一个新的差异对象，并将其添加到 diffs 中
		diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v4.DiffChanged)
		// 增加不同项的计数
		differentItems.Increment()
	}
	// 遍历未匹配的项
	for _, k := range notSeen {
		// 如果 diffs 中已经存在了相同的键值对，则跳过当前循环
		if _, exists := diffs[k.id+k.namespace]; exists {
			continue
		}
# 创建一个名为diffs的字典，键为k.id+k.namespace，值为使用createDiff函数创建的差异对象
diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v4.DiffRemoved)
# 增加不同项的计数
differentItems.Increment()
# 返回diffs字典的指针
return &diffs
}

# 定义一个名为MetadataSet的结构体
type MetadataSet struct {
    data map[storeKey]*storeMetadata
}

# 创建一个新的MetadataSet对象，参数为models的指针
func NewMetadataSet(models *[]v4.VulnerabilityMetadata) *MetadataSet {
    # 创建一个名为m的map，键为storeKey类型，值为storeMetadata类型的指针
    m := make(map[storeKey]*storeMetadata, len(*models))
    # 遍历models切片中的每个元素
    for _, mm := range *models {
        # 将mm赋值给model
        model := mm
        # 将getMetadataKey(model)作为键，创建一个storeMetadata对象作为值，并添加到m中
        m[getMetadataKey(model)] = &storeMetadata{
            item: &model,
            seen: false,
        }
    }
}
# 返回一个包含数据的MetadataSet结构体指针
return &MetadataSet{
    data: m,
}

# 检查给定的VulnerabilityMetadata是否存在于MetadataSet中
func (v *MetadataSet) in(item v4.VulnerabilityMetadata) bool {
    _, exists := v.data[getMetadataKey(item)]
    return exists
}

# 检查给定的VulnerabilityMetadata是否与MetadataSet中的元数据匹配
func (v *MetadataSet) match(item v4.VulnerabilityMetadata) bool {
    if baseModel, exists := v.data[getMetadataKey(item)]; exists {
        baseModel.seen = true
        return baseModel.item.Equal(item)
    }
    return false
}

# 获取未匹配的数据
func (v *MetadataSet) getUnmatched() []storeKey {
    notSeen := []storeKey{}
	// 遍历数据集合中的每个键值对
	for k, item := range v.data {
		// 如果项目未被标记为已见，则将其添加到未见列表中
		if !item.seen {
			notSeen = append(notSeen, k)
		}
	}
	// 返回未见列表
	return notSeen
}

// 比较漏洞元数据的差异，并返回差异结果
func diffVulnerabilityMetadata(baseModels, targetModels *[]v4.VulnerabilityMetadata, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v4.Diff {
	// 创建一个空的差异结果集合
	diffs := make(map[string]*v4.Diff)
	// 创建一个基础模型的元数据集合
	m := NewMetadataSet(baseModels)

	// 遍历目标模型集合中的每个模型
	for _, tModel := range *targetModels {
		// 复制目标模型，以便在循环中修改
		targetModel := tModel
		// 获取目标模型的元数据键
		k := getMetadataKey(targetModel)
		// 如果基础模型集合中存在目标模型
		if m.in(targetModel) {
			// 如果目标模型与基础模型不匹配
			if !m.match(targetModel) {
				// 如果差异结果集合中已存在相同键的差异结果，则跳过当前循环
				if _, exists := diffs[k.id+k.namespace]; exists {
					continue
				}
# 遍历一个数据结构，为每个元素创建一个差异对象，并将其添加到差异字典中
for _, k := range m.getUnmatched() {
    # 如果差异字典中已经存在相同的键，则跳过当前元素
    if _, exists := diffs[k.id+k.namespace]; exists {
        continue
    }
    # 创建一个差异对象，表示元素被移除，并将其添加到差异字典中
    diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v4.DiffRemoved)
    # 增加不同项的计数
    differentItems.Increment()
}

# 返回差异字典的指针
return &diffs
# 定义一个函数，接收一个v4.VulnerabilityMetadata类型的参数metadata，并返回一个storeKey类型的值
func getMetadataKey(metadata v4.VulnerabilityMetadata) storeKey {
    # 创建一个storeKey类型的值，其中包含metadata的ID和Namespace，最后一个参数为空字符串
    return storeKey{metadata.ID, metadata.Namespace, ""}
}
```