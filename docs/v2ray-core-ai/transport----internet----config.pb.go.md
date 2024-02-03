# `v2ray-core\transport\internet\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/config.proto

package internet

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type TransportProtocol int32  // 定义 TransportProtocol 类型为 int32

const (
    TransportProtocol_TCP          TransportProtocol = 0  // 定义 TransportProtocol_TCP 值为 0
    TransportProtocol_UDP          TransportProtocol = 1  // 定义 TransportProtocol_UDP 值为 1
    TransportProtocol_MKCP         TransportProtocol = 2  // 定义 TransportProtocol_MKCP 值为 2
    TransportProtocol_WebSocket    TransportProtocol = 3  // 定义 TransportProtocol_WebSocket 值为 3
    TransportProtocol_HTTP         TransportProtocol = 4  // 定义 TransportProtocol_HTTP 值为 4
    TransportProtocol_DomainSocket TransportProtocol = 5  // 定义 TransportProtocol_DomainSocket 值为 5
)

// TransportProtocol 的枚举值映射。
var (
    TransportProtocol_name = map[int32]string{  // 定义 TransportProtocol_name 映射
        0: "TCP",
        1: "UDP",
        2: "MKCP",
        3: "WebSocket",
        4: "HTTP",
        5: "DomainSocket",
    }
    TransportProtocol_value = map[string]int32{  // 定义 TransportProtocol_value 映射
        "TCP":          0,
        "UDP":          1,
        "MKCP":         2,
        "WebSocket":    3,
        "HTTP":         4,
        "DomainSocket": 5,
    }
)

func (x TransportProtocol) Enum() *TransportProtocol {  // 定义 TransportProtocol 的 Enum 方法
    p := new(TransportProtocol)
    *p = x
    return p
}

func (x TransportProtocol) String() string {  // 定义 TransportProtocol 的 String 方法
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}
// 返回传输协议的枚举描述符
func (TransportProtocol) Descriptor() protoreflect.EnumDescriptor {
    return file_transport_internet_config_proto_enumTypes[0].Descriptor()
}

// 返回传输协议的枚举类型
func (TransportProtocol) Type() protoreflect.EnumType {
    return &file_transport_internet_config_proto_enumTypes[0]
}

// 返回传输协议的枚举数值
func (x TransportProtocol) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: Use TransportProtocol.Descriptor instead.
// 返回传输协议的枚举描述符，已废弃
func (TransportProtocol) EnumDescriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{0}
}

// 定义 SocketConfig_TCPFastOpenState 类型
type SocketConfig_TCPFastOpenState int32

// 定义 SocketConfig_TCPFastOpenState 的枚举值
const (
    // AsIs 用于保持当前 TFO 状态不变
    SocketConfig_AsIs SocketConfig_TCPFastOpenState = 0
    // Enable 用于显式启用 TFO
    SocketConfig_Enable SocketConfig_TCPFastOpenState = 1
    // Disable 用于显式禁用 TFO
    SocketConfig_Disable SocketConfig_TCPFastOpenState = 2
)

// SocketConfig_TCPFastOpenState 的枚举值映射
var (
    SocketConfig_TCPFastOpenState_name = map[int32]string{
        0: "AsIs",
        1: "Enable",
        2: "Disable",
    }
    SocketConfig_TCPFastOpenState_value = map[string]int32{
        "AsIs":    0,
        "Enable":  1,
        "Disable": 2,
    }
)

// 返回 SocketConfig_TCPFastOpenState 的枚举值
func (x SocketConfig_TCPFastOpenState) Enum() *SocketConfig_TCPFastOpenState {
    p := new(SocketConfig_TCPFastOpenState)
    *p = x
    return p
}

// 返回 SocketConfig_TCPFastOpenState 的字符串表示
func (x SocketConfig_TCPFastOpenState) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

// 返回 SocketConfig_TCPFastOpenState 的枚举描述符
func (SocketConfig_TCPFastOpenState) Descriptor() protoreflect.EnumDescriptor {
    return file_transport_internet_config_proto_enumTypes[1].Descriptor()
}

// 返回 SocketConfig_TCPFastOpenState 的枚举类型
func (SocketConfig_TCPFastOpenState) Type() protoreflect.EnumType {
    return &file_transport_internet_config_proto_enumTypes[1]
}

