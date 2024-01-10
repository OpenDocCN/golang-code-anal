# `v2ray-core\infra\conf\json\reader.go`

```
// 导入必要的包
package json

import (
    "io"
    "v2ray.com/core/common/buf"
)

// State 是解析器的内部状态
type State byte

// 定义解析器的不同状态
const (
    StateContent State = iota
    StateEscape
    StateDoubleQuote
    StateDoubleQuoteEscape
    StateSingleQuote
    StateSingleQuoteEscape
    StateComment
    StateSlash
    StateMultilineComment
    StateMultilineCommentStar
)

// Reader 是用于过滤注释的读取器
// 它支持 Java 风格的单行和多行注释语法，以及 Python 风格的单行注释语法
type Reader struct {
    io.Reader  // 继承自 io.Reader 接口

    state State  // 解析器的当前状态
    br    *buf.BufferedReader  // 缓冲读取器
}

// Read 实现了 io.Reader.Read() 方法。缓冲区必须至少有 3 个字节
func (v *Reader) Read(b []byte) (int, error) {
    if v.br == nil {
        v.br = &buf.BufferedReader{Reader: buf.NewReader(v.Reader)}  // 初始化缓冲读取器
    }

    p := b[:0]  // 初始化 p 为空切片
    // 返回 p 的长度和无错误
    return len(p), nil
}
```