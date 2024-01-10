# `v2ray-core\app\proxyman\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/proxyman/config.proto

package proxyman

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    net "v2ray.com/core/common/net"  // 导入 net 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
    internet "v2ray.com/core/transport/internet"  // 导入 internet 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

type KnownProtocols int32

const (
    KnownProtocols_HTTP KnownProtocols = 0  // 定义 KnownProtocols 枚举值 HTTP
    KnownProtocols_TLS  KnownProtocols = 1  // 定义 KnownProtocols 枚举值 TLS
)

// KnownProtocols 的枚举值映射
var (
    KnownProtocols_name = map[int32]string{
        0: "HTTP",
        1: "TLS",
    }
    KnownProtocols_value = map[string]int32{
        "HTTP": 0,
        "TLS":  1,
    }
)

func (x KnownProtocols) Enum() *KnownProtocols {
    p := new(KnownProtocols)
    *p = x
    return p
}

func (x KnownProtocols) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (KnownProtocols) Descriptor() protoreflect.EnumDescriptor {
    return file_app_proxyman_config_proto_enumTypes[0].Descriptor()
}

func (KnownProtocols) Type() protoreflect.EnumType {
    return &file_app_proxyman_config_proto_enumTypes[0]
}

func (x KnownProtocols) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: Use KnownProtocols.Descriptor instead.
func (KnownProtocols) EnumDescriptor() ([]byte, []int) {
    # 返回 file_app_proxyman_config_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{0}
// 定义 AllocationStrategy_Type 枚举类型
type AllocationStrategy_Type int32

const (
    // 总是分配所有连接处理程序
    AllocationStrategy_Always AllocationStrategy_Type = 0
    // 随机分配特定范围的处理程序
    AllocationStrategy_Random AllocationStrategy_Type = 1
    // 外部分配策略，目前不支持
    AllocationStrategy_External AllocationStrategy_Type = 2
)

// AllocationStrategy_Type 的枚举值映射
var (
    AllocationStrategy_Type_name = map[int32]string{
        0: "Always",
        1: "Random",
        2: "External",
    }
    AllocationStrategy_Type_value = map[string]int32{
        "Always":   0,
        "Random":   1,
        "External": 2,
    }
)

// 返回 AllocationStrategy_Type 的枚举值
func (x AllocationStrategy_Type) Enum() *AllocationStrategy_Type {
    p := new(AllocationStrategy_Type)
    *p = x
    return p
}

// 返回 AllocationStrategy_Type 的字符串表示
func (x AllocationStrategy_Type) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

// 返回 AllocationStrategy_Type 的枚举描述符
func (AllocationStrategy_Type) Descriptor() protoreflect.EnumDescriptor {
    return file_app_proxyman_config_proto_enumTypes[1].Descriptor()
}

// 返回 AllocationStrategy_Type 的枚举类型
func (AllocationStrategy_Type) Type() protoreflect.EnumType {
    return &file_app_proxyman_config_proto_enumTypes[1]
}

// 返回 AllocationStrategy_Type 的枚举数值
func (x AllocationStrategy_Type) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: 使用 AllocationStrategy_Type.Descriptor 替代
func (AllocationStrategy_Type) EnumDescriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{1, 0}
}

// InboundConfig 结构体
type InboundConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置 InboundConfig 结构体
func (x *InboundConfig) Reset() {
    *x = InboundConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 InboundConfig 结构体的字符串表示
func (x *InboundConfig) String() string {
    # 返回给定对象的字符串表示形式
    return protoimpl.X.MessageStringOf(x)
// 空的方法，实现了 ProtoMessage 接口
func (*InboundConfig) ProtoMessage() {}

// 返回当前消息的反射信息
func (x *InboundConfig) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_proxyman_config_proto_msgTypes[0]
    // 如果启用了不安全的操作，并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息未加载，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use InboundConfig.ProtoReflect.Descriptor instead.
// 返回消息的原始描述符
func (*InboundConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{0}
}

// AllocationStrategy 结构体
type AllocationStrategy struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Type AllocationStrategy_Type `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.proxyman.AllocationStrategy_Type" json:"type,omitempty"`
    // 并发处理的数量，默认值为 3
    Concurrency *AllocationStrategy_AllocationStrategyConcurrency `protobuf:"bytes,2,opt,name=concurrency,proto3" json:"concurrency,omitempty"`
    // 处理器重新生成前的分钟数，默认值为 5
    Refresh *AllocationStrategy_AllocationStrategyRefresh `protobuf:"bytes,3,opt,name=refresh,proto3" json:"refresh,omitempty"`
}

// 重置 AllocationStrategy 结构体
func (x *AllocationStrategy) Reset() {
    *x = AllocationStrategy{}
    // 如果启用了不安全的操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_proxyman_config_proto_msgTypes[1]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 AllocationStrategy 结构体的字符串表示
func (x *AllocationStrategy) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 空的方法，实现了 ProtoMessage 接口
func (*AllocationStrategy) ProtoMessage() {}

// 返回 AllocationStrategy 结构体的反射信息
func (x *AllocationStrategy) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[1]
}
    # 检查是否启用了不安全的功能，并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil:
        # 获取 x 对象的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 如果消息状态中没有加载消息信息，则存储消息信息
        if ms.LoadMessageInfo() == nil:
            ms.StoreMessageInfo(mi)
        # 返回消息状态
        return ms
    # 返回 x 对象的消息信息
    return mi.MessageOf(x)
// Deprecated: Use AllocationStrategy.ProtoReflect.Descriptor instead.
// 使用 AllocationStrategy.ProtoReflect.Descriptor 替代
func (*AllocationStrategy) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{1}
}

