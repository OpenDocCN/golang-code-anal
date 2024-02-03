# `trojan-go\redirector\redirector_test.go`

```go
package redirector

import (
    "context"  // 导入上下文包，用于控制程序的上下文
    "fmt"  // 导入格式化包，用于格式化输出
    "net"  // 导入网络包，用于网络操作
    "net/http"  // 导入 HTTP 包，用于处理 HTTP 请求
    "strings"  // 导入字符串包，用于字符串操作
    "testing"  // 导入测试包，用于编写测试函数
    "time"  // 导入时间包，用于时间相关操作

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包，用于通用功能
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入测试工具包，用于测试工具函数
)

func TestRedirector(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消函数的上下文
    redir := NewRedirector(ctx)  // 创建一个重定向器对象
    redir.Redirect(&Redirection{  // 调用重定向器的重定向方法，传入重定向结构体
        Dial:        nil,  // 设置拨号函数为 nil
        RedirectTo:  nil,  // 设置重定向地址为 nil
        InboundConn: nil,  // 设置入站连接为 nil
    })
    var fakeAddr net.Addr  // 声明一个网络地址变量
    var fakeConn net.Conn  // 声明一个网络连接变量
    redir.Redirect(&Redirection{  // 再次调用重定向方法，传入新的重定向结构体
        Dial:        nil,  // 设置拨号函数为 nil
        RedirectTo:  fakeAddr,  // 设置重定向地址为 fakeAddr
        InboundConn: fakeConn,  // 设置入站连接为 fakeConn
    })
    redir.Redirect(&Redirection{  // 再次调用重定向方法，传入新的重定向结构体
        Dial:        nil,  // 设置拨号函数为 nil
        RedirectTo:  nil,  // 设置重定向地址为 nil
        InboundConn: fakeConn,  // 设置入站连接为 fakeConn
    })
    redir.Redirect(&Redirection{  // 再次调用重定向方法，传入新的重定向结构体
        Dial:        nil,  // 设置拨号函数为 nil
        RedirectTo:  fakeAddr,  // 设置重定向地址为 fakeAddr
        InboundConn: nil,  // 设置入站连接为 nil
    })
    l, err := net.Listen("tcp", "127.0.0.1:0")  // 在本地监听 TCP 连接
    common.Must(err)  // 检查错误并处理
    conn1, err := net.Dial("tcp", l.Addr().String())  // 通过 TCP 连接到监听的地址
    common.Must(err)  // 检查错误并处理
    conn2, err := l.Accept()  // 接受传入的连接
    common.Must(err)  // 检查错误并处理
    redirAddr, err := net.ResolveTCPAddr("tcp", util.HTTPAddr)  // 解析 TCP 地址
    common.Must(err)  // 检查错误并处理
    redir.Redirect(&Redirection{  // 再次调用重定向方法，传入新的重定向结构体
        Dial:        nil,  // 设置拨号函数为 nil
        RedirectTo:  redirAddr,  // 设置重定向地址为 redirAddr
        InboundConn: conn2,  // 设置入站连接为 conn2
    })
    time.Sleep(time.Second)  // 休眠一秒
    req, err := http.NewRequest("GET", "http://localhost/", nil)  // 创建一个 HTTP 请求
    common.Must(err)  // 检查错误并处理
    req.Write(conn1)  // 将请求写入连接
    buf := make([]byte, 1024)  // 创建一个字节切片
    conn1.Read(buf)  // 从连接中读取数据到字节切片
    fmt.Println(string(buf))  // 格式化输出字节切片的内容
    if !strings.HasPrefix(string(buf), "HTTP/1.1 200 OK") {  // 检查响应是否以指定字符串开头
        t.Fail()  // 测试失败
    }
    cancel()  // 取消上下文
    conn1.Close()  // 关闭连接1
    conn2.Close()  // 关闭连接2
}
```