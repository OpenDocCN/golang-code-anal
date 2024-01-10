# `v2ray-core\common\net\address_test.go`

```
package net_test

import (
    "net"  // 导入 net 包
    "testing"  // 导入 testing 包

    "github.com/google/go-cmp/cmp"  // 导入 google/go-cmp/cmp 包

    . "v2ray.com/core/common/net"  // 导入 v2ray.com/core/common/net 包，使用 . 表示在本文件中可以直接使用该包的函数和类型
)

func TestAddressProperty(t *testing.T) {
    type addrProprty struct {  // 定义名为 addrProprty 的结构体类型
        IP     []byte  // IP 字节切片
        Domain string  // 域名字符串
        Family AddressFamily  // AddressFamily 类型
        String string  // 字符串
    }

    testCases := []struct {  // 定义名为 testCases 的结构体切片
        Input  Address  // Address 类型
        Output addrProprty  // addrProprty 类型
    }

    for _, testCase := range testCases {  // 遍历 testCases 切片
        actual := addrProprty{  // 创建 addrProprty 类型的变量 actual
            Family: testCase.Input.Family(),  // 赋值 Family 字段
            String: testCase.Input.String(),  // 赋值 String 字段
        }
        if testCase.Input.Family().IsIP() {  // 判断 Family 类型是否为 IP
            actual.IP = testCase.Input.IP()  // 赋值 IP 字段
        } else {
            actual.Domain = testCase.Input.Domain()  // 赋值 Domain 字段
        }

        if r := cmp.Diff(actual, testCase.Output); r != "" {  // 比较 actual 和 testCase.Output 的差异
            t.Error("for input: ", testCase.Input, ":", r)  // 输出错误信息
        }
    }
}

func TestInvalidAddressConvertion(t *testing.T) {
    panics := func(f func()) (ret bool) {  // 定义名为 panics 的函数
        defer func() {  // 延迟执行
            if r := recover(); r != nil {  // 捕获 panic
                ret = true  // 设置 ret 为 true
            }
        }()
        f()  // 执行函数 f
        return false  // 返回 false
    }

    testCases := []func(){  // 定义名为 testCases 的函数切片
        func() { ParseAddress("8.8.8.8").Domain() },  // 匿名函数
        func() { ParseAddress("2001:4860:0:2001::68").Domain() },  // 匿名函数
        func() { ParseAddress("v2ray.com").IP() },  // 匿名函数
    }
    for idx, testCase := range testCases {  // 遍历 testCases 切片
        if !panics(testCase) {  // 判断是否发生 panic
            t.Error("case ", idx, " failed")  // 输出错误信息
        }
    }
}

func BenchmarkParseAddressIPv4(b *testing.B) {
    for i := 0; i < b.N; i++ {  // 循环执行 b.N 次
        addr := ParseAddress("8.8.8.8")  // 解析地址
        if addr.Family() != AddressFamilyIPv4 {  // 判断地址类型是否为 IPv4
            panic("not ipv4")  // 抛出 panic
        }
    }
}

func BenchmarkParseAddressIPv6(b *testing.B) {
    for i := 0; i < b.N; i++ {  // 循环执行 b.N 次
        addr := ParseAddress("2001:4860:0:2001::68")  // 解析地址
        if addr.Family() != AddressFamilyIPv6 {  // 判断地址类型是否为 IPv6
            panic("not ipv6")  // 抛出 panic
        }
    }
}

func BenchmarkParseAddressDomain(b *testing.B) {
    # 循环执行 b.N 次
    for i := 0; i < b.N; i++ {
        # 解析地址 "v2ray.com"，并将结果赋给 addr
        addr := ParseAddress("v2ray.com")
        # 如果地址的类型不是域名，则触发 panic
        if addr.Family() != AddressFamilyDomain {
            panic("not domain")
        }
    }
# 闭合前面的函数定义
```