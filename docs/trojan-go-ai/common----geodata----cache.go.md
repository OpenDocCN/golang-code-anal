# `trojan-go\common\geodata\cache.go`

```go
package geodata

import (
    "io/ioutil"
    "strings"

    v2router "github.com/v2fly/v2ray-core/v4/app/router"
    "google.golang.org/protobuf/proto"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/log"
)

// 定义一个名为geoipCache的类型别名，表示字符串到GeoIP对象的映射
type geoipCache map[string]*v2router.GeoIP

// 判断geoipCache中是否存在指定的key
func (g geoipCache) Has(key string) bool {
    return !(g.Get(key) == nil)
}

// 获取geoipCache中指定key对应的值
func (g geoipCache) Get(key string) *v2router.GeoIP {
    if g == nil {
        return nil
    }
    return g[key]
}

// 设置geoipCache中指定key对应的值
func (g geoipCache) Set(key string, value *v2router.GeoIP) {
    if g == nil {
        g = make(map[string]*v2router.GeoIP)
    }
    g[key] = value
}

// 从文件中解析并填充geoipCache
func (g geoipCache) Unmarshal(filename, code string) (*v2router.GeoIP, error) {
    // 获取文件的位置
    asset := common.GetAssetLocation(filename)
    // 构建索引
    idx := strings.ToLower(asset + ":" + code)
    // 如果geoipCache中存在索引，则直接返回对应的GeoIP对象
    if g.Has(idx) {
        log.Debugf("geoip cache HIT: %s -> %s", code, idx)
        return g.Get(idx), nil
    }

    // 解码文件内容
    geoipBytes, err := Decode(asset, code)
    switch err {
    case nil:
        // 反序列化GeoIP对象
        var geoip v2router.GeoIP
        if err := proto.Unmarshal(geoipBytes, &geoip); err != nil {
            return nil, err
        }
        // 将解析后的GeoIP对象存入geoipCache
        g.Set(idx, &geoip)
        return &geoip, nil

    case ErrCodeNotFound:
        return nil, common.NewError("country code " + code + " not found in " + filename)
    }
}
    # 处理四种特定的错误情况
    case ErrFailedToReadBytes, ErrFailedToReadExpectedLenBytes,
        ErrInvalidGeodataFile, ErrInvalidGeodataVarintLength:
        # 记录警告日志，指出无法解码地理位置文件，回退到原始的ReadFile方法
        log.Warnf("failed to decode geoip file: %s, fallback to the original ReadFile method", filename)
        # 读取地理位置文件的内容
        geoipBytes, err = ioutil.ReadFile(asset)
        # 如果读取出错，则返回空和错误
        if err != nil {
            return nil, err
        }
        # 创建地理位置列表对象
        var geoipList v2router.GeoIPList
        # 解码地理位置文件的内容到地理位置列表对象
        if err := proto.Unmarshal(geoipBytes, &geoipList); err != nil {
            return nil, err
        }
        # 遍历地理位置列表中的每个地理位置
        for _, geoip := range geoipList.GetEntry() {
            # 如果国家代码与地理位置的国家代码相同
            if strings.EqualFold(code, geoip.GetCountryCode()) {
                # 设置地理位置到指定索引
                g.Set(idx, geoip)
                # 返回地理位置和空错误
                return geoip, nil
            }
        }

    # 默认情况
    default:
        # 返回空和错误
        return nil, err
    }

    # 返回错误，指出指定国家代码在指定文件中未找到
    return nil, common.NewError("country code " + code + " not found in " + filename)
# 定义了一个名为geositeCache的类型，是一个字符串到*v2router.GeoSite指针的映射
type geositeCache map[string]*v2router.GeoSite

# 判断geositeCache中是否存在指定的key
func (g geositeCache) Has(key string) bool {
    return !(g.Get(key) == nil)
}

# 获取geositeCache中指定key对应的*v2router.GeoSite指针
func (g geositeCache) Get(key string) *v2router.GeoSite {
    if g == nil {
        return nil
    }
    return g[key]
}

# 设置geositeCache中指定key对应的*v2router.GeoSite指针
func (g geositeCache) Set(key string, value *v2router.GeoSite) {
    if g == nil {
        g = make(map[string]*v2router.GeoSite)
    }
    g[key] = value
}

# 从文件中解析出*v2router.GeoSite对象，并存入geositeCache中
func (g geositeCache) Unmarshal(filename, code string) (*v2router.GeoSite, error) {
    # 获取文件的位置
    asset := common.GetAssetLocation(filename)
    # 构建索引
    idx := strings.ToLower(asset + ":" + code)
    # 判断geositeCache中是否存在指定的索引
    if g.Has(idx) {
        log.Debugf("geosite cache HIT: %s -> %s", code, idx)
        return g.Get(idx), nil
    }

    # 解码文件内容
    geositeBytes, err := Decode(asset, code)
    switch err {
    case nil:
        # 解析出*v2router.GeoSite对象
        var geosite v2router.GeoSite
        if err := proto.Unmarshal(geositeBytes, &geosite); err != nil {
            return nil, err
        }
        # 将解析出的对象存入geositeCache中
        g.Set(idx, &geosite)
        return &geosite, nil

    case ErrCodeNotFound:
        return nil, common.NewError("list " + code + " not found in " + filename)

    case ErrFailedToReadBytes, ErrFailedToReadExpectedLenBytes,
        ErrInvalidGeodataFile, ErrInvalidGeodataVarintLength:
        log.Warnf("failed to decode geoip file: %s, fallback to the original ReadFile method", filename)
        # 读取文件内容
        geositeBytes, err = ioutil.ReadFile(asset)
        if err != nil {
            return nil, err
        }
        # 解析出*v2router.GeoSiteList对象
        var geositeList v2router.GeoSiteList
        if err := proto.Unmarshal(geositeBytes, &geositeList); err != nil {
            return nil, err
        }
        # 遍历对象列表，找到匹配的对象并存入geositeCache中
        for _, geosite := range geositeList.GetEntry() {
            if strings.EqualFold(code, geosite.GetCountryCode()) {
                g.Set(idx, geosite)
                return geosite, nil
            }
        }

    default:
        return nil, err
    }

    return nil, common.NewError("list " + code + " not found in " + filename)
}
```