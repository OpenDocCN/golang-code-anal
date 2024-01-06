# `grype\grype\db\v5\store\diff.go`

```
// 导入所需的包
package store

import (
	"github.com/wagoodman/go-partybus" // 导入go-partybus包
	"github.com/wagoodman/go-progress" // 导入go-progress包

	v5 "github.com/anchore/grype/grype/db/v5" // 导入v5版本的数据库包
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
// 定义存储漏洞列表的结构体
type storeVulnerabilityList struct {
	items map[storeKey][]storeVulnerability // 存储漏洞列表的映射
	seen  bool // 是否已经查看过
}
// 定义存储漏洞的结构体
type storeVulnerability struct {
	item *v5.Vulnerability // 漏洞项
	seen bool // 是否已经查看过
}
// 定义存储元数据的结构体
type storeMetadata struct {
	item *v5.VulnerabilityMetadata // 漏洞元数据项
	seen bool // 是否已经查看过
}

// 创建用于跟踪数据库差异进度的手动进度条
func trackDiff(total int64) (*progress.Manual, *progress.Manual, *progress.Stage) {
	stageProgress := &progress.Manual{} // 创建阶段进度的手动进度条
	stageProgress.SetTotal(total) // 设置阶段进度的总数
	differencesDiscovered := &progress.Manual{} // 创建发现差异的手动进度条
	stager := &progress.Stage{} // 创建阶段进度
	// 发布一个事件，表示数据库差异比较开始
	bus.Publish(partybus.Event{
		// 事件类型为数据库差异比较开始
		Type: event.DatabaseDiffingStarted,
		// 值为监视数据库差异的对象
		Value: monitor.DBDiff{
			Stager:                stager,  // 设置数据库差异监视器的阶段
			StageProgress:         progress.Progressable(stageProgress),  // 设置阶段进度
			DifferencesDiscovered: progress.Monitorable(differencesDiscovered),  // 设置发现的差异
		},
	})
	// 返回阶段进度、发现的差异和数据库差异监视器
	return stageProgress, differencesDiscovered, stager
}

// 创建一个从未打包的键到与之关联的所有软件包列表的映射
func buildVulnerabilityPkgsMap(models *[]v5.Vulnerability) *map[storeKey][]string {
	// 创建一个空的映射
	storeMap := make(map[storeKey][]string)
	// 遍历所有漏洞模型
	for _, m := range *models {
		model := m
		// 获取漏洞的父键
		k := getVulnerabilityParentKey(model)
		// 如果映射中已经存在该键，则将软件包名称追加到列表中
		if storeVuln, exists := storeMap[k]; exists {
			storeMap[k] = append(storeVuln, model.PackageName)
		} else {
// 创建一个空的字符串到字符串切片的映射
storeMap[k] = []string{model.PackageName}

// 返回 storeMap 的指针
return &storeMap
}

// 使用包映射信息创建一个给定键的差异，以填充受更新影响的相关包
func createDiff(baseStore, targetStore *PkgMap, key storeKey, reason v5.DiffReason) *v5.Diff {
    // 创建一个包映射
    pkgMap := make(map[string]struct{})

    // 将 key 的包名设置为空
    key.packageName = ""
    
    // 如果 baseStore 不为空
    if baseStore != nil {
        // 如果 key 在 baseStore 中存在
        if basePkgs, exists := (*baseStore)[key]; exists {
            // 遍历 basePkgs 中的包名，将其添加到 pkgMap 中
            for _, pkg := range basePkgs {
                pkgMap[pkg] = struct{}{}
            }
        }
    }
    
    // 如果 targetStore 不为空
    if targetStore != nil {
// 如果目标包存在于目标存储中，则将目标包中的每个包添加到pkgMap中
if targetPkgs, exists := (*targetStore)[key]; exists {
    for _, pkg := range targetPkgs {
        pkgMap[pkg] = struct{}{}
    }
}

// 创建一个空的字符串切片pkgs
pkgs := []string{}

// 遍历pkgMap中的包，并将它们添加到pkgs切片中
for pkg := range pkgMap {
    pkgs = append(pkgs, pkg)
}

// 返回一个v5.Diff结构体，包括原因、ID、命名空间和包列表
return &v5.Diff{
    Reason:    reason,
    ID:        key.id,
    Namespace: key.namespace,
    Packages:  pkgs,
}
```

