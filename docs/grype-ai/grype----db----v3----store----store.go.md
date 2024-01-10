# `grype\grype\db\v3\store\store.go`

```
package store

import (
    "fmt"
    "sort"

    _ "github.com/glebarez/sqlite" // 通过导入提供 sqlite 方言给 gorm 使用
    "github.com/go-test/deep"
    "gorm.io/gorm"

    "github.com/anchore/grype/grype/db/internal/gormadapter"
    v3 "github.com/anchore/grype/grype/db/v3"
    "github.com/anchore/grype/grype/db/v3/store/model"
    "github.com/anchore/grype/internal/stringutil"
)

// store holds an instance of the database connection
type store struct {
    db *gorm.DB
}

// New creates a new instance of the store.
func New(dbFilePath string, overwrite bool) (v3.Store, error) {
    // 打开数据库连接
    db, err := gormadapter.Open(dbFilePath, overwrite)
    if err != nil {
        return nil, err
    }

    if overwrite {
        // 如果需要覆盖数据库，则进行自动迁移
        // TODO: automigrate could write to the database,
        //  we should be validating the database is the correct database based on the version in the ID table before
        //  automigrating
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

    return &store{
        db: db,
    }, nil
}

// GetID fetches the metadata about the databases schema version and build time.
func (s *store) GetID() (*v3.ID, error) {
    var models []model.IDModel
    // 查询数据库中的 IDModel 表
    result := s.db.Find(&models)
    if result.Error != nil {
        return nil, result.Error
    }

    switch {
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple DB IDs")
    case len(models) == 1:
        // 解析 IDModel 数据为 ID 结构
        id, err := models[0].Inflate()
        if err != nil {
            return nil, err
        }
        return &id, nil
    }
}
    # 返回两个空值
    return nil, nil
// SetID函数用于存储数据库的模式版本和构建时间
func (s *store) SetID(id v3.ID) error {
    var ids []model.IDModel

    // 用给定的ID替换现有的ID
    s.db.Find(&ids).Delete(&ids)

    // 创建一个新的IDModel对象
    m := model.NewIDModel(id)
    // 将新对象插入数据库
    result := s.db.Create(&m)

    // 检查是否成功插入，如果不成功则返回错误
    if result.RowsAffected != 1 {
        return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
    }

    return result.Error
}

// GetVulnerability函数根据命名空间和包名检索一个或多个漏洞
func (s *store) GetVulnerability(namespace, packageName string) ([]v3.Vulnerability, error) {
    var models []model.VulnerabilityModel

    // 根据命名空间和包名从数据库中检索漏洞
    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

    // 创建一个漏洞切片，用于存储检索到的漏洞
    var vulnerabilities = make([]v3.Vulnerability, len(models))
    for idx, m := range models {
        // 将数据库中的漏洞模型转换为漏洞对象
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
    }

    return vulnerabilities, result.Error
}

// AddVulnerability函数将一个或多个漏洞保存到sqlite3存储中
func (s *store) AddVulnerability(vulnerabilities ...v3.Vulnerability) error {
    for _, vulnerability := range vulnerabilities {
        // 创建一个新的VulnerabilityModel对象
        m := model.NewVulnerabilityModel(vulnerability)

        // 将新对象插入数据库
        result := s.db.Create(&m)
        // 检查是否成功插入，如果不成功则返回错误
        if result.Error != nil {
            return result.Error
        }

        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
        }
    }
    return nil
}

// GetVulnerabilityMetadata函数根据特定记录源检索给定漏洞ID的元数据
func (s *store) GetVulnerabilityMetadata(id, namespace string) (*v3.VulnerabilityMetadata, error) {
    var models []model.VulnerabilityMetadataModel

    // 根据ID和命名空间从数据库中检索漏洞元数据
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, Namespace: namespace}).Find(&models)
    # 如果结果中包含错误信息，则返回空和错误信息
    if result.Error != nil:
        return nil, result.Error

    # 根据不同的情况进行处理
    switch:
        # 如果模型数量大于1，则返回空和错误信息
        case len(models) > 1:
            return nil, fmt.Errorf("found multiple metadatas for single ID=%q Namespace=%q", id, namespace)
        # 如果模型数量等于1，则解压模型并返回
        case len(models) == 1:
            metadata, err := models[0].Inflate()
            if err != nil:
                return nil, err
            return &metadata, nil
    # 如果没有符合条件的情况，则返回空和空
    return nil, nil
// AddVulnerabilityMetadata 将一个或多个漏洞元数据模型存储到 sqlite 数据库中
//
//nolint:gocognit
func (s *store) AddVulnerabilityMetadata(metadata ...v3.VulnerabilityMetadata) error {
    // 添加漏洞元数据模型到数据库
    return nil
}

// GetAllVulnerabilities 获取数据库中的所有漏洞
func (s *store) GetAllVulnerabilities() (*[]v3.Vulnerability, error) {
    var models []model.VulnerabilityModel
    // 从数据库中获取所有漏洞模型
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
    }
    // 创建漏洞切片
    vulns := make([]v3.Vulnerability, len(models))
    // 遍历漏洞模型，解压缩并存储到漏洞切片中
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
func (s *store) GetAllVulnerabilityMetadata() (*[]v3.VulnerabilityMetadata, error) {
    var models []model.VulnerabilityMetadataModel
    // 从数据库中获取所有漏洞元数据模型
    if result := s.db.Find(&models); result.Error != nil {
        return nil, result.Error
    }
    // 创建漏洞元数据切片
    metadata := make([]v3.VulnerabilityMetadata, len(models))
    // 遍历漏洞元数据模型，解压缩并存储到漏洞元数据切片中
    for idx, m := range models {
        data, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        metadata[idx] = data
    }
    return &metadata, nil
}

// DiffStore 在当前 SQL 数据库和给定存储之间创建差异
func (s *store) DiffStore(targetStore v3.StoreReader) (*[]v3.Diff, error) {
    // 7个阶段，每个阶段代表差异过程的一步（阶段）
    rowsProgress, diffItems, stager := trackDiff(7)

    stager.Current = "reading target vulnerabilities"
    // 读取目标存储中的所有漏洞
    targetVulns, err := targetStore.GetAllVulnerabilities()
    rowsProgress.Increment()
    if err != nil {
        return nil, err
    }

    stager.Current = "reading base vulnerabilities"
    // 读取当前存储中的所有漏洞
    baseVulns, err := s.GetAllVulnerabilities()
    rowsProgress.Increment()
    if err != nil {
        return nil, err
    }

    stager.Current = "preparing"
}
    # 构建基础漏洞包映射
    baseVulnPkgMap := buildVulnerabilityPkgsMap(baseVulns)
    # 构建目标漏洞包映射
    targetVulnPkgMap := buildVulnerabilityPkgsMap(targetVulns)
    
    # 设置当前进度为“比较漏洞”
    stager.Current = "comparing vulnerabilities"
    # 比较基础漏洞和目标漏洞，生成所有差异的映射
    allDiffsMap := diffVulnerabilities(baseVulns, targetVulns, baseVulnPkgMap, targetVulnPkgMap, diffItems)
    
    # 设置当前进度为“读取基础元数据”
    stager.Current = "reading base metadata"
    # 读取所有基础漏洞的元数据
    baseMetadata, err := s.GetAllVulnerabilityMetadata()
    # 如果出现错误，则返回空和错误信息
    if err != nil {
        return nil, err
    }
    # 增加行进度
    rowsProgress.Increment()
    
    # 设置当前进度为“读取目标元数据”
    stager.Current = "reading target metadata"
    # 读取所有目标漏洞的元数据
    targetMetadata, err := targetStore.GetAllVulnerabilityMetadata()
    # 如果出现错误，则返回空和错误信息
    if err != nil {
        return nil, err
    }
    # 增加行进度
    rowsProgress.Increment()
    
    # 设置当前进度为“比较元数据”
    stager.Current = "comparing metadata"
    # 比较基础元数据和目标元数据，生成所有差异的映射
    metaDiffsMap := diffVulnerabilityMetadata(baseMetadata, targetMetadata, baseVulnPkgMap, targetVulnPkgMap, diffItems)
    # 将元数据差异映射中的每个差异添加到所有差异映射中
    for k, diff := range *metaDiffsMap {
        (*allDiffsMap)[k] = diff
    }
    # 创建所有差异的切片
    allDiffs := []v3.Diff{}
    # 将所有差异映射中的每个差异添加到所有差异切片中
    for _, diff := range *allDiffsMap {
        allDiffs = append(allDiffs, *diff)
    }
    
    # 设置行进度为已完成
    rowsProgress.SetCompleted()
    # 设置差异项为已完成
    diffItems.SetCompleted()
    
    # 返回所有差异切片和空错误信息
    return &allDiffs, nil
# 关闭存储对象的方法
func (s *store) Close() {
    # 执行数据库的VACUUM操作，用于优化数据库性能
    s.db.Exec("VACUUM;")
}
```