// 返回 SocketConfig_TCPFastOpenState 的枚举数值
func (x SocketConfig_TCPFastOpenState) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}
// Deprecated: Use SocketConfig_TCPFastOpenState.Descriptor instead.
func (SocketConfig_TCPFastOpenState) EnumDescriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{3, 0}
}

type SocketConfig_TProxyMode int32

const (
    // TProxy is off.
    SocketConfig_Off SocketConfig_TProxyMode = 0
    // TProxy mode.
    SocketConfig_TProxy SocketConfig_TProxyMode = 1
    // Redirect mode.
    SocketConfig_Redirect SocketConfig_TProxyMode = 2
)

// Enum value maps for SocketConfig_TProxyMode.
var (
    // 定义枚举值名称到枚举值的映射
    SocketConfig_TProxyMode_name = map[int32]string{
        0: "Off",
        1: "TProxy",
        2: "Redirect",
    }
    // 定义枚举值到枚举值名称的映射
    SocketConfig_TProxyMode_value = map[string]int32{
        "Off":      0,
        "TProxy":   1,
        "Redirect": 2,
    }
)

// 返回枚举值的指针
func (x SocketConfig_TProxyMode) Enum() *SocketConfig_TProxyMode {
    p := new(SocketConfig_TProxyMode)
    *p = x
    return p
}

// 返回枚举值的字符串表示
func (x SocketConfig_TProxyMode) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

// 返回枚举值的描述符
func (SocketConfig_TProxyMode) Descriptor() protoreflect.EnumDescriptor {
    return file_transport_internet_config_proto_enumTypes[2].Descriptor()
}

// 返回枚举值的类型
func (SocketConfig_TProxyMode) Type() protoreflect.EnumType {
    return &file_transport_internet_config_proto_enumTypes[2]
}

// 返回枚举值的数字
func (x SocketConfig_TProxyMode) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: Use SocketConfig_TProxyMode.Descriptor instead.
func (SocketConfig_TProxyMode) EnumDescriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{3, 1}
}

type TransportConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Type of network that this settings supports.
    // Deprecated. Use the string form below.
}
    // 定义一个名为 TransportProtocol 的协议类型变量，使用 protobuf 标签指定了序列化和 JSON 标签
    Protocol TransportProtocol `protobuf:"varint,1,opt,name=protocol,proto3,enum=v2ray.core.transport.internet.TransportProtocol" json:"protocol,omitempty"`
    // 表示该设置支持的网络类型
    ProtocolName string `protobuf:"bytes,3,opt,name=protocol_name,json=protocolName,proto3" json:"protocol_name,omitempty"`
    // 具体的设置，必须是 transports 的类型
    Settings *serial.TypedMessage `protobuf:"bytes,2,opt,name=settings,proto3" json:"settings,omitempty"`
// 重置 TransportConfig 对象
func (x *TransportConfig) Reset() {
    // 将 TransportConfig 对象重置为空对象
    *x = TransportConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型
        mi := &file_transport_internet_config_proto_msgTypes[0]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 TransportConfig 对象的字符串表示
func (x *TransportConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*TransportConfig) ProtoMessage() {}

// 返回 TransportConfig 对象的反射信息
func (x *TransportConfig) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_config_proto_msgTypes[0]
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

// 已弃用：使用 TransportConfig.ProtoReflect.Descriptor 替代
func (*TransportConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{0}
}

// 获取 TransportConfig 对象的协议
func (x *TransportConfig) GetProtocol() TransportProtocol {
    if x != nil {
        return x.Protocol
    }
    return TransportProtocol_TCP
}

// 获取 TransportConfig 对象的协议名称
func (x *TransportConfig) GetProtocolName() string {
    if x != nil {
        return x.ProtocolName
    }
    return ""
}

// 获取 TransportConfig 对象的设置
func (x *TransportConfig) GetSettings() *serial.TypedMessage {
    if x != nil {
        return x.Settings
    }
    return nil
}

// StreamConfig 结构体
type StreamConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 有效网络。已弃用。使用下面的字符串形式。
    //
    // 已弃用：不要使用。
    Protocol TransportProtocol `protobuf:"varint,1,opt,name=protocol,proto3,enum=v2ray.core.transport.internet.TransportProtocol" json:"protocol,omitempty"`
    // 有效网络
    ProtocolName      string             `protobuf:"bytes,5,opt,name=protocol_name,json=protocolName,proto3" json:"protocol_name,omitempty"`
}
    // 传输设置的数组，每个元素是一个传输配置对象的指针
    TransportSettings []*TransportConfig `protobuf:"bytes,2,rep,name=transport_settings,json=transportSettings,proto3" json:"transport_settings,omitempty"`
    // 安全类型，必须是设置协议的消息名称
    SecurityType string `protobuf:"bytes,3,opt,name=security_type,json=securityType,proto3" json:"security_type,omitempty"`
    // 传输安全的设置，目前唯一的选择是 TLS
    SecuritySettings []*serial.TypedMessage `protobuf:"bytes,4,rep,name=security_settings,json=securitySettings,proto3" json:"security_settings,omitempty"`
    // 套接字设置，是套接字配置对象的指针
    SocketSettings   *SocketConfig          `protobuf:"bytes,6,opt,name=socket_settings,json=socketSettings,proto3" json:"socket_settings,omitempty"`
