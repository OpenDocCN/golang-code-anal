# `v2ray-core\common\protocol\http\sniff.go`

```go
package http

import (
    "bytes"  // 导入 bytes 包，用于处理字节切片
    "errors"  // 导入 errors 包，用于创建错误
    "strings"  // 导入 strings 包，用于处理字符串

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/net"  // 导入 v2ray.com/core/common/net 包
)

type version byte  // 定义 version 类型为 byte

const (
    HTTP1 version = iota  // 定义 HTTP1 常量为 version 类型的枚举值 iota
    HTTP2  // 定义 HTTP2 常量为 version 类型的枚举值 iota
)

type SniffHeader struct {  // 定义 SniffHeader 结构体
    version version  // 版本号
    host    string  // 主机名
}

func (h *SniffHeader) Protocol() string {  // 定义 SniffHeader 结构体的 Protocol 方法
    switch h.version {  // 根据版本号进行判断
    case HTTP1:  // 如果是 HTTP1 版本
        return "http1"  // 返回字符串 "http1"
    case HTTP2:  // 如果是 HTTP2 版本
        return "http2"  // 返回字符串 "http2"
    default:  // 默认情况
        return "unknown"  // 返回字符串 "unknown"
    }
}

func (h *SniffHeader) Domain() string {  // 定义 SniffHeader 结构体的 Domain 方法
    return h.host  // 返回主机名
}

var (
    methods = [...]string{"get", "post", "head", "put", "delete", "options", "connect"}  // 定义 methods 切片，包含 HTTP 方法

    errNotHTTPMethod = errors.New("not an HTTP method")  // 创建错误变量 errNotHTTPMethod
)

func beginWithHTTPMethod(b []byte) error {  // 定义 beginWithHTTPMethod 函数，接收字节切片参数
    for _, m := range &methods {  // 遍历 methods 切片
        if len(b) >= len(m) && strings.EqualFold(string(b[:len(m)]), m) {  // 判断是否以 HTTP 方法开头
            return nil  // 如果是，返回 nil
        }

        if len(b) < len(m) {  // 如果长度小于 HTTP 方法
            return common.ErrNoClue  // 返回 common 包中的错误变量
        }
    }

    return errNotHTTPMethod  // 返回错误变量 errNotHTTPMethod
}

func SniffHTTP(b []byte) (*SniffHeader, error) {  // 定义 SniffHTTP 函数，接收字节切片参数，返回 SniffHeader 结构体指针和错误
    if err := beginWithHTTPMethod(b); err != nil {  // 调用 beginWithHTTPMethod 函数，判断是否以 HTTP 方法开头
        return nil, err  // 如果不是，返回 nil 和错误
    }

    sh := &SniffHeader{  // 创建 SniffHeader 结构体指针
        version: HTTP1,  // 设置版本号为 HTTP1
    }

    headers := bytes.Split(b, []byte{'\n'})  // 使用换行符对字节切片进行分割
    for i := 1; i < len(headers); i++ {  // 遍历分割后的字节切片
        header := headers[i]  // 获取当前行的字节切片
        if len(header) == 0 {  // 如果长度为 0
            break  // 跳出循环
        }
        parts := bytes.SplitN(header, []byte{':'}, 2)  // 使用冒号对字节切片进行分割
        if len(parts) != 2 {  // 如果分割后的长度不为 2
            continue  // 继续下一次循环
        }
        key := strings.ToLower(string(parts[0]))  // 获取键，并转换为小写
        if key == "host" {  // 如果键为 "host"
            rawHost := strings.ToLower(string(bytes.TrimSpace(parts[1])))  // 获取值，并去除空格后转换为小写
            dest, err := ParseHost(rawHost, net.Port(80))  // 解析主机名和端口
            if err != nil {  // 如果有错误
                return nil, err  // 返回 nil 和错误
            }
            sh.host = dest.Address.String()  // 设置 SniffHeader 结构体的主机名
        }
    }

    if len(sh.host) > 0 {  // 如果主机名长度大于 0
        return sh, nil  // 返回 SniffHeader 结构体指针和 nil
    }

    return nil, common.ErrNoClue  // 返回 nil 和 common 包中的错误变量
}
```