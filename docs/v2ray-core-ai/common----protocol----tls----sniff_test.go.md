# `v2ray-core\common\protocol\tls\sniff_test.go`

```go
package tls_test

import (
    "testing"

    . "v2ray.com/core/common/protocol/tls"
)

func TestTLSHeaders(t *testing.T) {
    cases := []struct {  // 定义测试用例结构体数组
        input  []byte      // 输入数据
        domain string      // 域名
        err    bool        // 是否期望出现错误
    }

    for _, test := range cases {  // 遍历测试用例
        header, err := SniffTLS(test.input)  // 使用 SniffTLS 函数检测 TLS 头部
        if test.err {  // 如果期望出现错误
            if err == nil {  // 如果实际上没有错误
                t.Errorf("Exepct error but nil in test %v", test)  // 输出错误信息
            }
        } else {  // 如果不期望出现错误
            if err != nil {  // 如果实际上有错误
                t.Errorf("Expect no error but actually %s in test %v", err.Error(), test)  // 输出错误信息
            }
            if header.Domain() != test.domain {  // 检查返回的域名是否与期望的一致
                t.Error("expect domain ", test.domain, " but got ", header.Domain())  // 输出错误信息
            }
        }
    }
}
```