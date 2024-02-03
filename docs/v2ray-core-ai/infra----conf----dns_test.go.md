# `v2ray-core\infra\conf\dns_test.go`

```go
package conf_test

import (
    "encoding/json"  // 导入 JSON 编解码包
    "os"  // 导入操作系统相关包
    "path/filepath"  // 导入文件路径相关包
    "testing"  // 导入测试包

    "github.com/golang/protobuf/proto"  // 导入 Protobuf 包
    "v2ray.com/core/app/dns"  // 导入 DNS 应用包
    "v2ray.com/core/app/router"  // 导入路由应用包
    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/net"  // 导入网络相关包
    "v2ray.com/core/common/platform"  // 导入平台相关包
    "v2ray.com/core/common/platform/filesystem"  // 导入文件系统相关包
    . "v2ray.com/core/infra/conf"  // 导入配置包
)

func init() {
    wd, err := os.Getwd()  // 获取当前工作目录
    common.Must(err)  // 如果有错误则中断程序

    if _, err := os.Stat(platform.GetAssetLocation("geoip.dat")); err != nil && os.IsNotExist(err) {
        common.Must(filesystem.CopyFile(platform.GetAssetLocation("geoip.dat"), filepath.Join(wd, "..", "..", "release", "config", "geoip.dat")))  // 复制文件
    }

    geositeFilePath := filepath.Join(wd, "geosite.dat")  // 获取地理位置文件路径
    os.Setenv("v2ray.location.asset", wd)  // 设置环境变量
    geositeFile, err := os.OpenFile(geositeFilePath, os.O_CREATE|os.O_WRONLY, 0600)  // 打开或创建文件
    common.Must(err)  // 如果有错误则中断程序
    defer geositeFile.Close()  // 延迟关闭文件

    list := &router.GeoSiteList{  // 创建路由地理位置列表
        Entry: []*router.GeoSite{  // 创建路由地理位置条目
            {
                CountryCode: "TEST",  // 设置国家代码
                Domain: []*router.Domain{  // 创建域名列表
                    {Type: router.Domain_Full, Value: "example.com"},  // 设置域名类型和值
                },
            },
        },
    }

    listBytes, err := proto.Marshal(list)  // 序列化路由地理位置列表
    common.Must(err)  // 如果有错误则中断程序
    common.Must2(geositeFile.Write(listBytes))  // 写入序列化后的数据到文件
}

func TestDnsConfigParsing(t *testing.T) {
    geositePath := platform.GetAssetLocation("geosite.dat")  // 获取地理位置文件路径
    defer func() {
        os.Remove(geositePath)  // 延迟删除文件
        os.Unsetenv("v2ray.location.asset")  // 延迟取消环境变量设置
    }()

    parserCreator := func() func(string) (proto.Message, error) {  // 创建解析器
        return func(s string) (proto.Message, error) {  // 返回解析函数
            config := new(DnsConfig)  // 创建 DNS 配置对象
            if err := json.Unmarshal([]byte(s), config); err != nil {  // 解析 JSON 数据到配置对象
                return nil, err  // 如果有错误则返回错误
            }
            return config.Build()  // 构建配置对象
        }
    }

    })
}
```