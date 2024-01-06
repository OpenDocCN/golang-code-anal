# `grype\grype\db\v5\store\store.go`

```
package store

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"sort" // 导入 sort 包，用于对数据进行排序

	_ "github.com/glebarez/sqlite" // 通过导入该包，为 gorm 提供 sqlite 方言
	"github.com/go-test/deep" // 导入 deep 包，用于深度比较

	"gorm.io/gorm" // 导入 gorm 包，用于操作数据库

	"github.com/anchore/grype/grype/db/internal/gormadapter" // 导入 gormadapter 包
	v5 "github.com/anchore/grype/grype/db/v5" // 导入 v5 包
	"github.com/anchore/grype/grype/db/v5/store/model" // 导入 model 包
	"github.com/anchore/grype/internal/stringutil" // 导入 stringutil 包
)

// store holds an instance of the database connection
type store struct {
	db *gorm.DB // store 结构体包含一个 gorm 数据库连接实例
}
// New函数创建一个新的存储实例。
func New(dbFilePath string, overwrite bool) (v5.Store, error) {
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
		// 如果出现错误，自动迁移VulnerabilityMatchExclusionModel模型
		if err := db.AutoMigrate(&model.VulnerabilityMatchExclusionModel{}); err != nil {
			return nil, fmt.Errorf("unable to migrate Vulnerability Match Exclusion model: %w", err)
		}
	}

	// 返回一个包含数据库的存储结构指针
	return &store{
		db: db,
	}, nil
}

// GetID 获取关于数据库模式版本和构建时间的元数据
func (s *store) GetID() (*v5.ID, error) {
	// 创建一个空的IDModel切片
	var models []model.IDModel
	// 从数据库中查找IDModel并将结果存储在models切片中
	result := s.db.Find(&models)
	// 如果查找过程中出现错误，返回错误信息
	if result.Error != nil {
		return nil, result.Error
	}

	// 根据不同的情况进行处理
	switch {
// 根据模型数量进行不同的处理
switch {
    // 如果模型数量大于1，返回错误信息
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple DB IDs")
    // 如果模型数量等于1
    case len(models) == 1:
        // 解压模型数据并返回ID
        id, err := models[0].Inflate()
        // 如果解压出错，返回错误信息
        if err != nil {
            return nil, err
        }
        // 返回解压出的ID
        return &id, nil
    }

    // 如果模型数量为0，返回空值
    return nil, nil
}

// SetID 存储数据库的模式版本和构建时间
func (s *store) SetID(id v5.ID) error {
    var ids []model.IDModel

    // 用给定的ID替换现有的ID
    s.db.Find(&ids).Delete(&ids)
// 使用给定的 ID 创建一个新的 ID 模型对象
m := model.NewIDModel(id)
// 在数据库中创建新的记录，并将结果赋值给 result
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
	// 创建一个空的字符串数组
	var names []string
	// 从数据库中检索所有不同的命名空间，并将结果赋值给 names
	result := s.db.Model(&model.VulnerabilityMetadataModel{}).Distinct().Pluck("namespace", &names)
	// 返回命名空间数组和数据库操作的错误信息
	return names, result.Error
}

// 通过命名空间和 ID 检索漏洞
func (s *store) GetVulnerability(namespace, id string) ([]v5.Vulnerability, error) {
	// 创建一个空的漏洞模型数组
	var models []model.VulnerabilityModel
// 通过命名空间和ID在数据库中查找对应的记录，并将结果存储在models中
result := s.db.Where("namespace = ? AND id = ?", namespace, id).Find(&models)

// 创建一个长度为models长度的v5.Vulnerability切片
var vulnerabilities = make([]v5.Vulnerability, len(models))

// 遍历models切片，将每个记录解压缩为v5.Vulnerability对象，并存储在vulnerabilities切片中
for idx, m := range models {
    vulnerability, err := m.Inflate()
    if err != nil {
        return nil, err
    }
    vulnerabilities[idx] = vulnerability
}

// 返回vulnerabilities切片和result的错误信息
return vulnerabilities, result.Error
}

// 通过命名空间和包名在数据库中查找对应的记录，并将结果存储在models中
func (s *store) SearchForVulnerabilities(namespace, packageName string) ([]v5.Vulnerability, error) {
    var models []model.VulnerabilityModel

    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)
// 创建一个长度为 models 长度的 Vulnerability 切片
var vulnerabilities = make([]v5.Vulnerability, len(models))
// 遍历 models，将每个 model 转换为 Vulnerability，并存入 vulnerabilities 切片
for idx, m := range models {
    vulnerability, err := m.Inflate()
    if err != nil {
        return nil, err
    }
    vulnerabilities[idx] = vulnerability
}
// 返回 vulnerabilities 切片和 result.Error

// AddVulnerability 将一个或多个 vulnerabilities 保存到 sqlite3 存储中
func (s *store) AddVulnerability(vulnerabilities ...v5.Vulnerability) error {
    // 遍历 vulnerabilities，将每个 vulnerability 转换为 VulnerabilityModel
    for _, vulnerability := range vulnerabilities {
        m := model.NewVulnerabilityModel(vulnerability)
        // 将 VulnerabilityModel 存入数据库
        result := s.db.Create(&m)
        // 如果出现错误，返回错误
        if result.Error != nil {
// 返回错误结果
return result.Error
// 如果受影响的行数不等于1，则返回添加漏洞失败的错误
return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
// 获取给定漏洞ID相对于特定记录源的元数据
func (s *store) GetVulnerabilityMetadata(id, namespace string) (*v5.VulnerabilityMetadata, error) {
// 创建一个VulnerabilityMetadataModel切片
var models []model.VulnerabilityMetadataModel
// 在数据库中查找匹配给定漏洞ID和命名空间的VulnerabilityMetadataModel，并将结果存储在models切片中
result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, Namespace: namespace}).Find(&models)
// 如果发生错误，则返回nil和错误结果
if result.Error != nil {
    return nil, result.Error
}
// 使用switch语句处理不同的情况
switch {
// 检查是否存在多个元数据与单个ID和命名空间对应，如果是则返回错误
case len(models) > 1:
    return nil, fmt.Errorf("found multiple metadatas for single ID=%q Namespace=%q", id, namespace)
// 检查是否存在一个元数据与单个ID和命名空间对应，如果是则继续处理
case len(models) == 1:
    // 将元数据解压缩
    metadata, err := models[0].Inflate()
    if err != nil {
        return nil, err
    }
    // 返回解压缩后的元数据
    return &metadata, nil
}

// 将一个或多个漏洞元数据模型存储到sqlite数据库中
func (s *store) AddVulnerabilityMetadata(metadata ...v5.VulnerabilityMetadata) error {
    // 遍历元数据
    for _, m := range metadata {
        // 获取指定ID和命名空间的漏洞元数据
        existing, err := s.GetVulnerabilityMetadata(m.ID, m.Namespace)
		if err != nil {
			// 如果发生错误，返回带有错误信息的错误对象
			return fmt.Errorf("failed to verify existing entry: %w", err)
		}

		if existing != nil {
			// 与现有条目合并

			switch {
			case existing.Severity != m.Severity:
				// 如果现有元数据的严重性与新元数据不匹配，返回带有错误信息的错误对象
				return fmt.Errorf("existing metadata has mismatched severity (%q!=%q)", existing.Severity, m.Severity)
			case existing.Description != m.Description:
				// 如果现有元数据的描述与新元数据不匹配，返回带有错误信息的错误对象
				return fmt.Errorf("existing metadata has mismatched description (%q!=%q)", existing.Description, m.Description)
			}

		incoming:
			// 遍历所有传入的 CVSS，并查看它们是否已经存储。
			// 如果它们已经存在于数据库中，则跳过添加，防止重复
			for _, incomingCvss := range m.Cvss {
				for _, existingCvss := range existing.Cvss {
				if len(deep.Equal(incomingCvss, existingCvss)) == 0 {
					// 检查 incomingCvss 和 existingCvss 是否相同，如果相同则说明重复，不需要添加
					continue incoming
				}
				// 如果没有找到重复的 CVSS 条目，则将 incoming CVSS 添加到 existing.Cvss 中
				existing.Cvss = append(existing.Cvss, incomingCvss)
			}

			// 创建一个包含 existing.URLs 的字符串集合
			links := stringutil.NewStringSetFromSlice(existing.URLs)
			// 遍历 m.URLs，将其添加到 links 中
			for _, l := range m.URLs {
				links.Add(l)
			}

			// 将 links 转换为切片并赋值给 existing.URLs
			existing.URLs = links.ToSlice()
			// 对 existing.URLs 进行排序
			sort.Strings(existing.URLs)

			// 创建一个新的 VulnerabilityMetadataModel 对象
			newModel := model.NewVulnerabilityMetadataModel(*existing)
			// 将 newModel 保存到数据库中
			result := s.db.Save(&newModel)
			// 检查更新操作是否影响了一行数据，如果不是则返回错误
			if result.RowsAffected != 1 {
				return fmt.Errorf("unable to merge vulnerability metadata (%d rows affected)", result.RowsAffected)
			}

			// 检查更新操作是否出现错误，如果有则返回错误
			if result.Error != nil {
				return result.Error
			}
		} else {
			// 如果是新的条目
			// 创建一个新的 VulnerabilityMetadataModel 对象
			newModel := model.NewVulnerabilityMetadataModel(m)
			// 将新对象插入数据库
			result := s.db.Create(&newModel)
			// 检查插入操作是否出现错误，如果有则返回错误
			if result.Error != nil {
				return result.Error
			}

			// 检查插入操作是否影响了一行数据，如果不是则返回错误
			if result.RowsAffected != 1 {
				return fmt.Errorf("unable to add vulnerability metadata (%d rows affected)", result.RowsAffected)
			}
		}
	}
// 返回空值
return nil
}

// GetVulnerabilityMatchExclusion 根据漏洞标识符检索一个或多个漏洞匹配排除记录
func (s *store) GetVulnerabilityMatchExclusion(id string) ([]v5.VulnerabilityMatchExclusion, error) {
	// 定义一个模型数组
	var models []model.VulnerabilityMatchExclusionModel

	// 在数据库中查找符合条件的记录并将结果存储在models中
	result := s.db.Where("id = ?", id).Find(&models)

	// 定义一个漏洞匹配排除数组
	var exclusions []v5.VulnerabilityMatchExclusion
	// 遍历模型数组
	for _, m := range models {
		// 将模型转换为漏洞匹配排除对象
		exclusion, err := m.Inflate()
		// 如果转换过程中出现错误，则返回空值和错误
		if err != nil {
			return nil, err
		}

		// 如果排除对象不为空，则将其添加到排除数组中
		if exclusion != nil {
			exclusions = append(exclusions, *exclusion)
		}
	}
// AddVulnerabilityMatchExclusion 将一个或多个漏洞匹配排除记录保存到 sqlite3 存储中
func (s *store) AddVulnerabilityMatchExclusion(exclusions ...v5.VulnerabilityMatchExclusion) error {
	// 遍历传入的排除记录
	for _, exclusion := range exclusions {
		// 创建漏洞匹配排除模型
		m := model.NewVulnerabilityMatchExclusionModel(exclusion)

		// 将模型保存到数据库中
		result := s.db.Create(&m)
		// 检查保存过程中是否出现错误
		if result.Error != nil {
			return result.Error
		}

		// 检查保存的行数是否为1，如果不是则返回错误
		if result.RowsAffected != 1 {
			return fmt.Errorf("unable to add vulnerability match exclusion (%d rows affected)", result.RowsAffected)
		}
	}

	// 如果保存过程中没有出现错误，则返回 nil
	return nil
}
}

// Close closes the store and performs a VACUUM operation on the database
func (s *store) Close() {
	// Perform a VACUUM operation on the database to reclaim unused space
	s.db.Exec("VACUUM;")

	// Get the underlying SQL database connection
	sqlDB, err := s.db.DB()
	if err != nil {
		// Close the database connection if there is an error
		_ = sqlDB.Close()
	}
}

// GetAllVulnerabilities retrieves all vulnerabilities from the database
func (s *store) GetAllVulnerabilities() (*[]v5.Vulnerability, error) {
	// Define a slice to hold the retrieved vulnerability models
	var models []model.VulnerabilityModel
	// Query the database to retrieve all vulnerability models
	if result := s.db.Find(&models); result.Error != nil {
		// Return an error if there is a problem with the query
		return nil, result.Error
	}
	// Create a slice to hold the inflated vulnerabilities
	vulns := make([]v5.Vulnerability, len(models))
	// Iterate through the retrieved models and inflate each one
	for idx, m := range models {
		vuln, err := m.Inflate()
		// 如果发生错误，返回空值和错误信息
		if err != nil {
			return nil, err
		}
		// 将获取到的漏洞信息存入漏洞数组中
		vulns[idx] = vuln
	}
	// 返回漏洞数组和空错误信息
	return &vulns, nil
}

// GetAllVulnerabilityMetadata 获取数据库中的所有漏洞元数据
func (s *store) GetAllVulnerabilityMetadata() (*[]v5.VulnerabilityMetadata, error) {
	// 创建一个模型数组来存储漏洞元数据模型
	var models []model.VulnerabilityMetadataModel
	// 从数据库中获取所有漏洞元数据模型
	if result := s.db.Find(&models); result.Error != nil {
		// 如果发生错误，返回空值和错误信息
		return nil, result.Error
	}
	// 创建一个漏洞元数据数组，长度与模型数组相同
	metadata := make([]v5.VulnerabilityMetadata, len(models))
	// 遍历模型数组
	for idx, m := range models {
		// 解压缩数据
		data, err := m.Inflate()
		// 如果发生错误，返回空值和错误信息
		if err != nil {
			return nil, err
		}
		metadata[idx] = data
	}
	return &metadata, nil
}

// DiffStore creates a diff between the current sql database and the given store
func (s *store) DiffStore(targetStore v5.StoreReader) (*[]v5.Diff, error) {
	// 7 stages, one for each step of the diff process (stages)
	rowsProgress, diffItems, stager := trackDiff(7)

	// 设置当前阶段为“读取目标漏洞”
	stager.Current = "reading target vulnerabilities"
	// 从目标存储中获取所有漏洞信息
	targetVulns, err := targetStore.GetAllVulnerabilities()
	// 增加进度
	rowsProgress.Increment()
	// 如果出现错误，返回空和错误信息
	if err != nil {
		return nil, err
	}

	// 设置当前阶段为“读取基础漏洞”
	stager.Current = "reading base vulnerabilities"
	// 从当前存储中获取所有漏洞信息
	baseVulns, err := s.GetAllVulnerabilities()
	// 增加进度
	rowsProgress.Increment()
// 如果发生错误，返回空值和错误信息
if err != nil {
    return nil, err
}

// 设置当前进度为“准备中”
stager.Current = "preparing"

// 构建基础漏洞包映射
baseVulnPkgMap := buildVulnerabilityPkgsMap(baseVulns)

// 构建目标漏洞包映射
targetVulnPkgMap := buildVulnerabilityPkgsMap(targetVulns)

// 设置当前进度为“比较漏洞”
stager.Current = "comparing vulnerabilities"

// 比较基础漏洞和目标漏洞，生成所有差异的映射
allDiffsMap := diffVulnerabilities(baseVulns, targetVulns, baseVulnPkgMap, targetVulnPkgMap, diffItems)

// 设置当前进度为“读取基础元数据”
stager.Current = "reading base metadata"

// 读取所有基础漏洞元数据
baseMetadata, err := s.GetAllVulnerabilityMetadata()
if err != nil {
    return nil, err
}
rowsProgress.Increment()

// 设置当前进度为“读取目标元数据”
stager.Current = "reading target metadata"

// 读取所有目标漏洞元数据
targetMetadata, err := targetStore.GetAllVulnerabilityMetadata()
# 如果发生错误，返回空值和错误信息
if err != nil:
    return nil, err
# 增加行进度
rowsProgress.Increment()

# 设置当前阶段为“比较元数据”
stager.Current = "comparing metadata"
# 比较漏洞元数据，生成差异的映射
metaDiffsMap := diffVulnerabilityMetadata(baseMetadata, targetMetadata, baseVulnPkgMap, targetVulnPkgMap, diffItems)
# 将差异映射中的差异添加到所有差异映射中
for k, diff := range *metaDiffsMap:
    (*allDiffsMap)[k] = diff
# 将所有差异映射中的差异添加到所有差异列表中
allDiffs := []v5.Diff{}
for _, diff := range *allDiffsMap:
    allDiffs = append(allDiffs, *diff)

# 设置行进度为已完成
rowsProgress.SetCompleted()
# 设置差异项为已完成
diffItems.SetCompleted()

# 返回所有差异列表和空的错误信息
return &allDiffs, nil
抱歉，我无法为您提供代码注释，因为您没有提供需要注释的代码。如果您有任何代码需要解释，请提供给我，我将竭诚为您服务。
```