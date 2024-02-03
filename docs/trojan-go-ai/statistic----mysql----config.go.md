# `trojan-go\statistic\mysql\config.go`

```go
package mysql

import (
    "github.com/p4gefau1t/trojan-go/config"
)

type MySQLConfig struct {
    Enabled    bool   `json:"enabled" yaml:"enabled"`  // MySQL 是否启用
    ServerHost string `json:"server_addr" yaml:"server-addr"`  // MySQL 服务器地址
    ServerPort int    `json:"server_port" yaml:"server-port"`  // MySQL 服务器端口
    Database   string `json:"database" yaml:"database"`  // 数据库名称
    Username   string `json:"username" yaml:"username"`  // 用户名
    Password   string `json:"password" yaml:"password"`  // 密码
    CheckRate  int    `json:"check_rate" yaml:"check-rate"`  // 检查频率
}

type Config struct {
    MySQL MySQLConfig `json:"mysql" yaml:"mysql"`  // MySQL 配置
}

func init() {
    // 注册配置创建函数
    config.RegisterConfigCreator(Name, func() interface{} {
        // 返回配置对象
        return &Config{
            MySQL: MySQLConfig{
                ServerPort: 3306,  // 默认 MySQL 服务器端口
                CheckRate:  30,    // 默认检查频率
            },
        }
    })
}
```