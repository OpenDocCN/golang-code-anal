# `v2ray-core\app\policy\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: app/policy/config.proto

package policy

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保生成的代码足够更新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够更新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Second struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`  // 定义了一个 uint32 类型的字段 Value
}

func (x *Second) Reset() {
    *x = Second{}  // 重置 Second 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_app_policy_config_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Second) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Second) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Second) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Second.ProtoReflect.Descriptor instead.
func (*Second) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{0}  // 返回描述符
}

func (x *Second) GetValue() uint32 {
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的值
        return x.Value
    # 如果 x 为空
    # 返回 0
    return 0
// 定义 Policy 结构体，包含状态、大小缓存和未知字段
type Policy struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义 Timeout 字段，类型为 Policy_Timeout 结构体指针
    Timeout *Policy_Timeout `protobuf:"bytes,1,opt,name=timeout,proto3" json:"timeout,omitempty"`
    // 定义 Stats 字段，类型为 Policy_Stats 结构体指针
    Stats   *Policy_Stats   `protobuf:"bytes,2,opt,name=stats,proto3" json:"stats,omitempty"`
    // 定义 Buffer 字段，类型为 Policy_Buffer 结构体指针
    Buffer  *Policy_Buffer  `protobuf:"bytes,3,opt,name=buffer,proto3" json:"buffer,omitempty"`
}

// 重置 Policy 对象
func (x *Policy) Reset() {
    *x = Policy{}
    // 如果启用了不安全模式，存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_policy_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Policy 对象的字符串表示
func (x *Policy) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Policy) ProtoMessage() {}

// 返回 Policy 对象的反射信息
func (x *Policy) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Policy.ProtoReflect.Descriptor instead.
// 返回 Policy 对象的描述符
func (*Policy) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{1}
}

// 获取 Policy 对象的 Timeout 字段
func (x *Policy) GetTimeout() *Policy_Timeout {
    if x != nil {
        return x.Timeout
    }
    return nil
}

// 获取 Policy 对象的 Stats 字段
func (x *Policy) GetStats() *Policy_Stats {
    if x != nil {
        return x.Stats
    }
    return nil
}

// 获取 Policy 对象的 Buffer 字段
func (x *Policy) GetBuffer() *Policy_Buffer {
    if x != nil {
        return x.Buffer
    }
    return nil
}

// 定义 SystemPolicy 结构体，包含状态、大小缓存和未知字段
type SystemPolicy struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义 Stats 字段，类型为 SystemPolicy_Stats 结构体指针
    Stats *SystemPolicy_Stats `protobuf:"bytes,1,opt,name=stats,proto3" json:"stats,omitempty"`
}

// 重置 SystemPolicy 对象
func (x *SystemPolicy) Reset() {
    *x = SystemPolicy{}
}
    # 如果启用了不安全操作，执行以下操作
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_app_policy_config_proto_msgTypes[2]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 将 SystemPolicy 结构体转换为字符串
func (x *SystemPolicy) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*SystemPolicy) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *SystemPolicy) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use SystemPolicy.ProtoReflect.Descriptor instead.
func (*SystemPolicy) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{2}
}

// 获取 SystemPolicy 结构体中的 Stats 字段
func (x *SystemPolicy) GetStats() *SystemPolicy_Stats {
    if x != nil {
        return x.Stats
    }
    return nil
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Level 字段为 map 类型，存储 uint32 到 Policy 结构体的映射
    Level  map[uint32]*Policy `protobuf:"bytes,1,rep,name=level,proto3" json:"level,omitempty" protobuf_key:"varint,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
    // System 字段为 SystemPolicy 结构体类型
    System *SystemPolicy      `protobuf:"bytes,2,opt,name=system,proto3" json:"system,omitempty"`
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_policy_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 Config 结构体转换为字符串
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*Config) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[3]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 返回配置文件的原始描述和索引为3的元素
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{3}
}

// 返回级别与策略的映射
func (x *Config) GetLevel() map[uint32]*Policy {
    if x != nil {
        return x.Level
    }
    return nil
}

// 返回系统策略
func (x *Config) GetSystem() *SystemPolicy {
    if x != nil {
        return x.System
    }
    return nil
}

