# `trojan-go\log\golog\colorful\colorful_test.go`

```go
// The color engine for the go-log library
// 版权所有 2017 年 Fadhli Dzil Ikram
//
// 测试文件

package colorful

import (
    "testing"

    . "github.com/smartystreets/goconvey/convey"

    "github.com/p4gefau1t/trojan-go/log/golog/buffer"
)

func TestColorBuffer(t *testing.T) {
    Convey("给定空的颜色缓冲区和测试数据", t, func() {
        var cb ColorBuffer
        var result buffer.Buffer

        // 将颜色添加到结果缓冲区
        result.Append(colorRed)
        result.Append(colorGreen)
        result.Append(colorOrange)
        result.Append(colorBlue)
        result.Append(colorPurple)
        result.Append(colorCyan)
        result.Append(colorGray)
        result.Append(colorOff)

        Convey("当添加颜色时", func() {
            cb.Red()
            cb.Green()
            cb.Orange()
            cb.Blue()
            cb.Purple()
            cb.Cyan()
            cb.Gray()
            cb.Off()

            Convey("它应该与测试数据具有相同的内容", func() {
                So(result.Bytes(), ShouldResemble, cb.Bytes())
            })
        })
    })
}

func TestColorMixer(t *testing.T) {
    })
}
```