// 重置 StreamConfig 对象，将其重置为空对象
func (x *StreamConfig) Reset() {
    *x = StreamConfig{}
    // 如果启用了不安全操作，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 StreamConfig 对象的字符串表示形式
func (x *StreamConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*StreamConfig) ProtoMessage() {}

// 返回 StreamConfig 对象的反射信息
func (x *StreamConfig) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_config_proto_msgTypes[1]
    // 如果启用了不安全操作并且对象不为空，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 StreamConfig.ProtoReflect.Descriptor 替代
func (*StreamConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{1}
}

// 已弃用：不要使用
func (x *StreamConfig) GetProtocol() TransportProtocol {
    if x != nil {
        return x.Protocol
    }
    return TransportProtocol_TCP
}

// 返回 StreamConfig 对象的协议名称
func (x *StreamConfig) GetProtocolName() string {
    if x != nil {
        return x.ProtocolName
    }
    return ""
}

// 返回 StreamConfig 对象的传输设置
func (x *StreamConfig) GetTransportSettings() []*TransportConfig {
    if x != nil {
        return x.TransportSettings
    }
    return nil
}

// 返回 StreamConfig 对象的安全类型
func (x *StreamConfig) GetSecurityType() string {
    if x != nil {
        return x.SecurityType
    }
    return ""
}

// 返回 StreamConfig 对象的安全设置
func (x *StreamConfig) GetSecuritySettings() []*serial.TypedMessage {
    if x != nil {
        return x.SecuritySettings
    }
    return nil
}

// 返回 StreamConfig 对象的套接字设置
func (x *StreamConfig) GetSocketSettings() *SocketConfig {
    if x != nil {
        return x.SocketSettings
    }
    return nil
}

// 定义 ProxyConfig 结构体
type ProxyConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}
    # 定义一个名为Tag的字符串字段
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
// 重置 ProxyConfig 对象
func (x *ProxyConfig) Reset() {
    // 将 ProxyConfig 对象重置为空对象
    *x = ProxyConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_transport_internet_config_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ProxyConfig 对象的字符串表示
func (x *ProxyConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ProxyConfig) ProtoMessage() {}

// 返回 ProxyConfig 对象的反射信息
func (x *ProxyConfig) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_config_proto_msgTypes[2]
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

// Deprecated: Use ProxyConfig.ProtoReflect.Descriptor instead.
// 返回 ProxyConfig 对象的描述符
func (*ProxyConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{2}
}

// 获取 ProxyConfig 对象的标签
func (x *ProxyConfig) GetTag() string {
    // 如果对象不为空，则返回标签值，否则返回空字符串
    if x != nil {
        return x.Tag
    }
    return ""
}

// SocketConfig 是应用于网络套接字的选项
type SocketConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 连接的标记。如果非零，则该值将设置为 SO_MARK。
    Mark int32 `protobuf:"varint,1,opt,name=mark,proto3" json:"mark,omitempty"`
    // TFO 是 TFO 设置的状态。
    Tfo SocketConfig_TCPFastOpenState `protobuf:"varint,2,opt,name=tfo,proto3,enum=v2ray.core.transport.internet.SocketConfig_TCPFastOpenState" json:"tfo,omitempty"`
    // TProxy 用于启用 TProxy 套接字选项。
    Tproxy SocketConfig_TProxyMode `protobuf:"varint,3,opt,name=tproxy,proto3,enum=v2ray.core.transport.internet.SocketConfig_TProxyMode" json:"tproxy,omitempty"`
    // ReceiveOriginalDestAddress 用于启用 IP_RECVORIGDSTADDR 套接字选项。此选项仅适用于 UDP。
}
    # 布尔类型字段，表示是否接收原始目标地址
    ReceiveOriginalDestAddress bool   `protobuf:"varint,4,opt,name=receive_original_dest_address,json=receiveOriginalDestAddress,proto3" json:"receive_original_dest_address,omitempty"`
    # 字节切片类型字段，用于存储绑定地址
    BindAddress                []byte `protobuf:"bytes,5,opt,name=bind_address,json=bindAddress,proto3" json:"bind_address,omitempty"`
    # 32 位无符号整数类型字段，用于存储绑定端口
    BindPort                   uint32 `protobuf:"varint,6,opt,name=bind_port,json=bindPort,proto3" json:"bind_port,omitempty"`
