# `v2ray-core\app\router\command\command.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/router/command/command.proto

package command

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    net "v2ray.com/core/common/net"  // 导入 net 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// RoutingContext 是包含与路由过程相关信息的上下文。
// 它符合 v2ray.core.features.routing.Context 和 v2ray.core.features.routing.Route 的结构。
type RoutingContext struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    InboundTag        string            `protobuf:"bytes,1,opt,name=InboundTag,proto3" json:"InboundTag,omitempty"`  // 入站标签
    Network           net.Network       `protobuf:"varint,2,opt,name=Network,proto3,enum=v2ray.core.common.net.Network" json:"Network,omitempty"`  // 网络类型
    SourceIPs         [][]byte          `protobuf:"bytes,3,rep,name=SourceIPs,proto3" json:"SourceIPs,omitempty"`  // 源 IP 地址
    TargetIPs         [][]byte          `protobuf:"bytes,4,rep,name=TargetIPs,proto3" json:"TargetIPs,omitempty"`  // 目标 IP 地址
    SourcePort        uint32            `protobuf:"varint,5,opt,name=SourcePort,proto3" json:"SourcePort,omitempty"`  // 源端口
    TargetPort        uint32            `protobuf:"varint,6,opt,name=TargetPort,proto3" json:"TargetPort,omitempty"`  // 目标端口
    # 目标域名
    TargetDomain      string            `protobuf:"bytes,7,opt,name=TargetDomain,proto3" json:"TargetDomain,omitempty"`
    # 协议
    Protocol          string            `protobuf:"bytes,8,opt,name=Protocol,proto3" json:"Protocol,omitempty"`
    # 用户
    User              string            `protobuf:"bytes,9,opt,name=User,proto3" json:"User,omitempty"`
    # 属性，使用键值对表示
    Attributes        map[string]string `protobuf:"bytes,10,rep,name=Attributes,proto3" json:"Attributes,omitempty" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
    # 出站组标签
    OutboundGroupTags []string          `protobuf:"bytes,11,rep,name=OutboundGroupTags,proto3" json:"OutboundGroupTags,omitempty"`
    # 出站标签
    OutboundTag       string            `protobuf:"bytes,12,opt,name=OutboundTag,proto3" json:"OutboundTag,omitempty"`
// 重置 RoutingContext 对象
func (x *RoutingContext) Reset() {
    // 将 RoutingContext 对象重置为空对象
    *x = RoutingContext{}
    // 如果启用了不安全操作，则获取消息类型并存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_command_command_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RoutingContext 对象的字符串表示形式
func (x *RoutingContext) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*RoutingContext) ProtoMessage() {}

// 返回 RoutingContext 对象的反射信息
func (x *RoutingContext) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_command_command_proto_msgTypes[0]
    // 如果启用了不安全操作并且对象不为空，则获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 RoutingContext.ProtoReflect.Descriptor 替代
func (*RoutingContext) Descriptor() ([]byte, []int) {
    return file_app_router_command_command_proto_rawDescGZIP(), []int{0}
}

// 返回 RoutingContext 对象的 InboundTag 属性值
func (x *RoutingContext) GetInboundTag() string {
    if x != nil {
        return x.InboundTag
    }
    return ""
}

// 返回 RoutingContext 对象的 Network 属性值
func (x *RoutingContext) GetNetwork() net.Network {
    if x != nil {
        return x.Network
    }
    return net.Network_Unknown
}

// 返回 RoutingContext 对象的 SourceIPs 属性值
func (x *RoutingContext) GetSourceIPs() [][]byte {
    if x != nil {
        return x.SourceIPs
    }
    return nil
}

// 返回 RoutingContext 对象的 TargetIPs 属性值
func (x *RoutingContext) GetTargetIPs() [][]byte {
    if x != nil {
        return x.TargetIPs
    }
    return nil
}

// 返回 RoutingContext 对象的 SourcePort 属性值
func (x *RoutingContext) GetSourcePort() uint32 {
    if x != nil {
        return x.SourcePort
    }
    return 0
}

// 返回 RoutingContext 对象的 TargetPort 属性值
func (x *RoutingContext) GetTargetPort() uint32 {
    if x != nil {
        return x.TargetPort
    }
    return 0
}

// 返回 RoutingContext 对象的 TargetDomain 属性值
func (x *RoutingContext) GetTargetDomain() string {
    if x != nil {
        return x.TargetDomain
    }
    return ""
}

// 返回 RoutingContext 对象的 Protocol 属性值
func (x *RoutingContext) GetProtocol() string {
    if x != nil {
        return x.Protocol
    }
    return ""
}

// 返回 RoutingContext 对象的 User 属性值
func (x *RoutingContext) GetUser() string {
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的用户属性
        return x.User
    # 如果 x 为空，返回空字符串
    return ""
// 获取路由上下文的属性，返回属性名到属性值的映射
func (x *RoutingContext) GetAttributes() map[string]string {
    // 如果路由上下文不为空，则返回其属性
    if x != nil {
        return x.Attributes
    }
    // 如果路由上下文为空，则返回空值
    return nil
}

// 获取出站组标签，返回出站组标签的字符串数组
func (x *RoutingContext) GetOutboundGroupTags() []string {
    // 如果路由上下文不为空，则返回其出站组标签
    if x != nil {
        return x.OutboundGroupTags
    }
    // 如果路由上下文为空，则返回空值
    return nil
}

// 获取出站标签，返回出站标签的字符串
func (x *RoutingContext) GetOutboundTag() string {
    // 如果路由上下文不为空，则返回其出站标签
    if x != nil {
        return x.OutboundTag
    }
    // 如果路由上下文为空，则返回空字符串
    return ""
}

// SubscribeRoutingStatsRequest 订阅路由统计请求，如果由v2ray-core打开
// * FieldSelectors 选择要返回的路由统计字段的子集
// 有效的选择器:
//  - inbound: 选择连接的入站标签。
//  - network: 选择连接的网络。
//  - ip: 等同于"ip_source"和"ip_target"，选择源IP和目标IP。
//  - port: 等同于"port_source"和"port_target"，选择源端口和目标端口。
//  - domain: 选择目标域名。
//  - protocol: 选择连接的协议。
//  - user: 选择连接的入站用户电子邮件。
//  - attributes: 选择连接的附加属性。
//  - outbound: 等同于"outbound"和"outbound_group"，选择出站标签和出站组标签。
// * 如果FieldSelectors为空，则返回所有字段。
type SubscribeRoutingStatsRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    FieldSelectors []string `protobuf:"bytes,1,rep,name=FieldSelectors,proto3" json:"FieldSelectors,omitempty"`
}

// 重置SubscribeRoutingStatsRequest对象
func (x *SubscribeRoutingStatsRequest) Reset() {
    *x = SubscribeRoutingStatsRequest{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_command_command_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回SubscribeRoutingStatsRequest对象的字符串表示形式
func (x *SubscribeRoutingStatsRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现ProtoMessage接口
func (*SubscribeRoutingStatsRequest) ProtoMessage() {}
// 返回消息类型的反射信息
func (x *SubscribeRoutingStatsRequest) ProtoReflect() protoreflect.Message {
    // 获取消息类型的信息
    mi := &file_app_router_command_command_proto_msgTypes[1]
    // 如果启用了不安全操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        // 返回消息状态
        return ms
    }
    // 返回消息类型的信息
    return mi.MessageOf(x)
}

// Deprecated: Use SubscribeRoutingStatsRequest.ProtoReflect.Descriptor instead.
// 返回描述符的字节切片和路径
func (*SubscribeRoutingStatsRequest) Descriptor() ([]byte, []int) {
    return file_app_router_command_command_proto_rawDescGZIP(), []int{1}
}

// 返回字段选择器的字符串切片
func (x *SubscribeRoutingStatsRequest) GetFieldSelectors() []string {
    if x != nil {
        return x.FieldSelectors
    }
    return nil
}

// TestRouteRequest 手动测试路由结果，根据路由上下文消息。
// * RoutingContext 是不带出站信息的路由消息。
// * FieldSelectors 选择要在路由结果中返回的字段。如果为空，则返回所有字段。
// * 如果设置为 true，则将路由结果广播到路由统计通道。
type TestRouteRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    RoutingContext *RoutingContext `protobuf:"bytes,1,opt,name=RoutingContext,proto3" json:"RoutingContext,omitempty"`
    FieldSelectors []string        `protobuf:"bytes,2,rep,name=FieldSelectors,proto3" json:"FieldSelectors,omitempty"`
    PublishResult  bool            `protobuf:"varint,3,opt,name=PublishResult,proto3" json:"PublishResult,omitempty"`
}

// 重置 TestRouteRequest 对象
func (x *TestRouteRequest) Reset() {
    *x = TestRouteRequest{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型的信息
        mi := &file_app_router_command_command_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 TestRouteRequest 对象的字符串表示形式
func (x *TestRouteRequest) String() string {
    # 返回给定对象的字符串表示形式
    return protoimpl.X.MessageStringOf(x)
// 定义 TestRouteRequest 结构体的 ProtoMessage 方法
func (*TestRouteRequest) ProtoMessage() {}

// 定义 TestRouteRequest 结构体的 ProtoReflect 方法
func (x *TestRouteRequest) ProtoReflect() protoreflect.Message {
    // 获取 TestRouteRequest 对应的消息类型信息
    mi := &file_app_router_command_command_proto_msgTypes[2]
    // 如果启用了不安全操作并且 x 不为空
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

// Deprecated: Use TestRouteRequest.ProtoReflect.Descriptor instead.
// 定义 TestRouteRequest 结构体的 Descriptor 方法
func (*TestRouteRequest) Descriptor() ([]byte, []int) {
    return file_app_router_command_command_proto_rawDescGZIP(), []int{2}
}

// 获取 TestRouteRequest 结构体的 RoutingContext 字段
func (x *TestRouteRequest) GetRoutingContext() *RoutingContext {
    if x != nil {
        return x.RoutingContext
    }
    return nil
}

// 获取 TestRouteRequest 结构体的 FieldSelectors 字段
func (x *TestRouteRequest) GetFieldSelectors() []string {
    if x != nil {
        return x.FieldSelectors
    }
    return nil
}

// 获取 TestRouteRequest 结构体的 PublishResult 字段
func (x *TestRouteRequest) GetPublishResult() bool {
    if x != nil {
        return x.PublishResult
    }
    return false
}

// 定义 Config 结构体
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取 Config 对应的消息类型信息
        mi := &file_app_router_command_command_proto_msgTypes[3]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 获取 Config 结构体的字符串表示
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 定义 Config 结构体的 ProtoMessage 方法
func (*Config) ProtoMessage() {}

// 定义 Config 结构体的 ProtoReflect 方法
func (x *Config) ProtoReflect() protoreflect.Message {
    // 获取 Config 对应的消息类型信息
    mi := &file_app_router_command_command_proto_msgTypes[3]
    // 如果启用了不安全操作并且 x 不为空
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

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 定义 Config 结构体的 Descriptor 方法
func (*Config) Descriptor() ([]byte, []int) {
    # 返回 file_app_router_command_command_proto_rawDescGZIP() 函数的结果和空的整数列表
    return file_app_router_command_command_proto_rawDescGZIP(), []int{3}
// 定义变量 File_app_router_command_command_proto，用于存储协议描述符
var File_app_router_command_command_proto protoreflect.FileDescriptor

// 定义变量 file_app_router_command_command_proto_rawDesc，存储原始的协议描述符的字节流
var file_app_router_command_command_proto_rawDesc = []byte{
    // ...（省略部分内容）
}
    # 创建一个十六进制数值列表
    0x50, 0x6f, 0x72, 0x74, 0x12, 0x1e, 0x0a, 0x0a, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74, 0x50, 0x6f,
    0x72, 0x74, 0x18, 0x06, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x0a, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74,
    0x50, 0x6f, 0x72, 0x74, 0x12, 0x22, 0x0a, 0x0c, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74, 0x44, 0x6f,
    0x6d, 0x61, 0x69, 0x6e, 0x18, 0x07, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0c, 0x54, 0x61, 0x72, 0x67,
    0x65, 0x74, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x1a, 0x0a, 0x08, 0x50, 0x72, 0x6f, 0x74,
    0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x08, 0x20, 0x01, 0x28, 0x09, 0x52, 0x08, 0x50, 0x72, 0x6f, 0x74,
    0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x12, 0x0a, 0x04, 0x55, 0x73, 0x65, 0x72, 0x18, 0x09, 0x20, 0x01,
    0x28, 0x09, 0x52, 0x04, 0x55, 0x73, 0x65, 0x72, 0x12, 0x5d, 0x0a, 0x0a, 0x41, 0x74, 0x74, 0x72,
    0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x18, 0x0a, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x3d, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
    0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f, 0x75,
    0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x2e, 0x41, 0x74, 0x74, 0x72,
    0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x52, 0x0a, 0x41, 0x74, 0x74,
    0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x12, 0x2c, 0x0a, 0x11, 0x4f, 0x75, 0x74, 0x62, 0x6f,
    0x75, 0x6e, 0x64, 0x47, 0x72, 0x6f, 0x75, 0x70, 0x54, 0x61, 0x67, 0x73, 0x18, 0x0b, 0x20, 0x03,
    0x28, 0x09, 0x52, 0x11, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x47, 0x72, 0x6f, 0x75,
    0x70, 0x54, 0x61, 0x67, 0x73, 0x12, 0x20, 0x0a, 0x0b, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e,
    0x64, 0x54, 0x61, 0x67, 0x18, 0x0c, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0b, 0x4f, 0x75, 0x74, 0x62,
    0x6f, 0x75, 0x6e, 0x64, 0x54, 0x61, 0x67, 0x1a, 0x3d, 0x0a, 0x0f, 0x41, 0x74, 0x74, 0x72, 0x69,
    0x62, 0x75, 0x74, 0x65, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 12, 0x10, 0x0a, 0x03, 0x6b, 0x65,
    # 创建一个十六进制数值列表
    0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12, 0x14, 0x0a, 0x05,
    0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x05, 0x76, 0x61, 0x6c,
    0x75, 0x65, 0x3a, 0x02, 0x38, 0x01, 0x22, 0x46, 0x0a, 0x1c, 0x53, 0x75, 0x62, 0x73, 0x63, 0x72,
    0x69, 0x62, 0x65, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52,
    0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x26, 0x0a, 0x0e, 0x46, 0x69, 0x65, 0x6c, 0x64, 0x53,
    0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x73, 0x18, 0x01, 0x20, 0x03, 0x28, 0x09, 0x52, 0x0e,
    0x46, 0x69, 0x65, 0x6c, 0x64, 0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x73, 0x22, 0xb7,
    0x01, 0x0a, 0x10, 0x54, 0x65, 0x73, 0x74, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x52, 0x65, 0x71, 0x75,
    0x65, 0x73, 0x74, 0x12, 0x55, 0x0a, 0x0e, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f,
    0x6e, 0x74, 0x65, 0x78, 0x74, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75,
    0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f, 0x75, 0x74,
    0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x52, 0x0e, 0x52, 0x6f, 0x75, 0x74,
    0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x12, 0x26, 0x0a, 0x0e, 0x46, 0x69,
    0x65, 0x6c, 0x64, 0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x73, 0x18, 0x02, 0x20, 0x03,
    0x28, 0x09, 0x52, 0x0e, 0x46, 0x69, 0x65, 0x6c, 0x64, 0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f,
    0x72, 0x73, 0x12, 0x24, 0x0a, 0x0d, 0x50, 0x75, 0x62, 0x6c, 0x69, 0x73, 0x68, 0x52, 0x65, 0x73,
    0x75, 0x6c, 0x74, 0x18, 0x03, 0x20, 0x01, 0x28, 0x08, 0x52, 0x0d, 0x50, 0x75, 0x62, 0x6c, 0x69,
    0x73, 0x68, 0x52, 0x65, 0x73, 0x75, 0x6c, 0x74, 0x22, 0x08, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66,
    0x69, 0x67, 0x32, 0x89, 0x02, 0x0a, 0x0e, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x53, 0x65,
    # 以十六进制表示的字节码序列
    0x72, 0x76, 0x69, 0x63, 0x65, 0x12, 0x87, 0x01, 0x0a, 0x15, 0x53, 0x75, 0x62, 0x73, 0x63, 0x72,
    0x69, 0x62, 0x65, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x53, 0x74, 0x61, 0x74, 0x73, 0x12,
    0x3b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
    0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e,
    0x53, 0x75, 0x62, 0x73, 0x63, 0x72, 0x69, 0x62, 0x65, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67,
    0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x2d, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
    0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f, 0x75,
    0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x22, 0x00, 0x30, 0x01, 0x12,
    0x6d, 0x0a, 0x09, 0x54, 0x65, 0x73, 0x74, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x12, 0x2f, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
    0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x54, 0x65, 0x73,
    0x74, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x2d, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72,
    0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f,
    0x75, 0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x22, 0x00, 0x42, 0x68,
    0x0a, 0x21, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
    0x61, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d,
    0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
    # 这是一串十六进制数字，可能是某种编码或加密的数据
    # 无法确定具体含义，需要进一步分析和处理
// 定义变量，确保 file_app_router_command_command_proto_rawDescData 只被初始化一次
var (
    file_app_router_command_command_proto_rawDescOnce sync.Once
    file_app_router_command_command_proto_rawDescData = file_app_router_command_command_proto_rawDesc
)

// 使用 GZIP 压缩原始描述数据
func file_app_router_command_command_proto_rawDescGZIP() []byte {
    // 确保只执行一次 GZIP 压缩
    file_app_router_command_command_proto_rawDescOnce.Do(func() {
        file_app_router_command_command_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_router_command_command_proto_rawDescData)
    })
    return file_app_router_command_command_proto_rawDescData
}

// 定义消息类型
var file_app_router_command_command_proto_msgTypes = make([]protoimpl.MessageInfo, 5)
// 定义 Go 类型
var file_app_router_command_command_proto_goTypes = []interface{}{
    (*RoutingContext)(nil),               // 0: v2ray.core.app.router.command.RoutingContext
    (*SubscribeRoutingStatsRequest)(nil), // 1: v2ray.core.app.router.command.SubscribeRoutingStatsRequest
    (*TestRouteRequest)(nil),             // 2: v2ray.core.app.router.command.TestRouteRequest
    (*Config)(nil),                       // 3: v2ray.core.app.router.command.Config
    nil,                                  // 4: v2ray.core.app.router.command.RoutingContext.AttributesEntry
    (net.Network)(0),                     // 5: v2ray.core.common.net.Network
}
// 定义依赖索引
var file_app_router_command_command_proto_depIdxs = []int32{
    5, // 0: v2ray.core.app.router.command.RoutingContext.Network:type_name -> v2ray.core.common.net.Network
    4, // 1: v2ray.core.app.router.command.RoutingContext.Attributes:type_name -> v2ray.core.app.router.command.RoutingContext.AttributesEntry
    0, // 2: v2ray.core.app.router.command.TestRouteRequest.RoutingContext:type_name -> v2ray.core.app.router.command.RoutingContext
    3, // 3: v2ray.core.app.router.command.RoutingService.SubscribeRoutingStats:input_type -> v2ray.core.app.router.command.SubscribeRoutingStatsRequest
    2, // 4: v2ray.core.app.router.command.RoutingService.TestRoute:input_type -> v2ray.core.app.router.command.TestRouteRequest
    0, // 5: v2ray.core.app.router.command.RoutingService.SubscribeRoutingStats:output_type -> v2ray.core.app.router.command.RoutingContext
    0, // 6: v2ray.core.app.router.command.RoutingService.TestRoute:output_type -> v2ray.core.app.router.command.RoutingContext
    5, // [5:7] is the sub-list for method output_type
    3, // [3:5] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

// 初始化函数，调用 file_app_router_command_command_proto_init 函数
func init() { file_app_router_command_command_proto_init() }
// file_app_router_command_command_proto_init 函数，用于初始化一些消息类型的导出器
func file_app_router_command_command_proto_init() {
    // 如果 File_app_router_command_command_proto 不为空，则直接返回
    if File_app_router_command_command_proto != nil {
        return
    }
    // 如果 protoimpl.UnsafeEnabled 为 false，则设置消息类型的导出器
    if !protoimpl.UnsafeEnabled {
        file_app_router_command_command_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*RoutingContext); i {
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
        file_app_router_command_command_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*SubscribeRoutingStatsRequest); i {
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
        file_app_router_command_command_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*TestRouteRequest); i {
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
        file_app_router_command_command_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
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
    }
    // 定义一个空结构体 x
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_app_router_command_command_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 5
            NumMessages:   5,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 1
            NumServices:   1,
        },
        # 设置 GoTypes 属性为指定的 Go 类型
        GoTypes:           file_app_router_command_command_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_app_router_command_command_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_app_router_command_command_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_app_router_command_command_proto
    File_app_router_command_command_proto = out.File
    # 将 file_app_router_command_command_proto_rawDesc 设置为 nil
    file_app_router_command_command_proto_rawDesc = nil
    # 将 file_app_router_command_command_proto_goTypes 设置为 nil
    file_app_router_command_command_proto_goTypes = nil
    # 将 file_app_router_command_command_proto_depIdxs 设置为 nil
    file_app_router_command_command_proto_depIdxs = nil
# 闭合前面的函数定义
```