// Timeout 是各个阶段超时设置的消息，单位为秒
type Policy_Timeout struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Handshake      *Second `protobuf:"bytes,1,opt,name=handshake,proto3" json:"handshake,omitempty"`
    ConnectionIdle *Second `protobuf:"bytes,2,opt,name=connection_idle,json=connectionIdle,proto3" json:"connection_idle,omitempty"`
    UplinkOnly     *Second `protobuf:"bytes,3,opt,name=uplink_only,json=uplinkOnly,proto3" json:"uplink_only,omitempty"`
    DownlinkOnly   *Second `protobuf:"bytes,4,opt,name=downlink_only,json=downlinkOnly,proto3" json:"downlink_only,omitempty"`
}

// 重置 Policy_Timeout 对象
func (x *Policy_Timeout) Reset() {
    *x = Policy_Timeout{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_policy_config_proto_msgTypes[4]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Policy_Timeout 对象的字符串表示
func (x *Policy_Timeout) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*Policy_Timeout) ProtoMessage() {}

// 返回 Policy_Timeout 对象的反射信息
func (x *Policy_Timeout) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[4]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Policy_Timeout.ProtoReflect.Descriptor instead.
// 返回 Policy_Timeout 对象的原始描述和索引为1, 0的元素
func (*Policy_Timeout) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{1, 0}
}
# 获取 Policy_Timeout 结构体中的 Handshake 字段
func (x *Policy_Timeout) GetHandshake() *Second {
    # 如果 x 不为空，则返回 Handshake 字段的数值
    if x != nil {
        return x.Handshake
    }
    # 如果 x 为空，则返回空值
    return nil
}

# 获取 Policy_Timeout 结构体中的 ConnectionIdle 字段
func (x *Policy_Timeout) GetConnectionIdle() *Second {
    # 如果 x 不为空，则返回 ConnectionIdle 字段的数值
    if x != nil {
        return x.ConnectionIdle
    }
    # 如果 x 为空，则返回空值
    return nil
}

# 获取 Policy_Timeout 结构体中的 UplinkOnly 字段
func (x *Policy_Timeout) GetUplinkOnly() *Second {
    # 如果 x 不为空，则返回 UplinkOnly 字段的数值
    if x != nil {
        return x.UplinkOnly
    }
    # 如果 x 为空，则返回空值
    return nil
}

# 获取 Policy_Timeout 结构体中的 DownlinkOnly 字段
func (x *Policy_Timeout) GetDownlinkOnly() *Second {
    # 如果 x 不为空，则返回 DownlinkOnly 字段的数值
    if x != nil {
        return x.DownlinkOnly
    }
    # 如果 x 为空，则返回空值
    return nil
}

# 定义 Policy_Stats 结构体
type Policy_Stats struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    # 定义 UserUplink 字段
    UserUplink   bool `protobuf:"varint,1,opt,name=user_uplink,json=userUplink,proto3" json:"user_uplink,omitempty"`
    # 定义 UserDownlink 字段
    UserDownlink bool `protobuf:"varint,2,opt,name=user_downlink,json=userDownlink,proto3" json:"user_downlink,omitempty"`
}

# 重置 Policy_Stats 结构体
func (x *Policy_Stats) Reset() {
    *x = Policy_Stats{}
    # 如果启用了不安全操作，则重置消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_app_policy_config_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

# 返回 Policy_Stats 结构体的字符串表示形式
func (x *Policy_Stats) String() string {
    return protoimpl.X.MessageStringOf(x)
}

# 定义 Policy_Stats 结构体的 ProtoMessage 方法
func (*Policy_Stats) ProtoMessage() {}

# 返回 Policy_Stats 结构体的 ProtoReflect 方法
func (x *Policy_Stats) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[5]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

# 弃用方法：使用 Policy_Stats.ProtoReflect.Descriptor 替代
func (*Policy_Stats) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{1, 1}
}

# 获取 Policy_Stats 结构体中的 UserUplink 字段
func (x *Policy_Stats) GetUserUplink() bool {
    # 如果 x 不为空，则返回 UserUplink 字段的数值
    if x != nil {
        return x.UserUplink
    }
    # 如果 x 为空，则返回 false
    return false
}