// 重置 SocketConfig 对象
func (x *SocketConfig) Reset() {
    // 将 SocketConfig 对象重置为空对象
    *x = SocketConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取 SocketConfig 对象的消息类型信息
        mi := &file_transport_internet_config_proto_msgTypes[3]
        // 获取 SocketConfig 对象的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SocketConfig 对象的字符串表示
func (x *SocketConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SocketConfig) ProtoMessage() {}

// 返回 SocketConfig 对象的反射信息
func (x *SocketConfig) ProtoReflect() protoreflect.Message {
    // 获取 SocketConfig 对象的消息类型信息
    mi := &file_transport_internet_config_proto_msgTypes[3]
    // 如果启用了不安全操作并且 SocketConfig 对象不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取 SocketConfig 对象的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 SocketConfig.ProtoReflect.Descriptor 替代
func (*SocketConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_config_proto_rawDescGZIP(), []int{3}
}

// 获取 SocketConfig 对象的 Mark 属性值
func (x *SocketConfig) GetMark() int32 {
    if x != nil {
        return x.Mark
    }
    return 0
}

// 获取 SocketConfig 对象的 Tfo 属性值
func (x *SocketConfig) GetTfo() SocketConfig_TCPFastOpenState {
    if x != nil {
        return x.Tfo
    }
    return SocketConfig_AsIs
}

// 获取 SocketConfig 对象的 Tproxy 属性值
func (x *SocketConfig) GetTproxy() SocketConfig_TProxyMode {
    if x != nil {
        return x.Tproxy
    }
    return SocketConfig_Off
}

// 获取 SocketConfig 对象的 ReceiveOriginalDestAddress 属性值
func (x *SocketConfig) GetReceiveOriginalDestAddress() bool {
    if x != nil {
        return x.ReceiveOriginalDestAddress
    }
    return false
}

// 获取 SocketConfig 对象的 BindAddress 属性值
func (x *SocketConfig) GetBindAddress() []byte {
    if x != nil {
        return x.BindAddress
    }
    return nil
}

// 获取 SocketConfig 对象的 BindPort 属性值
func (x *SocketConfig) GetBindPort() uint32 {
    if x != nil {
        return x.BindPort
    }
    return 0
}

// 定义 File_transport_internet_config_proto 变量
var File_transport_internet_config_proto protoreflect.FileDescriptor

// 定义 file_transport_internet_config_proto_rawDesc 变量
var file_transport_internet_config_proto_rawDesc = []byte{
    0x0a, 0x1f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
}
    # 以十六进制表示的字节码序列，可能是某种配置信息或者协议定义
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x12, 0x1d, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
    0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2f,
    0x74, 0x79, 0x70, 0x65, 0x64, 0x5f, 0x6d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x2e, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x22, 0xc8, 0x01, 0x0a, 0x0f, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72,
    0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x4c, 0x0a, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x63, 0x6f, 0x6c, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x30, 0x2e, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74,
    0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x52, 0x08, 0x70, 0x72, 0x6f,
    0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x23, 0x0a, 0x0d, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x5f, 0x6e, 0x61, 0x6d, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0c, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x4e, 0x61, 0x6d, 0x65, 0x12, 0x42, 0x0a, 0x08, 0x73, 0x65,
    0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e,
    0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73,
    0x73, 0x61, 0x67, 0x65, 0x52, 0x08, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x22, 0xb4,
    0x03, 0x0a, 0x0c, 0x53, 0x74, 0x72, 0x65, 0x61, 0x6d, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x50, 0x0a, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x01, 0x20, 0x01, 0x28,
    # 以十六进制表示的字节序列，可能是某种配置或数据的编码
    0x0e, 0x32, 0x30, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74,
    0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65,
    0x74, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x50, 0x72, 0x6f, 0x74, 0x6f,
    0x63, 0x6f, 0x6c, 0x42, 0x02, 0x18, 0x01, 0x52, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x12, 0x23, 0x0a, 0x0d, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x5f, 0x6e, 0x61,
    0x6d, 0x65, 0x18, 0x05, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0c, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63,
    0x6f, 0x6c, 0x4e, 0x61, 0x6d, 0x65, 0x12, 0x5d, 0x0a, 0x12, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x02, 0x20, 0x03,
    0x28, 0x0b, 0x32, 0x2e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e,
    0x65, 0x74, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x43, 0x6f, 0x6e, 0x66,
    0x69, 0x67, 0x52, 0x11, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x53, 0x65, 0x74,
    0x74, 0x69, 0x6e, 0x67, 0x73, 0x12, 0x23, 0x0a, 0x0d, 0x73, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
    0x79, 0x5f, 0x74, 0x79, 0x70, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0c, 0x73, 0x65,
    0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x54, 0x79, 0x70, 0x65, 0x12, 0x53, 0x0a, 0x11, 0x73, 0x65,
    0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18,
    0x04, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c,
    0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52, 0x10, 0x73,
    0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x12,
    # 定义一系列十六进制数，可能是某种配置或数据的表示方式
    0x54, 0x0a, 0x0f, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e,
    0x0x0a, 0x0f, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e,
    # ...
    # 这里是一系列十六进制数，可能是某种配置或数据的表示方式
    # 创建一个十六进制数组，表示一些特定的数值
    0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x5f, 0x6f, 0x72, 0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x5f,
    0x64, 0x65, 0x73, 0x74, 0x5f, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x18, 0x04, 0x20, 0x01,
    0x28, 0x08, 0x52, 0x1a, 0x72, 0x65, 0x63, 0x65, 0x69, 0x76, 0x65, 0x4f, 0x72, 0x69, 0x67, 0x69,
    0x6e, 0x61, 0x6c, 0x44, 0x65, 0x73, 0x74, 0x41, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x12, 0x21,
    0x0a, 0x0c, 0x62, 0x69, 0x6e, 0x64, 0x5f, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x18, 0x05,
    0x20, 0x01, 0x28, 0x0c, 0x52, 0x0b, 0x62, 0x69, 0x6e, 0x64, 0x41, 0x64, 0x64, 0x72, 0x65, 0x73,
    0x73, 0x12, 0x1b, 0x0a, 0x09, 0x62, 0x69, 0x6e, 0x64, 0x5f, 0x70, 0x6f, 0x72, 0x74, 0x18, 0x06,
    0x20, 0x01, 0x28, 0x0d, 0x52, 0x08, 0x62, 0x69, 0x6e, 0x64, 0x50, 0x6f, 0x72, 0x74, 0x22, 0x35,
    0x0a, 0x10, 0x54, 0x43, 0x50, 0x46, 0x61, 0x73, 0x74, 0x4f, 0x70, 0x65, 0x6e, 0x53, 0x74, 0x61,
    0x74, 0x65, 0x12, 0x08, 0x0a, 0x04, 0x41, 0x73, 0x49, 0x73, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06,
    0x45, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x10, 0x01, 0x12, 0x0b, 0x0a, 0x07, 0x44, 0x69, 0x73, 0x61,
    0x62, 0x6c, 0x65, 0x10, 0x02, 0x22, 0x2f, 0x0a, 0x0a, 0x54, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x4d,
    0x6f, 0x64, 0x65, 0x12, 0x07, 0x0a, 0x03, 0x4f, 0x66, 0x66, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06,
    0x54, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x10, 0x01, 0x12, 0x0c, 0x0a, 0x08, 0x52, 0x65, 0x64, 0x69,
    0x72, 0x65, 0x63, 0x74, 0x10, 0x02, 0x2a, 0x5a, 0x0a, 0x11, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x07, 0x0a, 0x03, 0x54,
    0x43, 0x50, 0x10, 0x00, 0x12, 0x07, 0x0a, 0x03, 0x55, 0x44, 0x50, 0x10, 0x01, 0x12, 0x08, 0x0a,
    0x04, 0x4d, 0x4b, 0x43, 0x50, 0x10, 0x02, 0x12, 0x0d, 0x0a, 0x09, 0x57, 0x65, 0x62, 0x53, 0x6f,
    0x63, 0x6b, 0x65, 0x74, 0x10, 0x03, 0x12, 0x08, 0x0a, 0x04, 0x48, 0x54, 0x54, 0x50, 0x10, 0x04,
    0x12, 0x10, 0x0a, 0x0c, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x53, 0x6f, 0x63, 0x6b, 0x65, 0x74,
    # 以十六进制表示的数据，可能是某种编码或者密文
    0x10, 0x05, 0x42, 0x68, 0x0a, 0x21, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69,
    0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x50, 0x01, 0x5a, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0xaa, 0x02, 0x1d, 0x56,
    0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x33,
# 定义变量 file_transport_internet_config_proto_rawDescOnce，用于确保 file_transport_internet_config_proto_rawDescData 只被初始化一次
var (
    file_transport_internet_config_proto_rawDescOnce sync.Once
    file_transport_internet_config_proto_rawDescData = file_transport_internet_config_proto_rawDesc
)

# 定义函数 file_transport_internet_config_proto_rawDescGZIP，用于获取经过 GZIP 压缩的原始描述数据
func file_transport_internet_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保原始描述数据只被压缩一次
    file_transport_internet_config_proto_rawDescOnce.Do(func() {
        # 调用 protoimpl.X.CompressGZIP 对原始描述数据进行 GZIP 压缩
        file_transport_internet_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_config_proto_rawDescData)
    })
    # 返回经过 GZIP 压缩的原始描述数据
    return file_transport_internet_config_proto_rawDescData
}

