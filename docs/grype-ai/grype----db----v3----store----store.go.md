# `grype\grype\db\v3\store\store.go`

```
// 导入所需的包
package store

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"sort" // 导入 sort 包，用于对数据进行排序

	_ "github.com/glebarez/sqlite" // 通过导入该包，为 gorm 提供 sqlite 方言支持
	"github.com/go-test/deep" // 导入 deep 包，用于深度比较两个值是否相等
	"gorm.io/gorm" // 导入 gorm 包，用于数据库操作

	"github.com/anchore/grype/grype/db/internal/gormadapter" // 导入 gormadapter 包
	v3 "github.com/anchore/grype/grype/db/v3" // 导入 v3 包
	"github.com/anchore/grype/grype/db/v3/store/model" // 导入 model 包
	"github.com/anchore/grype/internal/stringutil" // 导入 stringutil 包
)

// store 结构体，包含一个数据库连接实例
type store struct {
	db *gorm.DB // 数据库连接实例
}
// New函数创建一个新的存储实例。
func New(dbFilePath string, overwrite bool) (v3.Store, error) {
    // 打开数据库连接
    db, err := gormadapter.Open(dbFilePath, overwrite)
    if err != nil {
        return nil, err
    }

    if overwrite {
        // 如果需要覆盖数据库，则进行自动迁移
        // TODO: automigrate可能会写入数据库，我们应该在自动迁移之前验证数据库是否是基于ID表中的版本的正确数据库
        if err := db.AutoMigrate(&model.IDModel{}); err != nil {
            return nil, fmt.Errorf("无法迁移ID模型：%w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityModel{}); err != nil {
            return nil, fmt.Errorf("无法迁移漏洞模型：%w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityMetadataModel{}); err != nil {
            return nil, fmt.Errorf("无法迁移漏洞元数据模型：%w", err)
        }
		}
	}

	return &store{
		db: db,
	}, nil
}

// GetID fetches the metadata about the databases schema version and build time.
// GetID方法用于获取数据库架构版本和构建时间的元数据。
func (s *store) GetID() (*v3.ID, error) {
	// 定义一个存储IDModel的切片
	var models []model.IDModel
	// 从数据库中查找IDModel并将结果存储在models切片中
	result := s.db.Find(&models)
	// 如果查找过程中出现错误，则返回错误
	if result.Error != nil {
		return nil, result.Error
	}

	// 根据查找到的models切片的长度进行不同的处理
	switch {
	case len(models) > 1:
		// 如果找到多个DB ID，则返回错误
		return nil, fmt.Errorf("found multiple DB IDs")
	case len(models) == 1:
		// 从模型数组中取出第一个模型，并解压缩
		id, err := models[0].Inflate()
		// 如果解压缩过程中出现错误，返回空和错误
		if err != nil {
			return nil, err
		}
		// 返回解压缩后的ID和空错误
		return &id, nil
	}

	// 如果没有模型，返回空和空错误
	return nil, nil
}

// SetID 存储数据库的模式版本和构建时间
func (s *store) SetID(id v3.ID) error {
	// 创建一个IDModel类型的数组
	var ids []model.IDModel

	// 用给定的ID替换现有的ID
	s.db.Find(&ids).Delete(&ids)

	// 创建一个新的IDModel，并将其存储到数据库中
	m := model.NewIDModel(id)
	result := s.db.Create(&m)
// 如果受影响的行数不等于1，则返回错误信息
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
}

// 返回结果的错误信息
return result.Error
}

// GetVulnerability 根据命名空间和包名检索一个或多个漏洞
func (s *store) GetVulnerability(namespace, packageName string) ([]v3.Vulnerability, error) {
    var models []model.VulnerabilityModel

    // 在数据库中根据命名空间和包名查找漏洞
    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

    // 创建一个与模型长度相同的漏洞切片
    var vulnerabilities = make([]v3.Vulnerability, len(models))
    for idx, m := range models {
        // 将模型转换为漏洞对象
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
// AddVulnerability 将一个或多个漏洞保存到 sqlite3 存储中
func (s *store) AddVulnerability(vulnerabilities ...v3.Vulnerability) error {
    // 遍历传入的漏洞列表
    for _, vulnerability := range vulnerabilities {
        // 创建漏洞模型对象
        m := model.NewVulnerabilityModel(vulnerability)

        // 将漏洞模型对象保存到数据库中
        result := s.db.Create(&m)
        // 如果保存过程中出现错误，则返回错误信息
        if result.Error != nil {
            return result.Error
        }

        // 如果保存的行数不等于1，则返回无法添加漏洞的错误信息
        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
        }
    }
    // 如果保存成功，则返回 nil
    return nil
}
// GetVulnerabilityMetadata 根据给定的漏洞ID和命名空间获取与特定记录源相关的元数据。
func (s *store) GetVulnerabilityMetadata(id, namespace string) (*v3.VulnerabilityMetadata, error) {
    // 定义一个VulnerabilityMetadataModel切片
    var models []model.VulnerabilityMetadataModel

    // 在数据库中查找匹配给定ID和命名空间的VulnerabilityMetadataModel，并将结果存储在models切片中
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, Namespace: namespace}).Find(&models)
    // 如果查找过程中出现错误，则返回错误
    if result.Error != nil {
        return nil, result.Error
    }

    // 根据查找结果的数量进行不同的处理
    switch {
    case len(models) > 1:
        // 如果找到多个相同ID和命名空间的元数据，则返回错误
        return nil, fmt.Errorf("found multiple metadatas for single ID=%q Namespace=%q", id, namespace)
    case len(models) == 1:
        // 如果找到一个匹配的元数据，则将其解压缩为VulnerabilityMetadata对象
        metadata, err := models[0].Inflate()
        // 如果解压缩过程中出现错误，则返回错误
        if err != nil {
            return nil, err
        }
		// 返回元数据和空错误，表示成功存储了漏洞元数据模型到sqlite数据库
		return &metadata, nil
	}

	// 如果没有元数据，则返回空和空错误
	return nil, nil
}

// AddVulnerabilityMetadata将一个或多个漏洞元数据模型存储到sqlite数据库中
//
//nolint:gocognit
func (s *store) AddVulnerabilityMetadata(metadata ...v3.VulnerabilityMetadata) error {
	// 遍历元数据
	for _, m := range metadata {
		// 获取现有的元数据
		existing, err := s.GetVulnerabilityMetadata(m.ID, m.Namespace)
		if err != nil {
			// 如果获取失败，则返回错误
			return fmt.Errorf("failed to verify existing entry: %w", err)
		}

		// 如果存在现有的元数据
		if existing != nil {
			// 与现有条目合并

			switch {
			// 检查现有元数据的严重性是否与新元数据匹配，如果不匹配则返回错误
			case existing.Severity != m.Severity:
				return fmt.Errorf("existing metadata has mismatched severity (%q!=%q)", existing.Severity, m.Severity)
			// 检查现有元数据的描述是否与新元数据匹配，如果不匹配则返回错误
			case existing.Description != m.Description:
				return fmt.Errorf("existing metadata has mismatched description (%q!=%q)", existing.Description, m.Description)
			}

		incoming:
			// 遍历所有新的 CVSS 来检查它们是否已经存储
			// 如果它们已经存在于数据库中，则跳过添加，防止重复
			for _, incomingCvss := range m.Cvss {
				for _, existingCvss := range existing.Cvss {
					// 检查新的 CVSS 是否与现有的 CVSS 相等
					if len(deep.Equal(incomingCvss, existingCvss)) == 0 {
						// 发现重复，因此不应添加新的 CVSS
						continue incoming
					}
				}
				// 没有找到重复的 CVSS 条目，因此将新的 CVSS 追加到现有的 CVSS 中
				existing.Cvss = append(existing.Cvss, incomingCvss)
			}
# 创建一个新的字符串集合，包含现有URLs
links := stringutil.NewStringSetFromSlice(existing.URLs)

# 遍历新的URLs列表，将每个URL添加到现有的URL集合中
for _, l := range m.URLs {
    links.Add(l)
}

# 将更新后的URL集合转换为切片，并按字母顺序排序
existing.URLs = links.ToSlice()
sort.Strings(existing.URLs)

# 创建一个新的漏洞元数据模型
newModel := model.NewVulnerabilityMetadataModel(*existing)

# 将新的漏洞元数据模型保存到数据库中
result := s.db.Save(&newModel)

# 检查保存操作是否成功
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to merge vulnerability metadata (%d rows affected)", result.RowsAffected)
}

# 检查保存操作是否出现错误
if result.Error != nil {
    return result.Error
}
// 创建一个新的漏洞元数据模型对象
newModel := model.NewVulnerabilityMetadataModel(m)
// 将新的漏洞元数据模型对象存储到数据库中
result := s.db.Create(&newModel)
// 检查存储操作是否出错，如果有错误则返回错误信息
if result.Error != nil {
    return result.Error
}

// 检查存储操作是否影响了一行数据，如果没有则返回错误信息
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to add vulnerability metadata (%d rows affected)", result.RowsAffected)
}

// 获取所有漏洞数据模型对象
func (s *store) GetAllVulnerabilities() (*[]v3.Vulnerability, error) {
    var models []model.VulnerabilityModel
    // 从数据库中获取所有漏洞数据模型对象
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
	}
	// 创建一个与 models 长度相同的空的 Vulnerability 切片
	vulns := make([]v3.Vulnerability, len(models))
	// 遍历 models，将每个 model 转换为 Vulnerability 对象，并存入 vulns 切片
	for idx, m := range models {
		vuln, err := m.Inflate()
		// 如果转换出错，返回错误
		if err != nil {
			return nil, err
		}
		vulns[idx] = vuln
	}
	// 返回 Vulnerability 切片的指针和空错误
	return &vulns, nil
}

// GetAllVulnerabilityMetadata 从数据库中获取所有漏洞元数据
func (s *store) GetAllVulnerabilityMetadata() (*[]v3.VulnerabilityMetadata, error) {
	// 创建一个空的 VulnerabilityMetadataModel 切片
	var models []model.VulnerabilityMetadataModel
	// 从数据库中获取所有漏洞元数据
	if result := s.db.Find(&models); result.Error != nil {
		return nil, result.Error
	}
	// 创建一个与 models 长度相同的空的 VulnerabilityMetadata 切片
	metadata := make([]v3.VulnerabilityMetadata, len(models))
	// 遍历 models，将每个 model 转换为 VulnerabilityMetadata 对象，并存入 metadata 切片
	for idx, m := range models {
		// 调用Inflate方法解压数据，如果出现错误则返回nil和错误
		data, err := m.Inflate()
		if err != nil {
			return nil, err
		}
		// 将解压后的数据存储到metadata中的指定索引位置
		metadata[idx] = data
	}
	// 返回metadata和nil，表示没有错误
	return &metadata, nil
}

// DiffStore方法用于创建当前SQL数据库和给定存储之间的差异
func (s *store) DiffStore(targetStore v3.StoreReader) (*[]v3.Diff, error) {
	// 7个阶段，每个阶段对应差异过程中的一个步骤
	rowsProgress, diffItems, stager := trackDiff(7)

	// 设置当前阶段为“读取目标漏洞”
	stager.Current = "reading target vulnerabilities"
	// 从目标存储中获取所有漏洞信息
	targetVulns, err := targetStore.GetAllVulnerabilities()
	// 增加进度
	rowsProgress.Increment()
	// 如果出现错误则返回nil和错误
	if err != nil {
		return nil, err
	}
	# 设置当前进度为“读取基本漏洞”
	baseVulns, err := s.GetAllVulnerabilities()  # 获取所有基本漏洞信息
	rowsProgress.Increment()  # 增加进度条
	if err != nil:  # 如果出现错误
		return nil, err  # 返回空和错误信息

	# 设置当前进度为“准备中”
	baseVulnPkgMap := buildVulnerabilityPkgsMap(baseVulns)  # 构建基本漏洞包映射
	targetVulnPkgMap := buildVulnerabilityPkgsMap(targetVulns)  # 构建目标漏洞包映射

	# 设置当前进度为“比较漏洞”
	allDiffsMap := diffVulnerabilities(baseVulns, targetVulns, baseVulnPkgMap, targetVulnPkgMap, diffItems)  # 比较漏洞信息

	# 设置当前进度为“读取基本元数据”
	baseMetadata, err := s.GetAllVulnerabilityMetadata()  # 获取所有基本漏洞元数据
	if err != nil:  # 如果出现错误
		return nil, err  # 返回空和错误信息
# 增加行进度
rowsProgress.Increment()

# 设置当前阶段为“读取目标元数据”
stager.Current = "reading target metadata"
# 从目标存储获取所有漏洞元数据
targetMetadata, err := targetStore.GetAllVulnerabilityMetadata()
if err != nil:
    return nil, err
# 增加行进度
rowsProgress.Increment()

# 设置当前阶段为“比较元数据”
# 比较基础元数据和目标元数据，生成差异的漏洞元数据映射
metaDiffsMap := diffVulnerabilityMetadata(baseMetadata, targetMetadata, baseVulnPkgMap, targetVulnPkgMap, diffItems)
# 将差异的漏洞元数据映射添加到总的差异映射中
for k, diff := range *metaDiffsMap:
    (*allDiffsMap)[k] = diff
# 将所有差异映射转换为数组
allDiffs := []v3.Diff{}
for _, diff := range *allDiffsMap:
    allDiffs = append(allDiffs, *diff)

# 设置行进度为已完成
rowsProgress.SetCompleted()
# 将 diffItems 标记为已完成
diffItems.SetCompleted()

# 返回所有的差异项和空值
return &allDiffs, nil
}

# 关闭数据库连接并执行 VACUUM 操作
func (s *store) Close() {
    s.db.Exec("VACUUM;")
}
```