# 获取 Policy_Stats 结构体中的 UserDownlink 字段
func (x *Policy_Stats) GetUserDownlink() bool {
    # 如果 x 不为空，则返回 UserDownlink 字段的数值
    if x != nil {
        return x.UserDownlink
    }
    # 如果 x 为空，则返回 false
    return false
}
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的 UserDownlink 属性
        return x.UserDownlink
    # 如果 x 为空，返回 false
    return false
}
// 定义 Policy_Buffer 结构体
type Policy_Buffer struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 每个连接的缓冲区大小，以字节为单位。-1 表示无限制缓冲区。
    Connection int32 `protobuf:"varint,1,opt,name=connection,proto3" json:"connection,omitempty"`
}

// 重置 Policy_Buffer 对象
func (x *Policy_Buffer) Reset() {
    *x = Policy_Buffer{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_policy_config_proto_msgTypes[6]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Policy_Buffer 对象的字符串表示
func (x *Policy_Buffer) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Policy_Buffer) ProtoMessage() {}

// 返回 Policy_Buffer 对象的反射信息
func (x *Policy_Buffer) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[6]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Policy_Buffer.ProtoReflect.Descriptor instead.
// 返回 Policy_Buffer 对象的描述符
func (*Policy_Buffer) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{1, 2}
}

// 获取 Policy_Buffer 对象的 Connection 字段值
func (x *Policy_Buffer) GetConnection() int32 {
    if x != nil {
        return x.Connection
    }
    return 0
}

// 定义 SystemPolicy_Stats 结构体
type SystemPolicy_Stats struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    InboundUplink    bool `protobuf:"varint,1,opt,name=inbound_uplink,json=inboundUplink,proto3" json:"inbound_uplink,omitempty"`
    InboundDownlink  bool `protobuf:"varint,2,opt,name=inbound_downlink,json=inboundDownlink,proto3" json:"inbound_downlink,omitempty"`
    OutboundUplink   bool `protobuf:"varint,3,opt,name=outbound_uplink,json=outboundUplink,proto3" json:"outbound_uplink,omitempty"`
    # 定义一个布尔类型的字段，用于表示出站下行链接的状态
    OutboundDownlink bool `protobuf:"varint,4,opt,name=outbound_downlink,json=outboundDownlink,proto3" json:"outbound_downlink,omitempty"`
