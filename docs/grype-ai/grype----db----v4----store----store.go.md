# `grype\grype\db\v4\store\store.go`

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
	v4 "github.com/anchore/grype/grype/db/v4" // 导入 v4 包
	"github.com/anchore/grype/grype/db/v4/store/model" // 导入 model 包
	"github.com/anchore/grype/internal/stringutil" // 导入 stringutil 包
)

// store 结构体，包含一个指向数据库连接的指针
type store struct {
	db *gorm.DB
}
// New函数创建一个新的存储实例。
func New(dbFilePath string, overwrite bool) (v4.Store, error) {
    // 使用给定的数据库文件路径和覆盖标志打开数据库连接
    db, err := gormadapter.Open(dbFilePath, overwrite)
    if err != nil {
        return nil, err
    }

    if overwrite {
        // 如果覆盖标志为true，则执行自动迁移操作，将模型映射到数据库表
        // TODO: automigrate可能会写入数据库，我们应该在自动迁移之前验证数据库是否是基于ID表中的版本正确的数据库
        if err := db.AutoMigrate(&model.IDModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate ID model: %w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate Vulnerability model: %w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityMetadataModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate Vulnerability Metadata model: %w", err)
        }
		}
		// 如果出现错误，返回无法迁移漏洞匹配排除模型的错误信息
		if err := db.AutoMigrate(&model.VulnerabilityMatchExclusionModel{}); err != nil {
			return nil, fmt.Errorf("unable to migrate Vulnerability Match Exclusion model: %w", err)
		}
	}

	// 返回存储对象，包含数据库连接
	return &store{
		db: db,
	}, nil
}

// GetID 获取关于数据库模式版本和构建时间的元数据。
func (s *store) GetID() (*v4.ID, error) {
	// 创建一个空的 IDModel 切片
	var models []model.IDModel
	// 从数据库中获取 IDModel 数据
	result := s.db.Find(&models)
	// 如果获取过程中出现错误，返回错误信息
	if result.Error != nil {
		return nil, result.Error
	}

	// 根据不同的情况进行处理
	switch {
// 根据模型数量进行不同的处理
switch {
    // 如果模型数量大于1，返回错误
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple DB IDs")
    // 如果模型数量等于1
    case len(models) == 1:
        // 解压模型数据
        id, err := models[0].Inflate()
        // 如果解压出错，返回错误
        if err != nil {
            return nil, err
        }
        // 返回解压后的ID
        return &id, nil
    }

    // 如果模型数量为0，返回空
    return nil, nil
}

// SetID 存储数据库的模式版本和构建时间
func (s *store) SetID(id v4.ID) error {
    var ids []model.IDModel

    // 用给定的ID替换现有的ID
    s.db.Find(&ids).Delete(&ids)
// 使用给定的 ID 创建一个新的 IDModel 对象
m := model.NewIDModel(id)
// 在数据库中创建一个新的记录，并将结果赋值给 result
result := s.db.Create(&m)

// 如果受影响的行数不等于 1，则返回错误信息
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
}

// 返回数据库操作的错误信息
return result.Error
}

// 从数据库中检索所有可能的命名空间
func (s *store) GetVulnerabilityNamespaces() ([]string, error) {
    var names []string
    // 从 VulnerabilityMetadataModel 表中检索唯一的命名空间，并将结果存储在 names 切片中
    result := s.db.Model(&model.VulnerabilityMetadataModel{}).Distinct().Pluck("namespace", &names)
    return names, result.Error
}

