# `v2ray-core\proxy\freedom\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/freedom/config.proto

package freedom

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

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config_DomainStrategy int32

const (
    Config_AS_IS   Config_DomainStrategy = 0
    Config_USE_IP  Config_DomainStrategy = 1
    Config_USE_IP4 Config_DomainStrategy = 2
    Config_USE_IP6 Config_DomainStrategy = 3
)

// Config_DomainStrategy 的枚举值映射。
var (
    Config_DomainStrategy_name = map[int32]string{
        0: "AS_IS",
        1: "USE_IP",
        2: "USE_IP4",
        3: "USE_IP6",
    }
    Config_DomainStrategy_value = map[string]int32{
        "AS_IS":   0,
        "USE_IP":  1,
        "USE_IP4": 2,
        "USE_IP6": 3,
    }
)

func (x Config_DomainStrategy) Enum() *Config_DomainStrategy {
    p := new(Config_DomainStrategy)
    *p = x
    return p
}

func (x Config_DomainStrategy) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Config_DomainStrategy) Descriptor() protoreflect.EnumDescriptor {
    return file_proxy_freedom_config_proto_enumTypes[0].Descriptor()
}

func (Config_DomainStrategy) Type() protoreflect.EnumType {
    return &file_proxy_freedom_config_proto_enumTypes[0]
}
// 返回枚举类型的数字值
func (x Config_DomainStrategy) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: Use Config_DomainStrategy.Descriptor instead.
// 返回枚举类型的描述符
func (Config_DomainStrategy) EnumDescriptor() ([]byte, []int) {
    return file_proxy_freedom_config_proto_rawDescGZIP(), []int{1, 0}
}

// 目的地覆盖结构体
type DestinationOverride struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Server *protocol.ServerEndpoint `protobuf:"bytes,1,opt,name=server,proto3" json:"server,omitempty"`
}

// 重置目的地覆盖结构体
func (x *DestinationOverride) Reset() {
    *x = DestinationOverride{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_freedom_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回目的地覆盖结构体的字符串表示
func (x *DestinationOverride) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*DestinationOverride) ProtoMessage() {}

// 返回目的地覆盖结构体的反射信息
func (x *DestinationOverride) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_freedom_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use DestinationOverride.ProtoReflect.Descriptor instead.
// 返回目的地覆盖结构体的描述符
func (*DestinationOverride) Descriptor() ([]byte, []int) {
    return file_proxy_freedom_config_proto_rawDescGZIP(), []int{0}
}

// 返回目的地覆盖结构体的服务器端点
func (x *DestinationOverride) GetServer() *protocol.ServerEndpoint {
    if x != nil {
        return x.Server
    }
    return nil
}

// 配置结构体
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
    # 定义一个名为 Config_DomainStrategy 的变量，类型为 DomainStrategy，使用 protobuf 标签进行序列化和反序列化
    Config_DomainStrategy Config_DomainStrategy `protobuf:"varint,1,opt,name=domain_strategy,json=domainStrategy,proto3,enum=v2ray.core.proxy.freedom.Config_DomainStrategy" json:"domain_strategy,omitempty"`
    # Timeout 已废弃，不建议使用
    Timeout             uint32               `protobuf:"varint,2,opt,name=timeout,proto3" json:"timeout,omitempty"`
    # 定义一个名为 DestinationOverride 的指针变量，类型为 DestinationOverride，使用 protobuf 标签进行序列化和反序列化
    DestinationOverride *DestinationOverride `protobuf:"bytes,3,opt,name=destination_override,json=destinationOverride,proto3" json:"destination_override,omitempty"`
    # 定义一个名为 UserLevel 的变量，类型为 uint32，使用 protobuf 标签进行序列化和反序列化
    UserLevel           uint32               `protobuf:"varint,4,opt,name=user_level,json=userLevel,proto3" json:"user_level,omitempty"`
// 重置 Config 对象，将其置为空对象
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，获取消息类型信息并存储
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_freedom_config_proto_msgTypes[1]
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
    mi := &file_proxy_freedom_config_proto_msgTypes[1]
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
// 返回 Config 对象的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_proxy_freedom_config_proto_rawDescGZIP(), []int{1}
}

// 返回 Config 对象的 DomainStrategy 属性
func (x *Config) GetDomainStrategy() Config_DomainStrategy {
    if x != nil {
        return x.DomainStrategy
    }
    return Config_AS_IS
}

// Deprecated: Do not use.
// 返回 Config 对象的 Timeout 属性
func (x *Config) GetTimeout() uint32 {
    if x != nil {
        return x.Timeout
    }
    return 0
}

// 返回 Config 对象的 DestinationOverride 属性
func (x *Config) GetDestinationOverride() *DestinationOverride {
    if x != nil {
        return x.DestinationOverride
    }
    return nil
}

// 返回 Config 对象的 UserLevel 属性
func (x *Config) GetUserLevel() uint32 {
    if x != nil {
        return x.UserLevel
    }
    return 0
}

// 定义 File_proxy_freedom_config_proto 和 file_proxy_freedom_config_proto_rawDesc 变量
var File_proxy_freedom_config_proto protoreflect.FileDescriptor

