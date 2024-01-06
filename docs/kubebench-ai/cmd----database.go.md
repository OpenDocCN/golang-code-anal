# `kubebench-aquasecurity\cmd\database.go`

```
package cmd

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"os" // 导入 os 包，用于操作系统功能
	"time" // 导入 time 包，用于时间相关功能

	"github.com/golang/glog" // 导入 glog 包，用于日志记录
	"github.com/jinzhu/gorm" // 导入 gorm 包，用于数据库操作
	_ "github.com/jinzhu/gorm/dialects/postgres" // 导入 postgres 数据库驱动，使用下划线表示只使用其初始化功能
	"github.com/spf13/viper" // 导入 viper 包，用于读取配置信息
)

func savePgsql(jsonInfo string) {
	// 创建环境变量映射
	envVars := map[string]string{
		"PGSQL_HOST":     viper.GetString("PGSQL_HOST"), // 从配置文件中获取 PGSQL_HOST 环境变量的值
		"PGSQL_USER":     viper.GetString("PGSQL_USER"), // 从配置文件中获取 PGSQL_USER 环境变量的值
		"PGSQL_DBNAME":   viper.GetString("PGSQL_DBNAME"), // 从配置文件中获取 PGSQL_DBNAME 环境变量的值
		"PGSQL_SSLMODE":  viper.GetString("PGSQL_SSLMODE"), // 从配置文件中获取 PGSQL_SSLMODE 环境变量的值
		"PGSQL_PASSWORD": viper.GetString("PGSQL_PASSWORD"), // 从配置文件中获取 PGSQL_PASSWORD 环境变量的值
	}

	// 遍历环境变量，检查是否有空值，如果有则报错
	for k, v := range envVars {
		if v == "" {
			exitWithError(fmt.Errorf("environment variable %s is missing", envVarsPrefix+"_"+k))
		}
	}

	// 根据环境变量拼接数据库连接信息
	connInfo := fmt.Sprintf("host=%s user=%s dbname=%s sslmode=%s password=%s",
		envVars["PGSQL_HOST"],
		envVars["PGSQL_USER"],
		envVars["PGSQL_DBNAME"],
		envVars["PGSQL_SSLMODE"],
		envVars["PGSQL_PASSWORD"],
	)

	// 获取主机名
	hostname, err := os.Hostname()
	if err != nil {
		exitWithError(fmt.Errorf("received error looking up hostname: %s", err))
	}
	// 获取当前时间戳
	timestamp := time.Now()

	// 定义 ScanResult 结构体，包含扫描主机、扫描时间和扫描信息
	type ScanResult struct {
		gorm.Model
		ScanHost string    `gorm:"type:varchar(63) not null"` // 定义扫描主机字段，类型为 varchar(63)，不能为空
		ScanTime time.Time `gorm:"not null"`                  // 定义扫描时间字段，不能为空
		ScanInfo string    `gorm:"type:jsonb not null"`        // 定义扫描信息字段，类型为 jsonb，不能为空
	}

	// 连接到 PostgreSQL 数据库
	db, err := gorm.Open("postgres", connInfo)
	if err != nil {
		// 如果连接出错，打印错误信息并退出程序
		exitWithError(fmt.Errorf("received error connecting to database: %s", err))
	}
	// 延迟关闭数据库连接
	defer db.Close()

	// 开启调试模式，自动迁移 ScanResult 结构体对应的表
	db.Debug().AutoMigrate(&ScanResult{})
	// 将扫描结果存储到数据库中
	db.Save(&ScanResult{ScanHost: hostname, ScanTime: timestamp, ScanInfo: jsonInfo})
	// 打印成功存储结果的信息
	glog.V(2).Info(fmt.Sprintf("successfully stored result to: %s", envVars["PGSQL_HOST"]))
抱歉，我无法为您提供代码注释，因为您没有提供任何代码。如果您有任何需要帮助的代码，请随时告诉我。我会尽力帮助您。
```