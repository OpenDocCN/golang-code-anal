# `v2ray-core\proxy\vmess\encoding\auth_test.go`

```
package encoding_test

import (
    "crypto/rand"  // 导入加密随机数生成包
    "testing"  // 导入测试包

    "github.com/google/go-cmp/cmp"  // 导入用于比较的包

    "v2ray.com/core/common"  // 导入通用包
    . "v2ray.com/core/proxy/vmess/encoding"  // 导入 VMess 编码包
)

func TestFnvAuth(t *testing.T) {
    fnvAuth := new(FnvAuthenticator)  // 创建 FnvAuthenticator 对象

    expectedText := make([]byte, 256)  // 创建长度为 256 的字节数组
    _, err := rand.Read(expectedText)  // 生成随机字节填充到 expectedText 中
    common.Must(err)  // 检查错误，如果有错误则终止程序

    buffer := make([]byte, 512)  // 创建长度为 512 的字节数组
    b := fnvAuth.Seal(buffer[:0], nil, expectedText, nil)  // 使用 FnvAuthenticator 对象对数据进行加密
    b, err = fnvAuth.Open(buffer[:0], nil, b, nil)  // 使用 FnvAuthenticator 对象对数据进行解密
    common.Must(err)  // 检查错误，如果有错误则终止程序
    if r := cmp.Diff(b, expectedText); r != "" {  // 比较解密后的数据和原始数据
        t.Error(r)  // 如果不相等则输出错误信息
    }
}
```