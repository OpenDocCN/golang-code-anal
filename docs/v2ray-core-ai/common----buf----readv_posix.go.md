# `v2ray-core\common\buf\readv_posix.go`

```go
// 以下是针对非 Windows、非 WebAssembly 和非 Illumos 系统的构建条件
// 定义包名为 buf
package buf

// 导入 syscall 和 unsafe 包
import (
    "syscall"
    "unsafe"
)

// 定义结构体 posixReader
type posixReader struct {
    iovecs []syscall.Iovec
}

// 初始化函数，接收一个 Buffer 切片作为参数
func (r *posixReader) Init(bs []*Buffer) {
    // 将 r.iovecs 赋值给 iovecs
    iovecs := r.iovecs
    // 如果 iovecs 为 nil，则创建一个长度为 len(bs) 的 syscall.Iovec 切片
    if iovecs == nil {
        iovecs = make([]syscall.Iovec, 0, len(bs))
    }
    // 遍历 bs 切片，将每个 Buffer 的 v[0] 地址作为 Base 添加到 iovecs 中
    for idx, b := range bs {
        iovecs = append(iovecs, syscall.Iovec{
            Base: &(b.v[0]),
        })
        iovecs[idx].SetLen(int(Size))
    }
    // 将更新后的 iovecs 赋值给 r.iovecs
    r.iovecs = iovecs
}

// 读取函数，接收一个文件描述符作为参数，返回读取的字节数
func (r *posixReader) Read(fd uintptr) int32 {
    // 调用 syscall.Syscall 执行系统调用 SYS_READV，读取数据到 r.iovecs 中
    n, _, e := syscall.Syscall(syscall.SYS_READV, fd, uintptr(unsafe.Pointer(&r.iovecs[0])), uintptr(len(r.iovecs)))
    // 如果出现错误，返回 -1
    if e != 0 {
        return -1
    }
    // 返回读取的字节数
    return int32(n)
}

// 清空函数，将 r.iovecs 中的 Base 全部置为 nil，并将 r.iovecs 切片长度置为 0
func (r *posixReader) Clear() {
    for idx := range r.iovecs {
        r.iovecs[idx].Base = nil
    }
    r.iovecs = r.iovecs[:0]
}

// 创建多重读取器的函数，返回一个 posixReader 实例
func newMultiReader() multiReader {
    return &posixReader{}
}
```