// 重置 SystemPolicy_Stats 对象
func (x *SystemPolicy_Stats) Reset() {
    // 将 SystemPolicy_Stats 对象重置为空对象
    *x = SystemPolicy_Stats{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_policy_config_proto_msgTypes[7]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SystemPolicy_Stats 对象的字符串表示
func (x *SystemPolicy_Stats) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SystemPolicy_Stats) ProtoMessage() {}

// 返回 SystemPolicy_Stats 对象的反射信息
func (x *SystemPolicy_Stats) ProtoReflect() protoreflect.Message {
    mi := &file_app_policy_config_proto_msgTypes[7]
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

// 已弃用：使用 SystemPolicy_Stats.ProtoReflect.Descriptor 替代
func (*SystemPolicy_Stats) Descriptor() ([]byte, []int) {
    return file_app_policy_config_proto_rawDescGZIP(), []int{2, 0}
}

// 获取 InboundUplink 字段的值
func (x *SystemPolicy_Stats) GetInboundUplink() bool {
    if x != nil {
        return x.InboundUplink
    }
    return false
}

// 获取 InboundDownlink 字段的值
func (x *SystemPolicy_Stats) GetInboundDownlink() bool {
    if x != nil {
        return x.InboundDownlink
    }
    return false
}

// 获取 OutboundUplink 字段的值
func (x *SystemPolicy_Stats) GetOutboundUplink() bool {
    if x != nil {
        return x.OutboundUplink
    }
    return false
}

// 获取 OutboundDownlink 字段的值
func (x *SystemPolicy_Stats) GetOutboundDownlink() bool {
    if x != nil {
        return x.OutboundDownlink
    }
    return false
}

// 定义 File_app_policy_config_proto 变量
var File_app_policy_config_proto protoreflect.FileDescriptor

// 定义 file_app_policy_config_proto_rawDesc 变量
var file_app_policy_config_proto_rawDesc = []byte{
    0x0a, 0x17, 0x61, 0x70, 0x70, 0x2f, 0x70, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x2f, 0x63, 0x6f, 0x6e,
    0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 6c, 0x69, 0x63, 0x79, 0x2f, 0x63, 0x6f, 6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 6c, 0x72, 0x6f, 0x74, 0x6f,
}
    # 创建一个十六进制数值为 0x22 的整数
    0x22, 
    # 创建一个十六进制数值为 0x1e 的整数
    0x1e, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x06 的整数
    0x06, 
    # 创建一个十六进制数值为 0x53 的整数
    0x53, 
    # 创建一个十六进制数值为 0x65 的整数
    0x65, 
    # 创建一个十六进制数值为 0x63 的整数
    0x63, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x64 的整数
    0x64, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x14 的整数
    0x14, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x05 的整数
    0x05, 
    # 创建一个十六进制数值为 0x76 的整数
    0x76,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x75 的整数
    0x75,
    # 创建一个十六进制数值为 0x65 的整数
    0x65,
    # 创建一个十六进制数值为 0x18 的整数
    0x18,
    # 创建一个十六进制数值为 0x01 的整数
    0x01,
    # 创建一个十六进制数值为 0x20 的整数
    0x20,
    # 创建一个十六进制数值为 0x01 的整数
    0x01,
    # 创建一个十六进制数值为 0x28 的整数
    0x28,
    # 创建一个十六进制数值为 0x0d 的整数
    0x0d,
    # 创建一个十六进制数值为 0x52 的整数
    0x52,
    # 创建一个十六进制数值为 0x05 的整数
    0x05,
    # 创建一个十六进制数值为 0x76 的整数
    0x76,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x75 的整数
    0x75,
    # 创建一个十六进制数值为 0x65 的整数
    0x65,
    # 创建一个十六进制数值为 0x22 的整数
    0x22,
    # 创建一个十六进制数值为 0xd0 的整数
    0xd0,
    # 创建一个十六进制数值为 0x04 的整数
    0x04,
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a,
    # 创建一个十六进制数值为 0x06 的整数
    0x06,
    # 创建一个十六进制数值为 0x50 的整数
    0x50,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x12 的整数
    0x12,
    # 创建一个十六进制数值为 0x3f 的整数
    0x3f,
    # ���建一个十六进制数值为 0x0a 的整数
    0x0a,
    # 创建一个十六进制数值为 0x07 的整数
    0x07,
    # 创建一个十六进制数值为 0x74 的整数
    0x74,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x6d 的整数
    0x6d,
    # 创建一个十六进制数值为 0x65 的整数
    0x65,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x75 的整数
    0x75,
    # 创建一个十六进制数值为 0x74 的整数
    0x74,
    # 创建一个十六进制数值为 0x18 的整数
    0x18,
    # 创建一个十六进制数值为 0x01 的整数
    0x01,
    # 创建一个十六进制数值为 0x20 的整数
    0x20,
    # 创建一个十六进制数值为 0x01 的整数
    0x01,
    # 创建一个十六进制数值为 0x28 的整数
    0x28,
    # 创建一个十六进制数值为 0x0b 的整数
    0x0b,
    # 创建一个十六进制数值为 0x32 的整数
    0x32,
    # 创建一个十六进制数值为 0x25 的整数
    0x25,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x76 的整数
    0x76,
    # 创建一个十六进制数值为 0x32 的整数
    0x32,
    # 创建一个十六进制数值为 0x72 的整数
    0x72,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x72 的整数
    0x72,
    # 创建一个十六进制数值为 0x65 的整数
    0x65,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x50 的整数
    0x50,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x12 的整数
    0x12,
    # 创建一个十六进制数值为 0x39 的整数
    0x39,
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a,
    # 创建一个十六进制数值为 0x05 的整数
    0x05,
    # 创建一个十六进制数值为 0x73 的整数
    0x73,
    # 创建一个十六进制数值为 0x74 的整数
    0x74,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x74 的整数
    0x74,
    # 创建一个十六进制数值为 0x73 的整数
    0x73,
    # 创建一个十六进制数值为 0x18 的整数
    0x18,
    # 创建一个十六进制数值为 0x02 的整数
    0x02,
    # 创建一个十六进制数值为 0x20 的整数
    0x20,
    # 创建一个十六进制数值为 0x01 的整数
    0x01,
    # 创建一个十六进制数值为 0x28 的整数
    0x28,
    # 创建一个十六进制数值为 0x23 的整数
    0x23,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x76 的整数
    0x76,
    # 创建一个十六进制数值为 0x32 的整数
    0x32,
    # 创建一个十六进制数值为 0x72 的整数
    0x72,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制��值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x72 的整数
    0x72,
    # 创建一个十六进制数值为 0x65 的整数
    0x65,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x50 的整数
    0x50,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x12 的整数
    0x12,
    # 创建一个十六进制数值为 0x39 的整数
    0x39,
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a,
    # 创建一个十六进制数值为 0x05 的整数
    0x05,
    # 创建一个十六进制数值为 0x73 的整数
    0x73,
    # 创建一个十六进制数值为 0x74 的整数
    0x74,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x74 的整数
    0x74,
    # 创建一个十六进制数值为 0x73 的整数
    0x73,
    # 创建一个十六进制数值为 0x18 的整数
    0x18,
    # 创建一个十六进制数值为 0x02 的整数
    0x02,
    # 创建一个十六进制数值为 0x20 的整数
    0x20,
    # 创建一个十六进制数值为 0x01 的整数
    0x01,
    # 创建一个十六进制数值为 0x28 的整数
    0x28,
    # 创建一个十六进制数值为 0x24 的整数
    0x24,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x76 的整数
    0x76,
    # 创建一个十六进制数值为 0x32 的整数
    0x32,
    # 创建一个十六进制数值为 0x72 的整数
    0x72,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x72 的整数
    0x72,
    # 创建一个十六进制数值为 0x65 的整数
    0x65,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数值为 0x70 的整数
    0x70,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f,
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c,
    # 创建一个十六进制数值为 0x69 的整数
    0x69,
    # 创建一个十六进制数值为 0x63 的整数
    0x63,
    # 创建一个十六进制数值为 0x79 的整数
    0x79,
    # 创建一个十六进制数值为 0x2e 的整数
    0x2e,
    # 创建一个十六进制数
    # 以十六进制表示的整数列表，可能是某种数据或配置信息
    0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x2e,
    0x53, 0x65, 0x63, 0x6f, 0x6e, 0x64, 0x52, 0x0e, 0x63, 0x6f, 0x6e, 0x6e, 0x65, 0x63, 0x74, 0x69,
    0x6f, 0x6e, 0x49, 0x64, 0x6c, 0x65,
    
    # 以十六进制表示的整数列表，可能是某种数据或配置信息
    0x12, 0x3e, 0x0a, 0x0b, 0x75, 0x70, 0x6c, 0x69, 0x6e, 0x6b, 0x5f, 0x6f, 0x6e, 0x6c, 0x79, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1d, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x6f, 0x6c,
    0x69, 0x63, 0x79, 0x2e, 0x53, 0x65, 0x63, 0x6f, 0x6e, 0x64, 0x52, 0x0a, 0x75, 0x70, 0x6c, 0x69,
    0x6e, 0x6b, 0x4f, 0x6e, 0x6c, 0x79,
    
    # 以十六进制表示的整数列表，可能是某种数据或配置信息
    0x12, 0x42, 0x0a, 0x0d, 0x64, 0x6f, 0x77, 0x6e, 0x6c, 0x69, 0x6e, 0x6b, 0x5f, 0x6f, 0x6e, 0x6c, 0x79, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1d, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70,
    0x6f, 0x6c, 0x69, 0x63, 0x79, 0x2e, 0x53, 0x65, 0x63, 0x6f, 0x6e, 0x64, 0x52, 0x0c, 0x64, 0x6f,
    0x77, 0x6e, 0x6c, 0x69, 0x6e, 0x6b, 0x4f, 0x6e, 0x6c, 0x79,
    
    # 以十六进制表示的整数列表，可能是某种数据或配置信息
    0x1a, 0x4d, 0x0a, 0x05, 0x53, 0x74, 0x61, 0x74, 0x73, 0x12, 0x1f, 0x0a, 0x0b, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x75, 0x70, 0x6c, 0x69, 0x6e, 0x6b, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x0a, 0x75, 0x73, 0x65, 0x72, 0x55, 0x70, 0x6c, 0x69, 0x6e, 0x6b, 0x12, 0x23, 0x0a, 0x0d, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x64, 0x6f, 0x77, 0x6e, 0x6c, 0x69, 0x6e, 0x6b, 0x18, 0x02, 0x20, 0x01, 0x28, 0x08, 0x52, 0x0c, 0x75, 0x73, 0x65, 0x72, 0x44, 0x6f, 0x77, 0x6e, 0x6c, 0x69, 0x6e, 0x6b, 0x1a, 0x28, 0x0a, 0x06, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x12, 0x1e, 0x0a, 0x0a, 0x63, 0x6f, 0x6e, 0x6e, 0x65, 0x63, 0x74, 0x69, 0x6f, 0x6e, 0x18, 0x01, 0x20, 0x01, 0x28, 0x05, 0x52, 0x0a, 0x63, 0x6f, 0x6e, 0x6e, 0x65, 0x63, 0x74, 0x69, 0x6f, 0x6e, 0x22, 0x81, 0x02, 0x0a, 0x0c, 0x53, 0x79, 0x73, 0x74, 0x65, 0x6d, 0x50, 0x6f,
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x63
    0x63, 
    # 创建一个十六进制整数 0x79
    0x79, 
    # 创建一个十六进制整数 0x12
    0x12, 
    # 创建一个十六进制整数 0x3f
    0x3f, 
    # 创建一个十六进制整数 0x0a
    0x0a, 
    # 创建一个十六进制整数 0x05
    0x05, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x61
    0x61, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x18
    0x18, 
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x20
    0x20,
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x28
    0x28, 
    # 创建一个十六进制整数 0x0b
    0x0b, 
    # 创建一个十六进制整数 0x32
    0x32, 
    # 创建一个十六进制整数 0x29
    0x29, 
    # 创建一个十六进制整数 0x2e
    0x2e, 
    # 创建一个十六进制整数 0x76
    0x76, 
    # 创建一个十六进制整数 0x32
    0x32, 
    # 创建一个十六进制整数 0x72
    0x72, 
    # 创建一个十六进制整数 0x61
    0x61, 
    # 创建一个十六进制整数 0x79
    0x79, 
    # 创建一个十六进制整数 0x2e
    0x2e, 
    # 创建一个十六进制整数 0x63
    0x63, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x72
    0x72, 
    # 创建一个十六进制整数 0x65
    0x65,
    # 创建一个十六进制整数 0x2e
    0x2e, 
    # 创建一个十六进制整数 0x61
    0x61, 
    # 创建一个十六进制整数 0x70
    0x70, 
    # 创建一个十六进制整数 0x70
    0x70, 
    # 创建一个十六进制整数 0x2e
    0x2e, 
    # 创建一个十六进制整数 0x70
    0x70, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x63
    0x63, 
    # 创建一个十六进制整数 0x79
    0x79, 
    # 创建一个十六进制整数 0x2e
    0x2e, 
    # 创建一个十六进制整数 0x53
    0x53, 
    # 创建一个十六进制整数 0x79
    0x79, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x65
    0x65, 
    # 创建一个十六进制整数 0x6d
    0x6d, 
    # 创建一个十六进制整数 0x50
    0x50, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x63
    0x63, 
    # 创建一个十六进制整数 0x79
    0x79, 
    # 创建一个十六进制整数 0x2e
    0x2e, 
    # 创建一个十六进制整数 0x53
    0x53, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x61
    0x61, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x52
    0x52, 
    # 创建一个十六进制整数 0x05
    0x05, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x61
    0x61, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x1a
    0x1a, 
    # 创建一个十六进制整数 0xaf
    0xaf, 
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x0a
    0x0a, 
    # 创建一个十六进制整数 0x05
    0x05, 
    # 创建一个十六进制整数 0x53
    0x53, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x61
    0x61, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x73
    0x73, 
    # 创建一个十六进制整数 0x12
    0x12, 
    # 创建一个十六进制整数 0x25
    0x25, 
    # 创建一个十六进制整数 0x0a
    0x0a, 
    # 创建一个十六进制整数 0x0e
    0x0e, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x62
    0x62, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x5f
    0x5f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x70
    0x70, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6b
    0x6b, 
    # 创建一个十六进制整数 0x18
    0x18, 
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x20
    0x20, 
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x28
    0x28, 
    # 创建一个十六进制整数 0x08
    0x08, 
    # 创建一个十六进制整数 0x52
    0x52, 
    # 创建一个十六进制整数 0x0d
    0x0d, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x62
    0x62, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x55
    0x55, 
    # 创建一个十六进制整数 0x70
    0x70, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6b
    0x6b, 
    # 创建一个十六进制整数 0x12
    0x12, 
    # 创建一个十六进制整数 0x29
    0x29, 
    # 创建一个十六进制整数 0x0a
    0x0a, 
    # 创建一个十六进制整数 0x10
    0x10, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x62
    0x62, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x5f
    0x5f, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x77
    0x77, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6b
    0x6b, 
    # 创建一个十六进制整数 0x18
    0x18, 
    # 创建一个十六进制整数 0x02
    0x02, 
    # 创建一个十六进制整数 0x20
    0x20, 
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x28
    0x28, 
    # 创建一个十六进制整数 0x08
    0x08, 
    # 创建一个十六进制整数 0x52
    0x52, 
    # 创建一个十六进制整数 0x0f
    0x0f, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x62
    0x62, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x44
    0x44, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x77
    0x77, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6b
    0x6b, 
    # 创建一个十六进制整数 0x12
    0x12, 
    # 创建一个十六进制整数 0x27
    0x27, 
    # 创建一个十六进制整数 0x0a
    0x0a, 
    # 创建一个十六进制整数 0x0f
    0x0f, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x62
    0x62, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x5f
    0x5f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x70
    0x70, 
    # 创建一个十六进制整数 0x6c
    0x6c, 
    # 创建一个十六进制整数 0x69
    0x69, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x6b
    0x6b, 
    # 创建一个十六进制整数 0x18
    0x18, 
    # 创建一个十六进制整数 0x03
    0x03, 
    # 创建一个十六进制整数 0x20
    0x20, 
    # 创建一个十六进制整数 0x01
    0x01, 
    # 创建一个十六进制整数 0x28
    0x28, 
    # 创建一个十六进制整数 0x08
    0x08, 
    # 创建一个十六进制整数 0x52
    0x52, 
    # 创建一个十六进制整数 0x0e
    0x0e, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x74
    0x74, 
    # 创建一个十六进制整数 0x62
    0x62, 
    # 创建一个十六进制整数 0x6f
    0x6f, 
    # 创建一个十六进制整数 0x75
    0x75, 
    # 创建一个十六进制整数 0x6e
    0x6e, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十六进制整数 0x5f
    0x5f, 
    # 创建一个十六进制整数 0x64
    0x64, 
    # 创建一个十
    # 以十六进制表示的整数列表，可能是某种配置或数据的编码
    0x65, 0x6c, 0x12, 0x3b, 0x0a, 0x06, 0x73, 0x79, 0x73, 0x74, 0x65, 0x6d, 0x18, 0x02, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x23, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x61, 0x70, 0x70, 0x2e, 0x70, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x2e, 0x53, 0x79, 0x73, 0x74, 0x65,
    0x6d, 0x50, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x52, 0x06, 0x73, 0x79, 0x73, 0x74, 0x65, 0x6d, 0x1a,
    0x57, 0x0a, 0x0a, 0x4c, 0x65, 0x76, 0x65, 0x6c, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x12, 0x10, 0x0a,
    0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12,
    0x33, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1d,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e,
    0x70, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x2e, 0x50, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x52, 0x05, 0x76,
    0x61, 0x6c, 0x75, 0x65, 0x3a, 0x02, 0x38, 0x01, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70,
    0x6f, 0x6c, 0x69, 0x63, 0x79, 0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x70, 0x6f, 0x6c, 0x69,
    0x63, 0x79, 0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
    0x41, 0x70, 0x70, 0x2e, 0x50, 0x6f, 0x6c, 0x69, 0x63, 0x79, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x33,
// 定义变量 file_app_policy_config_proto_rawDescOnce，类型为 sync.Once，用于确保 file_app_policy_config_proto_rawDescData 只被初始化一次
var (
    file_app_policy_config_proto_rawDescOnce sync.Once
    file_app_policy_config_proto_rawDescData = file_app_policy_config_proto_rawDesc
)

// 定义函数 file_app_policy_config_proto_rawDescGZIP，返回类型为 []byte，用于对 file_app_policy_config_proto_rawDescData 进行 GZIP 压缩
func file_app_policy_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_app_policy_config_proto_rawDescOnce.Do(func() {
        file_app_policy_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_policy_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的数据
    return file_app_policy_config_proto_rawDescData
}

// 定义变量 file_app_policy_config_proto_msgTypes，类型为 []protoimpl.MessageInfo，用于存储消息类型信息
var file_app_policy_config_proto_msgTypes = make([]protoimpl.MessageInfo, 9)
// 定义变量 file_app_policy_config_proto_goTypes，类型为 []interface{}，用于存储 Go 类型信息
var file_app_policy_config_proto_goTypes = []interface{}{
    (*Second)(nil),             // 0: v2ray.core.app.policy.Second
    (*Policy)(nil),             // 1: v2ray.core.app.policy.Policy
    (*SystemPolicy)(nil),       // 2: v2ray.core.app.policy.SystemPolicy
    (*Config)(nil),             // 3: v2ray.core.app.policy.Config
    (*Policy_Timeout)(nil),     // 4: v2ray.core.app.policy.Policy.Timeout
    (*Policy_Stats)(nil),       // 5: v2ray.core.app.policy.Policy.Stats
    (*Policy_Buffer)(nil),      // 6: v2ray.core.app.policy.Policy.Buffer
    (*SystemPolicy_Stats)(nil), // 7: v2ray.core.app.policy.SystemPolicy.Stats
    nil,                        // 8: v2ray.core.app.policy.Config.LevelEntry
}
// 定义变量 file_app_policy_config_proto_depIdxs，类型为 []int32，用于存储消息类型之间的依赖关系
var file_app_policy_config_proto_depIdxs = []int32{
    4,  // 0: v2ray.core.app.policy.Policy.timeout:type_name -> v2ray.core.app.policy.Policy.Timeout
    5,  // 1: v2ray.core.app.policy.Policy.stats:type_name -> v2ray.core.app.policy.Policy.Stats
    6,  // 2: v2ray.core.app.policy.Policy.buffer:type_name -> v2ray.core.app.policy.Policy.Buffer
    7,  // 3: v2ray.core.app.policy.SystemPolicy.stats:type_name -> v2ray.core.app.policy.SystemPolicy.Stats
    8,  // 4: v2ray.core.app.policy.Config.level:type_name -> v2ray.core.app.policy.Config.LevelEntry
    2,  // 5: v2ray.core.app.policy.Config.system:type_name -> v2ray.core.app.policy.SystemPolicy
    0,  // 6: v2ray.core.app.policy.Policy.Timeout.handshake:type_name -> v2ray.core.app.policy.Second
    0,  // 7: v2ray.core.app.policy.Policy.Timeout.connection_idle:type_name -> v2ray.core.app.policy.Second
    // 设置连接空闲超时时间为0，类型为v2ray.core.app.policy.Second

    0,  // 8: v2ray.core.app.policy.Policy.Timeout.uplink_only:type_name -> v2ray.core.app.policy.Second
    // 设置仅上行连接超时时间为0，类型为v2ray.core.app.policy.Second

    0,  // 9: v2ray.core.app.policy.Policy.Timeout.downlink_only:type_name -> v2ray.core.app.policy.Second
    // 设置仅下行连接超时时间为0，类型为v2ray.core.app.policy.Second

    1,  // 10: v2ray.core.app.policy.Config.LevelEntry.value:type_name -> v2ray.core.app.policy.Policy
    // 设置配置级别条目的值为1，类型为v2ray.core.app.policy.Policy

    11, // [11:11] is the sub-list for method output_type
    // 输出类型的子列表

    11, // [11:11] is the sub-list for method input_type
    // 输入类型的子列表

    11, // [11:11] is the sub-list for extension type_name
    // 扩展类型的子列表

    11, // [11:11] is the sub-list for extension extendee
    // 扩展的扩展类型的子列表

    0,  // [0:11] is the sub-list for field type_name
    // 字段类型的子列表
# 初始化函数，用于在包被导入时执行一些初始化操作
func init() { file_app_policy_config_proto_init() }

# 文件初始化函数，用于初始化文件的一些信息
func file_app_policy_config_proto_init() {
    # 如果文件已经被初始化过，则直接返回
    if File_app_policy_config_proto != nil {
        return
    }
    
    # 定义一个空结构体 x
    type x struct{}
    
    # 使用 protoimpl.TypeBuilder 构建类型信息
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_app_policy_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   9,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_app_policy_config_proto_goTypes,
        DependencyIndexes: file_app_policy_config_proto_depIdxs,
        MessageInfos:      file_app_policy_config_proto_msgTypes,
    }.Build()
    
    # 将构建好的类型信息赋值给 File_app_policy_config_proto
    File_app_policy_config_proto = out.File
    file_app_policy_config_proto_rawDesc = nil
    file_app_policy_config_proto_goTypes = nil
    file_app_policy_config_proto_depIdxs = nil
}
```