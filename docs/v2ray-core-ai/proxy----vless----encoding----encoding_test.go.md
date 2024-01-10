# `v2ray-core\proxy\vless\encoding\encoding_test.go`

```
package encoding_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/uuid"
    "v2ray.com/core/proxy/vless"
    . "v2ray.com/core/proxy/vless/encoding"
)

func toAccount(a *vless.Account) protocol.Account {
    // 将 vless.Account 转换为 protocol.Account
    account, err := a.AsAccount()
    common.Must(err)
    return account
}

func TestRequestSerialization(t *testing.T) {
    user := &protocol.MemoryUser{
        Level: 0,
        Email: "test@v2fly.org",
    }
    id := uuid.New()
    account := &vless.Account{
        Id: id.String(),
    }
    user.Account = toAccount(account)

    expectedRequest := &protocol.RequestHeader{
        Version: Version,
        User:    user,
        Command: protocol.RequestCommandTCP,
        Address: net.DomainAddress("www.v2fly.org"),
        Port:    net.Port(443),
    }
    expectedAddons := &Addons{}

    buffer := buf.StackNew()
    // 编码请求头部
    common.Must(EncodeRequestHeader(&buffer, expectedRequest, expectedAddons))

    Validator := new(vless.Validator)
    Validator.Add(user)

    // 解码请求头部
    actualRequest, actualAddons, err, _ := DecodeRequestHeader(false, nil, &buffer, Validator)
    common.Must(err)

    // 比较实际请求头部和预期请求头部
    if r := cmp.Diff(actualRequest, expectedRequest, cmp.AllowUnexported(protocol.ID{})); r != "" {
        t.Error(r)
    }
    // 比较实际附加信息和预期附加信息
    if r := cmp.Diff(actualAddons, expectedAddons); r != "" {
        t.Error(r)
    }
}

func TestInvalidRequest(t *testing.T) {
    user := &protocol.MemoryUser{
        Level: 0,
        Email: "test@v2fly.org",
    }
    id := uuid.New()
    account := &vless.Account{
        Id: id.String(),
    }
    user.Account = toAccount(account)

    expectedRequest := &protocol.RequestHeader{
        Version: Version,
        User:    user,
        Command: protocol.RequestCommand(100),
        Address: net.DomainAddress("www.v2fly.org"),
        Port:    net.Port(443),
    }
    # 创建一个 Addons 结构体的指针，用于存储预期的附加信息
    expectedAddons := &Addons{}
    
    # 创建一个新的缓冲区，用于存储数据
    buffer := buf.StackNew()
    
    # 使用 EncodeRequestHeader 函数将预期的请求和附加信息编码到缓冲区中
    common.Must(EncodeRequestHeader(&buffer, expectedRequest, expectedAddons))
    
    # 创建一个 vless.Validator 对象
    Validator := new(vless.Validator)
    
    # 向 Validator 对象中添加用户信息
    Validator.Add(user)
    
    # 使用 DecodeRequestHeader 函数解码缓冲区中的数据，并验证用户信息
    # 返回的第一个值是请求头信息，第二个值是错误信息
    _, _, err, _ := DecodeRequestHeader(false, nil, &buffer, Validator)
    
    # 如果错误信息为空，则输出测试错误信息
    if err == nil {
        t.Error("nil error")
    }
func TestMuxRequest(t *testing.T) {
    // 创建一个内存用户对象
    user := &protocol.MemoryUser{
        Level: 0,
        Email: "test@v2fly.org",
    }
    // 生成一个新的 UUID 作为账户 ID
    id := uuid.New()
    // 创建一个 vless 账户对象
    account := &vless.Account{
        Id: id.String(),
    }
    // 将账户对象转换为用户账户
    user.Account = toAccount(account)

    // 创建一个预期的请求头对象
    expectedRequest := &protocol.RequestHeader{
        Version: Version,
        User:    user,
        Command: protocol.RequestCommandMux,
        Address: net.DomainAddress("v1.mux.cool"),
    }
    // 创建一个预期的附加信息对象
    expectedAddons := &Addons{}

    // 创建一个缓冲区
    buffer := buf.StackNew()
    // 编码预期的请求头和附加信息到缓冲区
    common.Must(EncodeRequestHeader(&buffer, expectedRequest, expectedAddons))

    // 创建一个 vless 验证器对象
    Validator := new(vless.Validator)
    // 将用户添加到验证器中
    Validator.Add(user)

    // 解码缓冲区中的请求头和附加信息
    actualRequest, actualAddons, err, _ := DecodeRequestHeader(false, nil, &buffer, Validator)
    // 必须处理错误
    common.Must(err)

    // 检查实际请求头和预期请求头是否相同
    if r := cmp.Diff(actualRequest, expectedRequest, cmp.AllowUnexported(protocol.ID{})); r != "" {
        t.Error(r)
    }
    // 检查实际附加信息和预期附加信息是否相同
    if r := cmp.Diff(actualAddons, expectedAddons); r != "" {
        t.Error(r)
    }
}
```