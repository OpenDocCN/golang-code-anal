# `v2ray-core\common\strmatcher\benchmark_test.go`

```go
package strmatcher_test

import (
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "testing"  // 导入 testing 包，用于编写测试函数

    "v2ray.com/core/common"  // 导入 v2ray 核心 common 包
    . "v2ray.com/core/common/strmatcher"  // 导入 strmatcher 包，并将其所有公开的函数和变量引入当前命名空间
)

func BenchmarkDomainMatcherGroup(b *testing.B) {
    g := new(DomainMatcherGroup)  // 创建一个新的 DomainMatcherGroup 对象

    for i := 1; i <= 1024; i++ {  // 循环 1024 次
        g.Add(strconv.Itoa(i)+".v2ray.com", uint32(i))  // 向 DomainMatcherGroup 对象中添加域名和对应的值
    }

    b.ResetTimer()  // 重置计时器
    for i := 0; i < b.N; i++ {  // 循环 b.N 次
        _ = g.Match("0.v2ray.com")  // 调用 DomainMatcherGroup 对象的 Match 方法，匹配指定的域名
    }
}

func BenchmarkFullMatcherGroup(b *testing.B) {
    g := new(FullMatcherGroup)  // 创建一个新的 FullMatcherGroup 对象

    for i := 1; i <= 1024; i++ {  // 循环 1024 次
        g.Add(strconv.Itoa(i)+".v2ray.com", uint32(i))  // 向 FullMatcherGroup 对象中添加域名和对应的值
    }

    b.ResetTimer()  // 重置计时器
    for i := 0; i < b.N; i++ {  // 循环 b.N 次
        _ = g.Match("0.v2ray.com")  // 调用 FullMatcherGroup 对象的 Match 方法，匹配指定的域名
    }
}

func BenchmarkMarchGroup(b *testing.B) {
    g := new(MatcherGroup)  // 创建一个新的 MatcherGroup 对象
    for i := 1; i <= 1024; i++ {  // 循环 1024 次
        m, err := Domain.New(strconv.Itoa(i) + ".v2ray.com")  // 创建一个新的域名匹配器
        common.Must(err)  // 检查错误并处理
        g.Add(m)  // 向 MatcherGroup 对象中添加域名匹配器
    }

    b.ResetTimer()  // 重置计时器
    for i := 0; i < b.N; i++ {  // 循环 b.N 次
        _ = g.Match("0.v2ray.com")  // 调用 MatcherGroup 对象的 Match 方法，匹配指定的域名
    }
}
```