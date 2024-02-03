# `v2ray-core\transport\internet\udp\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本：
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件：transport/internet/udp/config.proto

package udp

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

// 这是一个编译时断言，用于确保正在使用足够更新的旧 proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

func (x *Config) Reset() {
    *x = Config{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_udp_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_udp_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 Config.ProtoReflect.Descriptor 代替。
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_udp_config_proto_rawDescGZIP(), []int{0}
}

var File_transport_internet_udp_config_proto protoreflect.FileDescriptor
# 定义一个包含字节的切片，用于存储文件传输配置的原始描述信息
var file_transport_internet_udp_config_proto_rawDesc = []byte{
    # 以下为字节数据，表示文件传输配置的原始描述信息
}

# 定义一个同步对象，用于确保文件传输配置的原始描述信息只被压缩一次
var (
    file_transport_internet_udp_config_proto_rawDescOnce sync.Once
    # 将文件传输配置的原始描述信息存储到file_transport_internet_udp_config_proto_rawDescData中
    file_transport_internet_udp_config_proto_rawDescData = file_transport_internet_udp_config_proto_rawDesc
)

# 定义一个函数，用于对文件传输配置的原始描述信息进行GZIP压缩
func file_transport_internet_udp_config_proto_rawDescGZIP() []byte {
    # 使用sync.Once确保只对原始描述信息进行一次GZIP压缩
    file_transport_internet_udp_config_proto_rawDescOnce.Do(func() {
        # 调用protoimpl.X.CompressGZIP对原始描述信息进行GZIP压缩，并存储到file_transport_internet_udp_config_proto_rawDescData中
        file_transport_internet_udp_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_udp_config_proto_rawDescData)
    })
    # 返回GZIP压缩后的文件传输配置的描述信息
    return file_transport_internet_udp_config_proto_rawDescData
}

# 定义一个切片，用于存储文件传输配置的消息类型信息
var file_transport_internet_udp_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
# 定义一个包含 Config 结构体指针的接口切片
var file_transport_internet_udp_config_proto_goTypes = []interface{}{
    (*Config)(nil), // 0: v2ray.core.transport.internet.udp.Config
}
# 定义一个包含索引的整型切片
var file_transport_internet_udp_config_proto_depIdxs = []int32{
    0, // [0:0] is the sub-list for method output_type
    0, // [0:0] is the sub-list for method input_type
    0, // [0:0] is the sub-list for extension type_name
    0, // [0:0] is the sub-list for extension extendee
    0, // [0:0] is the sub-list for field type_name
}

# 初始化函数
func init() { file_transport_internet_udp_config_proto_init() }
# 初始化配置文件
func file_transport_internet_udp_config_proto_init() {
    # 如果文件已经初始化，则直接返回
    if File_transport_internet_udp_config_proto != nil {
        return
    }
    # 如果不允许使用不安全的操作，则设置消息类型的导出器
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_udp_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    # 定义一个结构体 x
    type x struct{}
    # 构建类型生成器
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_transport_internet_udp_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_transport_internet_udp_config_proto_goTypes,
        DependencyIndexes: file_transport_internet_udp_config_proto_depIdxs,
        MessageInfos:      file_transport_internet_udp_config_proto_msgTypes,
    }.Build()
    # 将生成的文件赋值给全局变量
    File_transport_internet_udp_config_proto = out.File
    # 将原始描述设置为空
    file_transport_internet_udp_config_proto_rawDesc = nil
    file_transport_internet_udp_config_proto_goTypes = nil
    file_transport_internet_udp_config_proto_depIdxs = nil
}
```