```
// 从漏洞中获取一个未打包的密钥
// 根据漏洞获取其父级键值
func getVulnerabilityParentKey(vuln v5.Vulnerability) storeKey {
	return storeKey{vuln.ID, vuln.Namespace, ""}
}

// 从漏洞获取打包的键值
func getVulnerabilityKey(vuln v5.Vulnerability) storeKey {
	return storeKey{vuln.ID, vuln.Namespace, vuln.PackageName}
}

// 定义漏洞集合结构
type VulnerabilitySet struct {
	data map[storeKey]*storeVulnerabilityList
}

// 创建新的漏洞集合
func NewVulnerabilitySet(models *[]v5.Vulnerability) *VulnerabilitySet {
	// 初始化一个包含漏洞键值和存储漏洞列表的映射
	m := make(map[storeKey]*storeVulnerabilityList, len(*models))
	// 遍历漏洞模型列表
	for _, mm := range *models {
		model := mm
		// 获取漏洞的父级键值和漏洞键值
		parentKey := getVulnerabilityParentKey(model)
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
func (v *VulnerabilitySet) in(item v5.Vulnerability) bool {
	// 检查漏洞是否存在于漏洞集合中，如果存在则返回 true
	_, exists := v.data[getVulnerabilityParentKey(item)]
	return exists
}

// 检查给定的漏洞是否与漏洞集合中的某个漏洞匹配，返回布尔值
func (v *VulnerabilitySet) match(item v5.Vulnerability) bool {
	// 检查漏洞是否存在于漏洞集合中
	if parent, exists := v.data[getVulnerabilityParentKey(item)]; exists {
		// 如果存在，则将其标记为已见
		parent.seen = true
		// 获取漏洞的键
		key := getVulnerabilityKey(item)
		// 检查是否存在与给定漏洞匹配的子漏洞
		if children, exists := parent.items[key]; exists {
			// 遍历子漏洞列表
			for idx, child := range children {
				// 如果找到匹配的子漏洞，则将其标记为已见并返回 true
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

// diffVulnerabilities 函数用于比较两个版本的漏洞数据，返回不同的漏洞信息
func diffVulnerabilities(baseModels, targetModels *[]v5.Vulnerability, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v5.Diff {
	// 创建一个空的漏洞差异映射
	diffs := make(map[string]*v5.Diff)
	// 创建一个包含基准版本漏洞的漏洞集合
	m := NewVulnerabilitySet(baseModels)

	// 遍历目标版本的漏洞数据
	for _, tModel := range *targetModels {
		targetModel := tModel
		// 获取漏洞的唯一标识
		k := getVulnerabilityKey(targetModel)
		// 如果基准版本中存在相同的漏洞
		if m.in(targetModel) {
			// 在基准版本中查找匹配的漏洞
			matched := m.match(targetModel)
			// 如果没有匹配的漏洞
			if !matched {
				// 如果差异映射中已经存在相同的漏洞
				if _, exists := diffs[k.id+k.namespace]; exists {
					// 继续下一次循环
					continue
				}
				// 创建漏洞差异对象，并添加到差异映射中
				diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v5.DiffChanged)
				// 增加不同漏洞的计数
				differentItems.Increment()
			}
		} else {
			// 如果 diffs 中已经存在相同的键值对，则跳过
			if _, exists := diffs[k.id+k.namespace]; exists {
				continue
			}
			// 创建一个新的差异对象，并将其添加到 diffs 中
			diffs[k.id+k.namespace] = createDiff(nil, targetPkgsMap, k, v5.DiffAdded)
			// 增加不同项的计数
			differentItems.Increment()
		}
	}
	// 获取未匹配和部分匹配的项
	notSeen, partialSeen := m.getUnmatched()
	// 遍历部分匹配的项
	for _, k := range partialSeen {
		// 如果 diffs 中已经存在相同的键值对，则跳过
		if _, exists := diffs[k.id+k.namespace]; exists {
			continue
		}
		// 创建一个新的差异对象，并将其添加到 diffs 中
		diffs[k.id+k.namespace] = createDiff(basePkgsMap, targetPkgsMap, k, v5.DiffChanged)
		// 增加不同项的计数
		differentItems.Increment()
	}
	// 遍历未匹配的项
	for _, k := range notSeen {
		// 如果 diffs 中已经存在相同的键值对，则跳过
		if _, exists := diffs[k.id+k.namespace]; exists {
			continue
		}
		// 将差异数据存入diffs字典中，键为k.id+k.namespace，值为createDiff函数的返回值
		diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v5.DiffRemoved)
		// 增加不同项的计数
		differentItems.Increment()
	}

	// 返回存储差异数据的diffs字典的指针
	return &diffs
}

// MetadataSet结构体定义
type MetadataSet struct {
	data map[storeKey]*storeMetadata
}

// 创建新的MetadataSet对象
func NewMetadataSet(models *[]v5.VulnerabilityMetadata) *MetadataSet {
	// 使用models的长度创建一个map
	m := make(map[storeKey]*storeMetadata, len(*models))
	// 遍历models，将每个元素转换为storeMetadata对象，并存入map中
	for _, mm := range *models {
		model := mm
		m[getMetadataKey(model)] = &storeMetadata{
			item: &model,
			seen: false,
		}
	}
# 返回一个包含数据的MetadataSet结构体指针
return &MetadataSet{
    data: m,
}

# 检查给定的VulnerabilityMetadata是否存在于MetadataSet中
func (v *MetadataSet) in(item v5.VulnerabilityMetadata) bool {
    # 检查给定的VulnerabilityMetadata是否存在于MetadataSet中，并返回结果
    _, exists := v.data[getMetadataKey(item)]
    return exists
}

# 检查给定的VulnerabilityMetadata是否与MetadataSet中的元数据匹配
func (v *MetadataSet) match(item v5.VulnerabilityMetadata) bool {
    # 如果基础模型存在于MetadataSet中，则将其标记为已见，并检查是否与给定的元数据相等
    if baseModel, exists := v.data[getMetadataKey(item)]; exists {
        baseModel.seen = true
        return baseModel.item.Equal(item)
    }
    return false
}

# 获取未匹配的元数据
func (v *MetadataSet) getUnmatched() []storeKey {
    # 创建一个空的未匹配列表
    notSeen := []storeKey{}
	// 遍历数据集合中的每个键值对
	for k, item := range v.data {
		// 如果项目未被标记为已见
		if !item.seen {
			// 将未被标记为已见的项目添加到未见列表中
			notSeen = append(notSeen, k)
		}
	}
	// 返回未被标记为已见的项目列表
	return notSeen
}

// 比较漏洞元数据的差异，并返回差异结果
func diffVulnerabilityMetadata(baseModels, targetModels *[]v5.VulnerabilityMetadata, basePkgsMap, targetPkgsMap *PkgMap, differentItems *progress.Manual) *map[string]*v5.Diff {
	// 创建一个差异结果的映射
	diffs := make(map[string]*v5.Diff)
	// 创建一个基础模型的元数据集合
	m := NewMetadataSet(baseModels)

	// 遍历目标模型的每个元数据
	for _, tModel := range *targetModels {
		// 复制目标模型
		targetModel := tModel
		// 获取元数据的键
		k := getMetadataKey(targetModel)
		// 如果基础模型中存在目标模型
		if m.in(targetModel) {
			// 如果目标模型与基础模型不匹配
			if !m.match(targetModel) {
				// 如果差异结果中已存在相同键的差异
				if _, exists := diffs[k.id+k.namespace]; exists {
					// 继续下一次循环
					continue
				}
# 遍历一个数据结构，根据条件创建不同的diff对象，并将其存储在diffs字典中
for _, k := range m.getUnmatched() {
    # 如果diffs字典中已经存在相同的key，则跳过当前循环
    if _, exists := diffs[k.id+k.namespace]; exists {
        continue
    }
    # 根据条件创建不同的diff对象，并将其存储在diffs字典中
    diffs[k.id+k.namespace] = createDiff(basePkgsMap, nil, k, v5.DiffRemoved)
    # 增加不同项的计数
    differentItems.Increment()
}

# 返回存储了不同项的diffs字典
return &diffs
# 定义一个函数，接收一个v5.VulnerabilityMetadata类型的参数metadata，并返回一个storeKey类型的值
func getMetadataKey(metadata v5.VulnerabilityMetadata) storeKey {
    # 创建一个storeKey类型的值，其中包含metadata的ID和Namespace，以及一个空字符串
    return storeKey{metadata.ID, metadata.Namespace, ""}
}
```