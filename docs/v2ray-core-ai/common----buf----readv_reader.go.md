# `v2ray-core\common\buf\readv_reader.go`

```
// +build !wasm  // 标记此文件不适用于 WebAssembly 构建

package buf  // 声明包名为 buf

import (  // 导入所需的包
    "io"  // 导入 io 包
    "runtime"  // 导入 runtime 包
    "syscall"  // 导入 syscall 包

    "v2ray.com/core/common/platform"  // 导入 v2ray.com/core/common/platform 包
)

type allocStrategy struct {  // 定义 allocStrategy 结构体
    current uint32  // 声明 current 属性为 uint32 类型
}

func (s *allocStrategy) Current() uint32 {  // 定义 allocStrategy 结构体的 Current 方法
    return s.current  // 返回当前值
}

func (s *allocStrategy) Adjust(n uint32) {  // 定义 allocStrategy 结构体的 Adjust 方法
    if n >= s.current {  // 如果 n 大于等于当前值
        s.current *= 4  // 当前值乘以 4
    } else {  // 否则
        s.current = n  // 当前值等于 n
    }

    if s.current > 32 {  // 如果当前值大于 32
        s.current = 32  // 当前值等于 32
    }

    if s.current == 0 {  // 如果当前值等于 0
        s.current = 1  // 当前值等于 1
    }
}

func (s *allocStrategy) Alloc() []*Buffer {  // 定义 allocStrategy 结构体的 Alloc 方法
    bs := make([]*Buffer, s.current)  // 创建长度为当前值的 Buffer 切片
    for i := range bs {  // 遍历切片
        bs[i] = New()  // 初始化每个元素
    }
    return bs  // 返回切片
}

type multiReader interface {  // 定义 multiReader 接口
    Init([]*Buffer)  // 初始化方法
    Read(fd uintptr) int32  // 读取方法
    Clear()  // 清除方法
}

// ReadVReader is a Reader that uses readv(2) syscall to read data.
type ReadVReader struct {  // 定义 ReadVReader 结构体
    io.Reader  // 继承 io.Reader 接口
    rawConn syscall.RawConn  // 原始连接
    mr multiReader  // 多重读取接口
    alloc allocStrategy  // 分配策略
}

// NewReadVReader creates a new ReadVReader.
func NewReadVReader(reader io.Reader, rawConn syscall.RawConn) *ReadVReader {  // 定义创建 ReadVReader 的方法
    return &ReadVReader{  // 返回 ReadVReader 对象
        Reader:  reader,  // 初始化 Reader 属性
        rawConn: rawConn,  // 初始化 rawConn 属性
        alloc: allocStrategy{  // 初始化 alloc 属性
            current: 1,  // 当前值为 1
        },
        mr: newMultiReader(),  // 初始化 mr 属性
    }
}

func (r *ReadVReader) readMulti() (MultiBuffer, error) {  // 定义 readMulti 方法
    bs := r.alloc.Alloc()  // 分配 Buffer 切片

    r.mr.Init(bs)  // 初始化多重读取接口
    var nBytes int32  // 声明 nBytes 变量为 int32 类型
    err := r.rawConn.Read(func(fd uintptr) bool {  // 调用原始连接的读取方法
        n := r.mr.Read(fd)  // 读取数据
        if n < 0 {  // 如果读取失败
            return false  // 返回 false
        }

        nBytes = n  // 将读取的字节数赋值给 nBytes
        return true  // 返回 true
    })
    r.mr.Clear()  // 清除多重读取接口

    if err != nil {  // 如果有错误
        ReleaseMulti(MultiBuffer(bs))  // 释放多重缓冲
        return nil, err  // 返回空值和错误
    }

    if nBytes == 0 {  // 如果没有读取到数据
        ReleaseMulti(MultiBuffer(bs))  // 释放多重缓冲
        return nil, io.EOF  // 返回空值和文件结束错误
    }

    nBuf := 0  // 声明 nBuf 变量为 0
    for nBuf < len(bs) {  // 当 nBuf 小于 bs 的长度时循环
        if nBytes <= 0 {  // 如果没有剩余字节数
            break  // 跳出循环
        }
        end := nBytes  // 结束位置为当前剩余字节数
        if end > Size {  // 如果结束位置大于 Size
            end = Size  // 结束位置为 Size
        }
        bs[nBuf].end = end  // 设置当前 Buffer 的结束位置
        nBytes -= end  // 减去已经处理的字节数
        nBuf++  // nBuf 自增
    }
}
    # 从 nBuf 开始遍历 bs 数组
    for i := nBuf; i < len(bs); i++ {
        # 释放 bs[i] 的资源
        bs[i].Release()
        # 将 bs[i] 置为 nil
        bs[i] = nil
    }
    # 返回 bs 数组中前 nBuf 个元素组成的 MultiBuffer 对象和空值
    return MultiBuffer(bs[:nBuf]), nil
// ReadMultiBuffer 实现了 Reader 接口
func (r *ReadVReader) ReadMultiBuffer() (MultiBuffer, error) {
    // 如果分配器当前的缓冲区数量为1
    if r.alloc.Current() == 1 {
        // 读取一个缓冲区
        b, err := ReadBuffer(r.Reader)
        // 如果缓冲区已满
        if b.IsFull() {
            // 调整分配器的缓冲区数量为1
            r.alloc.Adjust(1)
        }
        // 返回一个包含该缓冲区的多缓冲区和可能的错误
        return MultiBuffer{b}, err
    }

    // 读取多个缓冲区
    mb, err := r.readMulti()
    // 如果有错误，返回空和错误
    if err != nil {
        return nil, err
    }
    // 调整分配器的缓冲区数量为多缓冲区的长度
    r.alloc.Adjust(uint32(len(mb)))
    // 返回多缓冲区和可能的错误
    return mb, nil
}

// 初始化函数
func init() {
    // 定义默认标志值
    const defaultFlagValue = "NOT_DEFINED_AT_ALL"
    // 获取环境标志值
    value := platform.NewEnvFlag("v2ray.buf.readv").GetValue(func() string { return defaultFlagValue })
    // 根据标志值进行判断
    switch value {
    // 如果标志值为默认值或者"auto"
    case defaultFlagValue, "auto":
        // 如果运行时架构为 "386"、"amd64"、"s390x"，操作系统为 "linux"、"darwin"、"windows"
        if (runtime.GOARCH == "386" || runtime.GOARCH == "amd64" || runtime.GOARCH == "s390x") && (runtime.GOOS == "linux" || runtime.GOOS == "darwin" || runtime.GOOS == "windows") {
            // 设置 useReadv 为 true
            useReadv = true
        }
    // 如果标志值为"enable"
    case "enable":
        // 设置 useReadv 为 true
        useReadv = true
    }
}
```