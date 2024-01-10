# `v2ray-core\common\buf\readv_unix.go`

```
// +build illumos
// 声明当前文件只在 illumos 平台下编译

package buf
// 声明包名为 buf

import "golang.org/x/sys/unix"
// 导入 golang.org/x/sys/unix 包

type unixReader struct {
    iovs [][]byte
}
// 定义结构体 unixReader，包含字段 iovs，类型为二维字节切片

func (r *unixReader) Init(bs []*Buffer) {
    iovs := r.iovs
    // 从结构体中获取字段 iovs
    if iovs == nil {
        iovs = make([][]byte, 0, len(bs))
        // 如果 iovs 为空，则创建一个初始容量为 bs 长度的二维字节切片
    }
    for _, b := range bs {
        iovs = append(iovs, b.v)
        // 遍历 bs，将每个 Buffer 的字节切片添加到 iovs 中
    }
    r.iovs = iovs
    // 将更新后的 iovs 赋值给结构体字段 iovs
}

func (r *unixReader) Read(fd uintptr) int32 {
    n, e := unix.Readv(int(fd), r.iovs)
    // 调用 unix 包的 Readv 方法读取文件描述符 fd 对应的数据到 iovs 中
    if e != nil {
        return -1
        // 如果出现错误，返回 -1
    }
    return int32(n)
    // 返回读取的字节数
}

func (r *unixReader) Clear() {
    r.iovs = r.iovs[:0]
    // 清空结构体字段 iovs
}

func newMultiReader() multiReader {
    return &unixReader{}
    // 返回一个新的 unixReader 结构体指针
}
```