var file_proxy_freedom_config_proto_rawDesc = []byte{
    0x0a, 0x1a, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x2f,
    0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x18, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x66,
    0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70,
}
    # 创建一个十六进制数组，表示一些数据
    0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x5f, 0x73,
    0x70, 0x65, 0x63, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x59, 0x0a, 0x13, 0x44, 0x65, 0x73,
    0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x4f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65,
    0x12, 0x42, 0x0a, 0x06, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x2a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f,
    0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65,
    0x72, 0x76, 0x65, 0x72, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x06, 0x73, 0x65,
    0x72, 0x76, 0x65, 0x72, 0x22, 0xc4, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x58, 0x0a, 0x0f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x5f, 0x73, 0x74, 0x72, 0x61, 0x74, 0x65,
    0x67, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x2f, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x65,
    0x64, 0x6f, 0x6d, 0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69,
    0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x52, 0x0e, 0x64, 0x6f, 0x6d, 0x61, 0x69,
    0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x1c, 0x0a, 0x07, 0x74, 0x69, 0x6d,
    0x65, 0x6f, 0x75, 0x74, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x42, 0x02, 0x18, 0x01, 0x52, 0x07,
    0x74, 0x69, 0x6d, 0x65, 0x6f, 0x75, 0x74, 0x12, 0x60, 0x0a, 0x14, 0x64, 0x65, 0x73, 0x74, 0x69,
    0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x5f, 0x6f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65, 0x18,
    0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d,
    0x2e, 0x44, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x4f, 0x76, 0x65, 0x72,
    # 创建一个十六进制数的列表，可能是某种数据的编码表示
    0x72, 0x69, 0x64, 0x65, 0x52, 0x13, 0x64, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f,
    0x6e, 0x4f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65, 0x12, 0x1d, 0x0a, 0x0a, 0x75, 0x73, 0x65,
    0x72, 0x5f, 0x6c, 0x65, 0x76, 0x65, 0x6c, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x09, 0x75,
    0x73, 0x65, 0x72, 0x4c, 0x65, 0x76, 0x65, 0x6c, 0x22, 0x41, 0x0a, 0x0e, 0x44, 0x6f, 0x6d, 0x61,
    0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x09, 0x0a, 0x05, 0x41, 0x53,
    0x5f, 0x49, 0x53, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x55, 0x53, 0x45, 0x5f, 0x49, 0x50, 0x10,
    0x01, 0x12, 0x0b, 0x0a, 0x07, 0x55, 0x53, 0x45, 0x5f, 0x49, 0x50, 0x34, 0x10, 0x02, 0x12, 0x0b,
    0x0a, 0x07, 0x55, 0x53, 0x45, 0x5f, 0x49, 0x50, 0x36, 0x10, 0x03, 0x42, 0x59, 0x0a, 0x1c, 0x63,
    0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72,
    0x6f, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x50, 0x01, 0x5a, 0x1c, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72,
    0x6f, 0x78, 0x79, 0x2f, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0xaa, 0x02, 0x18, 0x56, 0x32,
    0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x46,
    0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保 file_proxy_freedom_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_freedom_config_proto_rawDescOnce sync.Once
    file_proxy_freedom_config_proto_rawDescData = file_proxy_freedom_config_proto_rawDesc
)

// 使用 GZIP 压缩文件描述数据
func file_proxy_freedom_config_proto_rawDescGZIP() []byte {
    // 确保文件描述数据只被压缩一次
    file_proxy_freedom_config_proto_rawDescOnce.Do(func() {
        file_proxy_freedom_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_freedom_config_proto_rawDescData)
    })
    return file_proxy_freedom_config_proto_rawDescData
}

// 定义枚举类型信息
var file_proxy_freedom_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
// 定义消息类型信息
var file_proxy_freedom_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
// 定义 Go 语言类型信息
var file_proxy_freedom_config_proto_goTypes = []interface{}{
    (Config_DomainStrategy)(0),      // 0: v2ray.core.proxy.freedom.Config.DomainStrategy
    (*DestinationOverride)(nil),     // 1: v2ray.core.proxy.freedom.DestinationOverride
    (*Config)(nil),                  // 2: v2ray.core.proxy.freedom.Config
    (*protocol.ServerEndpoint)(nil), // 3: v2ray.core.common.protocol.ServerEndpoint
}
// 定义依赖索引信息
var file_proxy_freedom_config_proto_depIdxs = []int32{
    3, // 0: v2ray.core.proxy.freedom.DestinationOverride.server:type_name -> v2ray.core.common.protocol.ServerEndpoint
    0, // 1: v2ray.core.proxy.freedom.Config.domain_strategy:type_name -> v2ray.core.proxy.freedom.Config.DomainStrategy
    1, // 2: v2ray.core.proxy.freedom.Config.destination_override:type_name -> v2ray.core.proxy.freedom.DestinationOverride
    3, // [3:3] is the sub-list for method output_type
    3, // [3:3] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

// 初始化函数
func init() { file_proxy_freedom_config_proto_init() }
// 文件初始化函数
func file_proxy_freedom_config_proto_init() {
    // 如果文件描述不为空，则直接返回
    if File_proxy_freedom_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_freedom_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*DestinationOverride); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        file_proxy_freedom_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
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
    # 定义一个空结构体 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_proxy_freedom_config_proto_rawDesc,
            NumEnums:      1,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_freedom_config_proto_goTypes,
        DependencyIndexes: file_proxy_freedom_config_proto_depIdxs,
        EnumInfos:         file_proxy_freedom_config_proto_enumTypes,
        MessageInfos:      file_proxy_freedom_config_proto_msgTypes,
    }.Build()
    # 将构建的文件赋值给 File_proxy_freedom_config_proto
    File_proxy_freedom_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_proxy_freedom_config_proto_rawDesc = nil
    file_proxy_freedom_config_proto_goTypes = nil
    file_proxy_freedom_config_proto_depIdxs = nil
# 闭合前面的函数定义
```