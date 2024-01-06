# `grype\grype\db\v1\store\store.go`

```
// 导入所需的包
package store

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"sort" // 导入 sort 包，用于对数据进行排序

	_ "github.com/glebarez/sqlite" // 通过导入空白标识符来提供 sqlite 方言给 gorm
	"github.com/go-test/deep" // 导入 deep 包，用于深度比较

	"gorm.io/gorm" // 导入 gorm 包，用于数据库操作

	"github.com/anchore/grype/grype/db/internal/gormadapter" // 导入 gormadapter 包
	v1 "github.com/anchore/grype/grype/db/v1" // 导入 v1 包
	"github.com/anchore/grype/grype/db/v1/store/model" // 导入 model 包
	"github.com/anchore/grype/internal/stringutil" // 导入 stringutil 包
)

// store 持有数据库连接的实例
type store struct {
	db *gorm.DB // 数据库连接实例
}
// New函数创建一个新的存储实例。
func New(dbFilePath string, overwrite bool) (v1.Store, error) {
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
// GetID方法用于获取数据库模式版本和构建时间的元数据。
func (s *store) GetID() (*v1.ID, error) {
	// 创建一个空的IDModel切片
	var models []model.IDModel
	// 从数据库中查找IDModel并将结果存储在models切片中
	result := s.db.Find(&models)
	// 如果查找过程中出现错误，则返回错误
	if result.Error != nil {
		return nil, result.Error
	}

	// 根据models切片的长度进行不同的处理
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
func (s *store) SetID(id v1.ID) error {
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
func (s *store) GetVulnerability(namespace, packageName string) ([]v1.Vulnerability, error) {
    var models []model.VulnerabilityModel

    // 在数据库中根据命名空间和包名查找漏洞
    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

    // 创建一个与模型长度相同的漏洞切片
    var vulnerabilities = make([]v1.Vulnerability, len(models))
    for idx, m := range models {
        // 将模型转换为漏洞对象
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
	}

	return vulnerabilities, result.Error
}

// AddVulnerability saves one or more vulnerabilities into the sqlite3 store.
// AddVulnerability函数将一个或多个漏洞保存到sqlite3存储中
func (s *store) AddVulnerability(vulnerabilities ...v1.Vulnerability) error {
	// 遍历传入的漏洞列表
	for _, vulnerability := range vulnerabilities {
		// 创建漏洞模型
		m := model.NewVulnerabilityModel(vulnerability)

		// 将漏洞模型保存到数据库中
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
	// 如果保存成功，则返回nil
	return nil
}
// GetVulnerabilityMetadata 根据给定的漏洞ID和记录来源获取漏洞元数据
func (s *store) GetVulnerabilityMetadata(id, recordSource string) (*v1.VulnerabilityMetadata, error) {
    // 定义一个VulnerabilityMetadataModel切片
    var models []model.VulnerabilityMetadataModel

    // 在数据库中查找匹配给定ID和记录来源的漏洞元数据
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, RecordSource: recordSource}).Find(&models)
    // 如果查找过程中出现错误，则返回错误信息
    if result.Error != nil {
        return nil, result.Error
    }

    // 根据查找结果的数量进行不同的处理
    switch {
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple metadatas for single ID=%q RecordSource=%q", id, recordSource)
    case len(models) == 1:
        // 将第一个匹配的漏洞元数据解析为VulnerabilityMetadata对象
        metadata, err := models[0].Inflate()
        // 如果解析过程中出现错误，则返回错误信息
        if err != nil {
            return nil, err
        }
		return &metadata, nil
	}

	return nil, nil
}

// AddVulnerabilityMetadata stores one or more vulnerability metadata models into the sqlite DB.
func (s *store) AddVulnerabilityMetadata(metadata ...v1.VulnerabilityMetadata) error {
	// 遍历传入的元数据
	for _, m := range metadata {
		// 获取指定 ID 和记录来源的漏洞元数据
		existing, err := s.GetVulnerabilityMetadata(m.ID, m.RecordSource)
		if err != nil {
			// 如果获取失败，返回错误
			return fmt.Errorf("failed to verify existing entry: %w", err)
		}

		if existing != nil {
			// 如果存在相同的漏洞元数据，将新旧数据进行合并

			// 比较现有数据和新数据的 CVSS V3 字段的差异
			cvssV3Diffs := deep.Equal(existing.CvssV3, m.CvssV3)
			// 比较现有数据和新数据的 CVSS V2 字段的差异
			cvssV2Diffs := deep.Equal(existing.CvssV2, m.CvssV2)
// 检查现有元数据的严重性是否与新元数据的严重性匹配，如果不匹配则返回错误
switch {
case existing.Severity != m.Severity:
    return fmt.Errorf("existing metadata has mismatched severity (%q!=%q)", existing.Severity, m.Severity)
// 检查现有元数据的描述是否与新元数据的描述匹配，如果不匹配则返回错误
case existing.Description != m.Description:
    return fmt.Errorf("existing metadata has mismatched description (%q!=%q)", existing.Description, m.Description)
// 检查现有元数据的 CvssV2 是否存在且与新元数据的 CvssV2 有差异，如果有差异则返回错误
case existing.CvssV2 != nil && len(cvssV2Diffs) > 0:
    return fmt.Errorf("existing metadata has mismatched cvss-v2: %+v", cvssV2Diffs)
// 检查现有元数据的 CvssV3 是否存在且与新元数据的 CvssV3 有差异，如果有差异则返回错误
case existing.CvssV3 != nil && len(cvssV3Diffs) > 0:
    return fmt.Errorf("existing metadata has mismatched cvss-v3: %+v", cvssV3Diffs)
// 如果以上条件都不满足，则更新现有元数据的 CvssV2 和 CvssV3
default:
    existing.CvssV2 = m.CvssV2
    existing.CvssV3 = m.CvssV3
}

// 创建一个包含现有链接的字符串集合
links := stringutil.NewStringSetFromSlice(existing.Links)
// 遍历新链接列表，将新链接添加到现有链接集合中
for _, l := range m.Links {
    links.Add(l)
}
// 将更新后的链接集合转换为切片，并赋值给现有链接
existing.Links = links.ToSlice()
// 对现有链接列表进行排序
sort.Strings(existing.Links)

// 创建一个新的漏洞元数据模型
newModel := model.NewVulnerabilityMetadataModel(*existing)

// 将新的漏洞元数据模型保存到数据库中
result := s.db.Save(&newModel)

// 检查保存操作是否影响了一行数据，如果没有则返回错误
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to merge vulnerability metadata (%d rows affected)", result.RowsAffected)
}

// 检查保存操作是否出现错误，如果有则返回错误
if result.Error != nil {
    return result.Error
}
// 如果是新的条目，则创建一个新的漏洞元数据模型并保存到数据库中
} else {
    newModel := model.NewVulnerabilityMetadataModel(m)
    result := s.db.Create(&newModel)
    if result.Error != nil {
        return result.Error
    }
# 如果受影响的行数不等于1，则返回错误信息
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to add vulnerability metadata (%d rows affected)", result.RowsAffected)
}
# 关闭数据库连接
}

# 执行数据库的VACUUM操作，用于优化数据库性能
func (s *store) Close() {
    s.db.Exec("VACUUM;")
}
```