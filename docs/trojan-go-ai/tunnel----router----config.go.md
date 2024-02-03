# `trojan-go\tunnel\router\config.go`

```go
package router

import (
    "github.com/p4gefau1t/trojan-go/common"  // 导入common包
    "github.com/p4gefau1t/trojan-go/config"  // 导入config包
)

type Config struct {
    Router RouterConfig `json:"router" yaml:"router"`  // 定义Config结构体，包含RouterConfig字段
}

type RouterConfig struct {
    Enabled         bool     `json:"enabled" yaml:"enabled"`  // 定义RouterConfig结构体，包含Enabled字段
    Bypass          []string `json:"bypass" yaml:"bypass"`    // 定义Bypass字段为字符串数组
    Proxy           []string `json:"proxy" yaml:"proxy"`      // 定义Proxy字段为字符串数组
    Block           []string `json:"block" yaml:"block"`      // 定义Block字段为字符串数组
    DomainStrategy  string   `json:"domain_strategy" yaml:"domain-strategy"`  // 定义DomainStrategy字段为字符串
    DefaultPolicy   string   `json:"default_policy" yaml:"default-policy"`      // 定义DefaultPolicy字段为字符串
    GeoIPFilename   string   `json:"geoip" yaml:"geoip"`      // 定义GeoIPFilename字段为字符串
    GeoSiteFilename string   `json:"geosite" yaml:"geosite"`  // 定义GeoSiteFilename字段为字符串
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {  // 在config包中注册ConfigCreator
        cfg := &Config{  // 创建Config结构体实例
            Router: RouterConfig{  // 初始化Router字段
                DefaultPolicy:   "proxy",  // 设置DefaultPolicy字段的默认值为"proxy"
                DomainStrategy:  "as_is",  // 设置DomainStrategy字段的默认值为"as_is"
                GeoIPFilename:   common.GetAssetLocation("geoip.dat"),  // 设置GeoIPFilename字段的默认值为"geoip.dat"的位置
                GeoSiteFilename: common.GetAssetLocation("geosite.dat"),  // 设置GeoSiteFilename字段的默认值为"geosite.dat"的位置
            },
        }
        return cfg  // 返回Config结构体实例
    })
}
```