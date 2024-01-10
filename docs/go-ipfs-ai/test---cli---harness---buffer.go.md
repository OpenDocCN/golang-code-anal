# `kubo\test\cli\harness\buffer.go`

```
package harness

import (
    "strings"  // 导入字符串操作包
    "sync"     // 导入同步操作包

    "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
)

// Buffer is a thread-safe byte buffer.
type Buffer struct {  // 定义一个线程安全的字节缓冲区结构体
    b strings.Builder  // 使用字符串构建器作为缓冲区
    m sync.Mutex       // 使用互斥锁保证线程安全
}

func (b *Buffer) Write(p []byte) (n int, err error) {  // 定义向缓冲区写入数据的方法
    b.m.Lock()  // 加锁
    defer b.m.Unlock()  // 延迟解锁
    return b.b.Write(p)  // 将数据写入缓冲区
}

func (b *Buffer) String() string {  // 定义获取缓冲区内容的方法
    b.m.Lock()  // 加锁
    defer b.m.Unlock()  // 延迟解锁
    return b.b.String()  // 返回缓冲区内容
}

// Trimmed returns the bytes as a string, but with the trailing newline removed.
// This only removes a single trailing newline, not all whitespace.
func (b *Buffer) Trimmed() string {  // 定义获取去除尾部换行符的内容的方法
    b.m.Lock()  // 加锁
    defer b.m.Unlock()  // 延迟解锁
    s := b.b.String()  // 获取缓冲区内容
    if len(s) == 0 {  // 如果内容为空
        return s  // 直接返回
    }
    if s[len(s)-1] == '\n' {  // 如果最后一个字符是换行符
        return s[:len(s)-1]  // 返回去除换行符的内容
    }
    return s  // 返回原内容
}

func (b *Buffer) Bytes() []byte {  // 定义获取缓冲区内容的字节数组的方法
    b.m.Lock()  // 加锁
    defer b.m.Unlock()  // 延迟解锁
    return []byte(b.b.String())  // 返回缓冲区内容的字节数组
}

func (b *Buffer) Lines() []string {  // 定义获取缓冲区内容的每行字符串的方法
    b.m.Lock()  // 加锁
    defer b.m.Unlock()  // 延迟解锁
    return testutils.SplitLines(b.b.String())  // 使用测试工具包中的方法将缓冲区内容按行分割并返回
}
```