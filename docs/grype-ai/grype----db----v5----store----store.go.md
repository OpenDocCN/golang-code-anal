# `grype\grype\db\v5\store\store.go`

```
package store

import (
    "fmt"
    "sort"

    _ "github.com/glebarez/sqlite" // 通过导入提供 sqlite 方言给 gorm 使用
    "github.com/go-test/deep"
    "gorm.io/gorm"

    "github.com/anchore/grype/grype/db/internal/gormadapter"
    v5 "github.com/anchore/grype/grype/db/v5"
    "github.com/anchore/grype/grype/db/v5/store/model"
    "github.com/anchore/grype/internal/stringutil"
)

// store holds an instance of the database connection
type store struct {
    db *gorm.DB
}

// New creates a new instance of the store.
func New(dbFilePath string, overwrite bool) (v5.Store, error) {
    // 打开数据库连接
    db, err := gormadapter.Open(dbFilePath, overwrite)
    if err != nil {
        return nil, err
    }

    if overwrite {
        // TODO: automigrate could write to the database,
        //  we should be validating the database is the correct database based on the version in the ID table before
        //  automigrating
        // 自动迁移数据库表结构，确保数据库表结构与模型一致
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
func (s *store) GetID() (*v5.ID, error) {
    var models []model.IDModel
    // 查询数据库中的 IDModel 数据
    result := s.db.Find(&models)
    if result.Error != nil {
        return nil, result.Error
    }

    switch {
    // 检查模型数组的长度是否大于1，如果是则返回空和错误信息
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple DB IDs")
    // 检查模型数组的长度是否等于1，如果是则继续执行下面的代码
    case len(models) == 1:
        // 从模型数组中获取第一个模型的ID，并检查是否有错误
        id, err := models[0].Inflate()
        // 如果有错误则返回空和错误信息
        if err != nil {
            return nil, err
        }
        // 返回获取到的ID和空错误信息
        return &id, nil
    }

    // 如果模型数组长度不大于1也不等于1，则返回空和空错误信息
    return nil, nil
// SetID函数用于存储数据库的模式版本和构建时间
func (s *store) SetID(id v5.ID) error {
    var ids []model.IDModel

    // 用给定的ID替换现有的ID
    s.db.Find(&ids).Delete(&ids)

    // 创建一个新的IDModel对象
    m := model.NewIDModel(id)
    // 将新对象插入数据库
    result := s.db.Create(&m)

    // 检查插入操作是否成功
    if result.RowsAffected != 1 {
        return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
    }

    return result.Error
}

// GetVulnerabilityNamespaces函数从数据库中检索所有可能的命名空间
func (s *store) GetVulnerabilityNamespaces() ([]string, error) {
    var names []string
    // 从VulnerabilityMetadataModel表中检索不同的命名空间，并将结果存储在names切片中
    result := s.db.Model(&model.VulnerabilityMetadataModel{}).Distinct().Pluck("namespace", &names)
    return names, result.Error
}

// GetVulnerability函数通过命名空间和ID检索漏洞
func (s *store) GetVulnerability(namespace, id string) ([]v5.Vulnerability, error) {
    var models []model.VulnerabilityModel

    // 通过命名空间和ID从数据库中检索漏洞
    result := s.db.Where("namespace = ? AND id = ?", namespace, id).Find(&models)

    var vulnerabilities = make([]v5.Vulnerability, len(models))
    for idx, m := range models {
        // 将数据库模型转换为Vulnerability对象
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
    }

    return vulnerabilities, result.Error
}

// SearchForVulnerabilities函数通过命名空间和包名检索漏洞
func (s *store) SearchForVulnerabilities(namespace, packageName string) ([]v5.Vulnerability, error) {
    var models []model.VulnerabilityModel

    // 通过命名空间和包名从数据库中检索漏洞
    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

    var vulnerabilities = make([]v5.Vulnerability, len(models))
    for idx, m := range models {
        // 将数据库模型转换为Vulnerability对象
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
    }

    return vulnerabilities, result.Error
}
// AddVulnerability 将一个或多个漏洞保存到 sqlite3 存储中。
func (s *store) AddVulnerability(vulnerabilities ...v5.Vulnerability) error {
    // 遍历传入的漏洞列表
    for _, vulnerability := range vulnerabilities {
        // 创建漏洞模型对象
        m := model.NewVulnerabilityModel(vulnerability)

        // 将漏洞模型对象保存到数据库中
        result := s.db.Create(&m)
        // 检查保存过程中是否出现错误
        if result.Error != nil {
            return result.Error
        }

        // 检查保存的行数是否为1，如果不是则返回错误
        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
        }
    }
    return nil
}

// GetVulnerabilityMetadata 根据给定的漏洞 ID 和记录来源，检索漏洞的元数据。
func (s *store) GetVulnerabilityMetadata(id, namespace string) (*v5.VulnerabilityMetadata, error) {
    var models []model.VulnerabilityMetadataModel

    // 根据漏洞 ID 和记录来源从数据库中检索元数据
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, Namespace: namespace}).Find(&models)
    // 检查检索过程中是否出现错误
    if result.Error != nil {
        return nil, result.Error
    }

    switch {
    // 如果找到多个相同 ID 和记录来源的元数据，则返回错误
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple metadatas for single ID=%q Namespace=%q", id, namespace)
    // 如果找到一个符合条件的元数据，则返回该元数据
    case len(models) == 1:
        metadata, err := models[0].Inflate()
        if err != nil {
            return nil, err
        }

        return &metadata, nil
    }

    return nil, nil
}

// AddVulnerabilityMetadata 将一个或多个漏洞元数据模型保存到 sqlite 数据库中。
//
//nolint:gocognit
func (s *store) AddVulnerabilityMetadata(metadata ...v5.VulnerabilityMetadata) error {
    // 该函数暂时没有实现，直接返回 nil
    return nil
}

// GetVulnerabilityMatchExclusion 根据漏洞标识符检索一个或多个漏洞匹配排除记录。
func (s *store) GetVulnerabilityMatchExclusion(id string) ([]v5.VulnerabilityMatchExclusion, error) {
    var models []model.VulnerabilityMatchExclusionModel

    // 根据漏洞标识符从数据库中检索匹配排除记录
    result := s.db.Where("id = ?", id).Find(&models)

    var exclusions []v5.VulnerabilityMatchExclusion
    # 遍历 models 列表中的每个元素，使用 _ 作为索引，m 作为元素值
    for _, m := range models {
        # 调用 m 对象的 Inflate 方法，获取返回的 exclusion 和 err
        exclusion, err := m.Inflate()
        # 如果 err 不为空，返回 nil 和 err
        if err != nil {
            return nil, err
        }

        # 如果 exclusion 不为空，将其添加到 exclusions 列表中
        if exclusion != nil {
            exclusions = append(exclusions, *exclusion)
        }
    }

    # 返回 exclusions 列表和 result 对象的 Error 属性
    return exclusions, result.Error
// AddVulnerabilityMatchExclusion 将一个或多个漏洞匹配排除记录保存到 sqlite3 存储中
func (s *store) AddVulnerabilityMatchExclusion(exclusions ...v5.VulnerabilityMatchExclusion) error {
    // 遍历排除记录列表
    for _, exclusion := range exclusions {
        // 创建漏洞匹配排除模型
        m := model.NewVulnerabilityMatchExclusionModel(exclusion)

        // 将模型保存到数据库
        result := s.db.Create(&m)
        // 检查保存过程中是否出现错误
        if result.Error != nil {
            return result.Error
        }

        // 检查受影响的行数是否为1
        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability match exclusion (%d rows affected)", result.RowsAffected)
        }
    }

    return nil
}

// Close 关闭数据库连接
func (s *store) Close() {
    // 执行数据库的 VACUUM 操作
    s.db.Exec("VACUUM;")

    // 获取底层的 sql.DB 对象
    sqlDB, err := s.db.DB()
    if err != nil {
        // 如果出现错误，关闭数据库连接
        _ = sqlDB.Close()
    }
}

// GetAllVulnerabilities 获取数据库中的所有漏洞
func (s *store) GetAllVulnerabilities() (*[]v5.Vulnerability, error) {
    // 创建漏洞模型列表
    var models []model.VulnerabilityModel
    // 从数据库中获取所有漏洞模型
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
    }
    // 创建漏洞列表
    vulns := make([]v5.Vulnerability, len(models))
    // 遍历漏洞模型列表，填充漏洞列表
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
func (s *store) GetAllVulnerabilityMetadata() (*[]v5.VulnerabilityMetadata, error) {
    // 创建漏洞元数据模型列表
    var models []model.VulnerabilityMetadataModel
    // 从数据库中获取所有漏洞元数据模型
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
    }
    // 创建漏洞元数据列表
    metadata := make([]v5.VulnerabilityMetadata, len(models))
    // 遍历漏洞元数据模型列表，填充漏洞元数据列表
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
func (s *store) DiffStore(targetStore v5.StoreReader) (*[]v5.Diff, error) {
    // 创建用于跟踪 diff 过程的 7 个阶段的进度条
    rowsProgress, diffItems, stager := trackDiff(7)

    // 设置当前阶段为“读取目标漏洞”
    stager.Current = "reading target vulnerabilities"
    // 从目标存储中获取所有漏洞数据
    targetVulns, err := targetStore.GetAllVulnerabilities()
    // 增加进度条
    rowsProgress.Increment()
    // 如果出现错误，返回空和错误信息
    if err != nil {
        return nil, err
    }

    // 设置当前阶段为“读取基础漏洞”
    stager.Current = "reading base vulnerabilities"
    // 从当前存储中获取所有漏洞数据
    baseVulns, err := s.GetAllVulnerabilities()
    // 增加进度条
    rowsProgress.Increment()
    // 如果出现错误，返回空和错误信息
    if err != nil {
        return nil, err
    }

    // 设置当前阶段为“准备中”
    stager.Current = "preparing"
    // 构建基础漏洞数据的漏洞包映射
    baseVulnPkgMap := buildVulnerabilityPkgsMap(baseVulns)
    // 构建目标漏洞数据的漏洞包映射
    targetVulnPkgMap := buildVulnerabilityPkgsMap(targetVulns)

    // 设置当前阶段为“比较漏洞”
    stager.Current = "comparing vulnerabilities"
    // 比较基础漏洞和目标漏洞，生成所有差异的映射
    allDiffsMap := diffVulnerabilities(baseVulns, targetVulns, baseVulnPkgMap, targetVulnPkgMap, diffItems)

    // 设置当前阶段为“读取基础元数据”
    stager.Current = "reading base metadata"
    // 从当前存储中获取所有漏洞元数据
    baseMetadata, err := s.GetAllVulnerabilityMetadata()
    // 如果出现错误，返回空和错误信息
    if err != nil {
        return nil, err
    }
    // 增加进度条
    rowsProgress.Increment()

    // 设置当前阶段为“读取目标元数据”
    stager.Current = "reading target metadata"
    // 从目标存储中获取所有漏洞元数据
    targetMetadata, err := targetStore.GetAllVulnerabilityMetadata()
    // 如果出现错误，返回空和错误信息
    if err != nil {
        return nil, err
    }
    // 增加进度条
    rowsProgress.Increment()

    // 设置当前阶段为“比较元数据”
    stager.Current = "comparing metadata"
    // 比较基础元数据和目标元数据，生成所有差异的映射
    metaDiffsMap := diffVulnerabilityMetadata(baseMetadata, targetMetadata, baseVulnPkgMap, targetVulnPkgMap, diffItems)
    // 将元数据差异映射合并到所有差异映射中
    for k, diff := range *metaDiffsMap {
        (*allDiffsMap)[k] = diff
    }
    // 将所有差异映射转换为数组
    allDiffs := []v5.Diff{}
    for _, diff := range *allDiffsMap {
        allDiffs = append(allDiffs, *diff)
    }

    // 设置进度条为完成状态
    rowsProgress.SetCompleted()
    // 设置差异项为完成状态
    diffItems.SetCompleted()

    // 返回所有差异数组和空
    return &allDiffs, nil
# 闭合前面的函数定义
```