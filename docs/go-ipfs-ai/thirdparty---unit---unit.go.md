# `kubo\thirdparty\unit\unit.go`

```go
package unit

import "fmt"

type Information int64

const (
    _  Information = iota // ignore first value by assigning to blank identifier
    KB             = 1 << (10 * iota)  // 1KB的字节数
    MB                                  // 1MB的字节数
    GB                                  // 1GB的字节数
    TB                                  // 1TB的字节数
    PB                                  // 1PB的字节数
    EB                                  // 1EB的字节数
)

func (i Information) String() string {
    tmp := int64(i)

    // default
    d := tmp  // 默认值为tmp
    symbol := "B"  // 默认单位为字节

    switch {
    case i > EB:  // 如果大于1EB
        d = tmp / EB  // 将字节数转换为EB单位
        symbol = "EB"  // 单位为EB
    case i > PB:  // 如果大于1PB
        d = tmp / PB  // 将字节数转换为PB单位
        symbol = "PB"  // 单位为PB
    case i > TB:  // 如果大于1TB
        d = tmp / TB  // 将字节数转换为TB单位
        symbol = "TB"  // 单位为TB
    case i > GB:  // 如果大于1GB
        d = tmp / GB  // 将字节数转换为GB单位
        symbol = "GB"  // 单位为GB
    case i > MB:  // 如果大于1MB
        d = tmp / MB  // 将字节数转换为MB单位
        symbol = "MB"  // 单位为MB
    case i > KB:  // 如果大于1KB
        d = tmp / KB  // 将字节数转换为KB单位
        symbol = "KB"  // 单位为KB
    }
    return fmt.Sprintf("%d %s", d, symbol)  // 返回转换后的值和单位
}
```