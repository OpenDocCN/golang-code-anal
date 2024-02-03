# `v2ray-core\app\router\condition_test.go`

```go
package router_test

import (
    "os" // 导入操作系统相关的包
    "path/filepath" // 导入文件路径相关的包
    "strconv" // 导入字符串转换相关的包
    "testing" // 导入测试相关的包

    proto "github.com/golang/protobuf/proto" // 导入protobuf相关的包

    . "v2ray.com/core/app/router" // 导入v2ray路由相关的包
    "v2ray.com/core/common" // 导入v2ray通用相关的包
    "v2ray.com/core/common/errors" // 导入v2ray错误处理相关的包
    "v2ray.com/core/common/net" // 导入v2ray网络相关的包
    "v2ray.com/core/common/platform" // 导入v2ray平台相关的包
    "v2ray.com/core/common/platform/filesystem" // 导入v2ray文件系统相关的包
    "v2ray.com/core/common/protocol" // 导入v2ray协议相关的包
    "v2ray.com/core/common/protocol/http" // 导入v2ray HTTP协议相关的包
    "v2ray.com/core/common/session" // 导入v2ray会话相关的包
    "v2ray.com/core/features/routing" // 导入v2ray路由特性相关的包
    routing_session "v2ray.com/core/features/routing/session" // 导入v2ray路由会话相关的包
)

func init() {
    wd, err := os.Getwd() // 获取当前工作目录
    common.Must(err) // 检查错误

    if _, err := os.Stat(platform.GetAssetLocation("geoip.dat")); err != nil && os.IsNotExist(err) { // 检查geoip.dat文件是否存在
        common.Must(filesystem.CopyFile(platform.GetAssetLocation("geoip.dat"), filepath.Join(wd, "..", "..", "release", "config", "geoip.dat"))) // 复制geoip.dat文件
    }
    if _, err := os.Stat(platform.GetAssetLocation("geosite.dat")); err != nil && os.IsNotExist(err) { // 检查geosite.dat文件是否存在
        common.Must(filesystem.CopyFile(platform.GetAssetLocation("geosite.dat"), filepath.Join(wd, "..", "..", "release", "config", "geosite.dat"))) // 复制geosite.dat文件
    }
}

func withBackground() routing.Context {
    return &routing_session.Context{} // 返回一个带有背景的路由上下文
}

func withOutbound(outbound *session.Outbound) routing.Context {
    return &routing_session.Context{Outbound: outbound} // 返回一个带有出站连接的路由上下文
}

func withInbound(inbound *session.Inbound) routing.Context {
    return &routing_session.Context{Inbound: inbound} // 返回一个带有入站连接的路由上下文
}

func withContent(content *session.Content) routing.Context {
    return &routing_session.Context{Content: content} // 返回一个带有内容的路由上下文
}

func TestRoutingRule(t *testing.T) {
    type ruleTest struct {
        input  routing.Context // 定义路由上下文输入
        output bool // 定义预期输出
    }

    cases := []struct {
        rule *RoutingRule // 定义路由规则
        test []ruleTest // 定义测试用例
    }
    # 遍历测试用例集合
    for _, test := range cases:
        # 根据规则构建条件
        cond, err := test.rule.BuildCondition()
        # 必须处理错误
        common.Must(err)

        # 遍历每个子测试用例
        for _, subtest := range test.test:
            # 应用条件到子测试用例的输入，得到实际结果
            actual := cond.Apply(subtest.input)
            # 检查实际结果是否与预期输出一致，如果不一致则输出错误信息
            if actual != subtest.output:
                t.Error("test case failed: ", subtest.input, " expected ", subtest.output, " but got ", actual)
        # 结束子测试用例的遍历
    # 结束测试用例集合的遍历
func loadGeoSite(country string) ([]*Domain, error) {
    // 从文件系统中读取名为 "geosite.dat" 的文件内容
    geositeBytes, err := filesystem.ReadAsset("geosite.dat")
    // 如果读取文件内容出现错误，则返回空指针和错误信息
    if err != nil {
        return nil, err
    }
    // 创建一个空的 GeoSiteList 对象
    var geositeList GeoSiteList
    // 将读取的文件内容解析为 GeoSiteList 对象
    if err := proto.Unmarshal(geositeBytes, &geositeList); err != nil {
        return nil, err
    }

    // 遍历 GeoSiteList 中的每个条目
    for _, site := range geositeList.Entry {
        // 如果条目的国家代码与指定的国家相同，则返回该条目的域名列表
        if site.CountryCode == country {
            return site.Domain, nil
        }
    }

    // 如果未找到指定国家的条目，则返回错误信息
    return nil, errors.New("country not found: " + country)
}

func TestChinaSites(t *testing.T) {
    // 加载中国的域名列表
    domains, err := loadGeoSite("CN")
    // 如果加载出现错误，则终止测试
    common.Must(err)

    // 创建一个新的域名匹配器
    matcher, err := NewDomainMatcher(domains)
    // 如果创建匹配器出现错误，则终止测试
    common.Must(err)

    // 定义测试用例结构
    type TestCase struct {
        Domain string
        Output bool
    }
    // 创建测试用例列表
    testCases := []TestCase{
        {
            Domain: "163.com",
            Output: true,
        },
        {
            Domain: "163.com",
            Output: true,
        },
        {
            Domain: "164.com",
            Output: false,
        },
        {
            Domain: "164.com",
            Output: false,
        },
    }

    // 循环添加额外的测试用例
    for i := 0; i < 1024; i++ {
        testCases = append(testCases, TestCase{Domain: strconv.Itoa(i) + ".not-exists.com", Output: false})
    }

    // 遍历测试用例列表，对每个域名进行匹配测试
    for _, testCase := range testCases {
        r := matcher.ApplyDomain(testCase.Domain)
        // 如果匹配结果与预期输出不符，则输出错误信息
        if r != testCase.Output {
            t.Error("expected output ", testCase.Output, " for domain ", testCase.Domain, " but got ", r)
        }
    }
}

func BenchmarkMultiGeoIPMatcher(b *testing.B) {
    // 创建一个空的 GeoIP 列表
    var geoips []*GeoIP

    // 加载中国的 IP 地址列表
    {
        ips, err := loadGeoIP("CN")
        // 如果加载出现错误，则终止基准测试
        common.Must(err)
        // 将加载的 IP 地址列表添加到 GeoIP 列表中
        geoips = append(geoips, &GeoIP{
            CountryCode: "CN",
            Cidr:        ips,
        })
    }

    // 加载日本的 IP 地址列表
    {
        ips, err := loadGeoIP("JP")
        // 如果加载出现错误，则终止基准测试
        common.Must(err)
        // 将加载的 IP 地址列表添加到 GeoIP 列表中
        geoips = append(geoips, &GeoIP{
            CountryCode: "JP",
            Cidr:        ips,
        })
    }
}
    {
        // 加载国家代码为 "CA" 的地理IP信息，返回IP列表和错误
        ips, err := loadGeoIP("CA")
        // 如果有错误，立即终止程序
        common.Must(err)
        // 将加载的地理IP信息添加到 geoips 切片中
        geoips = append(geoips, &GeoIP{
            CountryCode: "CA",
            Cidr:        ips,
        })
    }

    {
        // 加载国家代码为 "US" 的地理IP信息，返回IP列表和错误
        ips, err := loadGeoIP("US")
        // 如果有错误，立即终止程序
        common.Must(err)
        // 将加载的地理IP信息添加到 geoips 切片中
        geoips = append(geoips, &GeoIP{
            CountryCode: "US",
            Cidr:        ips,
        })
    }

    // 创建一个包含所有地理IP信息的匹配器
    matcher, err := NewMultiGeoIPMatcher(geoips, false)
    // 如果有错误，立即终止程序
    common.Must(err)

    // 创建一个带有目标地址和端口的出站连接上下文
    ctx := withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("8.8.8.8"), 80)})

    // 重置计时器
    b.ResetTimer()

    // 循环执行匹配器的应用方法，测试性能
    for i := 0; i < b.N; i++ {
        _ = matcher.Apply(ctx)
    }
# 闭合前面的函数定义
```