# `v2ray-core\transport\internet\headers\srtp\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: transport/internet/headers/srtp/config.proto

package srtp

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保此生成的代码足够更新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够更新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言，用于确保使用了足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    Version     uint32 `protobuf:"varint,1,opt,name=version,proto3" json:"version,omitempty"`  // 版本号
    Padding     bool   `protobuf:"varint,2,opt,name=padding,proto3" json:"padding,omitempty"`  // 是否填充
    Extension   bool   `protobuf:"varint,3,opt,name=extension,proto3" json:"extension,omitempty"`  // 是否扩展
    CsrcCount   uint32 `protobuf:"varint,4,opt,name=csrc_count,json=csrcCount,proto3" json:"csrc_count,omitempty"`  // Csrc 数量
    Marker      bool   `protobuf:"varint,5,opt,name=marker,proto3" json:"marker,omitempty"`  // 标记
    PayloadType uint32 `protobuf:"varint,6,opt,name=payload_type,json=payloadType,proto3" json:"payload_type,omitempty"`  // 负载类型
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_srtp_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Config) ProtoMessage() {}  // Config 结构体实现 ProtoMessage 接口
// 返回配置对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_headers_srtp_config_proto_msgTypes[0]
    // 如果启用了不安全操作，并且配置对象不为空
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
    // 返回消息类型信息
    return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 返回配置对象的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_srtp_config_proto_rawDescGZIP(), []int{0}
}

// 获取配置对象的版本号
func (x *Config) GetVersion() uint32 {
    if x != nil {
        return x.Version
    }
    return 0
}

// 获取配置对象的填充标志
func (x *Config) GetPadding() bool {
    if x != nil {
        return x.Padding
    }
    return false
}

// 获取配置对象的扩展标志
func (x *Config) GetExtension() bool {
    if x != nil {
        return x.Extension
    }
    return false
}

// 获取配置对象的 CSRC 计数
func (x *Config) GetCsrcCount() uint32 {
    if x != nil {
        return x.CsrcCount
    }
    return 0
}

// 获取配置对象的标记
func (x *Config) GetMarker() bool {
    if x != nil {
        return x.Marker
    }
    return false
}

// 获取配置对象的载荷类型
func (x *Config) GetPayloadType() uint32 {
    if x != nil {
        return x.PayloadType
    }
    return 0
}

// 配置文件的描述符
var File_transport_internet_headers_srtp_config_proto protoreflect.FileDescriptor

// 配置文件的原始描述信息
var file_transport_internet_headers_srtp_config_proto_rawDesc = []byte{
    0x0a, 0x2c, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2f, 0x73, 0x72, 0x74,
    0x70, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x2a,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73,
    0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65,
    0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x73, 0x72, 0x74, 0x70, 0x22, 0xb4, 0x01, 0x0a, 0x06, 0x43,
}
    # 以十六进制表示的整数列表，可能是某种数据的编码
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x18, 0x0a, 0x07, 0x76, 0x65, 0x72, 0x73, 0x69, 0x6f, 0x6e,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x07, 0x76, 0x65, 0x72, 0x73, 0x69, 0x6f, 0x6e, 0x12,
    0x18, 0x0a, 0x07, 0x70, 0x61, 0x64, 0x64, 0x69, 0x6e, 0x67, 0x18, 0x02, 0x20, 0x01, 0x28, 0x08,
    0x52, 0x07, 0x70, 0x61, 0x64, 0x64, 0x69, 0x6e, 0x67, 0x12, 0x1c, 0x0a, 0x09, 0x65, 0x78, 0x74,
    0x65, 0x6e, 0x73, 0x69, 0x6f, 0x6e, 0x18, 0x03, 0x20, 0x01, 0x28, 0x08, 0x52, 0x09, 0x65, 0x78,
    0x74, 0x65, 0x6e, 0x73, 0x69, 0x6f, 0x6e, 0x12, 0x1d, 0x0a, 0x0a, 0x63, 0x73, 0x72, 0x63, 0x5f,
    0x63, 0x6f, 0x75, 0x6e, 0x74, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x09, 0x63, 0x73, 0x72,
    0x63, 0x43, 0x6f, 0x75, 0x6e, 0x74, 0x12, 0x16, 0x0a, 0x06, 0x6d, 0x61, 0x72, 0x6b, 0x65, 0x72,
    0x18, 0x05, 0x20, 0x01, 0x28, 0x08, 0x52, 0x06, 0x6d, 0x61, 0x72, 0x6b, 0x65, 0x72, 0x12, 0x21,
    0x0a, 0x0c, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64, 0x5f, 0x74, 0x79, 0x70, 0x65, 0x18, 0x06,
    0x20, 0x01, 0x28, 0x0d, 0x52, 0x0b, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64, 0x54, 0x79, 0x70,
    0x65, 0x42, 0x8f, 0x01, 0x0a, 0x2e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69,
    0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e,
    0x73, 0x72, 0x74, 0x70, 0x50, 0x01, 0x5a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74,
    0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72,
    0x73, 0x2f, 0x73, 0x72, 0x74, 0x70, 0xaa, 0x02, 0x2a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43,
    0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x53,
    # 这是一个十六进制数的列表，表示一组数据
    # 每个数字代表一个十六进制数
    # 0x72 表示十六进制数 72
    # 0x74 表示十六进制数 74
    # 0x70 表示十六进制数 70
    # 0x62 表示十六进制数 62
    # 0x06 表示十六进制数 06
    # 0x70 表示十六进制数 70
    # 0x72 表示十六进制数 72
    # 0x6f 表示十六进制数 6f
    # 0x74 表示十六进制数 74
    # 0x6f 表示十六进制数 6f
    # 0x33 表示十六进制数 33
