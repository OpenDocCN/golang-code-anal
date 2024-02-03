# `grype\grype\cpe\cpe.go`

```go
package cpe

import (
    "github.com/anchore/grype/internal/log"  // 导入日志模块
    "github.com/anchore/syft/syft/cpe"  // 导入CPE模块
)

func NewSlice(cpeStrs ...string) ([]cpe.CPE, error) {
    var cpes []cpe.CPE  // 声明一个CPE类型的切片
    for _, c := range cpeStrs {  // 遍历传入的CPE字符串
        value, err := cpe.New(c)  // 根据CPE字符串创建CPE对象
        if err != nil {  // 如果创建CPE对象出错
            log.Warnf("excluding invalid CPE %q: %v", c, err)  // 记录警告日志
            continue  // 继续下一次循环
        }

        cpes = append(cpes, value)  // 将创建的CPE对象添加到切片中
    }
    return cpes, nil  // 返回CPE对象切片和nil
}

func MatchWithoutVersion(c cpe.CPE, candidates []cpe.CPE) []cpe.CPE {
    matches := make([]cpe.CPE, 0)  // 声明一个CPE类型的切片
    for _, candidate := range candidates {  // 遍历候选CPE对象切片
        canCopy := candidate  // 复制候选CPE对象
        if c.MatchWithoutVersion(&canCopy) {  // 判断是否匹配（不考虑版本）
            matches = append(matches, candidate)  // 将匹配的CPE对象添加到切片中
        }
    }
    return matches  // 返回匹配的CPE对象切片
}
```