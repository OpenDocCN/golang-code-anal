# `grype\cmd\grype\cli\options\database.go`

```go
package options

import (
    "path"
    "time"

    "github.com/adrg/xdg"

    "github.com/anchore/clio"
    "github.com/anchore/grype/grype/db"
    "github.com/anchore/grype/internal"
)

type Database struct {
    Dir                   string        `yaml:"cache-dir" json:"cache-dir" mapstructure:"cache-dir"`  // 缓存目录路径
    UpdateURL             string        `yaml:"update-url" json:"update-url" mapstructure:"update-url"`  // 更新 URL
    CACert                string        `yaml:"ca-cert" json:"ca-cert" mapstructure:"ca-cert"`  // CA 证书路径
    AutoUpdate            bool          `yaml:"auto-update" json:"auto-update" mapstructure:"auto-update"`  // 是否自动更新
    ValidateByHashOnStart bool          `yaml:"validate-by-hash-on-start" json:"validate-by-hash-on-start" mapstructure:"validate-by-hash-on-start"`  // 启动时是否通过哈希验证
    ValidateAge           bool          `yaml:"validate-age" json:"validate-age" mapstructure:"validate-age"`  // 是否验证年龄
    MaxAllowedBuiltAge    time.Duration `yaml:"max-allowed-built-age" json:"max-allowed-built-age" mapstructure:"max-allowed-built-age"`  // 允许的最大构建年龄
}

func DefaultDatabase(id clio.Identification) Database {
    return Database{
        Dir:         path.Join(xdg.CacheHome, id.Name, "db"),  // 设置缓存目录路径
        UpdateURL:   internal.DBUpdateURL,  // 设置更新 URL
        AutoUpdate:  true,  // 默认开启自动更新
        ValidateAge: true,  // 默认开启验证年龄
        // After this period (5 days) the db data is considered stale
        MaxAllowedBuiltAge: time.Hour * 24 * 5,  // 设置允许的最大构建年龄为 5 天
    }
}

func (cfg Database) ToCuratorConfig() db.Config {
    return db.Config{
        DBRootDir:           cfg.Dir,  // 设置数据库根目录
        ListingURL:          cfg.UpdateURL,  // 设置列表 URL
        CACert:              cfg.CACert,  // 设置 CA 证书路径
        ValidateByHashOnGet: cfg.ValidateByHashOnStart,  // 获取时是否通过哈希验证
        ValidateAge:         cfg.ValidateAge,  // 是否验证年龄
        MaxAllowedBuiltAge:  cfg.MaxAllowedBuiltAge,  // 允许的最大构建年龄
    }
}
```