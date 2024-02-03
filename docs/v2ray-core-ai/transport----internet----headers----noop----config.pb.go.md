# `v2ray-core\transport\internet\headers\noop\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/headers/noop/config.proto

package noop

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // Config 结构体的状态
    sizeCache     protoimpl.SizeCache  // Config 结构体的大小缓存
    unknownFields protoimpl.UnknownFields  // Config 结构体的未知字段
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_noop_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Config 结构体的字符串表示
}

func (*Config) ProtoMessage() {}  // Config 结构体实现 ProtoMessage 接口

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_noop_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_noop_config_proto_rawDescGZIP(), []int{0}  // 返回 Config 结构体的描述符
}

type ConnectionConfig struct {
    state         protoimpl.MessageState  // ConnectionConfig 结构体的状态
    # 创建一个用于存储大小信息的缓存对象
    sizeCache     protoimpl.SizeCache
    # 创建一个用于存储未知字段的对象
    unknownFields protoimpl.UnknownFields
// 重置 ConnectionConfig 对象
func (x *ConnectionConfig) Reset() {
    // 将 ConnectionConfig 对象重置为空对象
    *x = ConnectionConfig{}
    // 如果启用了不安全模式
    if protoimpl.UnsafeEnabled {
        // 获取消息类型
        mi := &file_transport_internet_headers_noop_config_proto_msgTypes[1]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ConnectionConfig 对象的字符串表示
func (x *ConnectionConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ConnectionConfig) ProtoMessage() {}

// 返回 ConnectionConfig 对象的反射信息
func (x *ConnectionConfig) ProtoReflect() protoreflect.Message {
    // 获取消息类型
    mi := &file_transport_internet_headers_noop_config_proto_msgTypes[1]
    // 如果启用了不安全模式并且对象不为空
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

// 已弃用：使用 ConnectionConfig.ProtoReflect.Descriptor 代替
func (*ConnectionConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_noop_config_proto_rawDescGZIP(), []int{1}
}

// 定义文件描述符
var File_transport_internet_headers_noop_config_proto protoreflect.FileDescriptor

// 原始文件描述符
var file_transport_internet_headers_noop_config_proto_rawDesc = []byte{
    // 省略部分内容
    # 这是一个十六进制的数据序列，可能是用于某种数据传输或存储的编码
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e,
    0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68,
    0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x6e, 0x6f, 0x6f, 0x70, 0x50, 0x01, 0x5a, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
    0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2f, 0x6e, 0x6f, 0x6f, 0x70, 0xaa, 0x02, 0x2a,
    0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73,
    0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x48, 0x65,
    0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x4e, 0x6f, 0x6f, 0x70, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x33,
# 定义变量 file_transport_internet_headers_noop_config_proto_rawDescOnce，确保只执行一次初始化操作
var (
    file_transport_internet_headers_noop_config_proto_rawDescOnce sync.Once
    file_transport_internet_headers_noop_config_proto_rawDescData = file_transport_internet_headers_noop_config_proto_rawDesc
)

# 定义函数 file_transport_internet_headers_noop_config_proto_rawDescGZIP，用于获取压缩后的原始描述数据
func file_transport_internet_headers_noop_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保只执行一次压缩操作
    file_transport_internet_headers_noop_config_proto_rawDescOnce.Do(func() {
        # 调用 protoimpl.X.CompressGZIP 对原始描述数据进行 GZIP 压缩
        file_transport_internet_headers_noop_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_headers_noop_config_proto_rawDescData)
    })
    # 返回压缩后的原始描述数据
    return file_transport_internet_headers_noop_config_proto_rawDescData
}

# 定义变量 file_transport_internet_headers_noop_config_proto_msgTypes，存储消息类型信息
var file_transport_internet_headers_noop_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
# 定义变量 file_transport_internet_headers_noop_config_proto_goTypes，存储 Go 语言类型信息
var file_transport_internet_headers_noop_config_proto_goTypes = []interface{}{
    (*Config)(nil),           // 0: v2ray.core.transport.internet.headers.noop.Config
    (*ConnectionConfig)(nil), // 1: v2ray.core.transport.internet.headers.noop.ConnectionConfig
}
# 定义变量 file_transport_internet_headers_noop_config_proto_depIdxs，存储依赖索引信息
var file_transport_internet_headers_noop_config_proto_depIdxs = []int32{
    0, // [0:0] is the sub-list for method output_type
    0, // [0:0] is the sub-list for method input_type
    0, // [0:0] is the sub-list for extension type_name
    0, // [0:0] is the sub-list for extension extendee
    0, // [0:0] is the sub-list for field type_name
}

# 初始化函数，用于初始化配置文件
func init() { file_transport_internet_headers_noop_config_proto_init() }
func file_transport_internet_headers_noop_config_proto_init() {
    # 如果文件已经初始化，则直接返回
    if File_transport_internet_headers_noop_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_transport_internet_headers_noop_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Config); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_transport_internet_headers_noop_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*ConnectionConfig); i:
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
            RawDescriptor: file_transport_internet_headers_noop_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_transport_internet_headers_noop_config_proto_goTypes,
        DependencyIndexes: file_transport_internet_headers_noop_config_proto_depIdxs,
        MessageInfos:      file_transport_internet_headers_noop_config_proto_msgTypes,
    }.Build()
    # 设置全局变量 File_transport_internet_headers_noop_config_proto 为 out.File
    File_transport_internet_headers_noop_config_proto = out.File
    # 将全局变量 file_transport_internet_headers_noop_config_proto_rawDesc 置为 nil
    file_transport_internet_headers_noop_config_proto_rawDesc = nil
    # 将全局变量 file_transport_internet_headers_noop_config_proto_goTypes 置为 nil
    file_transport_internet_headers_noop_config_proto_goTypes = nil
    # 将全局变量 file_transport_internet_headers_noop_config_proto_depIdxs 置为 nil
    file_transport_internet_headers_noop_config_proto_depIdxs = nil
# 闭合前面的函数定义
```