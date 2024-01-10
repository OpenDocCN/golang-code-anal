# `v2ray-core\common\crypto\internal\chacha_core_gen.go`

```
// +build generate

package main

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "log" // 导入 log 包，用于记录错误日志
    "os"  // 导入 os 包，用于文件操作
)

func writeQuarterRound(file *os.File, a, b, c, d int) {
    add := "x%d+=x%d\n" // 定义字符串模板，用于格式化输出
    xor := "x=x%d^x%d\n" // 定义字符串模板，用于格式化输出
    rotate := "x%d=(x << %d) | (x >> (32 - %d))\n" // 定义字符串模板，用于格式化输出

    fmt.Fprintf(file, add, a, b) // 格式化输出字符串到文件
    fmt.Fprintf(file, xor, d, a) // 格式化输出字符串到文件
    fmt.Fprintf(file, rotate, d, 16, 16) // 格式化输出字符串到文件

    fmt.Fprintf(file, add, c, d) // 格式化输出字符串到文件
    fmt.Fprintf(file, xor, b, c) // 格式化输出字符串到文件
    fmt.Fprintf(file, rotate, b, 12, 12) // 格式化输出字符串到文件

    fmt.Fprintf(file, add, a, b) // 格式化输出字符串到文件
    fmt.Fprintf(file, xor, d, a) // 格式化输出字符串到文件
    fmt.Fprintf(file, rotate, d, 8, 8) // 格式化输出字符串到文件

    fmt.Fprintf(file, add, c, d) // 格式化输出字符串到文件
    fmt.Fprintf(file, xor, b, c) // 格式化输出字符串到文件
    fmt.Fprintf(file, rotate, b, 7, 7) // 格式化输出字符串到文件
}

func writeChacha20Block(file *os.File) {
    fmt.Fprintln(file, `
func ChaCha20Block(s *[16]uint32, out []byte, rounds int) {
  var x0,x1,x2,x3,x4,x5,x6,x7,x8,x9,x10,x11,x12,x13,x14,x15 = s[0],s[1],s[2],s[3],s[4],s[5],s[6],s[7],s[8],s[9],s[10],s[11],s[12],s[13],s[14],s[15]
    for i := 0; i < rounds; i+=2 {
    var x uint32
    `) // 格式化输出多行字符串到文件

    writeQuarterRound(file, 0, 4, 8, 12) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 1, 5, 9, 13) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 2, 6, 10, 14) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 3, 7, 11, 15) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 0, 5, 10, 15) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 1, 6, 11, 12) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 2, 7, 8, 13) // 调用函数，向文件中写入内容
    writeQuarterRound(file, 3, 4, 9, 14) // 调用函数，向文件中写入内容
    fmt.Fprintln(file, "}") // 向文件中写入内容
    for i := 0; i < 16; i++ {
        fmt.Fprintf(file, "binary.LittleEndian.PutUint32(out[%d:%d], s[%d]+x%d)\n", i*4, i*4+4, i, i) // 格式化输出字符串到文件
    }
    fmt.Fprintln(file, "}") // 向文件中写入内容
    fmt.Fprintln(file) // 向文件中写入空行
}

func main() {
    file, err := os.OpenFile("chacha_core.generated.go", os.O_WRONLY|os.O_TRUNC|os.O_CREATE, 0644) // 打开文件，如果不存在则创建，如果存在则清空内容
    if err != nil {
        log.Fatalf("Failed to generate chacha_core.go: %v", err) // 记录错误日志并退出程序
    }
    defer file.Close() // 延迟关闭文件

    fmt.Fprintln(file, "package internal") // 向文件中写入内容
    fmt.Fprintln(file) // 向文件中写入空行
    fmt.Fprintln(file, "import \"encoding/binary\"") // 向文件中写入内容
    fmt.Fprintln(file) // 向文件中写入空行
    writeChacha20Block(file) // 调用函数，向文件中写入内容
}
```