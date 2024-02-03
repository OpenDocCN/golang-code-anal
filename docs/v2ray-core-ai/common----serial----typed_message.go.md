# `v2ray-core\common\serial\typed_message.go`

```go
package serial

import (
    "errors"  // 导入 errors 包
    "reflect"  // 导入 reflect 包

    "github.com/golang/protobuf/proto"  // 导入第三方包 github.com/golang/protobuf/proto
)

// ToTypedMessage converts a proto Message into TypedMessage.
func ToTypedMessage(message proto.Message) *TypedMessage {
    if message == nil {  // 如果 message 为空
        return nil  // 返回空指针
    }
    settings, _ := proto.Marshal(message)  // 将 message 转换为字节流
    return &TypedMessage{  // 返回 TypedMessage 对象
        Type:  GetMessageType(message),  // 设置 Type 字段为 message 的类型
        Value: settings,  // 设置 Value 字段为 message 的字节流
    }
}

// GetMessageType returns the name of this proto Message.
func GetMessageType(message proto.Message) string {
    return proto.MessageName(message)  // 返回 message 的类型名
}

// GetInstance creates a new instance of the message with messageType.
func GetInstance(messageType string) (interface{}, error) {
    mType := proto.MessageType(messageType)  // 获取 messageType 对应的消息类型
    if mType == nil || mType.Elem() == nil {  // 如果消息类型为空
        return nil, errors.New("Serial: Unknown type: " + messageType)  // 返回错误信息
    }
    return reflect.New(mType.Elem()).Interface(), nil  // 创建消息类型的新实例并返回
}

// GetInstance converts current TypedMessage into a proto Message.
func (v *TypedMessage) GetInstance() (proto.Message, error) {
    instance, err := GetInstance(v.Type)  // 获取 TypedMessage 对应的消息类型实例
    if err != nil {  // 如果有错误
        return nil, err  // 返回错误信息
    }
    protoMessage := instance.(proto.Message)  // 将实例转换为 proto.Message 类型
    if err := proto.Unmarshal(v.Value, protoMessage); err != nil {  // 解析 Value 字段并将结果存入 protoMessage
        return nil, err  // 返回错误信息
    }
    return protoMessage, nil  // 返回 protoMessage
}
```