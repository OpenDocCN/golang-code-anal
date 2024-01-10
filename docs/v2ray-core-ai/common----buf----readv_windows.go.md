# `v2ray-core\common\buf\readv_windows.go`

```
package buf

import (
    "syscall"
)

type windowsReader struct {
    bufs []syscall.WSABuf
}

func (r *windowsReader) Init(bs []*Buffer) {
    // 如果 bufs 为空，则初始化为长度为 len(bs) 的 WSABuf 切片
    if r.bufs == nil {
        r.bufs = make([]syscall.WSABuf, 0, len(bs))
    }
    // 遍历 bs，将每个 Buffer 的内容封装成 WSABuf，并添加到 bufs 中
    for _, b := range bs {
        r.bufs = append(r.bufs, syscall.WSABuf{Len: uint32(Size), Buf: &b.v[0]})
    }
}

func (r *windowsReader) Clear() {
    // 将 bufs 中每个 WSABuf 的 Buf 置为 nil
    for idx := range r.bufs {
        r.bufs[idx].Buf = nil
    }
    // 清空 bufs
    r.bufs = r.bufs[:0]
}

func (r *windowsReader) Read(fd uintptr) int32 {
    var nBytes uint32
    var flags uint32
    // 从指定的文件描述符 fd 中读取数据到 bufs 中的缓冲区
    err := syscall.WSARecv(syscall.Handle(fd), &r.bufs[0], uint32(len(r.bufs)), &nBytes, &flags, nil, nil)
    // 如果出现错误，则返回 -1
    if err != nil {
        return -1
    }
    // 返回读取的字节数
    return int32(nBytes)
}

func newMultiReader() multiReader {
    // 创建并返回一个新的 windowsReader 对象
    return new(windowsReader)
}
```