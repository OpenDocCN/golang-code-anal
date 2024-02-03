# `v2ray-core\app\dispatcher\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/dispatcher/config.proto

// 定义 dispatcher 包
package dispatcher

// 导入必要的包
import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

// 常量定义
const (
    // 确保生成的代码足够新
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用足够新版本的 legacy proto 包
const _ = proto.ProtoPackageIsVersion4

// SessionConfig 结构体定义
type SessionConfig struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段
}

// 重置 SessionConfig 对象
func (x *SessionConfig) Reset() {
    *x = SessionConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_dispatcher_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SessionConfig 对象的字符串表示
func (x *SessionConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SessionConfig) ProtoMessage() {}

// 返回 SessionConfig 对象的反射信息
func (x *SessionConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_dispatcher_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use SessionConfig.ProtoReflect.Descriptor instead.
// 返回 SessionConfig 对象的描述符
func (*SessionConfig) Descriptor() ([]byte, []int) {
    return file_app_dispatcher_config_proto_rawDescGZIP(), []int{0}
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    # 声明一个 sizeCache 变量，用于缓存大小信息
    sizeCache     protoimpl.SizeCache
    # 声明一个 unknownFields 变量，用于存储未知字段信息
    unknownFields protoimpl.UnknownFields

    # 声明一个 Settings 变量，类型为 SessionConfig 结构体指针
    # 该变量使用 protobuf 标签指定了序列化时的字段编号和类型
    Settings *SessionConfig `protobuf:"bytes,1,opt,name=settings,proto3" json:"settings,omitempty"`
// 重置 Config 对象，将其值设为默认值
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，获取消息类型并存储消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_app_dispatcher_config_proto_msgTypes[1]
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
    mi := &file_app_dispatcher_config_proto_msgTypes[1]
    // 如果启用了不安全操作并且 Config 对象不为空，获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用方法：使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_dispatcher_config_proto_rawDescGZIP(), []int{1}
}

// 获取 Config 对象的 Settings 属性
func (x *Config) GetSettings() *SessionConfig {
    if x != nil {
        return x.Settings
    }
    return nil
}

// 定义 File_app_dispatcher_config_proto 变量
var File_app_dispatcher_config_proto protoreflect.FileDescriptor

// 定义 file_app_dispatcher_config_proto_rawDesc 变量
var file_app_dispatcher_config_proto_rawDesc = []byte{
    // 以下为一段二进制数据，表示文件描述符
}
    # 以十六进制表示的数据
    0x70, 0x61, 0x74, 0x63, 0x68, 0x65, 0x72, 0x2e, 0x53, 0x65, 0x73, 0x73, 0x69, 0x6f, 0x6e, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x08, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x42,
    0x5c, 0x0a, 0x1d, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x69, 0x73, 0x70, 0x61, 0x74, 0x63, 0x68, 0x65, 0x72,
    0x50, 0x01, 0x5a, 0x1d, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
    0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x64, 0x69, 0x73, 0x70, 0x61, 0x74, 0x63, 0x68, 0x65,
    0x72, 0xaa, 0x02, 0x19, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41,
    0x70, 0x70, 0x2e, 0x44, 0x69, 0x73, 0x70, 0x61, 0x74, 0x63, 0x68, 0x65, 0x72, 0x62, 0x06, 0x70,
    0x72, 0x6f, 0x74, 0x6f, 0x33,
# 定义变量 file_app_dispatcher_config_proto_rawDescOnce，用于确保 file_app_dispatcher_config_proto_rawDescData 只被初始化一次
var (
    file_app_dispatcher_config_proto_rawDescOnce sync.Once
    file_app_dispatcher_config_proto_rawDescData = file_app_dispatcher_config_proto_rawDesc
)

# 定义函数 file_app_dispatcher_config_proto_rawDescGZIP，用于获取经过 GZIP 压缩的原始描述数据
func file_app_dispatcher_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保原始描述数据只被压缩一次
    file_app_dispatcher_config_proto_rawDescOnce.Do(func() {
        file_app_dispatcher_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_dispatcher_config_proto_rawDescData)
    })
    # 返回经过 GZIP 压缩的原始描述数据
    return file_app_dispatcher_config_proto_rawDescData
}

# 定义变量 file_app_dispatcher_config_proto_msgTypes，存储消息类型信息
var file_app_dispatcher_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
# 定义变量 file_app_dispatcher_config_proto_goTypes，存储 Go 语言类型信息
var file_app_dispatcher_config_proto_goTypes = []interface{}{
    (*SessionConfig)(nil), // 0: v2ray.core.app.dispatcher.SessionConfig
    (*Config)(nil),        // 1: v2ray.core.app.dispatcher.Config
}
# 定义变量 file_app_dispatcher_config_proto_depIdxs，存储依赖索引信息
var file_app_dispatcher_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.app.dispatcher.Config.settings:type_name -> v2ray.core.app.dispatcher.SessionConfig
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

# 初始化函数，用于初始化文件 app_dispatcher_config_proto
func init() { file_app_dispatcher_config_proto_init() }
func file_app_dispatcher_config_proto_init() {
    # 如果文件 app_dispatcher_config_proto 不为空，则直接返回，避免重复初始化
    if File_app_dispatcher_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_app_dispatcher_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*SessionConfig); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_app_dispatcher_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
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
            RawDescriptor: file_app_dispatcher_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_app_dispatcher_config_proto_goTypes,
        DependencyIndexes: file_app_dispatcher_config_proto_depIdxs,
        MessageInfos:      file_app_dispatcher_config_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_app_dispatcher_config_proto
    File_app_dispatcher_config_proto = out.File
    # 清空原始描述和 Go 类型
    file_app_dispatcher_config_proto_rawDesc = nil
    file_app_dispatcher_config_proto_goTypes = nil
    file_app_dispatcher_config_proto_depIdxs = nil
# 闭合前面的函数定义
```