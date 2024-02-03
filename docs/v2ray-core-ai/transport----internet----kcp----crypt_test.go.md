# `v2ray-core\transport\internet\kcp\crypt_test.go`

```go
package kcp_test

import (
    "testing"  // 导入测试包
    "github.com/google/go-cmp/cmp"  // 导入用于比较的包
    "v2ray.com/core/common"  // 导入通用包
    . "v2ray.com/core/transport/internet/kcp"  // 导入 KCP 传输协议包
)

func TestSimpleAuthenticator(t *testing.T) {
    cache := make([]byte, 512)  // 创建一个长度为 512 的字节切片

    payload := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g'}  // 创建一个包含字符的字节切片

    auth := NewSimpleAuthenticator()  // 创建一个简单认证器对象
    b := auth.Seal(cache[:0], nil, payload, nil)  // 使用认证器对 payload 进行加密
    c, err := auth.Open(cache[:0], nil, b, nil)  // 使用认证器对加密后的数据进行解密
    common.Must(err)  // 检查是否有错误发生
    if r := cmp.Diff(c, payload); r != "" {  // 比较解密后的数据和原始数据是否一致
        t.Error(r)  // 如果不一致，则输出差异信息
    }
}

func TestSimpleAuthenticator2(t *testing.T) {
    cache := make([]byte, 512)  // 创建一个长度为 512 的字节切片

    payload := []byte{'a', 'b'}  // 创建一个包含字符的字节切片

    auth := NewSimpleAuthenticator()  // 创建一个简单认证器对象
    b := auth.Seal(cache[:0], nil, payload, nil)  // 使用认证器对 payload 进行加密
    c, err := auth.Open(cache[:0], nil, b, nil)  // 使用认证器对加密后的数据进行解密
    common.Must(err)  // 检查是否有错误发生
    if r := cmp.Diff(c, payload); r != "" {  // 比较解密后的数据和原始数据是否一致
        t.Error(r)  // 如果不一致，则输出差异信息
    }
}
```