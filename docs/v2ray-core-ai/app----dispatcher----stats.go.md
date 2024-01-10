# `v2ray-core\app\dispatcher\stats.go`

```
// +build !confonly
// 指定构建标签，表示该文件不仅仅是配置文件

package dispatcher
// 声明包名为dispatcher，用于组织代码

import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/features/stats"
)
// 导入所需的包

type SizeStatWriter struct {
    Counter stats.Counter
    Writer  buf.Writer
}
// 定义结构体SizeStatWriter，包含Counter和Writer两个字段

func (w *SizeStatWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    w.Counter.Add(int64(mb.Len()))
    return w.Writer.WriteMultiBuffer(mb)
}
// 定义方法WriteMultiBuffer，用于向Writer写入多个缓冲区，并更新Counter的值

func (w *SizeStatWriter) Close() error {
    return common.Close(w.Writer)
}
// 定义方法Close，用于关闭Writer

func (w *SizeStatWriter) Interrupt() {
    common.Interrupt(w.Writer)
}
// 定义方法Interrupt，用于中断Writer的操作
```