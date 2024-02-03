# `v2ray-core\transport\internet\tcp\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/tcp/config.proto

package tcp

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

const (
    // 确保生成的代码足够新
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    HeaderSettings      *serial.TypedMessage `protobuf:"bytes,2,opt,name=header_settings,json=headerSettings,proto3" json:"header_settings,omitempty"`  // 头部设置
    AcceptProxyProtocol bool                 `protobuf:"varint,3,opt,name=accept_proxy_protocol,json=acceptProxyProtocol,proto3" json:"accept_proxy_protocol,omitempty"`  // 是否接受代理协议
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_tcp_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Config) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_tcp_config_proto_msgTypes[0]  // 获取消息类型
    # 检查是否启用了不安全的功能，并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil:
        # 获取 x 对应的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 如果消息状态中没有加载消息信息，则存储消息信息
        if ms.LoadMessageInfo() == nil:
            ms.StoreMessageInfo(mi)
        # 返回消息状态
        return ms
    # 返回 x 对应的消息信息
    return mi.MessageOf(x)
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 返回配置的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_tcp_config_proto_rawDescGZIP(), []int{0}
}

// 返回配置的头部设置
func (x *Config) GetHeaderSettings() *serial.TypedMessage {
    if x != nil {
        return x.HeaderSettings
    }
    return nil
}

// 返回是否接受代理协议
func (x *Config) GetAcceptProxyProtocol() bool {
    if x != nil {
        return x.AcceptProxyProtocol
    }
    return false
}

// 定义文件描述符
var File_transport_internet_tcp_config_proto protoreflect.FileDescriptor

// 原始描述符的压缩版本
var file_transport_internet_tcp_config_proto_rawDesc = []byte{
    // 这里是一段二进制数据，表示文件描述符的内容
}
    # 这是一串十六进制数，可能是某种编码或者数据表示方式，需要进一步解析才能确定其作用
    0x18, 0x03, 0x20, 0x01, 0x28, 0x08, 0x52, 0x13, 0x61, 0x63, 0x63, 0x65, 0x70, 0x74, 0x50, 0x72,
    0x6f, 0x78, 0x79, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x4a, 0x04, 0x08, 0x01, 0x10,
    0x02, 0x42, 0x74, 0x0a, 0x25, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x74, 0x63, 0x70, 0x50, 0x01, 0x5a, 0x25, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61,
    0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f,
    0x74, 0x63, 0x70, 0xaa, 0x02, 0x21, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65,
    0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74, 0x65, 0x72,
    0x6e, 0x65, 0x74, 0x2e, 0x54, 0x63, 0x70, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保 file_transport_internet_tcp_config_proto_rawDescData 只被初始化一次
var (
    file_transport_internet_tcp_config_proto_rawDescOnce sync.Once
    file_transport_internet_tcp_config_proto_rawDescData = file_transport_internet_tcp_config_proto_rawDesc
)

// 使用 GZIP 压缩原始描述数据
func file_transport_internet_tcp_config_proto_rawDescGZIP() []byte {
    file_transport_internet_tcp_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_tcp_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_tcp_config_proto_rawDescData)
    })
    return file_transport_internet_tcp_config_proto_rawDescData
}

// 定义消息类型和 Go 类型
var file_transport_internet_tcp_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_transport_internet_tcp_config_proto_goTypes = []interface{}{
    (*Config)(nil),              // 0: v2ray.core.transport.internet.tcp.Config
    (*serial.TypedMessage)(nil), // 1: v2ray.core.common.serial.TypedMessage
}

// 定义依赖索引
var file_transport_internet_tcp_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.transport.internet.tcp.Config.header_settings:type_name -> v2ray.core.common.serial.TypedMessage
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数
func init() { file_transport_internet_tcp_config_proto_init() }
func file_transport_internet_tcp_config_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_transport_internet_tcp_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_tcp_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
            RawDescriptor: file_transport_internet_tcp_config_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 1
            NumMessages:   1,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为指定的 Go 类型
        GoTypes:           file_transport_internet_tcp_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_transport_internet_tcp_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_transport_internet_tcp_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_tcp_config_proto
    File_transport_internet_tcp_config_proto = out.File
    # 将 file_transport_internet_tcp_config_proto_rawDesc 置为 nil
    file_transport_internet_tcp_config_proto_rawDesc = nil
    # 将 file_transport_internet_tcp_config_proto_goTypes 置为 nil
    file_transport_internet_tcp_config_proto_goTypes = nil
    # 将 file_transport_internet_tcp_config_proto_depIdxs 置为 nil
    file_transport_internet_tcp_config_proto_depIdxs = nil
# 闭合前面的函数定义
```