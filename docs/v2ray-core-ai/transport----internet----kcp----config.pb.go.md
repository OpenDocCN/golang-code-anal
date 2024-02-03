# `v2ray-core\transport\internet\kcp\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/kcp/config.proto

package kcp

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

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// 最大传输单元，以字节为单位。
type MTU struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`  // MTU 值
}

func (x *MTU) Reset() {
    *x = MTU{}  // 重置 MTU 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *MTU) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 MTU 对象的字符串表示
}

func (*MTU) ProtoMessage() {}

func (x *MTU) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_kcp_config_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 MTU.ProtoReflect.Descriptor 代替。
func (*MTU) Descriptor() ([]byte, []int) {
    # 返回 file_transport_internet_kcp_config_proto_rawDescGZIP() 函数的结果和空的整型切片
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{0}
// 获取 MTU 结构体的值
func (x *MTU) GetValue() uint32 {
    // 如果 x 不为空，则返回其值，否则返回 0
    if x != nil {
        return x.Value
    }
    return 0
}

// 传输时间间隔，单位为毫秒
type TTI struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`
}

// 重置 TTI 结构体
func (x *TTI) Reset() {
    *x = TTI{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 TTI 结构体的字符串表示
func (x *TTI) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*TTI) ProtoMessage() {}

// 返回 TTI 结构体的反射信息
func (x *TTI) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_kcp_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use TTI.ProtoReflect.Descriptor instead.
// 返回 TTI 结构体的描述符
func (*TTI) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{1}
}

// 获取 UplinkCapacity 结构体的值
func (x *UplinkCapacity) GetValue() uint32 {
    // 如果 x 不为空，则返回其值，否则返回 0
    if x != nil {
        return x.Value
    }
    return 0
}

