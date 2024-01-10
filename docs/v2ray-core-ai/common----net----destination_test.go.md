# `v2ray-core\common\net\destination_test.go`

```
package net_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"

    . "v2ray.com/core/common/net"
)

func TestDestinationProperty(t *testing.T) {
    testCases := []struct {
        Input     Destination  // 输入的目的地
        Network   Network      // 网络类型
        String    string       // 目的地的字符串表示
        NetString string       // 网络地址的字符串表示
    }{
        {
            Input:     TCPDestination(IPAddress([]byte{1, 2, 3, 4}), 80),  // 创建一个 TCP 目的地
            Network:   Network_TCP,  // 设置网络类型为 TCP
            String:    "tcp:1.2.3.4:80",  // 目的地的字符串表示
            NetString: "1.2.3.4:80",  // 网络地址的字符串表示
        },
        {
            Input:     UDPDestination(IPAddress([]byte{0x20, 0x01, 0x48, 0x60, 0x48, 0x60, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x88, 0x88}), 53),  // 创建一个 UDP 目的地
            Network:   Network_UDP,  // 设置网络类型为 UDP
            String:    "udp:[2001:4860:4860::8888]:53",  // 目的地的字符串表示
            NetString: "[2001:4860:4860::8888]:53",  // 网络地址的字符串表示
        },
    }

    for _, testCase := range testCases {
        dest := testCase.Input
        if r := cmp.Diff(dest.Network, testCase.Network); r != "" {
            t.Error("unexpected Network in ", dest.String(), ": ", r)  // 检查网络类型是否符合预期
        }
        if r := cmp.Diff(dest.String(), testCase.String); r != "" {
            t.Error(r)  // 检查目的地字符串表示是否符合预期
        }
        if r := cmp.Diff(dest.NetAddr(), testCase.NetString); r != "" {
            t.Error(r)  // 检查网络地址字符串表示是否符合预期
        }
    }
}

func TestDestinationParse(t *testing.T) {
    cases := []struct {
        Input  string
        Output Destination
        Error  bool
    # 测试用例，包含输入和预期输出
    cases := []struct {
        Input  string   # 输入字符串
        Output Destination   # 预期输出目的地
        Error  bool     # 是否预期错误
    }{
        {
            Input:  "tcp:127.0.0.1:80",   # 输入为 TCP 地址
            Output: TCPDestination(LocalHostIP, Port(80)),   # 预期输出为本地主机IP和端口80的TCP目的地
        },
        {
            Input:  "udp:8.8.8.8:53",   # 输入为 UDP 地址
            Output: UDPDestination(IPAddress([]byte{8, 8, 8, 8}), Port(53)),   # 预期输出为IP地址8.8.8.8和端口53的UDP目的地
        },
        {
            Input: "8.8.8.8:53",   # 输入为IP地址和端口
            Output: Destination{   # 预期输出为IP地址8.8.8.8和端口53的目的地
                Address: IPAddress([]byte{8, 8, 8, 8}),
                Port:    Port(53),
            },
        },
        {
            Input: ":53",   # 输入为端口号
            Output: Destination{   # 预期输出为任意IP地址和端口53的目的地
                Address: AnyIP,
                Port:    Port(53),
            },
        },
        {
            Input: "8.8.8.8",   # 输入为无端口号的IP地址
            Error: true,   # 预期输出为错误
        },
        {
            Input: "8.8.8.8:http",   # 输入为带有非法端口号的IP地址
            Error: true,   # 预期输出为错误
        },
    }

    # 遍历测试用例
    for _, testcase := range cases {
        # 解析输入字符串为目的地对象
        d, err := ParseDestination(testcase.Input)
        # 如果不是预期错误
        if !testcase.Error {
            # 如果有错误
            if err != nil {
                t.Error("for test case: ", testcase.Input, " expected no error, but got ", err)   # 输出错误信息
            }
            # 如果输出不符合预期
            if d != testcase.Output {
                t.Error("for test case: ", testcase.Input, " expected output: ", testcase.Output.String(), " but got ", d.String())   # 输出错误信息
            }
        } else {
            # 如果预期有错误但是没有错误
            if err == nil {
                t.Error("for test case: ", testcase.Input, " expected error, but got nil")   # 输出错误信息
            }
        }
    }
# 闭合前面的函数定义
```