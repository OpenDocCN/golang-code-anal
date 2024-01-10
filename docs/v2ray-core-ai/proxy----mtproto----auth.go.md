# `v2ray-core\proxy\mtproto\auth.go`

```
package mtproto

import (
    "context"  // 导入 context 包，用于处理请求的上下文
    "crypto/rand"  // 导入 crypto/rand 包，用于生成随机数
    "crypto/sha256"  // 导入 crypto/sha256 包，用于 SHA-256 哈希算法
    "io"  // 导入 io 包，用于实现 I/O 操作
    "sync"  // 导入 sync 包，用于实现同步操作

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
)

const (
    HeaderSize = 64  // 定义常量 HeaderSize 为 64
)

type SessionContext struct {
    ConnectionType [4]byte  // 定义结构体 SessionContext 包含 ConnectionType 和 DataCenterID
    DataCenterID   uint16
}

func DefaultSessionContext() SessionContext {
    return SessionContext{  // 返回默认的 SessionContext 结构体
        ConnectionType: [4]byte{0xef, 0xef, 0xef, 0xef},  // 默认 ConnectionType 为 [0xef, 0xef, 0xef, 0xef]
        DataCenterID:   0,  // 默认 DataCenterID 为 0
    }
}

type contextKey int32  // 定义类型 contextKey 为 int32

const (
    sessionContextKey contextKey = iota  // 定义常量 sessionContextKey 为 contextKey 类型的枚举值 iota
)

func ContextWithSessionContext(ctx context.Context, c SessionContext) context.Context {
    return context.WithValue(ctx, sessionContextKey, c)  // 使用 context.WithValue 将 SessionContext 存储到上下文中
}

func SessionContextFromContext(ctx context.Context) SessionContext {
    if c := ctx.Value(sessionContextKey); c != nil {  // 从上下文中获取 SessionContext
        return c.(SessionContext)
    }
    return DefaultSessionContext()  // 如果不存在，则返回默认的 SessionContext
}

type Authentication struct {
    Header        [HeaderSize]byte  // 定义结构体 Authentication 包含 Header, DecodingKey, EncodingKey, DecodingNonce, EncodingNonce
    DecodingKey   [32]byte
    EncodingKey   [32]byte
    DecodingNonce [16]byte
    EncodingNonce [16]byte
}

func (a *Authentication) DataCenterID() uint16 {
    x := ((int16(a.Header[61]) << 8) | int16(a.Header[60]))  // 计算 DataCenterID
    if x < 0 {
        x = -x
    }
    return uint16(x) - 1
}

func (a *Authentication) ConnectionType() [4]byte {
    var x [4]byte
    copy(x[:], a.Header[56:60])  // 获取 ConnectionType
    return x
}

func (a *Authentication) ApplySecret(b []byte) {
    a.DecodingKey = sha256.Sum256(append(a.DecodingKey[:], b...))  // 应用密钥到解码密钥
    a.EncodingKey = sha256.Sum256(append(a.EncodingKey[:], b...))  // 应用密钥到编码密钥
}

func generateRandomBytes(random []byte, connType [4]byte) {
    // 生成随机字节
}
    # 无限循环，直到满足条件才会退出
    for {
        # 生成随机数填充到 random 数组中
        common.Must2(rand.Read(random))

        # 如果随机数组的第一个元素为 0xef，则继续下一次循环
        if random[0] == 0xef {
            continue
        }

        # 将随机数组中的四个字节转换为一个 uint32 类型的值
        val := (uint32(random[3]) << 24) | (uint32(random[2]) << 16) | (uint32(random[1]) << 8) | uint32(random[0])

        # 如果 val 符合指定的数值，则继续下一次循环
        if val == 0x44414548 || val == 0x54534f50 || val == 0x20544547 || val == 0x4954504f || val == 0xeeeeeeee {
            continue
        }

        # 如果随机数组中的四个字节转换为的 uint32 值为 0x00000000，则继续下一次循环
        if (uint32(random[7])<<24)|(uint32(random[6])<<16)|(uint32(random[5])<<8)|uint32(random[4]) == 0x00000000 {
            continue
        }

        # 将 connType 数组的内容复制到 random 数组的指定位置
        copy(random[56:60], connType[:])

        # 退出循环
        return
    }
// NewAuthentication 创建一个新的认证对象，并根据会话上下文生成随机数，设置编码和解码密钥和随机数，然后返回认证对象
func NewAuthentication(sc SessionContext) *Authentication {
    // 获取一个认证对象
    auth := getAuthenticationObject()
    // 从认证对象的头部获取随机数
    random := auth.Header[:]
    // 根据连接类型生成随机字节
    generateRandomBytes(random, sc.ConnectionType)
    // 将随机数的一部分复制到编码密钥
    copy(auth.EncodingKey[:], random[8:])
    // 将随机数的一部分复制到编码随机数
    copy(auth.EncodingNonce[:], random[8+32:])
    // 计算随机数的逆序，并将一部分复制到解码密钥
    keyivInverse := Inverse(random[8 : 8+32+16])
    copy(auth.DecodingKey[:], keyivInverse)
    // 将逆序随机数的一部分复制到解码随机数
    copy(auth.DecodingNonce[:], keyivInverse[32:])
    // 返回认证对象
    return auth
}

// ReadAuthentication 从读取器中读取认证对象的头部，并根据头部设置解码和编码密钥和随机数，然后返回认证对象
func ReadAuthentication(reader io.Reader) (*Authentication, error) {
    // 获取一个认证对象
    auth := getAuthenticationObject()
    // 从读取器中读取认证对象的头部
    if _, err := io.ReadFull(reader, auth.Header[:]); err != nil {
        // 如果读取失败，则释放认证对象并返回错误
        putAuthenticationObject(auth)
        return nil, err
    }
    // 将头部的一部分复制到解码密钥
    copy(auth.DecodingKey[:], auth.Header[8:])
    // 将头部的一部分复制到解码随机数
    copy(auth.DecodingNonce[:], auth.Header[8+32:])
    // 计算头部的逆序，并根据逆序设置编码密钥和编码随机数
    keyivInverse := Inverse(auth.Header[8 : 8+32+16])
    copy(auth.EncodingKey[:], keyivInverse)
    copy(auth.EncodingNonce[:], keyivInverse[32:])
    // 返回认证对象和空错误
    return auth, nil
}

// Inverse 返回一个新的字节数组，它是输入字节数组从末尾到开头的顺序
// 仅供测试可见
func Inverse(b []byte) []byte {
    // 获取字节数组的长度
    lenb := len(b)
    // 创建一个与输入字节数组相同长度的新字节数组
    b2 := make([]byte, lenb)
    // 将输入字节数组的内容逆序复制到新字节数组
    for i, v := range b {
        b2[lenb-i-1] = v
    }
    // 返回逆序后的新字节数组
    return b2
}

// authPool 是一个同步池，用于存储认证对象
var (
    authPool = sync.Pool{
        New: func() interface{} {
            return new(Authentication)
        },
    }
)

// getAuthenticationObject 从认证对象池中获取一个认证对象
func getAuthenticationObject() *Authentication {
    return authPool.Get().(*Authentication)
}

// putAuthenticationObject 将认证对象放回认证对象池
func putAuthenticationObject(auth *Authentication) {
    authPool.Put(auth)
}
```