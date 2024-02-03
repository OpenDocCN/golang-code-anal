# `v2ray-core\common\protocol\tls\sniff.go`

```go
package tls

import (
    "encoding/binary"  // 导入二进制编码包
    "errors"  // 导入错误处理包
    "strings"  // 导入字符串处理包

    "v2ray.com/core/common"  // 导入自定义包
)

type SniffHeader struct {
    domain string  // 定义域名字段
}

func (h *SniffHeader) Protocol() string {
    return "tls"  // 返回协议类型为 TLS
}

func (h *SniffHeader) Domain() string {
    return h.domain  // 返回域名字段的值
}

var errNotTLS = errors.New("not TLS header")  // 定义错误变量，表示不是 TLS 头部
var errNotClientHello = errors.New("not client hello")  // 定义错误变量，表示不是客户端 hello 消息

func IsValidTLSVersion(major, minor byte) bool {
    return major == 3  // 判断 TLS 版本是否为 3
}

// ReadClientHello returns server name (if any) from TLS client hello message.
// https://github.com/golang/go/blob/master/src/crypto/tls/handshake_messages.go#L300
func ReadClientHello(data []byte, h *SniffHeader) error {
    if len(data) < 42 {
        return common.ErrNoClue  // 如果数据长度小于 42，则返回无法确定的错误
    }
    sessionIDLen := int(data[38])  // 获取会话 ID 的长度
    if sessionIDLen > 32 || len(data) < 39+sessionIDLen {
        return common.ErrNoClue  // 如果会话 ID 长度大于 32或者数据长度小于 39+会话 ID 长度，则返回无法确定的错误
    }
    data = data[39+sessionIDLen:]  // 截取数据
    if len(data) < 2 {
        return common.ErrNoClue  // 如果数据长度小于 2，则返回无法确定的错误
    }
    // cipherSuiteLen is the number of bytes of cipher suite numbers. Since
    // they are uint16s, the number must be even.
    cipherSuiteLen := int(data[0])<<8 | int(data[1])  // 计算密码套件的长度
    if cipherSuiteLen%2 == 1 || len(data) < 2+cipherSuiteLen {
        return errNotClientHello  // 如果密码套件长度为奇数或者数据长度小于 2+密码套件长度，则返回不是客户端 hello 消息的错误
    }
    data = data[2+cipherSuiteLen:]  // 截取数据
    if len(data) < 1 {
        return common.ErrNoClue  // 如果数据长度小于 1，则返回无法确定的错误
    }
    compressionMethodsLen := int(data[0])  // 获取压缩方法的长度
    if len(data) < 1+compressionMethodsLen {
        return common.ErrNoClue  // 如果数据长度小于 1+压缩方法长度，则返回无法确定的错误
    }
    data = data[1+compressionMethodsLen:]  // 截取数据

    if len(data) == 0 {
        return errNotClientHello  // 如果数据长度为 0，则返回不是客户端 hello 消息的错误
    }
    if len(data) < 2 {
        return errNotClientHello  // 如果数据长度小于 2，则返回不是客户端 hello 消息的错误
    }

    extensionsLength := int(data[0])<<8 | int(data[1])  // 获取扩展长度
    data = data[2:]  // 截取数据
    if extensionsLength != len(data) {
        return errNotClientHello  // 如果扩展长度不等于数据长度，则返回不是客户端 hello 消息的错误
    }
    # 循环直到数据长度为0
    for len(data) != 0 {
        # 如果数据长度小于4，则返回错误
        if len(data) < 4 {
            return errNotClientHello
        }
        # 读取扩展字段和长度
        extension := uint16(data[0])<<8 | uint16(data[1])
        length := int(data[2])<<8 | int(data[3])
        data = data[4:]
        # 如果数据长度小于扩展字段长度，则返回错误
        if len(data) < length {
            return errNotClientHello
        }

        # 如果扩展字段为0x00（extensionServerName）
        if extension == 0x00 { 
            # 读取扩展字段中的数据
            d := data[:length]
            # 如果数据长度小于2，则返回错误
            if len(d) < 2 {
                return errNotClientHello
            }
            # 读取服务器名称列表的长度
            namesLen := int(d[0])<<8 | int(d[1])
            d = d[2:]
            # 如果数据长度不等于名称列表长度，则返回错误
            if len(d) != namesLen {
                return errNotClientHello
            }
            # 遍历服务器名称列表
            for len(d) > 0 {
                # 如果数据长度小于3，则返回错误
                if len(d) < 3 {
                    return errNotClientHello
                }
                # 读取名称类型和名称长度
                nameType := d[0]
                nameLen := int(d[1])<<8 | int(d[2])
                d = d[3:]
                # 如果数据长度小于名称长度，则返回错误
                if len(d) < nameLen {
                    return errNotClientHello
                }
                # 如果名称类型为0
                if nameType == 0 {
                    # 读取服务器名称并检查是否包含结尾的点
                    serverName := string(d[:nameLen])
                    if strings.HasSuffix(serverName, ".") {
                        return errNotClientHello
                    }
                    # 设置域名并返回
                    h.domain = serverName
                    return nil
                }
                d = d[nameLen:]
            }
        }
        # 更新数据
        data = data[length:]
    }
    # 返回TLS错误
    return errNotTLS
# 判断传入的字节流是否为 TLS 协议
func SniffTLS(b []byte) (*SniffHeader, error):
    # 如果字节流长度小于 5，则返回无法判断的错误
    if len(b) < 5:
        return nil, common.ErrNoClue
    # 如果字节流的第一个字节不是 0x16（TLS 握手），则返回不是 TLS 协议的错误
    if b[0] != 0x16 /* TLS Handshake */:
        return nil, errNotTLS
    # 如果字节流的第二个和第三个字节不是有效的 TLS 版本，则返回不是 TLS 协议的错误
    if !IsValidTLSVersion(b[1], b[2]):
        return nil, errNotTLS
    # 读取头部长度，并判断是否超出字节流长度，如果超出则返回无法判断的错误
    headerLen := int(binary.BigEndian.Uint16(b[3:5]))
    if 5+headerLen > len(b):
        return nil, common.ErrNoClue

    # 创建 SniffHeader 对象
    h := &SniffHeader{}
    # 读取客户端 Hello 消息，如果成功则返回 SniffHeader 对象，否则返回错误
    err := ReadClientHello(b[5:5+headerLen], h)
    if err == nil:
        return h, nil
    return nil, err
```