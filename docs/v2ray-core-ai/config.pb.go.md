# `v2ray-core\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: config.proto

package core

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
    transport "v2ray.com/core/transport"  // 导入 transport 包
)

const (
    // 确保此生成的代码足够更新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够更新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用了足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// Config 是 V2Ray 的主配置。V2Ray 将此配置作为输入并相应地运行。
type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    // 入站处理程序配置。必须至少有一项。
    Inbound []*InboundHandlerConfig `protobuf:"bytes,1,rep,name=inbound,proto3" json:"inbound,omitempty"`
    // 出站处理程序配置。必须至少有一项。第一项用作路由的默认项。
    Outbound []*OutboundHandlerConfig `protobuf:"bytes,2,rep,name=outbound,proto3" json:"outbound,omitempty"`
    // App 用于 V2Ray 中所有功能的配置。功能必须实现 Feature 接口，并且其配置类型必须通过 common.RegisterConfig 注册。
    App []*serial.TypedMessage `protobuf:"bytes,4,rep,name=app,proto3" json:"app,omitempty"`
    // 传输设置。
    // 已弃用。每个入站和出站应选择自己的传输配置。移除日期：2020-01-13
    //
    // 不推荐使用：已废弃的字段，不建议使用
    Transport *transport.Config `protobuf:"bytes,5,opt,name=transport,proto3" json:"transport,omitempty"`
    // 用于扩展的配置。如果对应的扩展没有加载到 V2Ray 中，该配置可能不起作用。V2Ray 在初始化时会忽略这样的配置。
    Extension []*serial.TypedMessage `protobuf:"bytes,6,rep,name=extension,proto3" json:"extension,omitempty"`
// 重置 Config 对象的值为默认值
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则获取消息类型信息并存储
    if protoimpl.UnsafeEnabled {
        mi := &file_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Config 对象的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回 Config 对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_config_proto_rawDescGZIP(), []int{0}
}

// 返回 Config 对象的 Inbound 字段
func (x *Config) GetInbound() []*InboundHandlerConfig {
    if x != nil {
        return x.Inbound
    }
    return nil
}

// 返回 Config 对象的 Outbound 字段
func (x *Config) GetOutbound() []*OutboundHandlerConfig {
    if x != nil {
        return x.Outbound
    }
    return nil
}

// 返回 Config 对象的 App 字段
func (x *Config) GetApp() []*serial.TypedMessage {
    if x != nil {
        return x.App
    }
    return nil
}

// Deprecated: 不要使用
// 返回 Config 对象的 Transport 字段
func (x *Config) GetTransport() *transport.Config {
    if x != nil {
        return x.Transport
    }
    return nil
}

// 返回 Config 对象的 Extension 字段
func (x *Config) GetExtension() []*serial.TypedMessage {
    if x != nil {
        return x.Extension
    }
    return nil
}

// InboundHandlerConfig 是入站处理程序的配置
type InboundHandlerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 入站处理程序的标签。标签必须在所有入站处理程序中是唯一的
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    // 用于处理此入站代理的设置
    // 用于接收器的设置，类型为 TypedMessage 结构体的指针，protobuf 标签指定了字段的类型和名称，json 标签指定了字段在 JSON 中的名称和是否可选
    ReceiverSettings *serial.TypedMessage `protobuf:"bytes,2,opt,name=receiver_settings,json=receiverSettings,proto3" json:"receiver_settings,omitempty"`
    // 用于入站代理的设置，类型为 TypedMessage 结构体的指针，protobuf 标签指定了字段的类型和名称，json 标签指定了字段在 JSON 中的名称和是否可选
    ProxySettings *serial.TypedMessage `protobuf:"bytes,3,opt,name=proxy_settings,json=proxySettings,proto3" json:"proxy_settings,omitempty"`
// 重置 InboundHandlerConfig 对象
func (x *InboundHandlerConfig) Reset() {
    // 将对象重置为空对象
    *x = InboundHandlerConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型
        mi := &file_config_proto_msgTypes[1]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 InboundHandlerConfig 对象的字符串表示形式
func (x *InboundHandlerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*InboundHandlerConfig) ProtoMessage() {}

// 返回 InboundHandlerConfig 对象的反射信息
func (x *InboundHandlerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_config_proto_msgTypes[1]
    // 如果启用了不安全操作并且对象不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use InboundHandlerConfig.ProtoReflect.Descriptor instead.
// 返回 InboundHandlerConfig 对象的描述符
func (*InboundHandlerConfig) Descriptor() ([]byte, []int) {
    return file_config_proto_rawDescGZIP(), []int{1}
}

// 获取 InboundHandlerConfig 对象的标签
func (x *InboundHandlerConfig) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取 InboundHandlerConfig 对象的接收器设置
func (x *InboundHandlerConfig) GetReceiverSettings() *serial.TypedMessage {
    if x != nil {
        return x.ReceiverSettings
    }
    return nil
}

// 获取 InboundHandlerConfig 对象的代理设置
func (x *InboundHandlerConfig) GetProxySettings() *serial.TypedMessage {
    if x != nil {
        return x.ProxySettings
    }
    return nil
}

// OutboundHandlerConfig 是出站处理程序的配置
type OutboundHandlerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 出站处理程序的标签
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    // 用于拨号连接的设置
    SenderSettings *serial.TypedMessage `protobuf:"bytes,2,opt,name=sender_settings,json=senderSettings,proto3" json:"sender_settings,omitempty"`
    // 出站代理的设置。必须是出站代理之一。
    # 代理设置的消息类型，使用protobuf序列化，存储在proxy_settings字段中
    ProxySettings *serial.TypedMessage `protobuf:"bytes,3,opt,name=proxy_settings,json=proxySettings,proto3" json:"proxy_settings,omitempty"`
    # 如果不为零，表示该出站将在多少秒后过期，目前未使用
    Expire int64 `protobuf:"varint,4,opt,name=expire,proto3" json:"expire,omitempty"`
    # 出站处理程序的注释，目前未使用
    Comment string `protobuf:"bytes,5,opt,name=comment,proto3" json:"comment,omitempty"`
// 重置 OutboundHandlerConfig 对象
func (x *OutboundHandlerConfig) Reset() {
    // 将对象重置为空值
    *x = OutboundHandlerConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_config_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 OutboundHandlerConfig 对象的字符串表示
func (x *OutboundHandlerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*OutboundHandlerConfig) ProtoMessage() {}

// 返回 OutboundHandlerConfig 对象的反射信息
func (x *OutboundHandlerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_config_proto_msgTypes[2]
    // 如果启用了不安全操作并且对象不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空
        if ms.LoadMessageInfo() == nil {
            // 存储消息信息
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 OutboundHandlerConfig.ProtoReflect.Descriptor 替代
func (*OutboundHandlerConfig) Descriptor() ([]byte, []int) {
    return file_config_proto_rawDescGZIP(), []int{2}
}

// 获取 Tag 字段的值
func (x *OutboundHandlerConfig) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取 SenderSettings 字段的值
func (x *OutboundHandlerConfig) GetSenderSettings() *serial.TypedMessage {
    if x != nil {
        return x.SenderSettings
    }
    return nil
}

// 获取 ProxySettings 字段的值
func (x *OutboundHandlerConfig) GetProxySettings() *serial.TypedMessage {
    if x != nil {
        return x.ProxySettings
    }
    return nil
}

// 获取 Expire 字段的值
func (x *OutboundHandlerConfig) GetExpire() int64 {
    if x != nil {
        return x.Expire
    }
    return 0
}

// 获取 Comment 字段的值
func (x *OutboundHandlerConfig) GetComment() string {
    if x != nil {
        return x.Comment
    }
    return ""
}

// 定义 File_config_proto 变量
var File_config_proto protoreflect.FileDescriptor

// 定义 file_config_proto_rawDesc 变量
var file_config_proto_rawDesc = []byte{
    0x0a, 0x0c, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x0a,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d,
    0x6f, 0x6e, 0x2f, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2f, 0x74, 0x79, 0x70, 0x65, 0x64, 0x5f,
}
    # 以下是十六进制数值，表示一些数据
    0x6d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x16, 0x74,
    0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0xc9, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67,
    12, 0x3a, 0x0a, 0x07, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x18, 0x01, 0x20, 0x03, 0x28,
    0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x49,
    0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x48, 0x61, 0x6e, 0x64, 0x6c, 0x65, 0x72, 0x43, 0x6f, 0x6e,
    0x66, 0x69, 0x67, 0x52, 0x07, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12, 0x3d, 0x0a, 0x08,
    0x6f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x21,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x4f, 0x75, 0x74, 0x62,
    0x6f, 0x75, 0x6e, 0x64, 0x48, 0x61, 0x6e, 0x64, 0x6c, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69,
    0x67, 0x52, 0x08, 0x6f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12, 0x38, 0x0a, 0x03, 0x61,
    0x70, 0x70, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72,
    0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65,
    0x52, 0x03, 0x61, 0x70, 0x70, 0x12, 0x3e, 0x0a, 0x09, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f,
    0x72, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e,
    0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x42, 0x02, 0x18, 0x01, 0x52, 0x09, 0x74, 0x72, 0x61, 0x6e,
    0x73, 0x70, 0x6f, 0x72, 0x74, 0x12, 0x44, 0x0a, 0x09, 0x65, 0x78, 0x74, 0x65, 0x6e, 0x73, 0x69,
    0x6f, 0x6e, 0x18, 0x06, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    # 以下是一系列十六进制数，可能是某种配置或数据的表示
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72,
    0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65,
    0x52, 0x09, 0x65, 0x78, 0x74, 0x65, 0x6e, 0x73, 0x69, 0x6f, 0x6e, 0x4a, 0x04, 0x08, 0x03, 0x10,
    0x04, 0x22, 0xcc, 0x01, 0x0a, 0x14, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x48, 0x61, 0x6e,
    0x64, 0x6c, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61,
    0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12, 0x53, 0x0a, 0x11,
    0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x72, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67,
    0x73, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69,
    0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52,
    0x10, 0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x72, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67,
    0x73, 0x12, 0x4d, 0x0a, 0x0e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69,
    0x6e, 0x67, 0x73, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65,
    0x72, 0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67,
    0x65, 0x52, 0x0d, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73,
    0x22, 0xfb, 0x01, 0x0a, 0x15, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x48, 0x61, 0x6e,
    0x64, 0x6c, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61,
    0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12, 0x4f, 0x0a, 0x0f,
    0x73, 0x65, 0x6e, 0x64, 0x65, 0x72, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18,
    # 定义一个十六进制数列表
    0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c,
    0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52, 0x0e, 0x73,
    0x65, 0x6e, 0x64, 0x65, 0x72, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x12, 0x4d, 0x0a,
    0x0e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18,
    0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c,
    0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52, 0x0d, 0x70,
    0x72, 0x6f, 0x78, 0x79, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x12, 0x16, 0x0a, 0x06,
    0x65, 0x78, 0x70, 0x69, 0x72, 0x65, 0x18, 0x04, 0x20, 0x01, 0x28, 0x03, 0x52, 0x06, 0x65, 0x78,
    0x70, 0x69, 0x72, 0x65, 0x12, 0x18, 0x0a, 0x07, 0x63, 0x6f, 0x6d, 0x6d, 0x65, 0x6e, 0x74, 0x18,
    0x05, 0x20, 0x01, 0x28, 0x09, 0x52, 0x07, 0x63, 0x6f, 0x6d, 0x6d, 0x65, 0x6e, 0x74, 0x42, 0x2f,
    0x0a, 0x0e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x50, 0x01, 0x5a, 0x0e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
    0x72, 0xaa, 0x02, 0x0a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x62, 0x06,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量 file_config_proto_rawDescOnce，类型为 sync.Once，用于确保 file_config_proto_rawDescData 只被初始化一次
var (
    file_config_proto_rawDescOnce sync.Once
    file_config_proto_rawDescData = file_config_proto_rawDesc
)

// 定义函数 file_config_proto_rawDescGZIP，返回类型为 []byte，用于对 file_config_proto_rawDescData 进行 GZIP 压缩
func file_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 file_config_proto_rawDescData 只被初始化一次，并对其进行 GZIP 压缩
    file_config_proto_rawDescOnce.Do(func() {
        file_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_config_proto_rawDescData)
    })
    // 返回经过 GZIP 压缩后的 file_config_proto_rawDescData
    return file_config_proto_rawDescData
}

// 定义变量 file_config_proto_msgTypes，类型为 []protoimpl.MessageInfo，用于存储消息类型信息
var file_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
// 定义变量 file_config_proto_goTypes，类型为 []interface{}，用于存储 Go 类型信息
var file_config_proto_goTypes = []interface{}{
    (*Config)(nil),                // 0: v2ray.core.Config
    (*InboundHandlerConfig)(nil),  // 1: v2ray.core.InboundHandlerConfig
    (*OutboundHandlerConfig)(nil), // 2: v2ray.core.OutboundHandlerConfig
    (*serial.TypedMessage)(nil),   // 3: v2ray.core.common.serial.TypedMessage
    (*transport.Config)(nil),      // 4: v2ray.core.transport.Config
}
// 定义变量 file_config_proto_depIdxs，类型为 []int32，用于存储消息类型之间的依赖关系
var file_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.Config.inbound:type_name -> v2ray.core.InboundHandlerConfig
    2, // 1: v2ray.core.Config.outbound:type_name -> v2ray.core.OutboundHandlerConfig
    3, // 2: v2ray.core.Config.app:type_name -> v2ray.core.common.serial.TypedMessage
    4, // 3: v2ray.core.Config.transport:type_name -> v2ray.core.transport.Config
    3, // 4: v2ray.core.Config.extension:type_name -> v2ray.core.common.serial.TypedMessage
    3, // 5: v2ray.core.InboundHandlerConfig.receiver_settings:type_name -> v2ray.core.common.serial.TypedMessage
    3, // 6: v2ray.core.InboundHandlerConfig.proxy_settings:type_name -> v2ray.core.common.serial.TypedMessage
    3, // 7: v2ray.core.OutboundHandlerConfig.sender_settings:type_name -> v2ray.core.common.serial.TypedMessage
    3, // 8: v2ray.core.OutboundHandlerConfig.proxy_settings:type_name -> v2ray.core.common.serial.TypedMessage
    9, // [9:9] is the sub-list for method output_type
    9, // [9:9] is the sub-list for method input_type
    9, // [9:9] is the sub-list for extension type_name
    9, // [9:9] is the sub-list for extension extendee
    # 0 表示字段类型为 type_name 的子列表的起始索引
    // [0:9] 表示子列表的范围，包括索引 0 不包括索引 9
}
// 在程序初始化时调用 file_config_proto_init 函数
func init() { file_config_proto_init() }
// 初始化 file_config_proto 函数
func file_config_proto_init() {
    // 如果 File_config_proto 不为空，则直接返回
    if File_config_proto != nil {
        return
    }
    // 如果 protoimpl.UnsafeEnabled 为假，则直接返回
    if !protoimpl.UnsafeEnabled {
        // 配置 Config 的消息类型导出器
        file_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*Config); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        }
        // 配置 InboundHandlerConfig 的消息类型导出器
        file_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*InboundHandlerConfig); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        }
        // 配置 OutboundHandlerConfig 的消息类型导出器
        file_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*OutboundHandlerConfig); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        }
    }
    // 创建一个类型为 x 的结构体
    type x struct{}
    // 构建 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   3,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_config_proto_goTypes,
        DependencyIndexes: file_config_proto_depIdxs,
        MessageInfos:      file_config_proto_msgTypes,
    }.Build()
    // 将构建的 out 赋值给 File_config_proto
    File_config_proto = out.File
    // 将 file_config_proto_rawDesc 置空
    file_config_proto_rawDesc = nil
    // 将 file_config_proto_goTypes 置空
    file_config_proto_goTypes = nil
    // 将 file_config_proto_depIdxs 置空
    file_config_proto_depIdxs = nil
}
```