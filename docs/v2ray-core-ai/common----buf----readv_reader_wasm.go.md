# `v2ray-core\common\buf\readv_reader_wasm.go`

```
// +build wasm
// 声明这个文件只在 WebAssembly 架构下编译

package buf
// 声明包名为 buf

import (
    "io"
    "syscall"
)
// 导入 io 和 syscall 包

const useReadv = false
// 声明一个名为 useReadv 的常量，并赋值为 false

func NewReadVReader(reader io.Reader, rawConn syscall.RawConn) Reader {
    // 定义一个名为 NewReadVReader 的函数，接受一个 io.Reader 和 syscall.RawConn 参数，并返回一个 Reader 接口
    panic("not implemented")
    // 抛出一个 panic，提示该函数尚未实现
}
```