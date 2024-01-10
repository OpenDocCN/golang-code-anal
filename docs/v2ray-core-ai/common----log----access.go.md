# `v2ray-core\common\log\access.go`

```
package log

import (
    "context"  // 导入 context 包
    "strings"  // 导入 strings 包

    "v2ray.com/core/common/serial"  // 导入自定义包

)

type logKey int  // 定义 logKey 类型为 int

const (
    accessMessageKey logKey = iota  // 定义 accessMessageKey 常量为 logKey 类型的枚举值 iota
)

type AccessStatus string  // 定义 AccessStatus 类型为 string

const (
    AccessAccepted = AccessStatus("accepted")  // 定义 AccessAccepted 常量为 AccessStatus 类型的字符串"accepted"
    AccessRejected = AccessStatus("rejected")  // 定义 AccessRejected 常量为 AccessStatus 类型的字符串"rejected"
)

type AccessMessage struct {  // 定义 AccessMessage 结构体
    From   interface{}  // 定义 From 字段为 interface{} 类型
    To     interface{}  // 定义 To 字段为 interface{} 类型
    Status AccessStatus  // 定义 Status 字段为 AccessStatus 类型
    Reason interface{}  // 定义 Reason 字段为 interface{} 类型
    Email  string  // 定义 Email 字段为 string 类型
    Detour string  // 定义 Detour 字段为 string 类型
}

func (m *AccessMessage) String() string {  // 定义 AccessMessage 结构体的 String 方法
    builder := strings.Builder{}  // 创建 strings.Builder 对象
    builder.WriteString(serial.ToString(m.From))  // 将 m.From 转换为字符串并写入 builder
    builder.WriteByte(' ')  // 写入空格
    builder.WriteString(string(m.Status))  // 将 m.Status 转换为字符串并写入 builder
    builder.WriteByte(' ')  // 写入空格
    builder.WriteString(serial.ToString(m.To))  // 将 m.To 转换为字符串并写入 builder
    builder.WriteByte(' ')  // 写入空格
    if len(m.Detour) > 0 {  // 如果 Detour 字段长度大于 0
        builder.WriteByte('[')  // 写入左方括号
        builder.WriteString(m.Detour)  // 将 Detour 字符串写入 builder
        builder.WriteString("] ")  // 写入右方括号和空格
    }
    builder.WriteString(serial.ToString(m.Reason))  // 将 m.Reason 转换为字符串并写入 builder

    if len(m.Email) > 0 {  // 如果 Email 字段长度大于 0
        builder.WriteString("email:")  // 写入"email:"
        builder.WriteString(m.Email)  // 将 Email 字符串写入 builder
        builder.WriteByte(' ')  // 写入空格
    }
    return builder.String()  // 返回 builder 中的字符串
}

func ContextWithAccessMessage(ctx context.Context, accessMessage *AccessMessage) context.Context {  // 定义 ContextWithAccessMessage 函数
    return context.WithValue(ctx, accessMessageKey, accessMessage)  // 使用 context.WithValue 将 accessMessage 存入 ctx 中并返回
}

func AccessMessageFromContext(ctx context.Context) *AccessMessage {  // 定义 AccessMessageFromContext 函数
    if accessMessage, ok := ctx.Value(accessMessageKey).(*AccessMessage); ok {  // 从 ctx 中获取 accessMessage
        return accessMessage  // 返回 accessMessage
    }
    return nil  // 返回空值
}
```