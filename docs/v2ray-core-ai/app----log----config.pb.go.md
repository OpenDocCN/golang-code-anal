# `v2ray-core\app\log\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/log/config.proto

package log

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    log "v2ray.com/core/common/log"  // 导入 log 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type LogType int32

const (
    LogType_None    LogType = 0
    LogType_Console LogType = 1
    LogType_File    LogType = 2
    LogType_Event   LogType = 3
)

// LogType 的枚举值映射。
var (
    LogType_name = map[int32]string{
        0: "None",
        1: "Console",
        2: "File",
        3: "Event",
    }
    LogType_value = map[string]int32{
        "None":    0,
        "Console": 1,
        "File":    2,
        "Event":   3,
    }
)

func (x LogType) Enum() *LogType {
    p := new(LogType)
    *p = x
    return p
}

func (x LogType) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (LogType) Descriptor() protoreflect.EnumDescriptor {
    return file_app_log_config_proto_enumTypes[0].Descriptor()
}

func (LogType) Type() protoreflect.EnumType {
    return &file_app_log_config_proto_enumTypes[0]
}

func (x LogType) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// 已弃用: 使用 LogType.Descriptor 代替。
func (LogType) EnumDescriptor() ([]byte, []int) {
    return file_app_log_config_proto_rawDescGZIP(), []int{0}
}
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    ErrorLogType  LogType      `protobuf:"varint,1,opt,name=error_log_type,json=errorLogType,proto3,enum=v2ray.core.app.log.LogType" json:"error_log_type,omitempty"`
    // 错误日志类型
    ErrorLogLevel log.Severity `protobuf:"varint,2,opt,name=error_log_level,json=errorLogLevel,proto3,enum=v2ray.core.common.log.Severity" json:"error_log_level,omitempty"`
    // 错误日志级别
    ErrorLogPath  string       `protobuf:"bytes,3,opt,name=error_log_path,json=errorLogPath,proto3" json:"error_log_path,omitempty"`
    // 错误日志路径
    AccessLogType LogType      `protobuf:"varint,4,opt,name=access_log_type,json=accessLogType,proto3,enum=v2ray.core.app.log.LogType" json:"access_log_type,omitempty"`
    // 访问日志类型
    AccessLogPath string       `protobuf:"bytes,5,opt,name=access_log_path,json=accessLogPath,proto3" json:"access_log_path,omitempty"`
    // 访问日志路径
}

