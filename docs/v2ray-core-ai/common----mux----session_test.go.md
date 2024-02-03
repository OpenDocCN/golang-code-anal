# `v2ray-core\common\mux\session_test.go`

```go
package mux_test

import (
    "testing"  // 导入测试包
    . "v2ray.com/core/common/mux"  // 导入 v2ray.com/core/common/mux 包，并使用 . 符号代替包名
)

func TestSessionManagerAdd(t *testing.T) {
    m := NewSessionManager()  // 创建一个新的 SessionManager 对象

    s := m.Allocate()  // 分配一个新的 Session 对象
    if s.ID != 1 {  // 如果 Session 对象的 ID 不等于 1
        t.Error("id: ", s.ID)  // 输出错误信息
    }
    if m.Size() != 1 {  // 如果 SessionManager 的大小不等于 1
        t.Error("size: ", m.Size())  // 输出错误信息
    }

    s = m.Allocate()  // 再次分配一个新的 Session 对象
    if s.ID != 2 {  // 如果 Session 对象的 ID 不等于 2
        t.Error("id: ", s.ID)  // 输出错误信息
    }
    if m.Size() != 2 {  // 如果 SessionManager 的大小不等于 2
        t.Error("size: ", m.Size())  // 输出错误信息
    }

    s = &Session{  // 创建一个新的 Session 对象
        ID: 4,  // 设置 ID 为 4
    }
    m.Add(s)  // 将新创建的 Session 对象添加到 SessionManager 中
    if s.ID != 4 {  // 如果 Session 对象的 ID 不等于 4
        t.Error("id: ", s.ID)  // 输出错误信息
    }
    if m.Size() != 3 {  // 如果 SessionManager 的大小不等于 3
        t.Error("size: ", m.Size())  // 输出错误信息
    }
}

func TestSessionManagerClose(t *testing.T) {
    m := NewSessionManager()  // 创建一个新的 SessionManager 对象
    s := m.Allocate()  // 分配一个新的 Session 对象

    if m.CloseIfNoSession() {  // 如果能够关闭 SessionManager
        t.Error("able to close")  // 输出错误信息
    }
    m.Remove(s.ID)  // 移除指定 ID 的 Session 对象
    if !m.CloseIfNoSession() {  // 如果不能关闭 SessionManager
        t.Error("not able to close")  // 输出错误信息
    }
}
```