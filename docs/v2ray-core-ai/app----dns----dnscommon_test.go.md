# `v2ray-core\app\dns\dnscommon_test.go`

```go
// +build !confonly

package dns

import (
    "math/rand"  // 导入 math/rand 包，用于生成随机数
    "testing"    // 导入 testing 包，用于编写测试函数
    "time"       // 导入 time 包，用于处理时间

    "github.com/google/go-cmp/cmp"  // 导入 github.com/google/go-cmp/cmp 包，用于比较数据结构
    "github.com/miekg/dns"           // 导入 github.com/miekg/dns 包，用于处理 DNS 相关操作
    "golang.org/x/net/dns/dnsmessage"  // 导入 golang.org/x/net/dns/dnsmessage 包，用于处理 DNS 消息
    "v2ray.com/core/common"          // 导入 v2ray.com/core/common 包，用于处理通用操作
    "v2ray.com/core/common/net"      // 导入 v2ray.com/core/common/net 包，用于处理网络相关操作
)

func Test_parseResponse(t *testing.T) {
    var p [][]byte  // 声明一个二维字节切片 p

    ans := new(dns.Msg)  // 创建一个新的 DNS 消息对象 ans
    ans.Id = 0  // 设置 ans 的 ID 为 0
    p = append(p, common.Must2(ans.Pack()).([]byte))  // 将 ans 打包成字节切片并添加到 p 中

    p = append(p, []byte{})  // 添加一个空的字节切片到 p 中

    ans = new(dns.Msg)  // 创建一个新的 DNS 消息对象 ans
    ans.Id = 1  // 设置 ans 的 ID 为 1
    ans.Answer = append(ans.Answer,
        common.Must2(dns.NewRR("google.com. IN CNAME m.test.google.com")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN CNAME fake.google.com")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN A 8.8.8.8")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN A 8.8.4.4")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
    )
    p = append(p, common.Must2(ans.Pack()).([]byte))  // 将 ans 打包成字节切片并添加到 p 中

    ans = new(dns.Msg)  // 创建一个新的 DNS 消息对象 ans
    ans.Id = 2  // 设置 ans 的 ID 为 2
    ans.Answer = append(ans.Answer,
        common.Must2(dns.NewRR("google.com. IN CNAME m.test.google.com")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN CNAME fake.google.com")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN CNAME m.test.google.com")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN CNAME test.google.com")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN AAAA 2001::123:8888")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
        common.Must2(dns.NewRR("google.com. IN AAAA 2001::123:8844")).(dns.RR),  // 向 ans 的 Answer 中添加一个 DNS 记录
    )
    p = append(p, common.Must2(ans.Pack()).([]byte))  // 将 ans 打包成字节切片并添加到 p 中

    tests := []struct {
        name    string  // 测试用例的名称
        want    *IPRecord  // 期望的 IP 记录
        wantErr bool  // 是否期望出现错误
    // 定义测试用例，包括名称、IPRecord、是否期望出错
    {
        "empty",
        &IPRecord{0, []net.Address(nil), time.Time{}, dnsmessage.RCodeSuccess},
        false,
    },
    {
        "error",
        nil,
        true,
    },
    {
        "a record",
        &IPRecord{1, []net.Address{net.ParseAddress("8.8.8.8"), net.ParseAddress("8.8.4.4")},
            time.Time{}, dnsmessage.RCodeSuccess},
        false,
    },
    {
        "aaaa record",
        &IPRecord{2, []net.Address{net.ParseAddress("2001::123:8888"), net.ParseAddress("2001::123:8844")}, time.Time{}, dnsmessage.RCodeSuccess},
        false,
    }
    // 遍历测试用例切片
    for i, tt := range tests {
        // 运行子测试，使用测试名称
        t.Run(tt.name, func(t *testing.T) {
            // 解析响应，获取结果和错误
            got, err := parseResponse(p[i])
            // 检查错误是否符合期望
            if (err != nil) != tt.wantErr {
                t.Errorf("handleResponse() error = %v, wantErr %v", err, tt.wantErr)
                return
            }

            // 如果结果不为空，重置时间
            if got != nil {
                got.Expire = time.Time{}
            }
            // 检查结果是否符合期望
            if cmp.Diff(got, tt.want) != "" {
                t.Errorf(cmp.Diff(got, tt.want))
                // t.Errorf("handleResponse() = %#v, want %#v", got, tt.want)
            }
        })
    }
// 测试函数，用于构建请求消息
func Test_buildReqMsgs(t *testing.T) {
    // 定义一个返回随机数的函数
    stubID := func() uint16 {
        return uint16(rand.Uint32())
    }
    // 定义参数结构体
    type args struct {
        domain  string
        option  IPOption
        reqOpts *dnsmessage.Resource
    }
    // 定义测试用例
    tests := []struct {
        name string
        args args
        want int
    }{
        {"dual stack", args{"test.com", IPOption{true, true}, nil}, 2},
        {"ipv4 only", args{"test.com", IPOption{true, false}, nil}, 1},
        {"ipv6 only", args{"test.com", IPOption{false, true}, nil}, 1},
        {"none/error", args{"test.com", IPOption{false, false}, nil}, 0},
    }
    // 遍历测试用例
    for _, tt := range tests {
        // 运行测试用例
        t.Run(tt.name, func(t *testing.T) {
            // 调用函数并检查结果
            if got := buildReqMsgs(tt.args.domain, tt.args.option, stubID, tt.args.reqOpts); !(len(got) == tt.want) {
                t.Errorf("buildReqMsgs() = %v, want %v", got, tt.want)
            }
        })
    }
}

// 测试函数，用于生成 EDNS0 选项
func Test_genEDNS0Options(t *testing.T) {
    // 定义参数结构体
    type args struct {
        clientIP net.IP
    }
    // 定义测试用例
    tests := []struct {
        name string
        args args
        want *dnsmessage.Resource
    }{
        // TODO: Add test cases.
        {"ipv4", args{net.ParseIP("4.3.2.1")}, nil},
        {"ipv6", args{net.ParseIP("2001::4321")}, nil},
    }
    // 遍历测试用例
    for _, tt := range tests {
        // 运行测试用例
        t.Run(tt.name, func(t *testing.T) {
            // 调用函数并检查结果
            if got := genEDNS0Options(tt.args.clientIP); got == nil {
                t.Errorf("genEDNS0Options() = %v, want %v", got, tt.want)
            }
        })
    }
}

// 测试函数，用于测试 Fqdn 函数
func TestFqdn(t *testing.T) {
    // 定义参数结构体
    type args struct {
        domain string
    }
    // 定义测试用例
    tests := []struct {
        name string
        args args
        want string
    }{
        {"with fqdn", args{"www.v2ray.com."}, "www.v2ray.com."},
        {"without fqdn", args{"www.v2ray.com"}, "www.v2ray.com."},
    }
}
    # 遍历测试用例数组
    for _, tt := range tests:
        # 使用测试用例的名称创建子测试，并运行
        t.Run(tt.name, func(t *testing.T) {
            # 调用 Fqdn 函数，检查返回值是否与期望值相等
            if got := Fqdn(tt.args.domain); got != tt.want {
                # 如果返回值与期望值不相等，则输出错误信息
                t.Errorf("Fqdn() = %v, want %v", got, tt.want)
            }
        })
    }
# 闭合前面的函数定义
```