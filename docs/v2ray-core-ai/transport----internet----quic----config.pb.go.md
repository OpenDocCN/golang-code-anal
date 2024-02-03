# `v2ray-core\transport\internet\quic\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/quic/config.proto

package quic

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

const (
    // 确保生成的代码足够新
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用了足够新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    Key      string                   `protobuf:"bytes,1,opt,name=key,proto3" json:"key,omitempty"`  // 字符串类型的 Key 字段
    Security *protocol.SecurityConfig `protobuf:"bytes,2,opt,name=security,proto3" json:"security,omitempty"`  // protocol.SecurityConfig 类型的 Security 字段
    Header   *serial.TypedMessage     `protobuf:"bytes,3,opt,name=header,proto3" json:"header,omitempty"`  // serial.TypedMessage 类型的 Header 字段
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_quic_config_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态信息
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Config 对象的字符串表示
}

func (*Config) ProtoMessage() {}  // Config 类型的 ProtoMessage 方法

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_quic_config_proto_msgTypes[0]  // 获取消息类型信息
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
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_quic_config_proto_rawDescGZIP(), []int{0}
}

// 获取配置的密钥
func (x *Config) GetKey() string {
    if x != nil {
        return x.Key
    }
    return ""
}

// 获取配置的安全性配置
func (x *Config) GetSecurity() *protocol.SecurityConfig {
    if x != nil {
        return x.Security
    }
    return nil
}

// 获取配置的头部信息
func (x *Config) GetHeader() *serial.TypedMessage {
    if x != nil {
        return x.Header
    }
    return nil
}

// 定义文件描述符
var File_transport_internet_quic_config_proto protoreflect.FileDescriptor

// 原始描述符的字节流
var file_transport_internet_quic_config_proto_rawDesc = []byte{
    // 这里是一段很长的字节流，省略具体内容
}
    # 以十六进制表示的字节码序列，可能是某种协议或数据格式的编码
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
    0x79, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x08, 0x73, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
    0x79, 0x12, 0x3e, 0x0a, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x18, 0x03, 0x20, 0x01, 0x28,
    0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63,
    0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72, 0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70,
    0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65, 0x52, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65,
    0x72, 0x42, 0x77, 0x0a, 0x26, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x71, 0x75, 0x69, 0x63, 0x50, 0x01, 0x5a, 0x26, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
    0x2f, 0x71, 0x75, 0x69, 0x63, 0xaa, 0x02, 0x22, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f,
    0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74,
    0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x51, 0x75, 0x69, 0x63, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x33,
# 定义变量 file_transport_internet_quic_config_proto_rawDescOnce，用于确保 file_transport_internet_quic_config_proto_rawDescData 只被初始化一次
var (
    file_transport_internet_quic_config_proto_rawDescOnce sync.Once
    file_transport_internet_quic_config_proto_rawDescData = file_transport_internet_quic_config_proto_rawDesc
)

# 定义函数 file_transport_internet_quic_config_proto_rawDescGZIP，用于对 file_transport_internet_quic_config_proto_rawDescData 进行 GZIP 压缩
func file_transport_internet_quic_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_transport_internet_quic_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_quic_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_quic_config_proto_rawDescData)
    })
    # 返回 GZIP 压缩后的数据
    return file_transport_internet_quic_config_proto_rawDescData
}

# 定义变量 file_transport_internet_quic_config_proto_msgTypes，用于存储消息类型信息
var file_transport_internet_quic_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
# 定义变量 file_transport_internet_quic_config_proto_goTypes，用于存储 Go 类型信息
var file_transport_internet_quic_config_proto_goTypes = []interface{}{
    (*Config)(nil),                  // 0: v2ray.core.transport.internet.quic.Config
    (*protocol.SecurityConfig)(nil), // 1: v2ray.core.common.protocol.SecurityConfig
    (*serial.TypedMessage)(nil),     // 2: v2ray.core.common.serial.TypedMessage
}
# 定义变量 file_transport_internet_quic_config_proto_depIdxs，用于存储依赖索引信息
var file_transport_internet_quic_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.transport.internet.quic.Config.security:type_name -> v2ray.core.common.protocol.SecurityConfig
    2, // 1: v2ray.core.transport.internet.quic.Config.header:type_name -> v2ray.core.common.serial.TypedMessage
    2, // [2:2] is the sub-list for method output_type
    2, // [2:2] is the sub-list for method input_type
    2, // [2:2] is the sub-list for extension type_name
    2, // [2:2] is the sub-list for extension extendee
    0, // [0:2] is the sub-list for field type_name
}

# 初始化函数，用于确保文件未被初始化
func init() { file_transport_internet_quic_config_proto_init() }
# 初始化函数，用于确保文件未被初始化
func file_transport_internet_quic_config_proto_init() {
    # 如果文件已经被初始化，则直接返回
    if File_transport_internet_quic_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_quic_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    # 定义一个空结构体 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_transport_internet_quic_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_transport_internet_quic_config_proto_goTypes,
        DependencyIndexes: file_transport_internet_quic_config_proto_depIdxs,
        MessageInfos:      file_transport_internet_quic_config_proto_msgTypes,
    }.Build()
    # 设置全局变量 File_transport_internet_quic_config_proto 为构建的类型
    File_transport_internet_quic_config_proto = out.File
    # 重置原始描述、Go 类型和依赖索引
    file_transport_internet_quic_config_proto_rawDesc = nil
    file_transport_internet_quic_config_proto_goTypes = nil
    file_transport_internet_quic_config_proto_depIdxs = nil
# 闭合前面的函数定义
```