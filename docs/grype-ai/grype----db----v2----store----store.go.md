# `grype\grype\db\v2\store\store.go`

```
// 导入所需的包
package store

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"sort" // 导入 sort 包，用于对数据进行排序

	_ "github.com/glebarez/sqlite" // 通过导入该包，为 gorm 提供 sqlite 方言支持
	"github.com/go-test/deep" // 导入 deep 包，用于深度比较数据
	"gorm.io/gorm" // 导入 gorm 包，用于数据库操作

	"github.com/anchore/grype/grype/db/internal/gormadapter" // 导入 gormadapter 包
	v2 "github.com/anchore/grype/grype/db/v2" // 导入 v2 包
	"github.com/anchore/grype/grype/db/v2/store/model" // 导入 model 包
	"github.com/anchore/grype/internal/stringutil" // 导入 stringutil 包
)

// store 结构体，包含一个数据库连接实例
type store struct {
	db *gorm.DB // 数据库连接实例
}
// New函数创建一个新的存储实例。
func New(dbFilePath string, overwrite bool) (v2.Store, error) {
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

	return &store{
		db: db,
	}, nil
}

// GetID fetches the metadata about the databases schema version and build time.
// GetID函数用于获取数据库模式版本和构建时间的元数据。
func (s *store) GetID() (*v2.ID, error) {
	// 定义一个存储IDModel的切片
	var models []model.IDModel
	// 从数据库中查找IDModel并将结果存储在models切片中
	result := s.db.Find(&models)
	// 如果查找过程中出现错误，则返回错误
	if result.Error != nil {
		return nil, result.Error
	}

	// 根据查找到的IDModel数量进行不同的处理
	switch {
	case len(models) > 1:
		return nil, fmt.Errorf("found multiple DB IDs")
	case len(models) == 1:
		// 将第一个IDModel解析为ID对象
		id, err := models[0].Inflate()
		// 如果发生错误，返回空和错误信息
		if err != nil {
			return nil, err
		}
		// 返回存储的数据库模式版本和构建时间
		return &id, nil
	}

	// 如果没有错误，返回空和空值
	return nil, nil
}

// SetID 存储数据库的模式版本和构建时间
func (s *store) SetID(id v2.ID) error {
	var ids []model.IDModel

	// 用给定的ID替换现有的ID
	s.db.Find(&ids).Delete(&ids)

	// 创建一个新的IDModel对象
	m := model.NewIDModel(id)
	// 在数据库中创建新的ID记录
	result := s.db.Create(&m)

	// 如果受影响的行数不等于1
	if result.RowsAffected != 1 {
// 返回一个错误，指示无法添加 id，并包含受影响的行数
return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
// 返回数据库操作的错误
}

// GetVulnerability 根据命名空间和包名检索一个或多个漏洞
func (s *store) GetVulnerability(namespace, packageName string) ([]v2.Vulnerability, error) {
	// 声明一个存储漏洞模型的切片
	var models []model.VulnerabilityModel

	// 在数据库中查找符合条件的漏洞模型
	result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

	// 声明一个存储漏洞的切片，长度为漏洞模型的数量
	var vulnerabilities = make([]v2.Vulnerability, len(models))
	// 遍历漏洞模型切片
	for idx, m := range models {
		// 调用漏洞模型的 Inflate 方法，将数据库模型转换为漏洞对象
		vulnerability, err := m.Inflate()
		// 如果转换过程中出现错误，返回 nil 和错误
		if err != nil {
			return nil, err
		}
		// 将转换后的漏洞对象存储到漏洞切片中
		vulnerabilities[idx] = vulnerability
	}
// AddVulnerability 将一个或多个漏洞保存到 sqlite3 存储中
func (s *store) AddVulnerability(vulnerabilities ...v2.Vulnerability) error {
	// 遍历传入的漏洞列表
	for _, vulnerability := range vulnerabilities {
		// 创建漏洞模型对象
		m := model.NewVulnerabilityModel(vulnerability)

		// 将漏洞模型对象保存到数据库中
		result := s.db.Create(&m)
		// 如果保存过程中出现错误，则返回错误
		if result.Error != nil {
			return result.Error
		}

		// 如果保存的行数不等于1，则返回错误
		if result.RowsAffected != 1 {
			return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
		}
	}
	// 没有出现错误，则返回空
	return nil
}
// GetVulnerabilityMetadata 根据给定的漏洞ID和记录来源获取漏洞元数据
func (s *store) GetVulnerabilityMetadata(id, recordSource string) (*v2.VulnerabilityMetadata, error) {
	// 定义一个存储漏洞元数据模型的切片
	var models []model.VulnerabilityMetadataModel

	// 在数据库中查找匹配给定漏洞ID和记录来源的漏洞元数据模型
	result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, RecordSource: recordSource}).Find(&models)
	// 如果查找过程中出现错误，则返回错误
	if result.Error != nil {
		return nil, result.Error
	}

	// 根据查找到的漏洞元数据模型数量进行不同的处理
	switch {
	case len(models) > 1:
		// 如果找到多个匹配的漏洞元数据模型，则返回错误
		return nil, fmt.Errorf("found multiple metadatas for single ID=%q RecordSource=%q", id, recordSource)
	case len(models) == 1:
		// 如果找到一个匹配的漏洞元数据模型，则解析该模型并返回漏洞元数据
		metadata, err := models[0].Inflate()
		if err != nil {
			return nil, err
		}

		return &metadata, nil
	}

	return nil, nil
}

// AddVulnerabilityMetadata 将一个或多个漏洞元数据模型存储到 sqlite 数据库中
func (s *store) AddVulnerabilityMetadata(metadata ...v2.VulnerabilityMetadata) error {
	// 遍历传入的漏洞元数据
	for _, m := range metadata {
		// 获取指定 ID 和记录来源的漏洞元数据
		existing, err := s.GetVulnerabilityMetadata(m.ID, m.RecordSource)
		if err != nil {
			// 如果获取失败，返回错误
			return fmt.Errorf("failed to verify existing entry: %w", err)
		}

		// 如果存在相同 ID 和记录来源的漏洞元数据
		if existing != nil {
			// 与现有条目合并

			// 比较现有条目的 CVSS V3 和传入数据的 CVSS V3
			cvssV3Diffs := deep.Equal(existing.CvssV3, m.CvssV3)
			// 比较现有条目的 CVSS V2 和传入数据的 CVSS V2
			cvssV2Diffs := deep.Equal(existing.CvssV2, m.CvssV2)

			// 根据比较结果进行处理
			switch {
			// 检查现有元数据的严重性是否匹配，如果不匹配则返回错误
			case existing.Severity != m.Severity:
				return fmt.Errorf("existing metadata has mismatched severity (%q!=%q)", existing.Severity, m.Severity)
			// 检查现有元数据的描述是否匹配，如果不匹配则返回错误
			case existing.Description != m.Description:
				return fmt.Errorf("existing metadata has mismatched description (%q!=%q)", existing.Description, m.Description)
			// 检查现有元数据的 CVSS-V2 是否存在且与新元数据不匹配，如果不匹配则返回错误
			case existing.CvssV2 != nil && len(cvssV2Diffs) > 0:
				return fmt.Errorf("existing metadata has mismatched cvss-v2: %+v", cvssV2Diffs)
			// 检查现有元数据的 CVSS-V3 是否存在且与新元数据不匹配，如果不匹配则返回错误
			case existing.CvssV3 != nil && len(cvssV3Diffs) > 0:
				return fmt.Errorf("existing metadata has mismatched cvss-v3: %+v", cvssV3Diffs)
			// 如果以上条件都不满足，则更新现有元数据的 CVSS-V2 和 CVSS-V3
			default:
				existing.CvssV2 = m.CvssV2
				existing.CvssV3 = m.CvssV3
			}

			// 创建现有链接的字符串集合
			links := stringutil.NewStringSetFromSlice(existing.Links)
			// 遍历新链接，将其添加到链接集合中
			for _, l := range m.Links {
				links.Add(l)
			}

			// 将链接集合转换为切片，并按字母顺序排序
			existing.Links = links.ToSlice()
			sort.Strings(existing.Links)
// 根据现有数据创建新的漏洞元数据模型
newModel := model.NewVulnerabilityMetadataModel(*existing)
// 将新的漏洞元数据模型保存到数据库中
result := s.db.Save(&newModel)

// 检查保存操作影响的行数，如果不为1，则返回错误
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to merge vulnerability metadata (%d rows affected)", result.RowsAffected)
}

// 检查保存操作是否出现错误，如果有则返回错误
if result.Error != nil {
    return result.Error
}
// 如果是新的条目
} else {
    // 创建新的漏洞元数据模型
    newModel := model.NewVulnerabilityMetadataModel(m)
    // 将新的漏洞元数据模型保存到数据库中
    result := s.db.Create(&newModel)
    // 检查保存操作是否出现错误，如果有则返回错误
    if result.Error != nil {
        return result.Error
    }
    // 检查保存操作影响的行数，如果不为1，则返回错误
    if result.RowsAffected != 1 {
# 返回一个错误信息，指示无法添加漏洞元数据，并包含受影响的行数
return fmt.Errorf("unable to add vulnerability metadata (%d rows affected)", result.RowsAffected)

# 关闭存储对象的数据库连接
func (s *store) Close() {
    # 执行 VACUUM 命令，用于清理数据库中的垃圾数据
    s.db.Exec("VACUUM;")
}
```