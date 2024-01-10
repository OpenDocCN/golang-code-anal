# `v2ray-core\app\dns\hosts_test.go`

```
package dns_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"

    . "v2ray.com/core/app/dns"  // 导入 v2ray.com/core/app/dns 包，使用 . 表示在后续代码中可以省略包名
    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/net"  // 导入 v2ray.com/core/common/net 包
)

func TestStaticHosts(t *testing.T) {
    pb := []*Config_HostMapping{  // 创建 Config_HostMapping 结构体的切片
        {
            Type:   DomainMatchingType_Full,  // 设置 Type 字段为 DomainMatchingType_Full
            Domain: "v2ray.com",  // 设置 Domain 字段为 "v2ray.com"
            Ip: [][]byte{  // 设置 Ip 字段为包含一个字节切片的切片
                {1, 1, 1, 1},  // 字节切片表示 IP 地址
            },
        },
        {
            Type:   DomainMatchingType_Subdomain,  // 设置 Type 字段为 DomainMatchingType_Subdomain
            Domain: "v2ray.cn",  // 设置 Domain 字段为 "v2ray.cn"
            Ip: [][]byte{  // 设置 Ip 字段为包含一个字节切片的切片
                {2, 2, 2, 2},  // 字节切片表示 IP 地址
            },
        },
        {
            Type:   DomainMatchingType_Subdomain,  // 设置 Type 字段为 DomainMatchingType_Subdomain
            Domain: "baidu.com",  // 设置 Domain 字段为 "baidu.com"
            Ip: [][]byte{  // 设置 Ip 字段为包含一个字节切片的切片
                {127, 0, 0, 1},  // 字节切片表示 IP 地址
            },
        },
    }

    hosts, err := NewStaticHosts(pb, nil)  // 调用 NewStaticHosts 方法创建静态主机映射，返回结果和错误
    common.Must(err)  // 检查错误，如果有错误则终止程序运行

    {
        ips := hosts.LookupIP("v2ray.com", IPOption{  // 调用 LookupIP 方法查找 "v2ray.com" 的 IP 地址
            IPv4Enable: true,  // 设置 IPv4Enable 字段为 true
            IPv6Enable: true,  // 设置 IPv6Enable 字段为 true
        })
        if len(ips) != 1 {  // 检查返回的 IP 地址数量是否为 1
            t.Error("expect 1 IP, but got ", len(ips))  // 输出错误信息
        }
        if diff := cmp.Diff([]byte(ips[0].IP()), []byte{1, 1, 1, 1}); diff != "" {  // 比较返回的 IP 地址和预期的 IP 地址
            t.Error(diff)  // 输出差异信息
        }
    }

    {
        ips := hosts.LookupIP("www.v2ray.cn", IPOption{  // 调用 LookupIP 方法查找 "www.v2ray.cn" 的 IP 地址
            IPv4Enable: true,  // 设置 IPv4Enable 字段为 true
            IPv6Enable: true,  // 设置 IPv6Enable 字段为 true
        })
        if len(ips) != 1 {  // 检查返回的 IP 地址数量是否为 1
            t.Error("expect 1 IP, but got ", len(ips))  // 输出错误信息
        }
        if diff := cmp.Diff([]byte(ips[0].IP()), []byte{2, 2, 2, 2}); diff != "" {  // 比较返回的 IP 地址和预期的 IP 地址
            t.Error(diff)  // 输出差异信息
        }
    }

    {
        ips := hosts.LookupIP("baidu.com", IPOption{  // 调用 LookupIP 方法查找 "baidu.com" 的 IP 地址
            IPv4Enable: false,  // 设置 IPv4Enable 字段为 false
            IPv6Enable: true,  // 设置 IPv6Enable 字段为 true
        })
        if len(ips) != 1 {  // 检查返回的 IP 地址数量是否为 1
            t.Error("expect 1 IP, but got ", len(ips))  // 输出错误信息
        }
        if diff := cmp.Diff([]byte(ips[0].IP()), []byte(net.LocalHostIPv6.IP())); diff != "" {  // 比较返回的 IP 地址和本地主机的 IPv6 地址
            t.Error(diff)  // 输出差异信息
        }
    }
}
```