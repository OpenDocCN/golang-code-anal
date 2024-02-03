# `grype\grype\db\v4\store\store.go`

```go
package store

import (
    "fmt"
    "sort"

    _ "github.com/glebarez/sqlite" // 通过导入提供 sqlite 方言给 gorm 使用
    "github.com/go-test/deep"
    "gorm.io/gorm"

    "github.com/anchore/grype/grype/db/internal/gormadapter"
    v4 "github.com/anchore/grype/grype/db/v4"
    "github.com/anchore/grype/grype/db/v4/store/model"
    "github.com/anchore/grype/internal/stringutil"
)

// store holds an instance of the database connection
type store struct {
    db *gorm.DB
}

// New creates a new instance of the store.
func New(dbFilePath string, overwrite bool) (v4.Store, error) {
    // 打开数据库连接
    db, err := gormadapter.Open(dbFilePath, overwrite)
    if err != nil {
        return nil, err
    }

    if overwrite {
        // TODO: automigrate could write to the database,
        //  we should be validating the database is the correct database based on the version in the ID table before
        //  automigrating
        // 自动迁移可能会写入数据库，我们应该在自动迁移之前验证数据库是否是正确的数据库，基于 ID 表中的版本
        if err := db.AutoMigrate(&model.IDModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate ID model: %w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate Vulnerability model: %w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityMetadataModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate Vulnerability Metadata model: %w", err)
        }
        if err := db.AutoMigrate(&model.VulnerabilityMatchExclusionModel{}); err != nil {
            return nil, fmt.Errorf("unable to migrate Vulnerability Match Exclusion model: %w", err)
        }
    }

    return &store{
        db: db,
    }, nil
}

// GetID fetches the metadata about the databases schema version and build time.
func (s *store) GetID() (*v4.ID, error) {
    var models []model.IDModel
    result := s.db.Find(&models)
    if result.Error != nil {
        return nil, result.Error
    }

    switch {
    # 检查模型数组的长度是否大于1
    case len(models) > 1:
        # 如果是，则返回空和错误信息，表示找到多个数据库ID
        return nil, fmt.Errorf("found multiple DB IDs")
    # 如果模型数组的长度等于1
    case len(models) == 1:
        # 解压模型数组的第一个元素，获取ID和错误信息
        id, err := models[0].Inflate()
        # 如果有错误，则返回空和错误信息
        if err != nil {
            return nil, err
        }
        # 返回ID和空错误信息
        return &id, nil
    }

    # 如果模型数组的长度为0，则返回空和空错误信息
    return nil, nil
// SetID函数用于存储数据库的模式版本和构建时间
func (s *store) SetID(id v4.ID) error {
    var ids []model.IDModel

    // 用给定的ID替换现有的ID
    s.db.Find(&ids).Delete(&ids)

    m := model.NewIDModel(id)
    result := s.db.Create(&m)

    if result.RowsAffected != 1 {
        return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
    }

    return result.Error
}

// GetVulnerabilityNamespaces函数从数据库中检索所有可能的命名空间
func (s *store) GetVulnerabilityNamespaces() ([]string, error) {
    var names []string
    result := s.db.Model(&model.VulnerabilityMetadataModel{}).Distinct().Pluck("namespace", &names)
    return names, result.Error
}

// GetVulnerability函数按命名空间和包名检索漏洞
func (s *store) GetVulnerability(namespace, packageName string) ([]v4.Vulnerability, error) {
    var models []model.VulnerabilityModel

    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

    var vulnerabilities = make([]v4.Vulnerability, len(models))
    for idx, m := range models {
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
    }

    return vulnerabilities, result.Error
}

// AddVulnerability函数将一个或多个漏洞保存到sqlite3存储中
func (s *store) AddVulnerability(vulnerabilities ...v4.Vulnerability) error {
    for _, vulnerability := range vulnerabilities {
        m := model.NewVulnerabilityModel(vulnerability)

        result := s.db.Create(&m)
        if result.Error != nil {
            return result.Error
        }

        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
        }
    }
    return nil
}
// GetVulnerabilityMetadata 根据给定的漏洞ID和命名空间获取与特定记录源相关的元数据。
func (s *store) GetVulnerabilityMetadata(id, namespace string) (*v4.VulnerabilityMetadata, error) {
    // 创建一个空的VulnerabilityMetadataModel切片
    var models []model.VulnerabilityMetadataModel

    // 在数据库中查找匹配ID和Namespace的VulnerabilityMetadataModel，并将结果存储在models切片中
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, Namespace: namespace}).Find(&models)
    // 如果发生错误，则返回nil和错误
    if result.Error != nil {
        return nil, result.Error
    }

    // 根据models切片的长度进行不同的处理
    switch {
    case len(models) > 1:
        // 如果找到多个相同ID和Namespace的元数据，则返回错误
        return nil, fmt.Errorf("found multiple metadatas for single ID=%q Namespace=%q", id, namespace)
    case len(models) == 1:
        // 如果找到一个匹配的元数据，则将其展开并返回
        metadata, err := models[0].Inflate()
        if err != nil {
            return nil, err
        }
        return &metadata, nil
    }

    // 如果没有找到匹配的元数据，则返回nil
    return nil, nil
}

// AddVulnerabilityMetadata 将一个或多个漏洞元数据模型存储到sqlite数据库中。
//
//nolint:gocognit
func (s *store) AddVulnerabilityMetadata(metadata ...v4.VulnerabilityMetadata) error {
    // 在这里添加漏洞元数据到sqlite数据库
    // 这里需要实现具体的逻辑
    return nil
}

// GetVulnerabilityMatchExclusion 根据漏洞标识符检索一个或多个漏洞匹配排除记录。
func (s *store) GetVulnerabilityMatchExclusion(id string) ([]v4.VulnerabilityMatchExclusion, error) {
    // 创建一个空的VulnerabilityMatchExclusionModel切片
    var models []model.VulnerabilityMatchExclusionModel

    // 在数据库中查找匹配ID的VulnerabilityMatchExclusionModel，并将结果存储在models切片中
    result := s.db.Where("id = ?", id).Find(&models)

    // 创建一个空的VulnerabilityMatchExclusion切片
    var exclusions []v4.VulnerabilityMatchExclusion
    // 遍历models切片，将每个元素展开并添加到exclusions切片中
    for _, m := range models {
        exclusion, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        if exclusion != nil {
            exclusions = append(exclusions, *exclusion)
        }
    }

    // 返回exclusions切片和结果的错误
    return exclusions, result.Error
}

// AddVulnerabilityMatchExclusion 将一个或多个漏洞匹配排除记录保存到sqlite3存储中。
func (s *store) AddVulnerabilityMatchExclusion(exclusions ...v4.VulnerabilityMatchExclusion) error {
    // 在这里添加漏洞匹配排除记录到sqlite3存储中
    // 这里需要实现具体的逻辑
    return nil
}
    # 遍历排除列表中的每个元素
    for _, exclusion := range exclusions {
        # 根据排除元素创建漏洞匹配排除模型
        m := model.NewVulnerabilityMatchExclusionModel(exclusion)

        # 将漏洞匹配排除模型添加到数据库
        result := s.db.Create(&m)
        # 如果添加过程中出现错误，则返回错误信息
        if result.Error != nil {
            return result.Error
        }

        # 如果添加成功的行数不为1，则返回错误信息
        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability match exclusion (%d rows affected)", result.RowsAffected)
        }
    }

    # 如果添加成功，则返回空
    return nil
// Close 关闭存储对象，执行数据库的 VACUUM 操作
func (s *store) Close() {
    s.db.Exec("VACUUM;")

    // 获取数据库连接
    sqlDB, err := s.db.DB()
    if err != nil {
        // 如果获取失败，则关闭数据库连接
        _ = sqlDB.Close()
    }
}

// GetAllVulnerabilities 获取数据库中的所有漏洞
func (s *store) GetAllVulnerabilities() (*[]v4.Vulnerability, error) {
    var models []model.VulnerabilityModel
    // 从数据库中获取所有漏洞数据
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
    }
    // 将数据库模型转换为漏洞对象
    vulns := make([]v4.Vulnerability, len(models))
    for idx, m := range models {
        vuln, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulns[idx] = vuln
    }
    return &vulns, nil
}

// GetAllVulnerabilityMetadata 获取数据库中的所有漏洞元数据
func (s *store) GetAllVulnerabilityMetadata() (*[]v4.VulnerabilityMetadata, error) {
    var models []model.VulnerabilityMetadataModel
    // 从数据库中获取所有漏洞元数据
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
    }
    // 将数据库模型转换为漏洞元数据对象
    metadata := make([]v4.VulnerabilityMetadata, len(models))
    for idx, m := range models {
        data, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        metadata[idx] = data
    }
    return &metadata, nil
}

// DiffStore 创建当前 SQL 数据库与给定存储之间的差异
func (s *store) DiffStore(targetStore v4.StoreReader) (*[]v4.Diff, error) {
    // 7个阶段，每个阶段代表差异过程的一步
    rowsProgress, diffItems, stager := trackDiff(7)

    stager.Current = "reading target vulnerabilities"
    // 读取目标存储中的漏洞数据
    targetVulns, err := targetStore.GetAllVulnerabilities()
    rowsProgress.Increment()
    if err != nil {
        return nil, err
    }

    stager.Current = "reading base vulnerabilities"
    // 读取当前存储中的漏洞数据
    baseVulns, err := s.GetAllVulnerabilities()
    rowsProgress.Increment()
    if err != nil {
        return nil, err
    }

    stager.Current = "preparing"
    // 构建漏洞包映射
    baseVulnPkgMap := buildVulnerabilityPkgsMap(baseVulns)
    # 构建目标漏洞包映射
    targetVulnPkgMap := buildVulnerabilityPkgsMap(targetVulns)
    
    # 设置当前阶段为“比较漏洞”
    stager.Current = "comparing vulnerabilities"
    
    # 比较基础漏洞和目标漏洞，生成所有差异的映射
    allDiffsMap := diffVulnerabilities(baseVulns, targetVulns, baseVulnPkgMap, targetVulnPkgMap, diffItems)
    
    # 设置当前阶段为“读取基础元数据”
    stager.Current = "reading base metadata"
    
    # 读取所有基础漏洞的元数据
    baseMetadata, err := s.GetAllVulnerabilityMetadata()
    if err != nil:
        return nil, err
    rowsProgress.Increment()
    
    # 设置当前阶段为“读取目标元数据”
    stager.Current = "reading target metadata"
    
    # 读取所有目标漏洞的元数据
    targetMetadata, err := targetStore.GetAllVulnerabilityMetadata()
    if err != nil:
        return nil, err
    rowsProgress.Increment()
    
    # 设置当前阶段为“比较元数据”
    stager.Current = "comparing metadata"
    
    # 比较基础元数据和目标元数据，生成所有差异的映射
    metaDiffsMap := diffVulnerabilityMetadata(baseMetadata, targetMetadata, baseVulnPkgMap, targetVulnPkgMap, diffItems)
    
    # 将元数据差异映射合并到所有差异映射中
    for k, diff := range *metaDiffsMap:
        (*allDiffsMap)[k] = diff
    
    # 初始化所有差异列表
    allDiffs := []v4.Diff{}
    
    # 将所有差异映射中的差异添加到所有差异列表中
    for _, diff := range *allDiffsMap:
        allDiffs = append(allDiffs, *diff)
    
    # 标记行进度为已完成
    rowsProgress.SetCompleted()
    
    # 标记差异项为已完成
    diffItems.SetCompleted()
    
    # 返回所有差异列表和空错误
    return &allDiffs, nil
# 闭合前面的函数定义
```