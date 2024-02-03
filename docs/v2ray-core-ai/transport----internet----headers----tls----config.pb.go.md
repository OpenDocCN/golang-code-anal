# `v2ray-core\transport\internet\headers\tls\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/headers/tls/config.proto

package tls

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

// 这是一个在编译时断言的声明，用于确保正在使用足够更新的旧 proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type PacketConfig struct {
    state         protoimpl.MessageState  // PacketConfig 的状态
    sizeCache     protoimpl.SizeCache  // PacketConfig 的大小缓存
    unknownFields protoimpl.UnknownFields  // PacketConfig 的未知字段
}

func (x *PacketConfig) Reset() {
    *x = PacketConfig{}  // 重置 PacketConfig
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_tls_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *PacketConfig) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 PacketConfig 的消息字符串
}

func (*PacketConfig) ProtoMessage() {}

func (x *PacketConfig) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_tls_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用: 使用 PacketConfig.ProtoReflect.Descriptor 代替。
func (*PacketConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_tls_config_proto_rawDescGZIP(), []int{0}  // 返回描述符
}
// 声明一个变量，用于存储描述文件的协议缓冲区
var File_transport_internet_headers_tls_config_proto protoreflect.FileDescriptor

// 存储描述文件的原始字节数据
var file_transport_internet_headers_tls_config_proto_rawDesc = []byte{
    // 这里是描述文件的原始字节数据，以十六进制表示
}

// 一次性初始化描述文件的原始字节数据
var (
    file_transport_internet_headers_tls_config_proto_rawDescOnce sync.Once
    file_transport_internet_headers_tls_config_proto_rawDescData = file_transport_internet_headers_tls_config_proto_rawDesc
}
# 返回经过 GZIP 压缩的文件内容
func file_transport_internet_headers_tls_config_proto_rawDescGZIP() []byte {
    # 一次性执行函数，对文件内容进行 GZIP 压缩
    file_transport_internet_headers_tls_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_headers_tls_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_headers_tls_config_proto_rawDescData)
    })
    # 返回经过 GZIP 压缩的文件内容
    return file_transport_internet_headers_tls_config_proto_rawDescData
}

# 定义消息类型的信息数组
var file_transport_internet_headers_tls_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
# 定义 Go 语言类型的数组
var file_transport_internet_headers_tls_config_proto_goTypes = []interface{}{
    (*PacketConfig)(nil), // 0: v2ray.core.transport.internet.headers.tls.PacketConfig
}
# 定义依赖索引的数组
var file_transport_internet_headers_tls_config_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型的子列表
}

# 初始化函数
func init() { file_transport_internet_headers_tls_config_proto_init() }
# 初始化文件的函数
func file_transport_internet_headers_tls_config_proto_init() {
    # 如果文件不为空，则返回
    if File_transport_internet_headers_tls_config_proto != nil {
        return
    }
    # 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_headers_tls_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*PacketConfig); i {
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
            RawDescriptor: file_transport_internet_headers_tls_config_proto_rawDesc,
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
        GoTypes:           file_transport_internet_headers_tls_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_transport_internet_headers_tls_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_transport_internet_headers_tls_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_headers_tls_config_proto
    File_transport_internet_headers_tls_config_proto = out.File
    # 将 file_transport_internet_headers_tls_config_proto_rawDesc 置为 nil
    file_transport_internet_headers_tls_config_proto_rawDesc = nil
    # 将 file_transport_internet_headers_tls_config_proto_goTypes 置为 nil
    file_transport_internet_headers_tls_config_proto_goTypes = nil
    # 将 file_transport_internet_headers_tls_config_proto_depIdxs 置为 nil
    file_transport_internet_headers_tls_config_proto_depIdxs = nil
# 闭合前面的函数定义
```