// 获取 AllocationStrategy 的类型
func (x *AllocationStrategy) GetType() AllocationStrategy_Type {
    if x != nil {
        return x.Type
    }
    return AllocationStrategy_Always
}

// 获取 AllocationStrategy 的并发性
func (x *AllocationStrategy) GetConcurrency() *AllocationStrategy_AllocationStrategyConcurrency {
    if x != nil {
        return x.Concurrency
    }
    return nil
}

// 获取 AllocationStrategy 的刷新策略
func (x *AllocationStrategy) GetRefresh() *AllocationStrategy_AllocationStrategyRefresh {
    if x != nil {
        return x.Refresh
    }
    return nil
}

// SniffingConfig 结构体
type SniffingConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 是否在入站连接上启用内容嗅探
    Enabled bool `protobuf:"varint,1,opt,name=enabled,proto3" json:"enabled,omitempty"`
    // 如果嗅探到的协议在给定列表中，则覆盖目标目的地
    // 支持的值为 "http", "tls"
    DestinationOverride []string `protobuf:"bytes,2,rep,name=destination_override,json=destinationOverride,proto3" json:"destination_override,omitempty"`
}

// 重置 SniffingConfig 结构体
func (x *SniffingConfig) Reset() {
    *x = SniffingConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SniffingConfig 结构体的字符串表示
func (x *SniffingConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SniffingConfig) ProtoMessage() {}

// 获取 SniffingConfig 结构体的反射信息
func (x *SniffingConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
}
    # 返回对象 x 的消息
    return mi.MessageOf(x)
// Deprecated: Use SniffingConfig.ProtoReflect.Descriptor instead.
// 返回 SniffingConfig 的描述符
func (*SniffingConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{2}
}

// 返回 SniffingConfig 的 Enabled 字段值
func (x *SniffingConfig) GetEnabled() bool {
    if x != nil {
        return x.Enabled
    }
    return false
}

// 返回 SniffingConfig 的 DestinationOverride 字段值
func (x *SniffingConfig) GetDestinationOverride() []string {
    if x != nil {
        return x.DestinationOverride
    }
    return nil
}

// ReceiverConfig 结构体定义
type ReceiverConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // PortRange 指定接收器应监听的端口
    PortRange *net.PortRange `protobuf:"bytes,1,opt,name=port_range,json=portRange,proto3" json:"port_range,omitempty"`
    // Listen 指定接收器应监听的 IP 地址
    Listen                     *net.IPOrDomain        `protobuf:"bytes,2,opt,name=listen,proto3" json:"listen,omitempty"`
    AllocationStrategy         *AllocationStrategy    `protobuf:"bytes,3,opt,name=allocation_strategy,json=allocationStrategy,proto3" json:"allocation_strategy,omitempty"`
    StreamSettings             *internet.StreamConfig `protobuf:"bytes,4,opt,name=stream_settings,json=streamSettings,proto3" json:"stream_settings,omitempty"`
    ReceiveOriginalDestination bool                   `protobuf:"varint,5,opt,name=receive_original_destination,json=receiveOriginalDestination,proto3" json:"receive_original_destination,omitempty"`
    // 给定协议的域名覆盖
    // 已弃用。使用 sniffing_settings。
    //
    // 已弃用：不要使用。
    DomainOverride   []KnownProtocols `protobuf:"varint,7,rep,packed,name=domain_override,json=domainOverride,proto3,enum=v2ray.core.app.proxyman.KnownProtocols" json:"domain_override,omitempty"`
    SniffingSettings *SniffingConfig  `protobuf:"bytes,8,opt,name=sniffing_settings,json=sniffingSettings,proto3" json:"sniffing_settings,omitempty"`
}
// 重置接收器配置为默认值
func (x *ReceiverConfig) Reset() {
    *x = ReceiverConfig{}
    // 如果启用了不安全操作，则获取消息类型信息并存储
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回接收器配置的字符串表示形式
func (x *ReceiverConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ReceiverConfig) ProtoMessage() {}

// 返回接收器配置的反射信息
func (x *ReceiverConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[3]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用 ReceiverConfig.ProtoReflect.Descriptor 替代
func (*ReceiverConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{3}
}

// 返回接收器配置的端口范围
func (x *ReceiverConfig) GetPortRange() *net.PortRange {
    if x != nil {
        return x.PortRange
    }
    return nil
}

// 返回接收器配置的监听地址
func (x *ReceiverConfig) GetListen() *net.IPOrDomain {
    if x != nil {
        return x.Listen
    }
    return nil
}

// 返回接收器配置的分配策略
func (x *ReceiverConfig) GetAllocationStrategy() *AllocationStrategy {
    if x != nil {
        return x.AllocationStrategy
    }
    return nil
}

// 返回接收器配置的流设置
func (x *ReceiverConfig) GetStreamSettings() *internet.StreamConfig {
    if x != nil {
        return x.StreamSettings
    }
    return nil
}

// 返回是否接收原始目标的布尔值
func (x *ReceiverConfig) GetReceiveOriginalDestination() bool {
    if x != nil {
        return x.ReceiveOriginalDestination
    }
    return false
}

// Deprecated: 不要使用
func (x *ReceiverConfig) GetDomainOverride() []KnownProtocols {
    if x != nil {
        return x.DomainOverride
    }
    return nil
}

// 返回接收器配置的嗅探设置
func (x *ReceiverConfig) GetSniffingSettings() *SniffingConfig {
    if x != nil {
        return x.SniffingSettings
    }
    return nil
}

// 入站处理程序配置
type InboundHandlerConfig struct {
    # 定义消息状态
    state         protoimpl.MessageState
    # 定义消息大小缓存
    sizeCache     protoimpl.SizeCache
    # 定义未知字段
    unknownFields protoimpl.UnknownFields

    # 定义标签，类型为字符串，protobuf 标签为1，可选字段
    Tag              string               `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    # 定义接收者设置，类型为 serial.TypedMessage，protobuf 标签为2，可选字段
    ReceiverSettings *serial.TypedMessage `protobuf:"bytes,2,opt,name=receiver_settings,json=receiverSettings,proto3" json:"receiver_settings,omitempty"`
    # 定义代理设置，类型为 serial.TypedMessage，protobuf 标签为3，可选字段
    ProxySettings    *serial.TypedMessage `protobuf:"bytes,3,opt,name=proxy_settings,json=proxySettings,proto3" json:"proxy_settings,omitempty"`
// 重置入站处理程序配置对象
func (x *InboundHandlerConfig) Reset() {
    // 将对象重置为空对象
    *x = InboundHandlerConfig{}
    // 如果启用了不安全操作，则获取消息类型信息并存储
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[4]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回入站处理程序配置对象的字符串表示形式
func (x *InboundHandlerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*InboundHandlerConfig) ProtoMessage() {}

// 返回入站处理程序配置对象的反射信息
func (x *InboundHandlerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[4]
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

// 已弃用：使用 InboundHandlerConfig.ProtoReflect.Descriptor 替代
func (*InboundHandlerConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{4}
}

// 获取入站处理程序配置对象的标签
func (x *InboundHandlerConfig) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取入站处理程序配置对象的接收器设置
func (x *InboundHandlerConfig) GetReceiverSettings() *serial.TypedMessage {
    if x != nil {
        return x.ReceiverSettings
    }
    return nil
}

// 获取入站处理程序配置对象的代理设置
func (x *InboundHandlerConfig) GetProxySettings() *serial.TypedMessage {
    if x != nil {
        return x.ProxySettings
    }
    return nil
}

// 重置出站配置对象
func (x *OutboundConfig) Reset() {
    // 将对象重置为空对象
    *x = OutboundConfig{}
    // 如果启用了不安全操作，则获取消息类型信息并存储
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回出站配置对象的字符串表示形式
func (x *OutboundConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*OutboundConfig) ProtoMessage() {}

// 返回出站配置对象的反射信息
func (x *OutboundConfig) ProtoReflect() protoreflect.Message {
    # 获取文件类型为file_app_proxyman_config_proto_msgTypes的第5个元素的指针
    mi := &file_app_proxyman_config_proto_msgTypes[5]
    # 如果启用了不安全操作并且x不为空
    if protoimpl.UnsafeEnabled && x != nil {
        # 获取x的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 如果消息状态中的消息信息为空
        if ms.LoadMessageInfo() == nil {
            # 存储消息信息
            ms.StoreMessageInfo(mi)
        }
        # 返回消息状态
        return ms
    }
    # 返回消息信息
    return mi.MessageOf(x)
// Deprecated: Use OutboundConfig.ProtoReflect.Descriptor instead.
// 使用 OutboundConfig.ProtoReflect.Descriptor 替代
func (*OutboundConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{5}
}

// SenderConfig 结构体定义
type SenderConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 通过给定的 IP 发送流量。只允许 IP。
    Via               *net.IPOrDomain        `protobuf:"bytes,1,opt,name=via,proto3" json:"via,omitempty"`
    // 流设置
    StreamSettings    *internet.StreamConfig `protobuf:"bytes,2,opt,name=stream_settings,json=streamSettings,proto3" json:"stream_settings,omitempty"`
    // 代理设置
    ProxySettings     *internet.ProxyConfig  `protobuf:"bytes,3,opt,name=proxy_settings,json=proxySettings,proto3" json:"proxy_settings,omitempty"`
    // 多路复用设置
    MultiplexSettings *MultiplexingConfig    `protobuf:"bytes,4,opt,name=multiplex_settings,json=multiplexSettings,proto3" json:"multiplex_settings,omitempty"`
}

// 重置 SenderConfig 结构体
func (x *SenderConfig) Reset() {
    *x = SenderConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[6]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SenderConfig 结构体的字符串表示
func (x *SenderConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// SenderConfig 结构体实现 ProtoMessage 接口
func (*SenderConfig) ProtoMessage() {}

// 返回 SenderConfig 结构体的 ProtoReflect
func (x *SenderConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[6]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use SenderConfig.ProtoReflect.Descriptor instead.
// 使用 SenderConfig.ProtoReflect.Descriptor 替代
func (*SenderConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{6}
}

// 获取 SenderConfig 结构体的 Via 字段
func (x *SenderConfig) GetVia() *net.IPOrDomain {
    if x != nil {
        return x.Via
    }
}
    # 返回空值
    return nil
// 获取 SenderConfig 结构体的流配置信息
func (x *SenderConfig) GetStreamSettings() *internet.StreamConfig {
    // 如果 SenderConfig 不为空，则返回其流配置信息
    if x != nil {
        return x.StreamSettings
    }
    // 否则返回空
    return nil
}

// 获取 SenderConfig 结构体的代理配置信息
func (x *SenderConfig) GetProxySettings() *internet.ProxyConfig {
    // 如果 SenderConfig 不为空，则返回其代理配置信息
    if x != nil {
        return x.ProxySettings
    }
    // 否则返回空
    return nil
}

// 获取 SenderConfig 结构体的多路复用配置信息
func (x *SenderConfig) GetMultiplexSettings() *MultiplexingConfig {
    // 如果 SenderConfig 不为空，则返回其多路复用配置信息
    if x != nil {
        return x.MultiplexSettings
    }
    // 否则返回空
    return nil
}

// MultiplexingConfig 结构体定义
type MultiplexingConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 是否启用多路复用
    Enabled bool `protobuf:"varint,1,opt,name=enabled,proto3" json:"enabled,omitempty"`
    // 单个多路复用连接可以处理的最大并发连接数
    Concurrency uint32 `protobuf:"varint,2,opt,name=concurrency,proto3" json:"concurrency,omitempty"`
}

// 重置 MultiplexingConfig 结构体
func (x *MultiplexingConfig) Reset() {
    *x = MultiplexingConfig{}
    // 如果启用了不安全操作，则设置消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[7]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 MultiplexingConfig 结构体的字符串表示
func (x *MultiplexingConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*MultiplexingConfig) ProtoMessage() {}

// 返回 MultiplexingConfig 结构体的反射信息
func (x *MultiplexingConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[7]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 MultiplexingConfig.ProtoReflect.Descriptor 替代
func (*MultiplexingConfig) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{7}
}

// 获取 MultiplexingConfig 结构体的 Enabled 字段值
func (x *MultiplexingConfig) GetEnabled() bool {
    // 如果 MultiplexingConfig 不为空，则返回其 Enabled 字段值
    if x != nil {
        return x.Enabled
    }
    // 否则返回 false
    return false
}
// 获取并发数配置，如果配置对象不为空则返回并发数，否则返回0
func (x *MultiplexingConfig) GetConcurrency() uint32 {
    if x != nil {
        return x.Concurrency
    }
    return 0
}

// AllocationStrategy_AllocationStrategyConcurrency 结构体定义
type AllocationStrategy_AllocationStrategyConcurrency struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`
}

// 重置 AllocationStrategy_AllocationStrategyConcurrency 结构体
func (x *AllocationStrategy_AllocationStrategyConcurrency) Reset() {
    *x = AllocationStrategy_AllocationStrategyConcurrency{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_config_proto_msgTypes[8]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 AllocationStrategy_AllocationStrategyConcurrency 结构体的字符串表示
func (x *AllocationStrategy_AllocationStrategyConcurrency) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*AllocationStrategy_AllocationStrategyConcurrency) ProtoMessage() {}

// 返回 AllocationStrategy_AllocationStrategyConcurrency 结构体的反射信息
func (x *AllocationStrategy_AllocationStrategyConcurrency) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_config_proto_msgTypes[8]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use AllocationStrategy_AllocationStrategyConcurrency.ProtoReflect.Descriptor instead.
// 返回 AllocationStrategy_AllocationStrategyConcurrency 结构体的描述符
func (*AllocationStrategy_AllocationStrategyConcurrency) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{1, 0}
}

// 获取 AllocationStrategy_AllocationStrategyConcurrency 结构体的值，如果结构体不为空则返回值，否则返回0
func (x *AllocationStrategy_AllocationStrategyConcurrency) GetValue() uint32 {
    if x != nil {
        return x.Value
    }
    return 0
}

// AllocationStrategy_AllocationStrategyRefresh 结构体定义
type AllocationStrategy_AllocationStrategyRefresh struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`
}
// 重置 AllocationStrategy_AllocationStrategyRefresh 对象
func (x *AllocationStrategy_AllocationStrategyRefresh) Reset() {
    // 将对象重置为空对象
    *x = AllocationStrategy_AllocationStrategyRefresh{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_proxyman_config_proto_msgTypes[9]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 AllocationStrategy_AllocationStrategyRefresh 对象的字符串表示
func (x *AllocationStrategy_AllocationStrategyRefresh) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*AllocationStrategy_AllocationStrategyRefresh) ProtoMessage() {}

// 返回 AllocationStrategy_AllocationStrategyRefresh 对象的反射信息
func (x *AllocationStrategy_AllocationStrategyRefresh) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_proxyman_config_proto_msgTypes[9]
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

// 已弃用：使用 AllocationStrategy_AllocationStrategyRefresh.ProtoReflect.Descriptor 替代
func (*AllocationStrategy_AllocationStrategyRefresh) Descriptor() ([]byte, []int) {
    return file_app_proxyman_config_proto_rawDescGZIP(), []int{1, 1}
}

// 获取 AllocationStrategy_AllocationStrategyRefresh 对象的值
func (x *AllocationStrategy_AllocationStrategyRefresh) GetValue() uint32 {
    // 如果对象不为空，则返回其值，否则返回 0
    if x != nil {
        return x.Value
    }
    return 0
}

// 定义 File_app_proxyman_config_proto 和 file_app_proxyman_config_proto_rawDesc
var File_app_proxyman_config_proto protoreflect.FileDescriptor

var file_app_proxyman_config_proto_rawDesc = []byte{
    // 以下为文件的原始描述信息的字节表示
    0x0a, 0x19, 0x61, 0x70, 0x70, 0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2f, 0x63,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x17, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78,
    0x79, 0x6d, 0x61, 0x6e, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74,
    0x2f, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x15,
    // ...
    # 以下是十六进制数值，可能是某种配置或数据的表示方式
    0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x70, 0x6f, 0x72, 0x74, 0x2e,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x1f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74,
    0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67,
    0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x73,
    0x65, 0x72, 0x69, 0x61, 0x6c, 0x2f, 0x74, 0x79, 0x70, 0x65, 0x64, 0x5f, 0x6d, 0x65, 0x73, 0x73,
    0x61, 0x67, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x0f, 0x0a, 0x0d, 0x49, 0x6e, 0x62,
    0x6f, 0x75, 0x6e, 0x64, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x22, 0xc0, 0x03, 0x0a, 0x12, 0x41,
    0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67,
    0x79, 0x12, 0x44, 0x0a, 0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32,
    0x30, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
    0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61,
    0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x2e, 0x54, 0x79, 0x70,
    0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65, 0x12, 0x6b, 0x0a, 0x0b, 0x63, 0x6f, 0x6e, 0x63, 0x75,
    0x72, 0x72, 0x65, 0x6e, 0x63, 0x79, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x49, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72,
    0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f,
    0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x2e, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61,
    0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x43, 0x6f, 0x6e, 0x63,
    0x75, 0x72, 0x72, 0x65, 0x6e, 0x63, 0x79, 0x52, 0x0b, 0x63, 0x6f, 0x6e, 0x63, 0x75, 0x72, 0x72,
    0x65, 0x6e, 0x63, 0x79, 0x12, 0x5f, 0x0a, 0x07, 0x72, 0x65, 0x66, 0x72, 0x65, 0x73, 0x68, 0x18,
    # 定义一个十六进制数组
    0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x45, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e,
    0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65,
    0x67, 0x79, 0x2e, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72,
    0x61, 0x74, 0x65, 0x67, 0x79, 0x52, 0x65, 0x66, 0x72, 0x65, 0x73, 0x68, 0x52, 0x07, 0x72, 0x65,
    0x66, 0x72, 0x65, 0x73, 0x68, 0x1a, 0x35, 0x0a, 0x1d, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74,
    0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x43, 0x6f, 0x6e, 0x63, 0x75,
    0x72, 0x72, 0x65, 0x6e, 0x63, 0x79, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18,
    0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x1a, 0x31, 0x0a, 0x19,
    0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65,
    0x67, 0x79, 0x52, 0x65, 0x66, 0x72, 0x65, 0x73, 0x68, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c,
    0x75, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22,
    0x2c, 0x0a, 0x04, 0x54, 0x79, 0x70, 0x65, 0x12, 0x0a, 0x0a, 0x06, 0x41, 0x6c, 0x77, 0x61, 0x79,
    0x73, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x52, 0x61, 0x6e, 0x64, 0x6f, 0x6d, 0x10, 0x01, 0x12,
    0x0c, 0x0a, 0x08, 0x45, 0x78, 0x74, 0x65, 0x72, 0x6e, 0x61, 0x6c, 0x10, 0x02, 0x22, 0x5d, 0x0a,
    0x0e, 0x53, 0x6e, 0x69, 0x66, 0x66, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x18, 0x0a, 0x07, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08,
    0x52, 0x07, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x12, 0x31, 0x0a, 0x14, 0x64, 0x65, 0x73,
    0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x5f, 0x6f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64,
    0x65, 0x18, 0x02, 0x20, 0x03, 0x28, 0x09, 0x52, 0x13, 0x64, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61,
    # 创建一个十六进制数列表，可能是某种配置或参数的表示方式
    0x74, 0x69, 0x6f, 0x6e, 0x4f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65, 0x22, 0xb4, 0x04, 0x0a,
    0x0e, 0x52, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x3f, 0x0a, 0x0a, 0x70, 0x6f, 0x72, 0x74, 0x5f, 0x72, 0x61, 0x6e, 0x67, 0x65, 0x18, 0x01, 0x20,
    0x01, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74,
    0x52, 0x61, 0x6e, 0x67, 0x65, 0x52, 0x09, 0x70, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65,
    0x12, 0x39, 0x0a, 0x06, 0x6c, 0x69, 0x73, 0x74, 0x65, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f,
    0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x49, 0x50, 0x4f, 0x72, 0x44, 0x6f, 0x6d,
    0x61, 0x69, 0x6e, 0x52, 0x06, 0x6c, 0x69, 0x73, 0x74, 0x65, 0x6e, 0x12, 0x5c, 0x0a, 0x13, 0x61,
    0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x5f, 0x73, 0x74, 0x72, 0x61, 0x74, 0x65,
    0x67, 0x79, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d,
    0x61, 0x6e, 0x2e, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x74, 0x72,
    0x61, 0x74, 0x65, 0x67, 0x79, 0x52, 0x12, 0x61, 0x6c, 0x6c, 0x6f, 0x63, 0x61, 0x74, 0x69, 0x6f,
    0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x54, 0x0a, 0x0f, 0x73, 0x74, 0x72,
    0x65, 0x61, 0x6d, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x04, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x2b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e,
    0x65, 0x74, 0x2e, 0x53, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52,
    # 创建一个十六进制数值列表
    0x0e, 0x73, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x12,
    0x40, 0x0a, 0x1c, 0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x5f, 0x6f, 0x72, 0x69, 0x67, 0x69,
    0x6e, 0x61, 0x6c, 0x5f, 0x64, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x18,
    0x05, 0x20, 0x01, 0x28, 0x08, 0x52, 0x1a, 0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x4f, 0x72,
    0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x44, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f,
    0x6e, 0x12, 0x54, 0x0a, 0x0f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x5f, 0x6f, 0x76, 0x65, 0x72,
    0x72, 0x69, 0x64, 0x65, 0x18, 0x07, 0x20, 0x03, 0x28, 0x0e, 0x32, 0x27, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78,
    0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x4b, 0x6e, 0x6f, 0x77, 0x6e, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63,
    0x6f, 0x6c, 0x73, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0e, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4f,
    0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65, 0x12, 0x54, 0x0a, 0x11, 0x73, 0x6e, 0x69, 0x66, 0x66,
    0x69, 0x6e, 0x67, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x08, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x27, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x53, 0x6e, 0x69,
    0x66, 0x66, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x10, 0x73, 0x6e, 0x69,
    0x66, 0x66, 0x69, 0x6e, 0x67, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x4a, 0x04, 0x08,
    0x06, 0x10, 0x07, 0x22, 0xcc, 0x01, 0x0a, 0x14, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x48,
    0x61, 0x6e, 0x64, 0x6c, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x10, 0x0a, 0x03,
    0x74, 0x61, 0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12, 0x53,
    0x0a, 0x11, 0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x72, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69,
    # 定义一组十六进制数，可能是某种配置或数据的表示
    0x6e, 0x67, 0x73, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65,
    0x72, 0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67,
    0x65, 0x52, 0x10, 0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x72, 0x53, 0x65, 0x74, 0x74, 0x69,
    0x6e, 0x67, 0x73, 0x12, 0x4d, 0x0a, 0x0e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x5f, 0x73, 0x65, 0x74,
    0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
    0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73,
    0x61, 0x67, 0x65, 0x52, 0x0d, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e,
    0x67, 0x73, 0x22, 0x10, 0x0a, 0x0e, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x43, 0x6f,
    0x6e, 0x66, 0x69, 0x67, 0x22, 0xc8, 0x02, 0x0a, 0x0c, 0x53, 0x65, 0x6e, 0x64, 0x65, 0x72, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x33, 0x0a, 0x03, 0x76, 0x69, 0x61, 0x18, 0x01, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x49, 0x50, 0x4f, 0x72, 0x44,
    0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x03, 0x76, 0x69, 0x61, 0x12, 0x54, 0x0a, 0x0f, 0x73, 0x74,
    0x72, 0x65, 0x61, 0x6d, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x02, 0x20,
    0x01, 0x28, 0x0b, 0x32, 0x2b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72,
    0x6e, 0x65, 0x74, 0x2e, 0x53, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67,
    0x52, 0x0e, 0x73, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73,
    # 以下是十六进制数的表示，可能是某种配置或数据的编码
    0x12, 0x51, 0x0a, 0x0e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e,
    0x67, 0x73, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e,
    0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x43, 0x6f,
    0x6e, 0x66, 0x69, 0x67, 0x52, 0x0d, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x53, 0x65, 0x74, 0x74, 0x69,
    0x6e, 0x67, 0x73, 0x12, 0x5a, 0x0a, 0x12, 0x6d, 0x75, 0x6c, 0x74, 0x69, 0x70, 0x6c, 0x65, 0x78,
    0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0b, 0x32,
    0x2b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
    0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x4d, 0x75, 0x6c, 0x74, 0x69, 0x70,
    0x6c, 0x65, 0x78, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x11, 0x6d, 0x75,
    0x6c, 0x74, 0x69, 0x70, 0x6c, 0x65, 0x78, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x22,
    0x50, 0x0a, 0x12, 0x4d, 0x75, 0x6c, 0x74, 0x69, 0x70, 0x6c, 0x65, 0x78, 0x69, 0x6e, 0x67, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x18, 0x0a, 0x07, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x12,
    0x20, 0x0a, 0x0b, 0x63, 0x6f, 0x6e, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x63, 0x79, 0x18, 0x02,
    0x20, 0x01, 0x28, 0x0d, 0x52, 0x0b, 0x63, 0x6f, 0x6e, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x63,
    0x79, 0x2a, 0x23, 0x0a, 0x0e, 0x4b, 0x6e, 0x6f, 0x77, 0x6e, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63,
    0x6f, 0x6c, 0x73, 0x12, 0x08, 0x0a, 0x04, 0x48, 0x54, 0x54, 0x50, 0x10, 0x00, 0x12, 0x07, 0x0a,
    0x03, 0x54, 0x4c, 0x53, 0x10, 0x01, 0x42, 0x56, 0x0a, 0x1b, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f,
    # 这是一串十六进制数，可能是用于某种数据加密或编码的值
    # 无法确定具体含义，需要根据上下文和使用情况来解释
// 定义变量file_app_proxyman_config_proto_rawDescOnce，类型为sync.Once，用于确保file_app_proxyman_config_proto_rawDescData只被初始化一次
var (
    file_app_proxyman_config_proto_rawDescOnce sync.Once
    file_app_proxyman_config_proto_rawDescData = file_app_proxyman_config_proto_rawDesc
)

// 定义函数file_app_proxyman_config_proto_rawDescGZIP，返回类型为[]byte，用于对file_app_proxyman_config_proto_rawDescData进行GZIP压缩
func file_app_proxyman_config_proto_rawDescGZIP() []byte {
    // 使用sync.Once确保file_app_proxyman_config_proto_rawDescData只被压缩一次
    file_app_proxyman_config_proto_rawDescOnce.Do(func() {
        file_app_proxyman_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_proxyman_config_proto_rawDescData)
    })
    return file_app_proxyman_config_proto_rawDescData
}

// 定义变量file_app_proxyman_config_proto_enumTypes，类型为[]protoimpl.EnumInfo，用于存储枚举类型信息
var file_app_proxyman_config_proto_enumTypes = make([]protoimpl.EnumInfo, 2)
// 定义变量file_app_proxyman_config_proto_msgTypes，类型为[]protoimpl.MessageInfo，用于存储消息类型信息
var file_app_proxyman_config_proto_msgTypes = make([]protoimpl.MessageInfo, 10)
// 定义变量file_app_proxyman_config_proto_goTypes，类型为[]interface{}，用于存储Go类型信息
var file_app_proxyman_config_proto_goTypes = []interface{}{
    (KnownProtocols)(0),                                      // 0: v2ray.core.app.proxyman.KnownProtocols
    (AllocationStrategy_Type)(0),                             // 1: v2ray.core.app.proxyman.AllocationStrategy.Type
    (*InboundConfig)(nil),                                    // 2: v2ray.core.app.proxyman.InboundConfig
    (*AllocationStrategy)(nil),                               // 3: v2ray.core.app.proxyman.AllocationStrategy
    (*SniffingConfig)(nil),                                   // 4: v2ray.core.app.proxyman.SniffingConfig
    (*ReceiverConfig)(nil),                                   // 5: v2ray.core.app.proxyman.ReceiverConfig
    (*InboundHandlerConfig)(nil),                             // 6: v2ray.core.app.proxyman.InboundHandlerConfig
    (*OutboundConfig)(nil),                                   // 7: v2ray.core.app.proxyman.OutboundConfig
    (*SenderConfig)(nil),                                     // 8: v2ray.core.app.proxyman.SenderConfig
    (*MultiplexingConfig)(nil),                               // 9: v2ray.core.app.proxyman.MultiplexingConfig
    (*AllocationStrategy_AllocationStrategyConcurrency)(nil), // 10: v2ray.core.app.proxyman.AllocationStrategy.AllocationStrategyConcurrency
    (*AllocationStrategy_AllocationStrategyRefresh)(nil),     // 11: 声明一个指向 AllocationStrategy_AllocationStrategyRefresh 类型的空指针
    (*net.PortRange)(nil),                                    // 12: 声明一个指向 PortRange 类型的空指针
    (*net.IPOrDomain)(nil),                                   // 13: 声明一个指向 IPOrDomain 类型的空指针
    (*internet.StreamConfig)(nil),                            // 14: 声明一个指向 StreamConfig 类型的空指针
    (*serial.TypedMessage)(nil),                              // 15: 声明一个指向 TypedMessage 类型的空指针
    (*internet.ProxyConfig)(nil),                             // 16: 声明一个指向 ProxyConfig 类型的空指针
// 定义一个名为file_app_proxyman_config_proto_depIdxs的变量，类型为int32的切片
var file_app_proxyman_config_proto_depIdxs = []int32{
    1,  // 0: v2ray.core.app.proxyman.AllocationStrategy.type:type_name -> v2ray.core.app.proxyman.AllocationStrategy.Type
    10, // 1: v2ray.core.app.proxyman.AllocationStrategy.concurrency:type_name -> v2ray.core.app.proxyman.AllocationStrategy.AllocationStrategyConcurrency
    11, // 2: v2ray.core.app.proxyman.AllocationStrategy.refresh:type_name -> v2ray.core.app.proxyman.AllocationStrategy.AllocationStrategyRefresh
    12, // 3: v2ray.core.app.proxyman.ReceiverConfig.port_range:type_name -> v2ray.core.common.net.PortRange
    13, // 4: v2ray.core.app.proxyman.ReceiverConfig.listen:type_name -> v2ray.core.common.net.IPOrDomain
    3,  // 5: v2ray.core.app.proxyman.ReceiverConfig.allocation_strategy:type_name -> v2ray.core.app.proxyman.AllocationStrategy
    14, // 6: v2ray.core.app.proxyman.ReceiverConfig.stream_settings:type_name -> v2ray.core.transport.internet.StreamConfig
    0,  // 7: v2ray.core.app.proxyman.ReceiverConfig.domain_override:type_name -> v2ray.core.app.proxyman.KnownProtocols
    4,  // 8: v2ray.core.app.proxyman.ReceiverConfig.sniffing_settings:type_name -> v2ray.core.app.proxyman.SniffingConfig
    15, // 9: v2ray.core.app.proxyman.InboundHandlerConfig.receiver_settings:type_name -> v2ray.core.common.serial.TypedMessage
    15, // 10: v2ray.core.app.proxyman.InboundHandlerConfig.proxy_settings:type_name -> v2ray.core.common.serial.TypedMessage
    13, // 11: v2ray.core.app.proxyman.SenderConfig.via:type_name -> v2ray.core.common.net.IPOrDomain
    14, // 12: v2ray.core.app.proxyman.SenderConfig.stream_settings:type_name -> v2ray.core.transport.internet.StreamConfig
    16, // 13: v2ray.core.app.proxyman.SenderConfig.proxy_settings:type_name -> v2ray.core.transport.internet.ProxyConfig
    9,  // 14: v2ray.core.app.proxyman.SenderConfig.multiplex_settings:type_name -> v2ray.core.app.proxyman.MultiplexingConfig
    15, // [15:15] is the sub-list for method output_type
}
    // 定义一个整数变量，表示方法的输入类型
    15, // [15:15] is the sub-list for method input_type
    // 定义一个整数变量，表示扩展类型的名称
    15, // [15:15] is the sub-list for extension type_name
    // 定义一个整数变量，表示扩展的扩展类型
    15, // [15:15] is the sub-list for extension extendee
    // 定义一个整数变量，表示字段的类型名称
    0,  // [0:15] is the sub-list for field type_name
func init() { file_app_proxyman_config_proto_init() }
func file_app_proxyman_config_proto_init() {
    // 如果 File_app_proxyman_config_proto 不为空，则直接返回，不进行初始化操作
    if File_app_proxyman_config_proto != nil {
        return
    }
    // 定义一个空结构体 x
    type x struct{}
    // 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            // 设置 GoPackagePath 为结构体 x 的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            // 设置 RawDescriptor 为 file_app_proxyman_config_proto_rawDesc
            RawDescriptor: file_app_proxyman_config_proto_rawDesc,
            // 设置 NumEnums 为 2，NumMessages 为 10，NumExtensions 和 NumServices 为 0
            NumEnums:      2,
            NumMessages:   10,
            NumExtensions: 0,
            NumServices:   0,
        },
        // 设置 GoTypes 为 file_app_proxyman_config_proto_goTypes
        GoTypes:           file_app_proxyman_config_proto_goTypes,
        // 设置 DependencyIndexes 为 file_app_proxyman_config_proto_depIdxs
        DependencyIndexes: file_app_proxyman_config_proto_depIdxs,
        // 设置 EnumInfos 为 file_app_proxyman_config_proto_enumTypes，MessageInfos 为 file_app_proxyman_config_proto_msgTypes
        EnumInfos:         file_app_proxyman_config_proto_enumTypes,
        MessageInfos:      file_app_proxyman_config_proto_msgTypes,
    }.Build()
    // 将构建好的类型赋值给 File_app_proxyman_config_proto
    File_app_proxyman_config_proto = out.File
    // 将 file_app_proxyman_config_proto_rawDesc、file_app_proxyman_config_proto_goTypes、file_app_proxyman_config_proto_depIdxs 置空
    file_app_proxyman_config_proto_rawDesc = nil
    file_app_proxyman_config_proto_goTypes = nil
    file_app_proxyman_config_proto_depIdxs = nil
}
```