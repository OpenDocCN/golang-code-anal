# `v2ray-core\app\log\command\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/log/command/config.proto

// 声明 command 包
package command

// 导入必要的包
import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

// 声明常量
const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

// 声明 Config 结构体
type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_app_log_command_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

// 返回 Config 结构体的字符串表示
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Config 结构体的字符串表示
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回 Config 结构体的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_log_command_config_proto_msgTypes[0]  // 获取消息类型
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
// 返回 Config 结构体的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_log_command_config_proto_rawDescGZIP(), []int{0}
}

// 声明 RestartLoggerRequest 结构体
type RestartLoggerRequest struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    # 定义一个变量 unknownFields，类型为 protoimpl.UnknownFields
// 重置 RestartLoggerRequest 对象
func (x *RestartLoggerRequest) Reset() {
    // 将 RestartLoggerRequest 对象重置为空对象
    *x = RestartLoggerRequest{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_log_command_config_proto_msgTypes[1]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RestartLoggerRequest 对象的字符串表示
func (x *RestartLoggerRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*RestartLoggerRequest) ProtoMessage() {}

// 返回 RestartLoggerRequest 对象的反射信息
func (x *RestartLoggerRequest) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_log_command_config_proto_msgTypes[1]
    // 如果启用了不安全操作并且对象不为空
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

// 弃用：使用 RestartLoggerRequest.ProtoReflect.Descriptor 替代
func (*RestartLoggerRequest) Descriptor() ([]byte, []int) {
    return file_app_log_command_config_proto_rawDescGZIP(), []int{1}
}

// 定义 RestartLoggerResponse 结构体
type RestartLoggerResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置 RestartLoggerResponse 对象
func (x *RestartLoggerResponse) Reset() {
    // 将 RestartLoggerResponse 对象重置为空对象
    *x = RestartLoggerResponse{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_log_command_config_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RestartLoggerResponse 对象的字符串表示
func (x *RestartLoggerResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*RestartLoggerResponse) ProtoMessage() {}

// 返回 RestartLoggerResponse 对象的反射信息
func (x *RestartLoggerResponse) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_log_command_config_proto_msgTypes[2]
    // 如果启用了不安全操作并且对象不为空
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
// Deprecated: Use RestartLoggerResponse.ProtoReflect.Descriptor instead.
// 返回文件描述符的字节切片和路径索引
func (*RestartLoggerResponse) Descriptor() ([]byte, []int) {
    return file_app_log_command_config_proto_rawDescGZIP(), []int{2}
}

// 定义文件描述符变量
var File_app_log_command_config_proto protoreflect.FileDescriptor

// 定义原始文件描述符的字节切片
var file_app_log_command_config_proto_rawDesc = []byte{
    // ...（省略部分代码）
}
    # 这是一个十六进制的数据序列，可能是用于某种数据传输或存储的编码
    0x5f, 0x0a, 0x1e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x6c, 0x6f, 0x67, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e,
    0x64, 0x50, 0x01, 0x5a, 0x1e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
    0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x6c, 0x6f, 0x67, 0x2f, 0x63, 0x6f, 0x6d, 0x6d,
    0x61, 0x6e, 0x64, 0xaa, 0x02, 0x1a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65,
    0x2e, 0x41, 0x70, 0x70, 0x2e, 0x4c, 0x6f, 0x67, 0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64,
    0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
# 定义变量，确保 file_app_log_command_config_proto_rawDescData 只被初始化一次
var (
    file_app_log_command_config_proto_rawDescOnce sync.Once
    file_app_log_command_config_proto_rawDescData = file_app_log_command_config_proto_rawDesc
)

# 使用 GZIP 压缩文件的原始描述数据
func file_app_log_command_config_proto_rawDescGZIP() []byte {
    # 确保原始描述数据只被压缩一次
    file_app_log_command_config_proto_rawDescOnce.Do(func() {
        file_app_log_command_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_log_command_config_proto_rawDescData)
    })
    return file_app_log_command_config_proto_rawDescData
}

# 定义消息类型的切片
var file_app_log_command_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
# 定义 Go 类型的切片
var file_app_log_command_config_proto_goTypes = []interface{}{
    (*Config)(nil),                // 0: v2ray.core.app.log.command.Config
    (*RestartLoggerRequest)(nil),  // 1: v2ray.core.app.log.command.RestartLoggerRequest
    (*RestartLoggerResponse)(nil), // 2: v2ray.core.app.log.command.RestartLoggerResponse
}
# 定义依赖索引的切片
var file_app_log_command_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.app.log.command.LoggerService.RestartLogger:input_type -> v2ray.core.app.log.command.RestartLoggerRequest
    2, // 1: v2ray.core.app.log.command.LoggerService.RestartLogger:output_type -> v2ray.core.app.log.command.RestartLoggerResponse
    1, // [1:2] is the sub-list for method output_type
    0, // [0:1] is the sub-list for method input_type
    0, // [0:0] is the sub-list for extension type_name
    0, // [0:0] is the sub-list for extension extendee
    0, // [0:0] is the sub-list for field type_name
}

# 初始化函数
func init() { file_app_log_command_config_proto_init() }
# 初始化文件
func file_app_log_command_config_proto_init() {
    # 如果文件已经初始化，则直接返回
    if File_app_log_command_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_app_log_command_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Config); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_app_log_command_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*RestartLoggerRequest); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_app_log_command_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*RestartLoggerResponse); i:
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
            RawDescriptor: file_app_log_command_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   3,
            NumExtensions: 0,
            NumServices:   1,
        },
        GoTypes:           file_app_log_command_config_proto_goTypes,
        DependencyIndexes: file_app_log_command_config_proto_depIdxs,
        MessageInfos:      file_app_log_command_config_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_app_log_command_config_proto
    File_app_log_command_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_app_log_command_config_proto_rawDesc = nil
    file_app_log_command_config_proto_goTypes = nil
    # 初始化一个变量file_app_log_command_config_proto_depIdxs，并赋值为nil
    file_app_log_command_config_proto_depIdxs = nil
# 闭合前面的函数定义
```