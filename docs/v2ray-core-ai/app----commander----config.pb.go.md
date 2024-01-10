# `v2ray-core\app\commander\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/commander/config.proto

package commander

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// Config 是 Commander 的设置。
type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    // 处理 grpc 连接的出站处理程序的标签。
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    // 该服务器支持的服务。所有服务必须实现 Service 接口。
    Service []*serial.TypedMessage `protobuf:"bytes,2,rep,name=service,proto3" json:"service,omitempty"`
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_app_commander_config_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态信息
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Config) ProtoMessage() {}  // Config 实现 ProtoMessage 接口

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_commander_config_proto_msgTypes[0]  // 获取消息类型信息
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
    return file_app_commander_config_proto_rawDescGZIP(), []int{0}
}

// 获取配置的标签
func (x *Config) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取配置的服务
func (x *Config) GetService() []*serial.TypedMessage {
    if x != nil {
        return x.Service
    }
    return nil
}

// 定义文件描述符
var File_app_commander_config_proto protoreflect.FileDescriptor

// 原始文件描述符的 GZIP 压缩版本
var file_app_commander_config_proto_rawDesc = []byte{
    // 这里是一段二进制数据，表示文件描述符的内容
}
    # 这是一个十六进制的数据序列，可能是用于某种编码或加密的数据
    # 无法确定具体含义，需要根据上下文和需求来解析和处理
# 定义变量，用于确保 file_app_commander_config_proto_rawDescData 只被初始化一次
var (
    file_app_commander_config_proto_rawDescOnce sync.Once
    file_app_commander_config_proto_rawDescData = file_app_commander_config_proto_rawDesc
)

# 定义函数，用于将 file_app_commander_config_proto_rawDescData 压缩为 GZIP 格式的字节数组
func file_app_commander_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保只有一个 goroutine 执行压缩操作
    file_app_commander_config_proto_rawDescOnce.Do(func() {
        file_app_commander_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_commander_config_proto_rawDescData)
    })
    return file_app_commander_config_proto_rawDescData
}

# 定义变量，存储消息类型信息
var file_app_commander_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
# 定义变量，存储 Go 类型信息
var file_app_commander_config_proto_goTypes = []interface{}{
    (*Config)(nil),              // 0: v2ray.core.app.commander.Config
    (*serial.TypedMessage)(nil), // 1: v2ray.core.common.serial.TypedMessage
}
# 定义变量，存储依赖索引信息
var file_app_commander_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.app.commander.Config.service:type_name -> v2ray.core.common.serial.TypedMessage
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

# 初始化函数，用于初始化文件 app_commander_config_proto
func init() { file_app_commander_config_proto_init() }
func file_app_commander_config_proto_init() {
    # 如果文件 app_commander_config_proto 已经被初始化，则直接返回
    if File_app_commander_config_proto != nil {
        return
    }
    # 如果 protoimpl.UnsafeEnabled 未启用，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_app_commander_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_app_commander_config_proto_rawDesc,
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
        GoTypes:           file_app_commander_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_app_commander_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_app_commander_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_app_commander_config_proto
    File_app_commander_config_proto = out.File
    # 将 file_app_commander_config_proto_rawDesc 设置为 nil
    file_app_commander_config_proto_rawDesc = nil
    # 将 file_app_commander_config_proto_goTypes 设置为 nil
    file_app_commander_config_proto_goTypes = nil
    # 将 file_app_commander_config_proto_depIdxs 设置为 nil
    file_app_commander_config_proto_depIdxs = nil
# 闭合前面的函数定义
```