// 通过命名空间和包名检索漏洞信息
func (s *store) GetVulnerability(namespace, packageName string) ([]v4.Vulnerability, error) {
    var models []model.VulnerabilityModel
```

// 通过查询条件查找数据库中符合条件的记录，并将结果存储到models中
result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

// 创建一个长度为models长度的vulnerabilities切片
var vulnerabilities = make([]v4.Vulnerability, len(models))

// 遍历models切片，将每个记录转换为vulnerability对象存储到vulnerabilities切片中
for idx, m := range models {
    vulnerability, err := m.Inflate()
    if err != nil {
        return nil, err
    }
    vulnerabilities[idx] = vulnerability
}

// 返回vulnerabilities切片和查询结果的错误信息
return vulnerabilities, result.Error
}

// AddVulnerability将一个或多个漏洞保存到sqlite3存储中
func (s *store) AddVulnerability(vulnerabilities ...v4.Vulnerability) error {
    // 遍历vulnerabilities切片，将每个漏洞对象转换为对应的数据库模型对象
    for _, vulnerability := range vulnerabilities {
        m := model.NewVulnerabilityModel(vulnerability)
// 使用给定的数据创建新记录，并将结果存储在result变量中
result := s.db.Create(&m)
// 如果创建过程中出现错误，则返回该错误
if result.Error != nil {
    return result.Error
}

// 如果受影响的行数不等于1，则返回一个格式化的错误信息
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
}

// 如果以上条件都不满足，则返回nil，表示没有错误发生
}

// GetVulnerabilityMetadata 根据给定的漏洞ID和命名空间检索与特定记录源相关的元数据
func (s *store) GetVulnerabilityMetadata(id, namespace string) (*v4.VulnerabilityMetadata, error) {
    // 创建一个空的VulnerabilityMetadataModel切片
    var models []model.VulnerabilityMetadataModel

    // 在数据库中查找符合给定条件的VulnerabilityMetadataModel，并将结果存储在result变量中
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, Namespace: namespace}).Find(&models)
    // 如果查找过程中出现错误，则返回该错误
    if result.Error != nil {
        return nil, result.Error
    }
// 根据 models 的长度进行不同的处理
switch {
    // 如果 models 的长度大于1，返回错误，表示找到了多个相同 ID 和 Namespace 的元数据
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple metadatas for single ID=%q Namespace=%q", id, namespace)
    // 如果 models 的长度等于1，继续处理
    case len(models) == 1:
        // 将第一个元数据解压缩为 metadata 对象
        metadata, err := models[0].Inflate()
        // 如果解压缩过程中出现错误，返回错误
        if err != nil {
            return nil, err
        }
        // 返回解压缩后的 metadata 对象
        return &metadata, nil
    }
    
    // 如果 models 的长度为0，返回空值
    return nil, nil
}

// AddVulnerabilityMetadata 将一个或多个 vulnerability metadata 模型存储到 sqlite 数据库中
//
//nolint:gocognit
func (s *store) AddVulnerabilityMetadata(metadata ...v4.VulnerabilityMetadata) error {
	// 遍历元数据列表
	for _, m := range metadata {
		// 获取指定漏洞的现有元数据
		existing, err := s.GetVulnerabilityMetadata(m.ID, m.Namespace)
		// 如果出现错误，返回错误信息
		if err != nil {
			return fmt.Errorf("failed to verify existing entry: %w", err)
		}

		// 如果存在现有元数据
		if existing != nil {
			// 与现有条目合并

			// 检查严重性是否匹配
			switch {
			case existing.Severity != m.Severity:
				return fmt.Errorf("existing metadata has mismatched severity (%q!=%q)", existing.Severity, m.Severity)
			// 检查描述是否匹配
			case existing.Description != m.Description:
				return fmt.Errorf("existing metadata has mismatched description (%q!=%q)", existing.Description, m.Description)
			}

		incoming:
			// 遍历所有传入的 CVSS，并查看它们是否已经存储。
			// 如果它们已经存在于数据库中，则跳过添加它们，防止重复
			// 遍历传入的 CVSS 列表
			for _, incomingCvss := range m.Cvss {
				// 遍历已存在的 CVSS 列表
				for _, existingCvss := range existing.Cvss {
					// 检查是否存在重复的 CVSS 条目
					if len(deep.Equal(incomingCvss, existingCvss)) == 0 {
						// 如果存在重复的 CVSS 条目，则不添加传入的 CVSS
						continue incoming
					}
				}
				// 如果没有找到重复的 CVSS 条目，则将传入的 CVSS 添加到已存在的列表中
				existing.Cvss = append(existing.Cvss, incomingCvss)
			}

			// 创建一个包含已存在的 URL 的字符串集合
			links := stringutil.NewStringSetFromSlice(existing.URLs)
			// 遍历传入的 URL 列表
			for _, l := range m.URLs {
				// 将传入的 URL 添加到已存在的集合中
				links.Add(l)
			}

			// 将更新后的 URL 列表赋值给已存在的 URL 列表
			existing.URLs = links.ToSlice()
			// 对 URL 列表进行排序
			sort.Strings(existing.URLs)

			// 创建一个新的漏洞元数据模型
			newModel := model.NewVulnerabilityMetadataModel(*existing)
// 保存新的模型数据到数据库中
result := s.db.Save(&newModel)

// 检查保存操作影响的行数，如果不为1，则返回错误
if result.RowsAffected != 1 {
    return fmt.Errorf("unable to merge vulnerability metadata (%d rows affected)", result.RowsAffected)
}

// 检查保存操作是否有错误，如果有则返回该错误
if result.Error != nil {
    return result.Error
}
// 如果不是更新操作，而是新的条目
} else {
    // 创建新的漏洞元数据模型
    newModel := model.NewVulnerabilityMetadataModel(m)
    // 将新模型数据添加到数据库中
    result := s.db.Create(&newModel)
    // 检查添加操作是否有错误，如果有则返回该错误
    if result.Error != nil {
        return result.Error
    }

    // 检查添加操作影响的行数，如果不为1，则返回错误
    if result.RowsAffected != 1 {
        return fmt.Errorf("unable to add vulnerability metadata (%d rows affected)", result.RowsAffected)
    }
}
		}
	}
	return nil
}

// GetVulnerabilityMatchExclusion retrieves one or more vulnerability match exclusion records given a vulnerability identifier.
// 根据漏洞标识符检索一个或多个漏洞匹配排除记录
func (s *store) GetVulnerabilityMatchExclusion(id string) ([]v4.VulnerabilityMatchExclusion, error) {
	// 定义一个模型数组
	var models []model.VulnerabilityMatchExclusionModel

	// 在数据库中查找符合条件的记录并将结果存储到models中
	result := s.db.Where("id = ?", id).Find(&models)

	// 定义一个漏洞匹配排除数组
	var exclusions []v4.VulnerabilityMatchExclusion
	// 遍历模型数组
	for _, m := range models {
		// 将模型转换为漏洞匹配排除对象
		exclusion, err := m.Inflate()
		if err != nil {
			return nil, err
		}

		// 如果排除对象不为空，则添加到排除数组中
		if exclusion != nil {
			exclusions = append(exclusions, *exclusion)
// AddVulnerabilityMatchExclusion 将一个或多个漏洞匹配排除记录保存到sqlite3存储中
func (s *store) AddVulnerabilityMatchExclusion(exclusions ...v4.VulnerabilityMatchExclusion) error {
	// 遍历传入的排除记录
	for _, exclusion := range exclusions {
		// 创建一个新的漏洞匹配排除模型
		m := model.NewVulnerabilityMatchExclusionModel(exclusion)

		// 将模型保存到数据库中
		result := s.db.Create(&m)
		// 检查是否有错误发生
		if result.Error != nil {
			return result.Error
		}

		// 检查是否只影响了一行数据
		if result.RowsAffected != 1 {
			return fmt.Errorf("unable to add vulnerability match exclusion (%d rows affected)", result.RowsAffected)
		}
	}
}
// 返回空值
return nil
}

// 关闭数据库连接
func (s *store) Close() {
	// 执行数据库的VACUUM操作
	s.db.Exec("VACUUM;")

	// 获取底层的SQL数据库连接
	sqlDB, err := s.db.DB()
	if err != nil {
		// 如果出错，关闭数据库连接
		_ = sqlDB.Close()
	}
}

// 获取数据库中的所有漏洞
func (s *store) GetAllVulnerabilities() (*[]v4.Vulnerability, error) {
	// 定义漏洞模型数组
	var models []model.VulnerabilityModel
	// 从数据库中获取所有漏洞模型
	if result := s.db.Find(&models); result.Error != nil {
		// 如果出错，返回空值和错误
		return nil, result.Error
	}
	// 创建漏洞结构体切片
	vulns := make([]v4.Vulnerability, len(models))
		// 遍历模型数组，获取每个模型的索引和值
		for idx, m := range models {
			// 调用模型的Inflate方法，获取漏洞数据和可能的错误
			vuln, err := m.Inflate()
			// 如果有错误，返回空和错误
			if err != nil {
				return nil, err
			}
			// 将获取的漏洞数据存入漏洞数组中
			vulns[idx] = vuln
		}
		// 返回漏洞数组和空错误
		return &vulns, nil
	}

	// GetAllVulnerabilityMetadata从数据库中获取所有漏洞元数据
	func (s *store) GetAllVulnerabilityMetadata() (*[]v4.VulnerabilityMetadata, error) {
		// 创建一个模型数组来存储漏洞元数据模型
		var models []model.VulnerabilityMetadataModel
		// 从数据库中获取所有漏洞元数据模型
		if result := s.db.Find(&models); result.Error != nil {
			// 如果有错误，返回空和错误
			return nil, result.Error
		}
		// 创建一个漏洞元数据数组，长度为模型数组的长度
		metadata := make([]v4.VulnerabilityMetadata, len(models))
		// 遍历模型数组，获取每个模型的索引和值
		for idx, m := range models {
			// 调用模型的Inflate方法，获取数据和可能的错误
			data, err := m.Inflate()
			// 如果有错误，返回空和错误
			if err != nil {
		// 如果发生错误，返回空值和错误信息
		return nil, err
	}
	// 创建一个存储差异的切片
	metadata[idx] = data
}
// 返回存储元数据的切片和空值
return &metadata, nil
}

// DiffStore 创建当前 SQL 数据库和给定存储之间的差异
func (s *store) DiffStore(targetStore v4.StoreReader) (*[]v4.Diff, error) {
	// 跟踪差异过程的7个阶段，每个阶段对应一个步骤
	rowsProgress, diffItems, stager := trackDiff(7)

	// 设置当前阶段为“读取目标漏洞”
	stager.Current = "reading target vulnerabilities"
	// 从目标存储中获取所有漏洞
	targetVulns, err := targetStore.GetAllVulnerabilities()
	// 增加进度
	rowsProgress.Increment()
	// 如果发生错误，返回空值和错误信息
	if err != nil {
		return nil, err
	}

	// 设置当前阶段为“读取基础漏洞”
// 获取所有基础漏洞信息
baseVulns, err := s.GetAllVulnerabilities()
// 更新进度条
rowsProgress.Increment()
// 如果出现错误，返回空和错误信息
if err != nil {
    return nil, err
}

// 设置当前状态为“准备中”
stager.Current = "preparing"
// 构建基础漏洞包映射
baseVulnPkgMap := buildVulnerabilityPkgsMap(baseVulns)
// 构建目标漏洞包映射
targetVulnPkgMap := buildVulnerabilityPkgsMap(targetVulns)

// 设置当前状态为“比较漏洞”
stager.Current = "comparing vulnerabilities"
// 比较基础漏洞和目标漏洞，生成所有差异的映射
allDiffsMap := diffVulnerabilities(baseVulns, targetVulns, baseVulnPkgMap, targetVulnPkgMap, diffItems)

// 设置当前状态为“读取基础元数据”
baseMetadata, err := s.GetAllVulnerabilityMetadata()
// 如果出现错误，返回空和错误信息
if err != nil {
    return nil, err
}
// 更新进度条
rowsProgress.Increment()
# 设置当前阶段为“读取目标元数据”
stager.Current = "reading target metadata"
# 从目标存储中获取所有漏洞元数据
targetMetadata, err := targetStore.GetAllVulnerabilityMetadata()
# 如果出现错误，返回空和错误信息
if err != nil:
    return nil, err
# 增加进度条的行数
rowsProgress.Increment()

# 设置当前阶段为“比较元数据”
stager.Current = "comparing metadata"
# 比较基础元数据和目标元数据，生成元数据差异的映射
metaDiffsMap := diffVulnerabilityMetadata(baseMetadata, targetMetadata, baseVulnPkgMap, targetVulnPkgMap, diffItems)
# 遍历元数据差异的映射，将差异添加到总差异映射中
for k, diff := range *metaDiffsMap:
    (*allDiffsMap)[k] = diff
# 将所有差异映射中的差异转换为数组
allDiffs := []v4.Diff{}
for _, diff := range *allDiffsMap:
    allDiffs = append(allDiffs, *diff)

# 设置进度条为已完成状态
rowsProgress.SetCompleted()
# 设置差异项为已完成状态
diffItems.SetCompleted()
# 返回allDiffs和nil。
```