// 上行容量，单位为 MB
type UplinkCapacity struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`
}

// 重置 UplinkCapacity 结构体
func (x *UplinkCapacity) Reset() {
    *x = UplinkCapacity{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 UplinkCapacity 结构体的字符串表示
func (x *UplinkCapacity) String() string {
    return protoimpl.X.MessageStringOf(x)
}
# 实现 ProtoMessage 方法
func (*UplinkCapacity) ProtoMessage() {}

# 实现 ProtoReflect 方法
func (x *UplinkCapacity) ProtoReflect() protoreflect.Message {
    # 获取消息类型信息
    mi := &file_transport_internet_kcp_config_proto_msgTypes[2]
    # 如果启用了不安全的操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        # 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 如果消息信息未加载，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

# Deprecated: 使用 UplinkCapacity.ProtoReflect.Descriptor 替代
func (*UplinkCapacity) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{2}
}

# 获取值
func (x *UplinkCapacity) GetValue() uint32 {
    if x != nil {
        return x.Value
    }
    return 0
}

# 下行容量，以 MB 为单位
type DownlinkCapacity struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value uint32 `protobuf:"varint,1,opt,name=value,proto3" json:"value,omitempty"`
}

# 重置消息
func (x *DownlinkCapacity) Reset() {
    *x = DownlinkCapacity{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

# 返回消息的字符串表示形式
func (x *DownlinkCapacity) String() string {
    return protoimpl.X.MessageStringOf(x)
}

# 实现 ProtoMessage 方法
func (*DownlinkCapacity) ProtoMessage() {}

# 实现 ProtoReflect 方法
func (x *DownlinkCapacity) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_kcp_config_proto_msgTypes[3]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

# Deprecated: 使用 DownlinkCapacity.ProtoReflect.Descriptor 替代
func (*DownlinkCapacity) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{3}
}
// GetValue 返回 DownlinkCapacity 结构体中的 Value 值，如果结构体为空则返回 0
func (x *DownlinkCapacity) GetValue() uint32 {
    if x != nil {
        return x.Value
    }
    return 0
}

// WriteBuffer 结构体定义
type WriteBuffer struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Buffer size in bytes.
    Size uint32 `protobuf:"varint,1,opt,name=size,proto3" json:"size,omitempty"`
}

// Reset 重置 WriteBuffer 结构体
func (x *WriteBuffer) Reset() {
    *x = WriteBuffer{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[4]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// String 返回 WriteBuffer 结构体的字符串表示形式
func (x *WriteBuffer) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// ProtoMessage 实现 ProtoMessage 接口
func (*WriteBuffer) ProtoMessage() {}

// ProtoReflect 实现 ProtoReflect 接口
func (x *WriteBuffer) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_kcp_config_proto_msgTypes[4]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use WriteBuffer.ProtoReflect.Descriptor instead.
// Descriptor 返回 WriteBuffer 结构体的描述符
func (*WriteBuffer) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{4}
}

// GetSize 返回 WriteBuffer 结构体中的 Size 值，如果结构体为空则返回 0
func (x *WriteBuffer) GetSize() uint32 {
    if x != nil {
        return x.Size
    }
    return 0
}

// ReadBuffer 结构体定义
type ReadBuffer struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Buffer size in bytes.
    Size uint32 `protobuf:"varint,1,opt,name=size,proto3" json:"size,omitempty"`
}

// Reset 重置 ReadBuffer 结构体
func (x *ReadBuffer) Reset() {
    *x = ReadBuffer{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// String 返回 ReadBuffer 结构体的字符串表示形式
func (x *ReadBuffer) String() string {
    # 返回给定对象的字符串表示形式
    return protoimpl.X.MessageStringOf(x)
// 空的方法，表示 ReadBuffer 类型实现了 ProtoMessage 接口
func (*ReadBuffer) ProtoMessage() {}

// 返回 ReadBuffer 类型的反射信息
func (x *ReadBuffer) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_kcp_config_proto_msgTypes[5]
    // 如果启用了不安全操作，并且 x 不为空
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

// Deprecated: Use ReadBuffer.ProtoReflect.Descriptor instead.
// 返回 ReadBuffer 类型的描述符
func (*ReadBuffer) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{5}
}

// 返回 ReadBuffer 类型的大小
func (x *ReadBuffer) GetSize() uint32 {
    if x != nil {
        return x.Size
    }
    return 0
}

// ConnectionReuse 结构体
type ConnectionReuse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Enable bool `protobuf:"varint,1,opt,name=enable,proto3" json:"enable,omitempty"`
}

// 重置 ConnectionReuse 结构体
func (x *ConnectionReuse) Reset() {
    *x = ConnectionReuse{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_transport_internet_kcp_config_proto_msgTypes[6]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ConnectionReuse 结构体的字符串表示
func (x *ConnectionReuse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 空的方法，表示 ConnectionReuse 类型实现了 ProtoMessage 接口
func (*ConnectionReuse) ProtoMessage() {}

// 返回 ConnectionReuse 类型的反射信息
func (x *ConnectionReuse) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_kcp_config_proto_msgTypes[6]
    // 如果启用了不安全操作，并且 x 不为空
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

// Deprecated: Use ConnectionReuse.ProtoReflect.Descriptor instead.
// 返回 ConnectionReuse 类型的描述符
func (*ConnectionReuse) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{6}
}

// 返回 ConnectionReuse 结构体的 Enable 字段值
func (x *ConnectionReuse) GetEnable() bool {
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的 Enable 属性
        return x.Enable
    # 如果 x 为空，返回 false
    return false
// 定义一个结构体，用于存储加密种子信息
type EncryptionSeed struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Seed string `protobuf:"bytes,1,opt,name=seed,proto3" json:"seed,omitempty"`
}

// 重置 EncryptionSeed 结构体
func (x *EncryptionSeed) Reset() {
    *x = EncryptionSeed{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[7]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 EncryptionSeed 结构体的字符串表示形式
func (x *EncryptionSeed) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*EncryptionSeed) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *EncryptionSeed) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_kcp_config_proto_msgTypes[7]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use EncryptionSeed.ProtoReflect.Descriptor instead.
// 返回 EncryptionSeed 结构体的描述符
func (*EncryptionSeed) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{7}
}

// 获取 EncryptionSeed 结构体中的 Seed 字段值
func (x *EncryptionSeed) GetSeed() string {
    if x != nil {
        return x.Seed
    }
    return ""
}

// 定义一个结构体，用于存储配置信息
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Mtu              *MTU                 `protobuf:"bytes,1,opt,name=mtu,proto3" json:"mtu,omitempty"`
    Tti              *TTI                 `protobuf:"bytes,2,opt,name=tti,proto3" json:"tti,omitempty"`
    UplinkCapacity   *UplinkCapacity      `protobuf:"bytes,3,opt,name=uplink_capacity,json=uplinkCapacity,proto3" json:"uplink_capacity,omitempty"`
    # 定义一个指向 DownlinkCapacity 结构体的指针变量，用于存储下行容量信息
    DownlinkCapacity *DownlinkCapacity    `protobuf:"bytes,4,opt,name=downlink_capacity,json=downlinkCapacity,proto3" json:"downlink_capacity,omitempty"`
    # 定义一个布尔类型变量，用于表示是否出现拥塞
    Congestion       bool                 `protobuf:"varint,5,opt,name=congestion,proto3" json:"congestion,omitempty"`
    # 定义一个指向 WriteBuffer 结构体的指针变量，用于存储写缓冲区信息
    WriteBuffer      *WriteBuffer         `protobuf:"bytes,6,opt,name=write_buffer,json=writeBuffer,proto3" json:"write_buffer,omitempty"`
    # 定义一个指向 ReadBuffer 结构体的指针变量，用于存储读缓冲区信息
    ReadBuffer       *ReadBuffer          `protobuf:"bytes,7,opt,name=read_buffer,json=readBuffer,proto3" json:"read_buffer,omitempty"`
    # 定义一个指向 serial.TypedMessage 结构体的指针变量，用于存储头部配置信息
    HeaderConfig     *serial.TypedMessage `protobuf:"bytes,8,opt,name=header_config,json=headerConfig,proto3" json:"header_config,omitempty"`
    # 定义一个指向 EncryptionSeed 结构体的指针变量，用于存储加密种子信息
    Seed             *EncryptionSeed      `protobuf:"bytes,10,opt,name=seed,proto3" json:"seed,omitempty"`
// 重置 Config 对象，将其重新赋值为空的 Config 对象
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则获取消息类型并存储消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_kcp_config_proto_msgTypes[8]
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
    mi := &file_transport_internet_kcp_config_proto_msgTypes[8]
    // 如果启用了不安全操作并且 Config 对象不为空，则获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_kcp_config_proto_rawDescGZIP(), []int{8}
}

// 返回 Config 对象的 MTU 字段
func (x *Config) GetMtu() *MTU {
    if x != nil {
        return x.Mtu
    }
    return nil
}

// 返回 Config 对象的 TTI 字段
func (x *Config) GetTti() *TTI {
    if x != nil {
        return x.Tti
    }
    return nil
}

// 返回 Config 对象的 UplinkCapacity 字段
func (x *Config) GetUplinkCapacity() *UplinkCapacity {
    if x != nil {
        return x.UplinkCapacity
    }
    return nil
}

// 返回 Config 对象的 DownlinkCapacity 字段
func (x *Config) GetDownlinkCapacity() *DownlinkCapacity {
    if x != nil {
        return x.DownlinkCapacity
    }
    return nil
}

// 返回 Config 对象的 Congestion 字段
func (x *Config) GetCongestion() bool {
    if x != nil {
        return x.Congestion
    }
    return false
}

// 返回 Config 对象的 WriteBuffer 字段
func (x *Config) GetWriteBuffer() *WriteBuffer {
    if x != nil {
        return x.WriteBuffer
    }
    return nil
}

// 返回 Config 对象的 ReadBuffer 字段
func (x *Config) GetReadBuffer() *ReadBuffer {
    if x != nil {
        return x.ReadBuffer
    }
    return nil
}

// 返回 Config 对象的 HeaderConfig 字段
func (x *Config) GetHeaderConfig() *serial.TypedMessage {
    if x != nil {
        return x.HeaderConfig
    }
    return nil
}

// 返回 Config 对象的 Seed 字段
func (x *Config) GetSeed() *EncryptionSeed {
    if x != nil {
        return x.Seed
    }
    return nil
}
// 定义变量 File_transport_internet_kcp_config_proto，用于存储协议描述符
var File_transport_internet_kcp_config_proto protoreflect.FileDescriptor

// 定义变量 file_transport_internet_kcp_config_proto_rawDesc，存储原始协议描述符的字节流
var file_transport_internet_kcp_config_proto_rawDesc = []byte{
    // 字节流内容
    0x0a, 0x23, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x6b, 0x63, 0x70, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e,
    0x2f, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2f, 0x74, 0x79, 0x70, 0x65, 0x64, 0x5f, 0x6d, 0x65,
    0x73, 0x73, 0x61, 0x67, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x1b, 0x0a, 0x03, 0x4d,
    0x54, 0x55, 0x12, 0x14, 0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28,
    0x0d, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22, 0x1b, 0x0a, 0x03, 0x54, 0x54, 0x49, 0x12,
    0x14, 0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05,
    0x76, 0x61, 0x6c, 0x75, 0x65, 0x22, 0x26, 0x0a, 0x0e, 0x55, 0x70, 0x6c, 0x69, 0x6e, 0x6b, 0x43,
    0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x12, 0x14, 0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22, 0x28, 0x0a,
    0x10, 0x44, 0x6f, 0x77, 0x6e, 0x6c, 0x69, 0x6e, 0x6b, 0x43, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74,
    0x79, 0x12, 0x14, 0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0d,
    0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22, 0x21, 0x0a, 0x0b, 0x57, 0x72, 0x69, 0x74, 0x65,
    0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x12, 0x12, 0a, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x18, 0x01,
    0x20, 0x01, 0x28, 0x0d, 0x52, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x22, 0x20, 0x0a, 0x0a, 0x52, 0x65,
    // ... 更多字节流内容
    # 创建一个十六进制数值列表
    0x61, 0x64, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x12, 0x12, 0x0a, 0x04, 0x73, 0x69, 0x7a, 0x65,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x22, 0x29, 0x0a, 0x0f,
    0x43, 0x6f, 0x6e, 0x6e, 0x65, 0x63, 0x74, 0x69, 0x6f, 0x6e, 0x52, 0x65, 0x75, 0x73, 0x65, 0x12,
    0x16, 0x0a, 0x06, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52,
    0x06, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x22, 0x24, 0x0a, 0x0e, 0x45, 0x6e, 0x63, 0x72, 0x79,
    0x70, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x65, 0x65, 0x64, 0x12, 0x12, 0x0a, 0x04, 0x73, 0x65, 0x65,
    0x64, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x73, 0x65, 0x65, 0x64, 0x22, 0x97, 0x05,
    0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x38, 0x0a, 0x03, 0x6d, 0x74, 0x75, 0x18,
    0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74,
    0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x2e, 0x4d, 0x54, 0x55, 0x52, 0x03, 0x6d,
    0x74, 0x75, 0x12, 0x38, 0x0a, 0x03, 0x74, 0x74, 0x69, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32,
    0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61,
    0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e,
    0x6b, 0x63, 0x70, 0x2e, 0x54, 0x54, 0x49, 0x52, 0x03, 0x74, 0x74, 0x69, 0x12, 0x5a, 0x0a, 0x0f,
    0x75, 0x70, 0x6c, 0x69, 0x6e, 0x6b, 0x5f, 0x63, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x18,
    0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x31, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74,
    0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x2e, 0x55, 0x70, 0x6c, 0x69, 0x6e, 0x6b,
    0x43, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x52, 0x0e, 0x75, 0x70, 0x6c, 0x69, 0x6e, 0x6b,
    # 创建一个十六进制数值列表，可能是某种配置或参数的表示
    0x43, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x12, 0x60, 0x0a, 0x11, 0x64, 0x6f, 0x77, 0x6e,
    0x6c, 0x69, 0x6e, 0x6b, 0x5f, 0x63, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x18, 0x04, 0x20,
    0x01, 0x28, 0x0b, 0x32, 0x33, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72,
    0x6e, 0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x2e, 0x44, 0x6f, 0x77, 0x6e, 0x6c, 0x69, 0x6e, 0x6b,
    0x43, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x52, 0x10, 0x64, 0x6f, 0x77, 0x6e, 0x6c, 0x69,
    0x6e, 0x6b, 0x43, 0x61, 0x70, 0x61, 0x63, 0x69, 0x74, 0x79, 0x12, 0x1e, 0x0a, 0x0a, 0x63, 0x6f,
    0x6e, 0x67, 0x65, 0x73, 0x74, 0x69, 0x6f, 0x6e, 0x18, 0x05, 0x20, 0x01, 0x28, 0x08, 0x52, 0x0a,
    0x63, 0x6f, 0x6e, 0x67, 0x65, 0x73, 0x74, 0x69, 0x6f, 0x6e, 0x12, 0x51, 0x0a, 0x0c, 0x77, 0x72,
    0x69, 0x74, 0x65, 0x5f, 0x62, 0x75, 0x66, 0x66, 0x65, 0x72, 0x18, 0x06, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x2e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
    0x2e, 0x6b, 0x63, 0x70, 0x2e, 0x57, 0x72, 0x69, 0x74, 0x65, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72,
    0x52, 0x0b, 0x77, 0x72, 0x69, 0x74, 0x65, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x12, 0x4e, 0x0a,
    0x0b, 0x72, 0x65, 0x61, 0x64, 0x5f, 0x62, 0x75, 0x66, 0x66, 0x65, 0x72, 0x18, 0x07, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e,
    0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x2e, 0x52, 0x65, 0x61, 0x64, 0x42, 0x75, 0x66, 0x66, 0x65,
    0x72, 0x52, 0x0a, 0x72, 0x65, 0x61, 0x64, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x12, 0x4b, 0x0a,
    0x0d, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x5f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x18, 0x08,
    # 定义一系列十六进制数，可能是某种数据或配置信息
    0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2e,
    0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52, 0x0c, 0x68, 0x65,
    0x61, 0x64, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x45, 0x0a, 0x04, 0x73, 0x65,
    0x65, 0x64, 0x18, 0x0a, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x31, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e,
    0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x45, 0x6e, 0x63, 0x72,
    0x79, 0x70, 0x74, 0x69, 0x6f, 0x6e, 0x53, 0x65, 0x65, 0x64, 0x52, 0x04, 0x73, 0x65, 0x65, 0x64,
    0x4a, 0x04, 0x08, 0x09, 0x10, 0x0a, 0x42, 0x74, 0x25, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72,
    0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x6b, 0x63, 0x70, 0x50, 0x01,
    0x5a, 0x25, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65,
    0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72,
    0x6e, 0x65, 0x74, 0x2f, 0x6b, 0x63, 0x70, 0x21, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f,
    0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74,
    0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x4b, 0x63, 0x70, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x33,
// 定义一个变量，用于确保 file_transport_internet_kcp_config_proto_rawDescData 只被初始化一次
var (
    file_transport_internet_kcp_config_proto_rawDescOnce sync.Once
    // 存储 file_transport_internet_kcp_config_proto_rawDesc 的原始数据
    file_transport_internet_kcp_config_proto_rawDescData = file_transport_internet_kcp_config_proto_rawDesc
)

// 定义一个函数，用于对 file_transport_internet_kcp_config_proto_rawDescData 进行 GZIP 压缩
func file_transport_internet_kcp_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_transport_internet_kcp_config_proto_rawDescOnce.Do(func() {
        // 调用 protoimpl.X.CompressGZIP 对原始数据进行 GZIP 压缩
        file_transport_internet_kcp_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_kcp_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的数据
    return file_transport_internet_kcp_config_proto_rawDescData
}

// 定义一个变量，存储消息类型的信息
var file_transport_internet_kcp_config_proto_msgTypes = make([]protoimpl.MessageInfo, 9)
// 定义一个变量，存储 Go 语言类型的信息
var file_transport_internet_kcp_config_proto_goTypes = []interface{}{
    (*MTU)(nil),                 // 0: v2ray.core.transport.internet.kcp.MTU
    (*TTI)(nil),                 // 1: v2ray.core.transport.internet.kcp.TTI
    (*UplinkCapacity)(nil),      // 2: v2ray.core.transport.internet.kcp.UplinkCapacity
    (*DownlinkCapacity)(nil),    // 3: v2ray.core.transport.internet.kcp.DownlinkCapacity
    (*WriteBuffer)(nil),         // 4: v2ray.core.transport.internet.kcp.WriteBuffer
    (*ReadBuffer)(nil),          // 5: v2ray.core.transport.internet.kcp.ReadBuffer
    (*ConnectionReuse)(nil),     // 6: v2ray.core.transport.internet.kcp.ConnectionReuse
    (*EncryptionSeed)(nil),      // 7: v2ray.core.transport.internet.kcp.EncryptionSeed
    (*Config)(nil),              // 8: v2ray.core.transport.internet.kcp.Config
    (*serial.TypedMessage)(nil), // 9: v2ray.core.common.serial.TypedMessage
}
// 定义一个变量，存储依赖的索引信息
var file_transport_internet_kcp_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.transport.internet.kcp.Config.mtu:type_name -> v2ray.core.transport.internet.kcp.MTU
    1, // 1: v2ray.core.transport.internet.kcp.Config.tti:type_name -> v2ray.core.transport.internet.kcp.TTI
    2, // 2: v2ray.core.transport.internet.kcp.Config.uplink_capacity:type_name -> v2ray.core.transport.internet.kcp.UplinkCapacity
    3, // 3: v2ray.core.transport.internet.kcp.Config.downlink_capacity:type_name -> v2ray.core.transport.internet.kcp.DownlinkCapacity
    // 第3个元素：v2ray.core.transport.internet.kcp.Config.downlink_capacity 的类型为 v2ray.core.transport.internet.kcp.DownlinkCapacity

    4, // 4: v2ray.core.transport.internet.kcp.Config.write_buffer:type_name -> v2ray.core.transport.internet.kcp.WriteBuffer
    // 第4个元素：v2ray.core.transport.internet.kcp.Config.write_buffer 的类型为 v2ray.core.transport.internet.kcp.WriteBuffer

    5, // 5: v2ray.core.transport.internet.kcp.Config.read_buffer:type_name -> v2ray.core.transport.internet.kcp.ReadBuffer
    // 第5个元素：v2ray.core.transport.internet.kcp.Config.read_buffer 的类型为 v2ray.core.transport.internet.kcp.ReadBuffer

    9, // 6: v2ray.core.transport.internet.kcp.Config.header_config:type_name -> v2ray.core.common.serial.TypedMessage
    // 第6个元素：v2ray.core.transport.internet.kcp.Config.header_config 的类型为 v2ray.core.common.serial.TypedMessage

    7, // 7: v2ray.core.transport.internet.kcp.Config.seed:type_name -> v2ray.core.transport.internet.kcp.EncryptionSeed
    // 第7个元素：v2ray.core.transport.internet.kcp.Config.seed 的类型为 v2ray.core.transport.internet.kcp.EncryptionSeed

    8, // [8:8] is the sub-list for method output_type
    // [8:8] 是用于方法输出类型的子列表

    8, // [8:8] is the sub-list for method input_type
    // [8:8] 是用于方法输入类型的子列表

    8, // [8:8] is the sub-list for extension type_name
    // [8:8] 是用于扩展类型名称的子列表

    8, // [8:8] is the sub-list for extension extendee
    // [8:8] 是用于扩展 extendee 的子列表

    0, // [0:8] is the sub-list for field type_name
    // [0:8] 是用于字段类型名称的子列表
func init() { file_transport_internet_kcp_config_proto_init() }
func file_transport_internet_kcp_config_proto_init() {
    // 如果 File_transport_internet_kcp_config_proto 不为空，则直接返回，不进行初始化
    if File_transport_internet_kcp_config_proto != nil {
        return
    }
    // 定义一个空结构体 x
    type x struct{}
    // 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            // 设置 GoPackagePath 为结构体 x 的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            // 设置 RawDescriptor 为 file_transport_internet_kcp_config_proto_rawDesc
            RawDescriptor: file_transport_internet_kcp_config_proto_rawDesc,
            // 设置 NumEnums 为 0
            NumEnums:      0,
            // 设置 NumMessages 为 9
            NumMessages:   9,
            // 设置 NumExtensions 为 0
            NumExtensions: 0,
            // 设置 NumServices 为 0
            NumServices:   0,
        },
        // 设置 GoTypes 为 file_transport_internet_kcp_config_proto_goTypes
        GoTypes:           file_transport_internet_kcp_config_proto_goTypes,
        // 设置 DependencyIndexes 为 file_transport_internet_kcp_config_proto_depIdxs
        DependencyIndexes: file_transport_internet_kcp_config_proto_depIdxs,
        // 设置 MessageInfos 为 file_transport_internet_kcp_config_proto_msgTypes
        MessageInfos:      file_transport_internet_kcp_config_proto_msgTypes,
    }.Build()
    // 将 out.File 赋值给 File_transport_internet_kcp_config_proto
    File_transport_internet_kcp_config_proto = out.File
    // 将 file_transport_internet_kcp_config_proto_rawDesc 置空
    file_transport_internet_kcp_config_proto_rawDesc = nil
    // 将 file_transport_internet_kcp_config_proto_goTypes 置空
    file_transport_internet_kcp_config_proto_goTypes = nil
    // 将 file_transport_internet_kcp_config_proto_depIdxs 置空
    file_transport_internet_kcp_config_proto_depIdxs = nil
}
```