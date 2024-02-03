# `kubo\cmd\ipfs\kubo\dnsresolve_test.go`

```go
package kubo

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "fmt"  // 导入格式化包，用于格式化输出
    "net"  // 导入网络包，用于网络相关操作
    "strings"  // 导入字符串包，用于处理字符串
    "testing"  // 导入测试包，用于编写测试函数

    ma "github.com/multiformats/go-multiaddr"  // 导入多地址包，并重命名为ma
    madns "github.com/multiformats/go-multiaddr-dns"  // 导入多地址DNS包，并重命名为madns
)

var (
    ctx         = context.Background()  // 创建一个上下文对象
    testAddr, _ = ma.NewMultiaddr("/dns4/example.com/tcp/5001")  // 创建一个测试地址对象
)

func makeResolver(t *testing.T, n uint8) *madns.Resolver {
    results := make([]net.IPAddr, n)  // 创建一个IP地址数组
    for i := uint8(0); i < n; i++ {  // 循环n次
        results[i] = net.IPAddr{IP: net.ParseIP(fmt.Sprintf("192.0.2.%d", i))}  // 将IP地址添加到数组中
    }

    backend := &madns.MockResolver{  // 创建一个模拟解析器对象
        IP: map[string][]net.IPAddr{  // 创建一个IP地址映射
            "example.com": results,  // 将结果添加到映射中
        },
    }

    resolver, err := madns.NewResolver(madns.WithDefaultResolver(backend))  // 创建一个新的解析器对象
    if err != nil {  // 如果有错误
        t.Fatal(err)  // 输出错误信息并终止测试
    }
    return resolver  // 返回解析器对象
}

func TestApiEndpointResolveDNSOneResult(t *testing.T) {
    dnsResolver = makeResolver(t, 1)  // 使用makeResolver函数创建解析器对象

    addr, err := resolveAddr(ctx, testAddr)  // 解析地址
    if err != nil {  // 如果有错误
        t.Error(err)  // 输出错误信息
    }

    if ref, _ := ma.NewMultiaddr("/ip4/192.0.2.0/tcp/5001"); !addr.Equal(ref) {  // 如果解析后的地址不等于预期地址
        t.Errorf("resolved address was different than expected")  // 输出错误信息
    }
}

func TestApiEndpointResolveDNSMultipleResults(t *testing.T) {
    dnsResolver = makeResolver(t, 4)  // 使用makeResolver函数创建解析器对象

    addr, err := resolveAddr(ctx, testAddr)  // 解析地址
    if err != nil {  // 如果有错误
        t.Error(err)  // 输出错误信息
    }

    if ref, _ := ma.NewMultiaddr("/ip4/192.0.2.0/tcp/5001"); !addr.Equal(ref) {  // 如果解析后的地址不等于预期地址
        t.Errorf("resolved address was different than expected")  // 输出错误信息
    }
}

func TestApiEndpointResolveDNSNoResults(t *testing.T) {
    dnsResolver = makeResolver(t, 0)  // 使用makeResolver函数创建解析器对象

    addr, err := resolveAddr(ctx, testAddr)  // 解析地址
    if addr != nil || err == nil {  // 如果地址不为空或者没有错误
        t.Error("expected test address not to resolve, and to throw an error")  // 输出错误信息
    }

    if !strings.HasPrefix(err.Error(), "non-resolvable API endpoint") {  // 如果错误信息不以指定字符串开头
        t.Errorf("expected error not thrown; actual: %v", err)  // 输出错误信息
    }
}
```