# 定义变量 file_transport_internet_config_proto_enumTypes，用于存储枚举类型信息
var file_transport_internet_config_proto_enumTypes = make([]protoimpl.EnumInfo, 3)
# 定义变量 file_transport_internet_config_proto_msgTypes，用于存储消息类型信息
var file_transport_internet_config_proto_msgTypes = make([]protoimpl.MessageInfo, 4)
# 定义变量 file_transport_internet_config_proto_goTypes，用于存储 Go 语言类型信息
var file_transport_internet_config_proto_goTypes = []interface{}{
    (TransportProtocol)(0),             // 0: v2ray.core.transport.internet.TransportProtocol
    (SocketConfig_TCPFastOpenState)(0), // 1: v2ray.core.transport.internet.SocketConfig.TCPFastOpenState
    (SocketConfig_TProxyMode)(0),       // 2: v2ray.core.transport.internet.SocketConfig.TProxyMode
    (*TransportConfig)(nil),            // 3: v2ray.core.transport.internet.TransportConfig
    (*StreamConfig)(nil),               // 4: v2ray.core.transport.internet.StreamConfig
    (*ProxyConfig)(nil),                // 5: v2ray.core.transport.internet.ProxyConfig
    (*SocketConfig)(nil),               // 6: v2ray.core.transport.internet.SocketConfig
    (*serial.TypedMessage)(nil),        // 7: v2ray.core.common.serial.TypedMessage
}
# 定义变量 file_transport_internet_config_proto_depIdxs，用于存储依赖索引信息
var file_transport_internet_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.transport.internet.TransportConfig.protocol:type_name -> v2ray.core.transport.internet.TransportProtocol
    7, // 1: v2ray.core.transport.internet.TransportConfig.settings:type_name -> v2ray.core.common.serial.TypedMessage
    0, // 2: v2ray.core.transport.internet.StreamConfig.protocol:type_name -> v2ray.core.transport.internet.TransportProtocol
    3, // 3: v2ray.core.transport.internet.StreamConfig.transport_settings:type_name -> v2ray.core.transport.internet.TransportConfig
    // 设置索引3对应的值，指定流配置的传输设置类型名称为v2ray.core.transport.internet.TransportConfig

    7, // 4: v2ray.core.transport.internet.StreamConfig.security_settings:type_name -> v2ray.core.common.serial.TypedMessage
    // 设置索引7对应的值，指定流配置的安全设置类型名称为v2ray.core.common.serial.TypedMessage

    6, // 5: v2ray.core.transport.internet.StreamConfig.socket_settings:type_name -> v2ray.core.transport.internet.SocketConfig
    // 设置索引6对应的值，指定流配置的套接字设置类型名称为v2ray.core.transport.internet.SocketConfig

    1, // 6: v2ray.core.transport.internet.SocketConfig.tfo:type_name -> v2ray.core.transport.internet.SocketConfig.TCPFastOpenState
    // 设置索引1对应的值，指定套接字配置的TCP快速打开类型名称为v2ray.core.transport.internet.SocketConfig.TCPFastOpenState

    2, // 7: v2ray.core.transport.internet.SocketConfig.tproxy:type_name -> v2ray.core.transport.internet.SocketConfig.TProxyMode
    // 设置索引2对应的值，指定套接字配置的透明代理类型名称为v2ray.core.transport.internet.SocketConfig.TProxyMode

    8, // [8:8] is the sub-list for method output_type
    // 设置索引8对应的值，用于方法输出类型

    8, // [8:8] is the sub-list for method input_type
    // 设置索引8对应的值，用于方法输入类型

    8, // [8:8] is the sub-list for extension type_name
    // 设置索引8对应的值，用于扩展类型名称

    8, // [8:8] is the sub-list for extension extendee
    // 设置索引8对应的值，用于扩展扩展对象

    0, // [0:8] is the sub-list for field type_name
    // 设置索引0对应的值，用于字段类型名称
