# `trojan-go\common\geodata\loader.go`

```go
package geodata

import (
    "runtime"

    v2router "github.com/v2fly/v2ray-core/v4/app/router"
)

type geodataCache struct {
    geoipCache
    geositeCache
}

// NewGeodataLoader 返回一个 GeodataLoader 接口的实现
func NewGeodataLoader() GeodataLoader {
    // 创建并返回一个 geodataCache 对象
    return &geodataCache{
        make(map[string]*v2router.GeoIP),  // 初始化一个空的 geoipCache
        make(map[string]*v2router.GeoSite),  // 初始化一个空的 geositeCache
    }
}

// LoadIP 从指定文件中加载 IP 地址数据
func (g *geodataCache) LoadIP(filename, country string) ([]*v2router.CIDR, error) {
    // 从 geoipCache 中解析指定国家的 IP 地址数据
    geoip, err := g.geoipCache.Unmarshal(filename, country)
    if err != nil {
        return nil, err
    }
    // 手动触发垃圾回收
    runtime.GC()
    return geoip.Cidr, nil
}

// LoadSite 从指定文件中加载网站数据
func (g *geodataCache) LoadSite(filename, list string) ([]*v2router.Domain, error) {
    // 从 geositeCache 中解析指定列表的网站数据
    geosite, err := g.geositeCache.Unmarshal(filename, list)
    if err != nil {
        return nil, err
    }
    // 手动触发垃圾回收
    runtime.GC()
    return geosite.Domain, nil
}

// LoadGeoIP 加载指定国家的 IP 地址数据
func (g *geodataCache) LoadGeoIP(country string) ([]*v2router.CIDR, error) {
    // 调用 LoadIP 方法加载指定国家的 IP 地址数据
    return g.LoadIP("geoip.dat", country)
}

// LoadGeoSite 加载指定列表的网站数据
func (g *geodataCache) LoadGeoSite(list string) ([]*v2router.Domain, error) {
    // 调用 LoadSite 方法加载指定列表的网站数据
    return g.LoadSite("geosite.dat", list)
}
```