# `kubo\config\migration_test.go`

```go
package config

import (
    "encoding/json"  // 导入 JSON 编解码包
    "testing"  // 导入测试包
)

func TestMigrationDecode(t *testing.T) {
    str := `
        {
            "DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
            "Keep": "cache"
        }
    `

    var cfg Migration  // 声明一个 Migration 类型的变量 cfg
    if err := json.Unmarshal([]byte(str), &cfg); err != nil {  // 使用 JSON 解码将字符串解析为 cfg 变量
        t.Errorf("failed while unmarshalling migration struct: %s", err)  // 如果解析失败则输出错误信息
    }

    if len(cfg.DownloadSources) != 3 {  // 检查 DownloadSources 的长度是否为 3
        t.Fatal("wrong number of DownloadSources")  // 如果长度不为 3 则输出致命错误
    }
    expect := []string{"IPFS", "HTTP", "127.0.0.1"}  // 期望的 DownloadSources 数组
    for i := range expect {  // 遍历期望的数组
        if cfg.DownloadSources[i] != expect[i] {  // 检查实际的 DownloadSources 数组与期望的数组是否一致
            t.Errorf("wrong DownloadSource at %d", i)  // 如果不一致则输出错误信息
        }
    }

    if cfg.Keep != "cache" {  // 检查 Keep 字段的值是否为 "cache"
        t.Error("wrong value for Keep")  // 如果不是则输出错误信息
    }
}
```