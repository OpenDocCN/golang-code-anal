# `v2ray-core\app\router\condition_geoip_test.go`

```go
package router_test

import (
    "os" // 导入操作系统相关的包
    "path/filepath" // 导入处理文件路径的包
    "testing" // 导入测试相关的包

    proto "github.com/golang/protobuf/proto" // 导入protobuf相关的包
    "v2ray.com/core/app/router" // 导入路由相关的包
    "v2ray.com/core/common" // 导入通用的包
    "v2ray.com/core/common/net" // 导入网络相关的包
    "v2ray.com/core/common/platform" // 导入平台相关的包
    "v2ray.com/core/common/platform/filesystem" // 导入文件系统相关的包
)

func init() {
    wd, err := os.Getwd() // 获取当前工作目录
    common.Must(err) // 检查错误

    if _, err := os.Stat(platform.GetAssetLocation("geoip.dat")); err != nil && os.IsNotExist(err) { // 检查是否存在指定文件
        common.Must(filesystem.CopyFile(platform.GetAssetLocation("geoip.dat"), filepath.Join(wd, "..", "..", "release", "config", "geoip.dat"))) // 复制文件
    }
    if _, err := os.Stat(platform.GetAssetLocation("geosite.dat")); err != nil && os.IsNotExist(err) { // 检查是否存在指定文件
        common.Must(filesystem.CopyFile(platform.GetAssetLocation("geosite.dat"), filepath.Join(wd, "..", "..", "release", "config", "geosite.dat"))) // 复制文件
    }
}

func TestGeoIPMatcherContainer(t *testing.T) {
    container := &router.GeoIPMatcherContainer{} // 创建路由的地理位置匹配器容器

    m1, err := container.Add(&router.GeoIP{ // 向容器中添加地理位置匹配器
        CountryCode: "CN", // 设置国家代码
    })
    common.Must(err) // 检查错误

    m2, err := container.Add(&router.GeoIP{ // 向容器中添加地理位置匹配器
        CountryCode: "US", // 设置国家代码
    })
    common.Must(err) // 检查错误

    m3, err := container.Add(&router.GeoIP{ // 向容器中添加地理位置匹配器
        CountryCode: "CN", // 设置国家代码
    })
    common.Must(err) // 检查错误

    if m1 != m3 { // 检查是否为同一个匹配器
        t.Error("expect same matcher for same geoip, but not") // 输出错误信息
    }

    if m1 == m2 { // 检查是否为不同的匹配器
        t.Error("expect different matcher for different geoip, but actually same") // 输出错误信息
    }
}

func TestGeoIPMatcher(t *testing.T) {
    # 创建一个CIDR列表，包含多个IP地址和前缀的组合
    cidrList := router.CIDRList{
        {Ip: []byte{0, 0, 0, 0}, Prefix: 8},
        {Ip: []byte{10, 0, 0, 0}, Prefix: 8},
        {Ip: []byte{100, 64, 0, 0}, Prefix: 10},
        {Ip: []byte{127, 0, 0, 0}, Prefix: 8},
        {Ip: []byte{169, 254, 0, 0}, Prefix: 16},
        {Ip: []byte{172, 16, 0, 0}, Prefix: 12},
        {Ip: []byte{192, 0, 0, 0}, Prefix: 24},
        {Ip: []byte{192, 0, 2, 0}, Prefix: 24},
        {Ip: []byte{192, 168, 0, 0}, Prefix: 16},
        {Ip: []byte{192, 18, 0, 0}, Prefix: 15},
        {Ip: []byte{198, 51, 100, 0}, Prefix: 24},
        {Ip: []byte{203, 0, 113, 0}, Prefix: 24},
        {Ip: []byte{8, 8, 8, 8}, Prefix: 32},
        {Ip: []byte{91, 108, 4, 0}, Prefix: 16},
    }

    # 创建一个GeoIPMatcher对象
    matcher := &router.GeoIPMatcher{}
    # 初始化matcher对象，必须成功
    common.Must(matcher.Init(cidrList))

    # 创建测试用例列表，每个测试用例包含输入和预期输出
    testCases := []struct {
        Input  string
        Output bool
    }{
        {
            Input:  "192.168.1.1",
            Output: true,
        },
        {
            Input:  "192.0.0.0",
            Output: true,
        },
        {
            Input:  "192.0.1.0",
            Output: false,
        }, {
            Input:  "0.1.0.0",
            Output: true,
        },
        {
            Input:  "1.0.0.1",
            Output: false,
        },
        {
            Input:  "8.8.8.7",
            Output: false,
        },
        {
            Input:  "8.8.8.8",
            Output: true,
        },
        {
            Input:  "2001:cdba::3257:9652",
            Output: false,
        },
        {
            Input:  "91.108.255.254",
            Output: true,
        },
    }

    # 遍历测试用例列表
    for _, testCase := range testCases {
        # 解析测试用例的输入IP地址
        ip := net.ParseAddress(testCase.Input).IP()
        # 使用matcher对象匹配IP地址，得到实际输出
        actual := matcher.Match(ip)
        # 检查实际输出是否与预期输出一致，如果不一致则输出错误信息
        if actual != testCase.Output {
            t.Error("expect input", testCase.Input, "to be", testCase.Output, ", but actually", actual)
        }
    }
// 测试中国地理位置匹配器
func TestGeoIPMatcher4CN(t *testing.T) {
    // 加载中国地理位置信息
    ips, err := loadGeoIP("CN")
    // 检查错误
    common.Must(err)

    // 创建地理位置匹配器
    matcher := &router.GeoIPMatcher{}
    // 初始化地理位置匹配器
    common.Must(matcher.Init(ips))

    // 检查是否匹配到指定 IP 地址
    if matcher.Match([]byte{8, 8, 8, 8}) {
        t.Error("expect CN geoip doesn't contain 8.8.8.8, but actually does")
    }
}

// 测试美国地理位置匹配器
func TestGeoIPMatcher6US(t *testing.T) {
    // 加载美国地理位置信息
    ips, err := loadGeoIP("US")
    // 检查错误
    common.Must(err)

    // 创建地理位置匹配器
    matcher := &router.GeoIPMatcher{}
    // 初始化地理位置匹配器
    common.Must(matcher.Init(ips))

    // 检查是否匹配到指定 IP 地址
    if !matcher.Match(net.ParseAddress("2001:4860:4860::8888").IP()) {
        t.Error("expect US geoip contain 2001:4860:4860::8888, but actually not")
    }
}

// 加载地理位置信息
func loadGeoIP(country string) ([]*router.CIDR, error) {
    // 读取地理位置信息文件
    geoipBytes, err := filesystem.ReadAsset("geoip.dat")
    // 检查错误
    if err != nil {
        return nil, err
    }
    // 反序列化地理位置信息
    var geoipList router.GeoIPList
    if err := proto.Unmarshal(geoipBytes, &geoipList); err != nil {
        return nil, err
    }

    // 遍历地理位置信息，查找指定国家的地理位置信息
    for _, geoip := range geoipList.Entry {
        if geoip.CountryCode == country {
            return geoip.Cidr, nil
        }
    }

    // 如果未找到指定国家的地理位置信息，则抛出异常
    panic("country not found: " + country)
}

// 测试中国地理位置匹配器性能
func BenchmarkGeoIPMatcher4CN(b *testing.B) {
    // 加载中国地理位置信息
    ips, err := loadGeoIP("CN")
    // 检查错误
    common.Must(err)

    // 创建地理位置匹配器
    matcher := &router.GeoIPMatcher{}
    // 初始化地理位置匹配器
    common.Must(matcher.Init(ips))

    // 重置计时器
    b.ResetTimer()

    // 循环测试地理位置匹配性能
    for i := 0; i < b.N; i++ {
        _ = matcher.Match([]byte{8, 8, 8, 8})
    }
}

// 测试美国地理位置匹配器性能
func BenchmarkGeoIPMatcher6US(b *testing.B) {
    // 加载美国地理位置信息
    ips, err := loadGeoIP("US")
    // 检查错误
    common.Must(err)

    // 创建地理位置匹配器
    matcher := &router.GeoIPMatcher{}
    // 初始化地理位置匹配器
    common.Must(matcher.Init(ips))

    // 重置计时器
    b.ResetTimer()

    // 循环测试地理位置匹配性能
    for i := 0; i < b.N; i++ {
        _ = matcher.Match(net.ParseAddress("2001:4860:4860::8888").IP())
    }
}
```