func init() { file_transport_internet_config_proto_init() }
func file_transport_internet_config_proto_init() {
    // 如果 File_transport_internet_config_proto 不为空，则直接返回
    if File_transport_internet_config_proto != nil {
        return
    }
    // 如果 protoimpl.UnsafeEnabled 为假，则直接返回
    if !protoimpl.UnsafeEnabled {
        // 设置消息类型的导出函数
        file_transport_internet_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*TransportConfig); i {
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
        file_transport_internet_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*StreamConfig); i {
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
        file_transport_internet_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*ProxyConfig); i {
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
        file_transport_internet_config_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*SocketConfig); i {
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
            # 设置 RawDescriptor 属性为 file_transport_internet_config_proto_rawDesc
            RawDescriptor: file_transport_internet_config_proto_rawDesc,
            # 设置 NumEnums 属性为 3
            NumEnums:      3,
            # 设置 NumMessages 属性为 4
            NumMessages:   4,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为 file_transport_internet_config_proto_goTypes
        GoTypes:           file_transport_internet_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为 file_transport_internet_config_proto_depIdxs
        DependencyIndexes: file_transport_internet_config_proto_depIdxs,
        # 设置 EnumInfos 属性为 file_transport_internet_config_proto_enumTypes
        EnumInfos:         file_transport_internet_config_proto_enumTypes,
        # 设置 MessageInfos 属性为 file_transport_internet_config_proto_msgTypes
        MessageInfos:      file_transport_internet_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_config_proto
    File_transport_internet_config_proto = out.File
    # 将 file_transport_internet_config_proto_rawDesc 设置为 nil
    file_transport_internet_config_proto_rawDesc = nil
    # 将 file_transport_internet_config_proto_goTypes 设置为 nil
    file_transport_internet_config_proto_goTypes = nil
    # 将 file_transport_internet_config_proto_depIdxs 设置为 nil
    file_transport_internet_config_proto_depIdxs = nil
# 闭合前面的函数定义
```