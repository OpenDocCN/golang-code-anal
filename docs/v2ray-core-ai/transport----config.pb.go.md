# `v2ray-core\transport\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/config.proto

package transport

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    internet "v2ray.com/core/transport/internet"  // 导入 internet 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用的 legacy proto 包足够新。
const _ = proto.ProtoPackageIsVersion4

// 全局传输设置。这影响所有通过 V2Ray 进行的连接类型。已弃用。请使用 StreamConfig 中的每个设置。
type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    TransportSettings []*internet.TransportConfig `protobuf:"bytes,1,rep,name=transport_settings,json=transportSettings,proto3" json:"transport_settings,omitempty"`  // 传输设置的切片
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Config) ProtoMessage() {}  // Config 类型的 ProtoMessage 方法

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
}
    # 返回对象 x 的消息
    return mi.MessageOf(x)
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_config_proto_rawDescGZIP(), []int{0}
}

// 获取传输设置
func (x *Config) GetTransportSettings() []*internet.TransportConfig {
    if x != nil {
        return x.TransportSettings
    }
    return nil
}

// 定义文件描述符
var File_transport_config_proto protoreflect.FileDescriptor

// 原始文件描述符的 GZIP 压缩版本
var file_transport_config_proto_rawDesc = []byte{
    // 这里是一段二进制数据，表示文件描述符的内容
}
    # 定义一个包含十六进制数值的列表
    0xaa, 0x02, 0x14, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量 file_transport_config_proto_rawDescOnce，类型为 sync.Once，用于确保 file_transport_config_proto_rawDescData 只被初始化一次
var (
    file_transport_config_proto_rawDescOnce sync.Once
    file_transport_config_proto_rawDescData = file_transport_config_proto_rawDesc
)

// 定义函数 file_transport_config_proto_rawDescGZIP，返回类型为 []byte，用于获取经过 GZIP 压缩的文件描述数据
func file_transport_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 file_transport_config_proto_rawDescData 只被压缩一次
    file_transport_config_proto_rawDescOnce.Do(func() {
        file_transport_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_config_proto_rawDescData)
    })
    return file_transport_config_proto_rawDescData
}

// 定义变量 file_transport_config_proto_msgTypes，类型为 []protoimpl.MessageInfo，用于存储消息类型信息
var file_transport_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// 定义变量 file_transport_config_proto_goTypes，类型为 []interface{}，用于存储 Go 类型信息
var file_transport_config_proto_goTypes = []interface{}{
    (*Config)(nil),                   // 0: v2ray.core.transport.Config
    (*internet.TransportConfig)(nil), // 1: v2ray.core.transport.internet.TransportConfig
}
// 定义变量 file_transport_config_proto_depIdxs，类型为 []int32，用于存储依赖索引信息
var file_transport_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.transport.Config.transport_settings:type_name -> v2ray.core.transport.internet.TransportConfig
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数，用于初始化文件传输配置协议
func init() { file_transport_config_proto_init() }
func file_transport_config_proto_init() {
    // 如果 File_transport_config_proto 不为空，则直接返回
    if File_transport_config_proto != nil {
        return
    }
    // 如果 protoimpl.UnsafeEnabled 未启用，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    // 定义类型 x
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为 file_transport_config_proto_rawDesc
            RawDescriptor: file_transport_config_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 1
            NumMessages:   1,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为 file_transport_config_proto_goTypes
        GoTypes:           file_transport_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为 file_transport_config_proto_depIdxs
        DependencyIndexes: file_transport_config_proto_depIdxs,
        # 设置 MessageInfos 属性为 file_transport_config_proto_msgTypes
        MessageInfos:      file_transport_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_config_proto
    File_transport_config_proto = out.File
    # 将 file_transport_config_proto_rawDesc 设置为 nil
    file_transport_config_proto_rawDesc = nil
    # 将 file_transport_config_proto_goTypes 设置为 nil
    file_transport_config_proto_goTypes = nil
    # 将 file_transport_config_proto_depIdxs 设置为 nil
    file_transport_config_proto_depIdxs = nil
# 闭合前面的函数定义
```