# `grype\cmd\grype\cli\options\database.go`

```
package options

import (
	"path" // 导入 path 包，用于处理文件路径
	"time" // 导入 time 包，用于处理时间

	"github.com/adrg/xdg" // 导入 xdg 包，用于处理 XDG 目录规范

	"github.com/anchore/clio" // 导入 anchore/clio 包，用于处理命令行输入输出
	"github.com/anchore/grype/grype/db" // 导入 anchore/grype/grype/db 包，用于处理 grype 数据库
	"github.com/anchore/grype/internal" // 导入 anchore/grype/internal 包，用于处理 grype 内部逻辑
)

type Database struct {
	Dir                   string        `yaml:"cache-dir" json:"cache-dir" mapstructure:"cache-dir"` // 数据库目录路径
	UpdateURL             string        `yaml:"update-url" json:"update-url" mapstructure:"update-url"` // 数据库更新 URL
	CACert                string        `yaml:"ca-cert" json:"ca-cert" mapstructure:"ca-cert"` // CA 证书路径
	AutoUpdate            bool          `yaml:"auto-update" json:"auto-update" mapstructure:"auto-update"` // 是否自动更新数据库
	ValidateByHashOnStart bool          `yaml:"validate-by-hash-on-start" json:"validate-by-hash-on-start" mapstructure:"validate-by-hash-on-start"` // 启动时是否通过哈希验证
	ValidateAge           bool          `yaml:"validate-age" json:"validate-age" mapstructure:"validate-age"` // 是否验证数据库的年龄
```
// MaxAllowedBuiltAge表示构建数据的最大允许年龄
MaxAllowedBuiltAge    time.Duration `yaml:"max-allowed-built-age" json:"max-allowed-built-age" mapstructure:"max-allowed-built-age"`
}

// DefaultDatabase返回一个默认的数据库配置
func DefaultDatabase(id clio.Identification) Database {
	return Database{
		Dir:         path.Join(xdg.CacheHome, id.Name, "db"), // 设置数据库目录
		UpdateURL:   internal.DBUpdateURL, // 设置更新URL
		AutoUpdate:  true, // 启用自动更新
		ValidateAge: true, // 启用年龄验证
		// 设置构建数据的最大允许年龄为5天
		MaxAllowedBuiltAge: time.Hour * 24 * 5,
	}
}

// 将数据库配置转换为Curator配置
func (cfg Database) ToCuratorConfig() db.Config {
	return db.Config{
		DBRootDir:           cfg.Dir, // 设置数据库根目录
		ListingURL:          cfg.UpdateURL, // 设置列表URL
		CACert:              cfg.CACert, // 设置CA证书
		ValidateByHashOnGet: cfg.ValidateByHashOnStart, // 在获取时通过哈希验证
		# 将cfg.ValidateAge赋值给ValidateAge
		ValidateAge:         cfg.ValidateAge,
		# 将cfg.MaxAllowedBuiltAge赋值给MaxAllowedBuiltAge
		MaxAllowedBuiltAge:  cfg.MaxAllowedBuiltAge,
	}
}
```