# 定义变量，用于确保文件的原始描述只被初始化一次
var (
    file_transport_internet_headers_srtp_config_proto_rawDescOnce sync.Once
    file_transport_internet_headers_srtp_config_proto_rawDescData = file_transport_internet_headers_srtp_config_proto_rawDesc
)

# 定义函数，用于对文件的原始描述进行 GZIP 压缩
func file_transport_internet_headers_srtp_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保原始描述只被压缩一次
    file_transport_internet_headers_srtp_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_headers_srtp_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_headers_srtp_config_proto_rawDescData)
    })
    # 返回压缩后的原始描述数据
    return file_transport_internet_headers_srtp_config_proto_rawDescData
}

# 定义变量，存储消息类型信息
var file_transport_internet_headers_srtp_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
# 定义变量，存储 Go 类型信息
var file_transport_internet_headers_srtp_config_proto_goTypes = []interface{}{
    (*Config)(nil), // 0: v2ray.core.transport.internet.headers.srtp.Config
}
# 定义变量，存储依赖索引信息
var file_transport_internet_headers_srtp_config_proto_depIdxs = []int32{
    0, // [0:0] 是方法输出类型的子列表
    0, // [0:0] 是方法输入类型的子列表
    0, // [0:0] 是扩展类型名称的子列表
    0, // [0:0] 是扩展扩展的子列表
    0, // [0:0] 是字段类型名称的子列表
}

# 初始化函数，用于初始化文件
func init() { file_transport_internet_headers_srtp_config_proto_init() }
func file_transport_internet_headers_srtp_config_proto_init() {
    # 如果文件已经被初始化，则直接返回
    if File_transport_internet_headers_srtp_config_proto != nil {
        return
    }
    # 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_headers_srtp_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    # 定义一个空结构体
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_transport_internet_headers_srtp_config_proto_rawDesc,
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
        GoTypes:           file_transport_internet_headers_srtp_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_transport_internet_headers_srtp_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_transport_internet_headers_srtp_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_headers_srtp_config_proto
    File_transport_internet_headers_srtp_config_proto = out.File
    # 将 file_transport_internet_headers_srtp_config_proto_rawDesc 置为 nil
    file_transport_internet_headers_srtp_config_proto_rawDesc = nil
    # 将 file_transport_internet_headers_srtp_config_proto_goTypes 置为 nil
    file_transport_internet_headers_srtp_config_proto_goTypes = nil
    # 将 file_transport_internet_headers_srtp_config_proto_depIdxs 置为 nil
    file_transport_internet_headers_srtp_config_proto_depIdxs = nil
# 闭合前面的函数定义
```