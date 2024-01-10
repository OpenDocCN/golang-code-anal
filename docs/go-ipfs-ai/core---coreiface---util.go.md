# `kubo\core\coreiface\util.go`

```
# 定义一个名为iface的包

# 导入context和io包
import (
    "context"
    "io"
)

# 定义一个接口类型Reader
type Reader interface {
    # 继承ReadSeekCloser接口
    ReadSeekCloser
    # 返回文件大小
    Size() uint64
    # 从上下文中读取指定长度的数据
    CtxReadFull(context.Context, []byte) (int, error)
}

# 定义一个接口类型ReadSeekCloser
# 实现了读取、复制、寻址和关闭的接口
type ReadSeekCloser interface {
    # 继承io.Reader接口
    io.Reader
    # 继承io.Seeker接口
    io.Seeker
    # 继承io.Closer接口
    io.Closer
    # 继承io.WriterTo接口
    io.WriterTo
}
```