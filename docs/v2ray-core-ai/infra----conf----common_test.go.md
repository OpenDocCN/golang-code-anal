# `v2ray-core\infra\conf\common_test.go`

```go
package conf_test

import (
    "encoding/json"  // 导入 JSON 包
    "github.com/google/go-cmp/cmp/cmpopts"  // 导入 Google 的比较包
    "os"  // 导入操作系统包
    "testing"  // 导入测试包

    "github.com/google/go-cmp/cmp"  // 导入 Google 的比较包
    "v2ray.com/core/common/protocol"  // 导入 V2Ray 的协议包

    "v2ray.com/core/common"  // 导入 V2Ray 的通用包
    "v2ray.com/core/common/net"  // 导入 V2Ray 的网络包
    . "v2ray.com/core/infra/conf"  // 导入 V2Ray 的配置包
)

func TestStringListUnmarshalError(t *testing.T) {
    rawJson := `1234`  // 定义一个 JSON 字符串
    list := new(StringList)  // 创建一个 StringList 对象
    err := json.Unmarshal([]byte(rawJson), list)  // 使用 JSON 解析字符串到 StringList 对象
    if err == nil {  // 如果没有错误
        t.Error("expected error, but got nil")  // 输出错误信息
    }
}

func TestStringListLen(t *testing.T) {
    rawJson := `"a, b, c, d"`  // 定义一个 JSON 字符串
    var list StringList  // 创建一个 StringList 对象
    err := json.Unmarshal([]byte(rawJson), &list)  // 使用 JSON 解析字符串到 StringList 对象
    common.Must(err)  // 检查是否有错误
    if r := cmp.Diff([]string(list), []string{"a", " b", " c", " d"}); r != "" {  // 比较解析结果和预期结果
        t.Error(r)  // 输出错误信息
    }
}

func TestIPParsing(t *testing.T) {
    rawJson := "\"8.8.8.8\""  // 定义一个 JSON 字符串
    var address Address  // 创建一个 Address 对象
    err := json.Unmarshal([]byte(rawJson), &address)  // 使用 JSON 解析字符串到 Address 对象
    common.Must(err)  // 检查是否有错误
    if r := cmp.Diff(address.IP(), net.IP{8, 8, 8, 8}); r != "" {  // 比较解析结果和预期结果
        t.Error(r)  // 输出错误信息
    }
}

func TestDomainParsing(t *testing.T) {
    rawJson := "\"v2ray.com\""  // 定义一个 JSON 字符串
    var address Address  // 创建一个 Address 对象
    common.Must(json.Unmarshal([]byte(rawJson), &address))  // 使用 JSON 解析字符串到 Address 对象
    if address.Domain() != "v2ray.com" {  // 检查解析结果是否符合预期
        t.Error("domain: ", address.Domain())  // 输出错误信息
    }
}

func TestURLParsing(t *testing.T) {
    {
        rawJson := "\"https://dns.google/dns-query\""  // 定义一个 JSON 字符串
        var address Address  // 创建一个 Address 对象
        common.Must(json.Unmarshal([]byte(rawJson), &address))  // 使用 JSON 解析字符串到 Address 对象
        if address.Domain() != "https://dns.google/dns-query" {  // 检查解析结果是否符合预期
            t.Error("URL: ", address.Domain())  // 输出错误信息
        }
    }
    {
        rawJson := "\"https+local://dns.google/dns-query\""  // 定义一个 JSON 字符串
        var address Address  // 创建一个 Address 对象
        common.Must(json.Unmarshal([]byte(rawJson), &address))  // 使用 JSON 解析字符串到 Address 对象
        if address.Domain() != "https+local://dns.google/dns-query" {  // 检查解析结果是否符合预期
            t.Error("URL: ", address.Domain())  // 输出错误信息
        }
    }
}

func TestInvalidAddressJson(t *testing.T) {
    rawJson := "1234"  // 定义一个 JSON 字符串
    var address Address  // 创建一个 Address 对象
    # 使用 json.Unmarshal 将原始 JSON 数据解析为 address 结构体
    err := json.Unmarshal([]byte(rawJson), &address)
    # 如果 err 为 nil，表示解析成功，输出错误信息
    if err == nil {
        t.Error("nil error")
    }
func TestStringNetwork(t *testing.T) {
    // 创建一个 Network 类型的变量
    var network Network
    // 将字符串 `"tcp"` 解析为 Network 类型的变量
    common.Must(json.Unmarshal([]byte(`"tcp"`), &network))
    // 如果 network.Build() 的返回值不等于 net.Network_TCP，则输出错误信息
    if v := network.Build(); v != net.Network_TCP {
        t.Error("network: ", v)
    }
}

func TestArrayNetworkList(t *testing.T) {
    // 创建一个 NetworkList 类型的变量
    var list NetworkList
    // 将字符串 `["Tcp"]` 解析为 NetworkList 类型的变量
    common.Must(json.Unmarshal([]byte("[\"Tcp\"]"), &list))

    // 调用 list.Build() 方法
    nlist := list.Build()
    // 如果 nlist 中不包含 net.Network_TCP，则输出错误信息
    if !net.HasNetwork(nlist, net.Network_TCP) {
        t.Error("no tcp network")
    }
    // 如果 nlist 中包含 net.Network_UDP，则输出错误信息
    if net.HasNetwork(nlist, net.Network_UDP) {
        t.Error("has udp network")
    }
}

func TestStringNetworkList(t *testing.T) {
    // 创建一个 NetworkList 类型的变量
    var list NetworkList
    // 将字符串 `"TCP, ip"` 解析为 NetworkList 类型的变量
    common.Must(json.Unmarshal([]byte("\"TCP, ip\""), &list))

    // 调用 list.Build() 方法
    nlist := list.Build()
    // 如果 nlist 中不包含 net.Network_TCP，则输出错误信息
    if !net.HasNetwork(nlist, net.Network_TCP) {
        t.Error("no tcp network")
    }
    // 如果 nlist 中包含 net.Network_UDP，则输出错误信息
    if net.HasNetwork(nlist, net.Network_UDP) {
        t.Error("has udp network")
    }
}

func TestInvalidNetworkJson(t *testing.T) {
    // 创建一个 NetworkList 类型的变量
    var list NetworkList
    // 将字符串 "0" 解析为 NetworkList 类型的变量
    err := json.Unmarshal([]byte("0"), &list)
    // 如果 err 为 nil，则输出错误信息
    if err == nil {
        t.Error("nil error")
    }
}

func TestIntPort(t *testing.T) {
    // 创建一个 PortRange 类型的变量
    var portRange PortRange
    // 将字符串 "1234" 解析为 PortRange 类型的变量
    common.Must(json.Unmarshal([]byte("1234"), &portRange))

    // 比较 portRange 和 PortRange{From: 1234, To: 1234} 的差异
    if r := cmp.Diff(portRange, PortRange{
        From: 1234, To: 1234,
    }); r != "" {
        t.Error(r)
    }
}

func TestOverRangeIntPort(t *testing.T) {
    // 创建一个 PortRange 类型的变量
    var portRange PortRange
    // 将字符串 "70000" 解析为 PortRange 类型的变量
    err := json.Unmarshal([]byte("70000"), &portRange)
    // 如果 err 为 nil，则输出错误信息
    if err == nil {
        t.Error("nil error")
    }

    // 将字符串 "-1" 解析为 PortRange 类型的变量
    err = json.Unmarshal([]byte("-1"), &portRange)
    // 如果 err 为 nil，则输出错误信息
    if err == nil {
        t.Error("nil error")
    }
}

func TestEnvPort(t *testing.T) {
    // 设置环境变量 "PORT" 为 "1234"
    common.Must(os.Setenv("PORT", "1234"))

    // 创建一个 PortRange 类型的变量
    var portRange PortRange
    // 将字符串 "env:PORT" 解析为 PortRange 类型的变量
    common.Must(json.Unmarshal([]byte("\"env:PORT\""), &portRange))

    // 比较 portRange 和 PortRange{From: 1234, To: 1234} 的差异
    if r := cmp.Diff(portRange, PortRange{
        From: 1234, To: 1234,
    }); r != "" {
        t.Error(r)
    }
}

func TestSingleStringPort(t *testing.T) {
    // 创建一个 PortRange 类型的变量
    var portRange PortRange
    # 使用 json.Unmarshal 将字符串转换为字节流，再解析到 portRange 变量中
    common.Must(json.Unmarshal([]byte("\"1234\""), &portRange))

    # 使用 cmp.Diff 比较 portRange 和 PortRange 结构体的差异
    if r := cmp.Diff(portRange, PortRange{
        From: 1234, To: 1234,
    }); r != "" {
        # 如果有差异，则输出错误信息
        t.Error(r)
    }
// 测试字符串转换为端口范围的函数
func TestStringPairPort(t *testing.T) {
    // 创建端口范围对象
    var portRange PortRange
    // 解析 JSON 字符串并将结果存入端口范围对象
    common.Must(json.Unmarshal([]byte("\"1234-5678\""), &portRange))

    // 检查解析结果是否符合预期
    if r := cmp.Diff(portRange, PortRange{
        From: 1234, To: 5678,
    }); r != "" {
        t.Error(r)
    }
}

// 测试无效的端口范围字符串
func TestOverRangeStringPort(t *testing.T) {
    // 创建端口范围对象
    var portRange PortRange
    // 解析 JSON 字符串并将结果存入端口范围对象
    err := json.Unmarshal([]byte("\"65536\""), &portRange)
    // 检查是否返回了预期的错误
    if err == nil {
        t.Error("nil error")
    }

    // 重复上述步骤，测试其他无效的端口范围字符串
}

// 测试用户信息解析函数
func TestUserParsing(t *testing.T) {
    // 创建用户对象
    user := new(User)
    // 解析 JSON 字符串并将结果存入用户对象
    common.Must(json.Unmarshal([]byte(`{
    "id": "96edb838-6d68-42ef-a933-25f7ac3a9d09",
    "email": "love@v2ray.com",
    "level": 1,
    "alterId": 100
  }`), user))

    // 构建新的用户对象
    nUser := user.Build()
    // 检查构建结果是否符合预期
    if r := cmp.Diff(nUser, &protocol.User{
        Level: 1,
        Email: "love@v2ray.com",
    }, cmpopts.IgnoreUnexported(protocol.User{})); r != "" {
        t.Error(r)
    }
}

// 测试无效的用户信息 JSON
func TestInvalidUserJson(t *testing.T) {
    // 创建用户对象
    user := new(User)
    // 解析无效的 JSON 字符串并将结果存入用户对象
    err := json.Unmarshal([]byte(`{"email": 1234}`), user)
    // 检查是否返回了预期的错误
    if err == nil {
        t.Error("nil error")
    }
}
```