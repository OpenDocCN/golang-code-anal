# `v2ray-core\app\dns\nameserver_test.go`

```
package dns_test

import (
    "context"  // 导入 context 包，用于处理上下文
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"     // 导入 time 包，用于处理时间

    . "v2ray.com/core/app/dns"  // 导入 v2ray.com/core/app/dns 包，并使用 . 符号表示在后续代码中可以省略包名
    "v2ray.com/core/common"     // 导入 v2ray.com/core/common 包
)

func TestLocalNameServer(t *testing.T) {
    s := NewLocalNameServer()  // 创建一个本地域名服务器对象
    ctx, cancel := context.WithTimeout(context.Background(), time.Second*2)  // 使用 context 包创建一个带有超时的上下文
    ips, err := s.QueryIP(ctx, "google.com", IPOption{  // 使用域名服务器对象查询指定域名的 IP 地址
        IPv4Enable: true,  // 启用 IPv4 地址查询
        IPv6Enable: true,  // 启用 IPv6 地址查询
    })
    cancel()  // 取消上下文
    common.Must(err)  // 检查错误，如果有错误则触发 panic
    if len(ips) == 0 {  // 如果查询到的 IP 地址数量为 0
        t.Error("expect some ips, but got 0")  // 输出错误信息
    }
}
```