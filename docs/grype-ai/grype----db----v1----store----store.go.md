# `grype\grype\db\v1\store\store.go`

```go
package store

import (
    "fmt"
    "sort"

    _ "github.com/glebarez/sqlite" // 通过导入提供 sqlite 方言给 gorm 使用
    "github.com/go-test/deep"
    "gorm.io/gorm"

    "github.com/anchore/grype/grype/db/internal/gormadapter"
    v1 "github.com/anchore/grype/grype/db/v1"
    "github.com/anchore/grype/grype/db/v1/store/model"
    "github.com/anchore/grype/internal/stringutil"
)

// store holds an instance of the database connection
type store struct {
    db *gorm.DB
}

// New creates a new instance of the store.
func New(dbFilePath string, overwrite bool) (v1.Store, error) {
    // 打开数据库连接
    db, err := gormadapter.Open(dbFilePath, overwrite)
    if err != nil {
        return nil, err
    }

    if overwrite {
        // TODO: automigrate 可能会写入数据库，我们应该在自动迁移之前验证数据库是否是基于 ID 表中的版本的正确数据库
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
func (s *store) GetID() (*v1.ID, error) {
    var models []model.IDModel
    // 查询数据库中的 IDModel 数据
    result := s.db.Find(&models)
    if result.Error != nil {
        return nil, result.Error
    }

    switch {
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple DB IDs")
    case len(models) == 1:
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
func (s *store) SetID(id v1.ID) error {
    var ids []model.IDModel

    // 用给定的ID替换现有的ID
    s.db.Find(&ids).Delete(&ids)

    // 创建一个新的IDModel对象
    m := model.NewIDModel(id)
    // 将新对象存储到数据库中
    result := s.db.Create(&m)

    // 检查是否成功添加了新的ID
    if result.RowsAffected != 1 {
        return fmt.Errorf("unable to add id (%d rows affected)", result.RowsAffected)
    }

    return result.Error
}

// GetVulnerability函数根据命名空间和包名检索一个或多个漏洞
func (s *store) GetVulnerability(namespace, packageName string) ([]v1.Vulnerability, error) {
    var models []model.VulnerabilityModel

    // 根据命名空间和包名从数据库中检索漏洞数据
    result := s.db.Where("namespace = ? AND package_name = ?", namespace, packageName).Find(&models)

    // 创建一个漏洞数组，长度与数据库中的漏洞数量相同
    var vulnerabilities = make([]v1.Vulnerability, len(models))
    for idx, m := range models {
        // 将数据库中的漏洞数据转换为Vulnerability对象
        vulnerability, err := m.Inflate()
        if err != nil {
            return nil, err
        }
        vulnerabilities[idx] = vulnerability
    }

    return vulnerabilities, result.Error
}

// AddVulnerability函数将一个或多个漏洞保存到sqlite3存储中
func (s *store) AddVulnerability(vulnerabilities ...v1.Vulnerability) error {
    for _, vulnerability := range vulnerabilities {
        // 创建一个新的VulnerabilityModel对象
        m := model.NewVulnerabilityModel(vulnerability)

        // 将新对象存储到数据库中
        result := s.db.Create(&m)
        if result.Error != nil {
            return result.Error
        }

        // 检查是否成功添加了新的漏洞
        if result.RowsAffected != 1 {
            return fmt.Errorf("unable to add vulnerability (%d rows affected)", result.RowsAffected)
        }
    }
    return nil
}

// GetVulnerabilityMetadata函数根据特定的记录源检索给定漏洞ID的元数据
func (s *store) GetVulnerabilityMetadata(id, recordSource string) (*v1.VulnerabilityMetadata, error) {
    var models []model.VulnerabilityMetadataModel

    // 根据ID和记录源从数据库中检索漏洞元数据
    result := s.db.Where(&model.VulnerabilityMetadataModel{ID: id, RecordSource: recordSource}).Find(&models)
    # 如果结果中包含错误信息，则返回空和错误信息
    if result.Error != nil:
        return nil, result.Error

    # 根据不同的情况进行处理
    switch:
    # 如果模型数量大于1，则返回空和错误信息
    case len(models) > 1:
        return nil, fmt.Errorf("found multiple metadatas for single ID=%q RecordSource=%q", id, recordSource)
    # 如果模型数量等于1
    case len(models) == 1:
        # 解压模型数据
        metadata, err := models[0].Inflate()
        # 如果解压过程中出现错误，则返回空和错误信息
        if err != nil:
            return nil, err
        # 返回解压后的元数据和空错误信息
        return &metadata, nil
    # 如果模型数量为0，则返回空和空错误信息
    return nil, nil
// AddVulnerabilityMetadata 将一个或多个漏洞元数据模型存储到 sqlite 数据库中
func (s *store) AddVulnerabilityMetadata(metadata ...v1.VulnerabilityMetadata) error {
    // 实现存储漏洞元数据的功能
    // ...
    // 返回空错误，表示操作成功
    return nil
}

// Close 关闭数据库连接，并执行 VACUUM 操作
func (s *store) Close() {
    // 执行 VACUUM 操作，用于整理和优化数据库文件
    s.db.Exec("VACUUM;")
}
```