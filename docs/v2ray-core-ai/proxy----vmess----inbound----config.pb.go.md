# `v2ray-core\proxy\vmess\inbound\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/vmess/inbound/config.proto

package inbound

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言，用于确保使用足够新版本的 legacy proto 包。
const _ = proto.ProtoPackageIsVersion4

type DetourConfig struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    To string `protobuf:"bytes,1,opt,name=to,proto3" json:"to,omitempty"`  // 字段 To 的定义
}

func (x *DetourConfig) Reset() {
    *x = DetourConfig{}  // 重置 DetourConfig 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vmess_inbound_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *DetourConfig) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*DetourConfig) ProtoMessage() {}  // DetourConfig 的 ProtoMessage 方法

func (x *DetourConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vmess_inbound_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 DetourConfig.ProtoReflect.Descriptor 代替。
func (*DetourConfig) Descriptor() ([]byte, []int) {
    # 返回 file_proxy_vmess_inbound_config_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_proxy_vmess_inbound_config_proto_rawDescGZIP(), []int{0}
// 获取 DetourConfig 结构体的 To 字段值
func (x *DetourConfig) GetTo() string {
    if x != nil {
        return x.To
    }
    return ""
}

// DefaultConfig 结构体定义
type DefaultConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义 AlterId 字段
    AlterId uint32 `protobuf:"varint,1,opt,name=alter_id,json=alterId,proto3" json:"alter_id,omitempty"`
    // 定义 Level 字段
    Level   uint32 `protobuf:"varint,2,opt,name=level,proto3" json:"level,omitempty"`
}

// 重置 DefaultConfig 结构体
func (x *DefaultConfig) Reset() {
    *x = DefaultConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vmess_inbound_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 DefaultConfig 结构体的字符串表示
func (x *DefaultConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*DefaultConfig) ProtoMessage() {}

// 返回 DefaultConfig 结构体的反射信息
func (x *DefaultConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vmess_inbound_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use DefaultConfig.ProtoReflect.Descriptor instead.
// 返回 DefaultConfig 结构体的描述信息
func (*DefaultConfig) Descriptor() ([]byte, []int) {
    return file_proxy_vmess_inbound_config_proto_rawDescGZIP(), []int{1}
}

// 获取 DefaultConfig 结构体的 AlterId 字段值
func (x *DefaultConfig) GetAlterId() uint32 {
    if x != nil {
        return x.AlterId
    }
    return 0
}

// 获取 DefaultConfig 结构体的 Level 字段值
func (x *DefaultConfig) GetLevel() uint32 {
    if x != nil {
        return x.Level
    }
    return 0
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义 User 字段，类型为 []*protocol.User
    User                 []*protocol.User `protobuf:"bytes,1,rep,name=user,proto3" json:"user,omitempty"`
    // 定义 Default 字段，类型为 *DefaultConfig
    Default              *DefaultConfig   `protobuf:"bytes,2,opt,name=default,proto3" json:"default,omitempty"`
    # 定义一个名为Detour的指针类型的变量DetourConfig，用于存储detour的配置信息
    Detour               *DetourConfig    `protobuf:"bytes,3,opt,name=detour,proto3" json:"detour,omitempty"`
    # 定义一个布尔类型的变量SecureEncryptionOnly，用于表示是否只使用安全加密
    SecureEncryptionOnly bool             `protobuf:"varint,4,opt,name=secure_encryption_only,json=secureEncryptionOnly,proto3" json:"secure_encryption_only,omitempty"`
// 重置 Config 对象，将其值重置为空
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vmess_inbound_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Config 对象的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*Config) ProtoMessage() {}

// 返回 Config 对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vmess_inbound_config_proto_msgTypes[2]
    // 如果启用了不安全操作并且 Config 对象不为空，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_proxy_vmess_inbound_config_proto_rawDescGZIP(), []int{2}
}

// 返回 Config 对象的 User 字段
func (x *Config) GetUser() []*protocol.User {
    if x != nil {
        return x.User
    }
    return nil
}

// 返回 Config 对象的 Default 字段
func (x *Config) GetDefault() *DefaultConfig {
    if x != nil {
        return x.Default
    }
    return nil
}

// 返回 Config 对象的 Detour 字段
func (x *Config) GetDetour() *DetourConfig {
    if x != nil {
        return x.Detour
    }
    return nil
}

// 返回 Config 对象的 SecureEncryptionOnly 字段
func (x *Config) GetSecureEncryptionOnly() bool {
    if x != nil {
        return x.SecureEncryptionOnly
    }
    return false
}

// 定义 FileDescriptor 变量
var File_proxy_vmess_inbound_config_proto protoreflect.FileDescriptor

// 定义原始描述信息的字节切片
var file_proxy_vmess_inbound_config_proto_rawDesc = []byte{
    // 这里是原始描述信息的字节切片，省略部分内容
    # 创建一个十六进制数组，可能用于定义某种配置或数据结构
    0x6e, 0x64, 0x1a, 0x1a, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x63, 0x6f, 0x6c, 0x2f, 0x75, 0x73, 0x65, 0x72, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x1e,
    0x0a, 0x0c, 0x44, 0x65, 0x74, 0x6f, 0x75, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x0e,
    0x0a, 0x02, 0x74, 0x6f, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x02, 0x74, 0x6f, 0x22, 0x40,
    0x0a, 0x0d, 0x44, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x19, 0x0a, 0x08, 0x61, 0x6c, 0x74, 0x65, 0x72, 0x5f, 0x69, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28,
    0x0d, 0x52, 0x07, 0x61, 0x6c, 0x74, 0x65, 0x72, 0x49, 0x64, 0x12, 0x14, 0x0a, 0x05, 0x6c, 0x65,
    0x76, 0x65, 0x6c, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05, 0x6c, 0x65, 0x76, 0x65, 0x6c,
    0x22, 0x83, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x34, 0x0a, 0x04, 0x75,
    0x73, 0x65, 0x72, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65,
    0x72, 0x12, 0x47, 0x0a, 0x07, 0x64, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x18, 0x02, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6d, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f,
    0x75, 0x6e, 0x64, 0x2e, 0x44, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69,
    0x67, 0x52, 0x07, 0x64, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x12, 0x44, 0x0a, 0x06, 0x64, 0x65,
    0x74, 0x6f, 0x75, 0x72, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2c, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6d,
    0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x2e, 0x44, 0x65, 0x74, 0x6f,
    # 以十六进制表示的字节码序列
    0x75, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x06, 0x64, 0x65, 0x74, 0x6f, 0x75, 0x72,
    0x12, 0x34, 0x0a, 0x16, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x5f, 0x65, 0x6e, 0x63, 0x72, 0x79,
    0x70, 0x74, 0x69, 0x6f, 0x6e, 0x5f, 0x6f, 0x6e, 0x6c, 0x79, 0x18, 0x04, 0x20, 0x01, 0x28, 0x08,
    0x52, 0x14, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x45, 0x6e, 0x63, 0x72, 0x79, 0x70, 0x74, 0x69,
    0x6f, 0x6e, 0x4f, 0x6e, 0x6c, 0x79, 0x42, 0x6b, 0x0a, 0x22, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76,
    0x6d, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x22,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70,
    0x72, 0x6f, 0x78, 0x79, 0x2f, 0x76, 0x6d, 0x65, 0x73, 0x73, 0x2f, 0x69, 0x6e, 0x62, 0x6f, 0x75,
    0x6e, 0x64, 0xaa, 0x02, 0x1e, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
    0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x56, 0x6d, 0x65, 0x73, 0x73, 0x2e, 0x49, 0x6e, 0x62, 0x6f,
    0x75, 0x6e, 0x64, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
# 定义变量，确保只执行一次初始化操作
var (
    file_proxy_vmess_inbound_config_proto_rawDescOnce sync.Once
    file_proxy_vmess_inbound_config_proto_rawDescData = file_proxy_vmess_inbound_config_proto_rawDesc
)

# 使用 GZIP 压缩原始描述数据
func file_proxy_vmess_inbound_config_proto_rawDescGZIP() []byte {
    # 确保只执行一次 GZIP 压缩操作
    file_proxy_vmess_inbound_config_proto_rawDescOnce.Do(func() {
        file_proxy_vmess_inbound_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vmess_inbound_config_proto_rawDescData)
    })
    return file_proxy_vmess_inbound_config_proto_rawDescData
}

# 定义消息类型和 Go 类型
var file_proxy_vmess_inbound_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
var file_proxy_vmess_inbound_config_proto_goTypes = []interface{}{
    (*DetourConfig)(nil),  // 0: v2ray.core.proxy.vmess.inbound.DetourConfig
    (*DefaultConfig)(nil), // 1: v2ray.core.proxy.vmess.inbound.DefaultConfig
    (*Config)(nil),        // 2: v2ray.core.proxy.vmess.inbound.Config
    (*protocol.User)(nil), // 3: v2ray.core.common.protocol.User
}

# 定义依赖索引
var file_proxy_vmess_inbound_config_proto_depIdxs = []int32{
    3, // 0: v2ray.core.proxy.vmess.inbound.Config.user:type_name -> v2ray.core.common.protocol.User
    1, // 1: v2ray.core.proxy.vmess.inbound.Config.default:type_name -> v2ray.core.proxy.vmess.inbound.DefaultConfig
    0, // 2: v2ray.core.proxy.vmess.inbound.Config.detour:type_name -> v2ray.core.proxy.vmess.inbound.DetourConfig
    3, // [3:3] is the sub-list for method output_type
    3, // [3:3] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

# 初始化操作
func init() { file_proxy_vmess_inbound_config_proto_init() }
func file_proxy_vmess_inbound_config_proto_init() {
    # 如果文件已经初始化，则直接返回
    if File_proxy_vmess_inbound_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_vmess_inbound_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*DetourConfig); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_vmess_inbound_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*DefaultConfig); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_vmess_inbound_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Config); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
    # 定义一个空结构体 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_proxy_vmess_inbound_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   3,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_vmess_inbound_config_proto_goTypes,
        DependencyIndexes: file_proxy_vmess_inbound_config_proto_depIdxs,
        MessageInfos:      file_proxy_vmess_inbound_config_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_proxy_vmess_inbound_config_proto
    File_proxy_vmess_inbound_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_proxy_vmess_inbound_config_proto_rawDesc = nil
    file_proxy_vmess_inbound_config_proto_goTypes = nil
    # 初始化一个变量，用于存储文件代理的 vmess 入站配置的 proto 依赖索引
    file_proxy_vmess_inbound_config_proto_depIdxs = nil
# 闭合前面的函数定义
```