func (x *Config) Reset() {
    *x = Config{}
    // 重置配置对象
    if protoimpl.UnsafeEnabled {
        mi := &file_app_log_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_log_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_log_config_proto_rawDescGZIP(), []int{0}
}

func (x *Config) GetErrorLogType() LogType {
    if x != nil {
        return x.ErrorLogType
    }
    return LogType_None
}

func (x *Config) GetErrorLogLevel() log.Severity {
    // 获取错误日志级别
    if x != nil {
        return x.ErrorLogLevel
    }
    return log.Severity_Unknown
}
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的错误日志级别
        return x.ErrorLogLevel
    # 如果 x 为空，返回未知严重性
    return log.Severity_Unknown
// 获取错误日志路径的方法
func (x *Config) GetErrorLogPath() string {
    // 如果配置对象不为空，则返回错误日志路径
    if x != nil {
        return x.ErrorLogPath
    }
    // 否则返回空字符串
    return ""
}

// 获取访问日志类型的方法
func (x *Config) GetAccessLogType() LogType {
    // 如果配置对象不为空，则返回访问日志类型
    if x != nil {
        return x.AccessLogType
    }
    // 否则返回无日志类型
    return LogType_None
}

// 获取访问日志路径的方法
func (x *Config) GetAccessLogPath() string {
    // 如果配置对象不为空，则返回访问日志路径
    if x != nil {
        return x.AccessLogPath
    }
    // 否则返回空字符串
    return ""
}

// 定义文件描述符
var File_app_log_config_proto protoreflect.FileDescriptor

// 原始文件描述符的字节流
var file_app_log_config_proto_rawDesc = []byte{
    // 这里是一段二进制数据，表示文件描述符的原始信息
    // ...
}
    # 定义一个十六进制数组，表示一些数据
    0x0c, 0x65, 0x72, 0x72, 0x6f, 0x72, 0x4c, 0x6f, 0x67, 0x50, 0x61, 0x74, 0x68, 0x12, 0x43, 0x0a,
    0x0f, 0x61, 0x63, 0x63, 0x65, 0x73, 0x73, 0x5f, 0x6c, 0x6f, 0x67, 0x5f, 0x74, 0x79, 0x70, 0x65,
    0x18, 0x04, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x1b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x6c, 0x6f, 0x67, 0x2e, 0x4c, 0x6f, 0x67, 0x54,
    0x79, 0x70, 0x65, 0x52, 0x0d, 0x61, 0x63, 0x63, 0x65, 0x73, 0x73, 0x4c, 0x6f, 0x67, 0x54, 0x79,
    0x70, 0x65, 0x12, 0x26, 0x0a, 0x0f, 0x61, 0x63, 0x63, 0x65, 0x73, 0x73, 0x5f, 0x6c, 0x6f, 0x67,
    0x5f, 0x70, 0x61, 0x74, 0x68, 0x18, 0x05, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0d, 0x61, 0x63, 0x63,
    0x65, 0x73, 0x73, 0x4c, 0x6f, 0x67, 0x50, 0x61, 0x74, 0x68, 0x2a, 0x35, 0x0a, 0x07, 0x4c, 0x6f,
    0x67, 0x54, 0x79, 0x70, 0x65, 0x12, 0x08, 0x0a, 0x04, 0x4e, 0x6f, 0x6e, 0x65, 0x10, 0x00, 0x12,
    0x0b, 0x0a, 0x07, 0x43, 0x6f, 0x6e, 0x73, 0x6f, 0x6c, 0x65, 0x10, 0x01, 0x12, 0x08, 0x0a, 0x04,
    0x46, 0x69, 0x6c, 0x65, 0x10, 0x02, 0x12, 0x09, 0x0a, 0x05, 0x45, 0x76, 0x65, 0x6e, 0x74, 0x10,
    0x03, 0x42, 0x47, 0x0a, 0x16, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x6c, 0x6f, 0x67, 0x50, 0x01, 0x5a, 0x16, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70,
    0x70, 0x2f, 0x6c, 0x6f, 0x67, 0xaa, 0x02, 0x12, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f,
    0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x4c, 0x6f, 0x67, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x33,
// 定义变量，确保 file_app_log_config_proto_rawDescData 只被初始化一次
var (
    file_app_log_config_proto_rawDescOnce sync.Once
    file_app_log_config_proto_rawDescData = file_app_log_config_proto_rawDesc
)

// 使用 GZIP 压缩文件的原始描述数据
func file_app_log_config_proto_rawDescGZIP() []byte {
    // 确保 file_app_log_config_proto_rawDescData 只被初始化一次，并对其进行 GZIP 压缩
    file_app_log_config_proto_rawDescOnce.Do(func() {
        file_app_log_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_log_config_proto_rawDescData)
    })
    return file_app_log_config_proto_rawDescData
}

// 定义枚举类型和消息类型的切片
var file_app_log_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_app_log_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_app_log_config_proto_goTypes = []interface{}{
    (LogType)(0),      // 0: v2ray.core.app.log.LogType
    (*Config)(nil),    // 1: v2ray.core.app.log.Config
    (log.Severity)(0), // 2: v2ray.core.common.log.Severity
}

// 定义依赖索引的切片
var file_app_log_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.app.log.Config.error_log_type:type_name -> v2ray.core.app.log.LogType
    2, // 1: v2ray.core.app.log.Config.error_log_level:type_name -> v2ray.core.common.log.Severity
    0, // 2: v2ray.core.app.log.Config.access_log_type:type_name -> v2ray.core.app.log.LogType
    3, // [3:3] is the sub-list for method output_type
    3, // [3:3] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

// 初始化函数
func init() { file_app_log_config_proto_init() }
func file_app_log_config_proto_init() {
    // 如果 File_app_log_config_proto 不为空，则直接返回
    if File_app_log_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_app_log_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
            # 获取 Go 包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置原始描述符
            RawDescriptor: file_app_log_config_proto_rawDesc,
            # 设置枚举数量
            NumEnums:      1,
            # 设置消息数量
            NumMessages:   1,
            # 设置扩展数量
            NumExtensions: 0,
            # 设置服务数量
            NumServices:   0,
        },
        # 设置 Go 类型
        GoTypes:           file_app_log_config_proto_goTypes,
        # 设置依赖索引
        DependencyIndexes: file_app_log_config_proto_depIdxs,
        # 设置枚举信息
        EnumInfos:         file_app_log_config_proto_enumTypes,
        # 设置消息信息
        MessageInfos:      file_app_log_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_app_log_config_proto
    File_app_log_config_proto = out.File
    # 将原始描述符设置为 nil
    file_app_log_config_proto_rawDesc = nil
    # 将 Go 类型设置为 nil
    file_app_log_config_proto_goTypes = nil
    # 将依赖索引设置为 nil
    file_app_log_config_proto_depIdxs = nil
# 闭合前面的函数定义
```