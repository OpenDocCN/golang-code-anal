# `kubebench-aquasecurity\cmd\database.go`

```go
package cmd

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "os" // 导入 os 包，用于操作系统功能
    "time" // 导入 time 包，用于时间相关功能

    "github.com/golang/glog" // 导入 glog 包，用于日志记录
    "github.com/jinzhu/gorm" // 导入 gorm 包，用于数据库操作
    _ "github.com/jinzhu/gorm/dialects/postgres" // 导入 postgres 数据库驱动，使用空白标识符表示只使用其初始化函数
    "github.com/spf13/viper" // 导入 viper 包，用于读取配置信息
)

func savePgsql(jsonInfo string) {
    // 定义环境变量映射
    envVars := map[string]string{
        "PGSQL_HOST":     viper.GetString("PGSQL_HOST"), // 从配置文件中获取 PGSQL_HOST 环境变量值
        "PGSQL_USER":     viper.GetString("PGSQL_USER"), // 从配置文件中获取 PGSQL_USER 环境变量值
        "PGSQL_DBNAME":   viper.GetString("PGSQL_DBNAME"), // 从配置文件中获取 PGSQL_DBNAME 环境变量值
        "PGSQL_SSLMODE":  viper.GetString("PGSQL_SSLMODE"), // 从配置文件中获取 PGSQL_SSLMODE 环境变量值
        "PGSQL_PASSWORD": viper.GetString("PGSQL_PASSWORD"), // 从配置文件中获取 PGSQL_PASSWORD 环境变量值
    }

    // 遍历环境变量映射
    for k, v := range envVars {
        // 如果环境变量值为空
        if v == "" {
            // 输出错误信息并退出程序
            exitWithError(fmt.Errorf("environment variable %s is missing", envVarsPrefix+"_"+k))
        }
    }

    // 格式化连接信息
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
        // 输出错误信息并退出程序
        exitWithError(fmt.Errorf("received error looking up hostname: %s", err))
    }

    // 获取当前时间戳
    timestamp := time.Now()

    // 定义数据库表结构
    type ScanResult struct {
        gorm.Model // gorm 模型
        ScanHost string    `gorm:"type:varchar(63) not null"` // 定义字段类型和约束
        ScanTime time.Time `gorm:"not null"` // 定义字段类型和约束
        ScanInfo string    `gorm:"type:jsonb not null"` // 定义字段类型和约束
    }

    // 连接数据库
    db, err := gorm.Open("postgres", connInfo)
    if err != nil {
        // 输出错误信息并退出程序
        exitWithError(fmt.Errorf("received error connecting to database: %s", err))
    }
    defer db.Close() // 延迟关闭数据库连接

    // 调试模式下自动迁移表结构
    db.Debug().AutoMigrate(&ScanResult{})
    // 保存扫描结果到数据库
    db.Save(&ScanResult{ScanHost: hostname, ScanTime: timestamp, ScanInfo: jsonInfo})
    // 记录日志信息
    glog.V(2).Info(fmt.Sprintf("successfully stored result to: %s", envVars["PGSQL_HOST"]))
}
```