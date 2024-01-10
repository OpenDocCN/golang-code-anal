# `trojan-go\log\golog\colorful\colorful.go`

```
// go-log库的颜色引擎
// 版权所有 2017年 Fadhli Dzil Ikram

package colorful

import (
    "runtime"

    "github.com/p4gefau1t/trojan-go/log/golog/buffer"
)

// ColorBuffer为缓冲区附加颜色选项
type ColorBuffer struct {
    buffer.Buffer
}

// 颜色调色板映射
var (
    colorOff    = []byte("\033[0m")  // 关闭颜色
    colorRed    = []byte("\033[0;31m")  // 红色
    colorGreen  = []byte("\033[0;32m")  // 绿色
    colorOrange = []byte("\033[0;33m")  // 橙色
    colorBlue   = []byte("\033[0;34m")  // 蓝色
    colorPurple = []byte("\033[0;35m")  // 紫色
    colorCyan   = []byte("\033[0;36m")  // 青色
    colorGray   = []byte("\033[0;37m")  // 灰色
)

func init() {
    if runtime.GOOS != "linux" {
        colorOff = []byte("")  // 关闭颜色
        colorRed = []byte("")  // 红色
        colorGreen = []byte("")  // 绿色
        colorOrange = []byte("")  // 橙色
        colorBlue = []byte("")  // 蓝色
        colorPurple = []byte("")  // 紫色
        colorCyan = []byte("")  // 青色
        colorGray = []byte("")  // 灰色
    }
}

// Off将数据应用无颜色
func (cb *ColorBuffer) Off() {
    cb.Append(colorOff)
}

// Red将数据应用红色
func (cb *ColorBuffer) Red() {
    cb.Append(colorRed)
}

// Green将数据应用绿色
func (cb *ColorBuffer) Green() {
    cb.Append(colorGreen)
}

// Orange将数据应用橙色
func (cb *ColorBuffer) Orange() {
    cb.Append(colorOrange)
}

// Blue将数据应用蓝色
func (cb *ColorBuffer) Blue() {
    cb.Append(colorBlue)
}

// Purple将数据应用紫色
func (cb *ColorBuffer) Purple() {
    cb.Append(colorPurple)
}

// Cyan将数据应用青色
func (cb *ColorBuffer) Cyan() {
    cb.Append(colorCyan)
}

// Gray将数据应用灰色
func (cb *ColorBuffer) Gray() {
    cb.Append(colorGray)
}

// mixer将颜色打开和关闭字节与实际数据混合
func mixer(data []byte, color []byte) []byte {
    var result []byte
    return append(append(append(result, color...), data...), colorOff...)
}

// Red将数据应用红色
func Red(data []byte) []byte {
    # 调用mixer函数，传入data和colorRed作为参数，并返回结果
    return mixer(data, colorRed)
// Green 将数据应用绿色
func Green(data []byte) []byte {
    return mixer(data, colorGreen)
}

// Orange 将数据应用橙色
func Orange(data []byte) []byte {
    return mixer(data, colorOrange)
}

// Blue 将数据应用蓝色
func Blue(data []byte) []byte {
    return mixer(data, colorBlue)
}

// Purple 将数据应用紫色
func Purple(data []byte) []byte {
    return mixer(data, colorPurple)
}

// Cyan 将数据应用青色
func Cyan(data []byte) []byte {
    return mixer(data, colorCyan)
}

// Gray 将数据应用灰色
func Gray(data []byte) []byte {
    return mixer(data, colorGray)
}
```