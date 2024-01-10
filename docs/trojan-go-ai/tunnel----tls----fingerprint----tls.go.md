# `trojan-go\tunnel\tls\fingerprint\tls.go`

```
package fingerprint

import (
    "crypto/tls"  // 导入加密通信包

    "github.com/p4gefau1t/trojan-go/log"  // 导入日志包
)

func ParseCipher(s []string) []uint16 {
    all := tls.CipherSuites()  // 获取所有加密套件
    var result []uint16  // 定义存放结果的数组
    for _, p := range s {  // 遍历输入的加密套件列表
        found := true  // 初始化找到标志为真
        for _, q := range all {  // 遍历所有加密套件
            if q.Name == p {  // 如果找到匹配的加密套件
                result = append(result, q.ID)  // 将加密套件 ID 添加到结果数组中
                break  // 跳出内层循环
            }
            if !found {  // 如果未找到匹配的加密套件
                log.Warn("invalid cipher suite", p, "skipped")  // 记录警告日志
            }
        }
    }
    return result  // 返回结果数组
}
```