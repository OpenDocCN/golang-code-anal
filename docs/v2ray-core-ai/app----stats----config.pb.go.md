# `v2ray-core\app\stats\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/stats/config.proto

package stats

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保生成的代码足够更新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够更新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保正在使用足够更新的旧版 proto 包。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // Config 结构体的状态
    sizeCache     protoimpl.SizeCache  // Config 结构体的大小缓存
    unknownFields protoimpl.UnknownFields  // Config 结构体的未知字段
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_app_stats_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Config 结构体的字符串表示
}

func (*Config) ProtoMessage() {}  // Config 结构体实现 ProtoMessage 接口

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用: 使用 Config.ProtoReflect.Descriptor 代替。
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_stats_config_proto_rawDescGZIP(), []int{0}  // 返回原始描述符和索引
}

type ChannelConfig struct {
    state         protoimpl.MessageState  // ChannelConfig 结构体的状态
    sizeCache     protoimpl.SizeCache  // ChannelConfig 结构体的大小缓存
    unknownFields protoimpl.UnknownFields  // ChannelConfig 结构体的未知字段
    # 定义一个布尔类型的字段，用于指示是否阻塞
    Blocking        bool  `protobuf:"varint,1,opt,name=Blocking,proto3" json:"Blocking,omitempty"`
    # 定义一个整型字段，用于指定订阅者的数量限制
    SubscriberLimit int32 `protobuf:"varint,2,opt,name=SubscriberLimit,proto3" json:"SubscriberLimit,omitempty"`
    # 定义一个整型字段，用于指定缓冲区的大小
    BufferSize      int32 `protobuf:"varint,3,opt,name=BufferSize,proto3" json:"BufferSize,omitempty"`
// 重置 ChannelConfig 对象
func (x *ChannelConfig) Reset() {
    // 将 ChannelConfig 对象重置为空对象
    *x = ChannelConfig{}
    // 如果启用了不安全操作，则获取消息类型信息并存储
    if protoimpl.UnsafeEnabled {
        mi := &file_app_stats_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ChannelConfig 对象的字符串表示
func (x *ChannelConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ChannelConfig) ProtoMessage() {}

// 返回 ChannelConfig 对象的反射信息
func (x *ChannelConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_config_proto_msgTypes[1]
    // 如果启用了不安全操作并且对象不为空，则获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 ChannelConfig.ProtoReflect.Descriptor 代替
func (*ChannelConfig) Descriptor() ([]byte, []int) {
    return file_app_stats_config_proto_rawDescGZIP(), []int{1}
}

// 获取 ChannelConfig 对象的 Blocking 属性
func (x *ChannelConfig) GetBlocking() bool {
    if x != nil {
        return x.Blocking
    }
    return false
}

// 获取 ChannelConfig 对象的 SubscriberLimit 属性
func (x *ChannelConfig) GetSubscriberLimit() int32 {
    if x != nil {
        return x.SubscriberLimit
    }
    return 0
}

// 获取 ChannelConfig 对象的 BufferSize 属性
func (x *ChannelConfig) GetBufferSize() int32 {
    if x != nil {
        return x.BufferSize
    }
    return 0
}

// 定义 File_app_stats_config_proto 的全局变量
var File_app_stats_config_proto protoreflect.FileDescriptor

// 定义 File_app_stats_config_proto 的原始描述信息
var file_app_stats_config_proto_rawDesc = []byte{
    // 省略部分内容
}
    # 这是一串十六进制数，可能是某种数据或者代码的表示方式
    0x63, 0x6b, 0x69, 0x6e, 0x67, 0x12, 0x28, 0x0a, 0x0f, 0x53, 0x75, 0x62, 0x73, 0x63, 0x72, 0x69,
    0x62, 0x65, 0x72, 0x4c, 0x69, 0x6d, 0x69, 0x74, 0x18, 0x02, 0x20, 0x01, 0x28, 0x05, 0x52, 0x0f,
    0x53, 0x75, 0x62, 0x73, 0x63, 0x72, 0x69, 0x62, 0x65, 0x72, 0x4c, 0x69, 0x6d, 0x69, 0x74, 0x12,
    0x1e, 0x0a, 0x0a, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x53, 0x69, 0x7a, 0x65, 0x18, 0x03, 0x20,
    0x01, 0x28, 0x05, 0x52, 0x0a, 0x42, 0x75, 0x66, 0x66, 0x65, 0x72, 0x53, 0x69, 0x7a, 0x65, 0x42,
    0x4d, 0x0a, 0x18, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x73, 0x74, 0x61, 0x74, 0x73, 0x50, 0x01, 0x5a, 0x18, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70,
    0x70, 0x2f, 0x73, 0x74, 0x61, 0x74, 0x73, 0xaa, 0x02, 0x14, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e,
    0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x53, 0x74, 0x61, 0x74, 0x73, 0x62, 0x06,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义一个同步锁，确保只有一个 goroutine 能够执行 file_app_stats_config_proto_rawDescOnce.Do 方法
var (
    file_app_stats_config_proto_rawDescOnce sync.Once
    // 定义一个变量用于存储原始的文件描述数据
    file_app_stats_config_proto_rawDescData = file_app_stats_config_proto_rawDesc
)

// 定义一个函数，用于对原始文件描述数据进行 GZIP 压缩
func file_app_stats_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保只有一个 goroutine 执行 GZIP 压缩操作
    file_app_stats_config_proto_rawDescOnce.Do(func() {
        // 调用 protoimpl.X.CompressGZIP 方法对原始文件描述数据进行 GZIP 压缩
        file_app_stats_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_stats_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的文件描述数据
    return file_app_stats_config_proto_rawDescData
}

// 定义消息类型的切片
var file_app_stats_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
// 定义 Go 类型的切片
var file_app_stats_config_proto_goTypes = []interface{}{
    (*Config)(nil),        // 0: v2ray.core.app.stats.Config
    (*ChannelConfig)(nil), // 1: v2ray.core.app.stats.ChannelConfig
}
// 定义依赖索引的切片
var file_app_stats_config_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型的子列表
}

// 初始化函数
func init() { file_app_stats_config_proto_init() }
func file_app_stats_config_proto_init() {
    // 如果文件描述不为空，则直接返回
    if File_app_stats_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出器
    if !protoimpl.UnsafeEnabled {
        file_app_stats_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
        file_app_stats_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*ChannelConfig); i {
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
}
    # 定义一个结构体类型 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建一个类型描述对象 out
    out := protoimpl.TypeBuilder{
        # 使用 protoimpl.DescBuilder 构建一个文件描述对象
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 为结构体 x 的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 为预定义的文件原始描述
            RawDescriptor: file_app_stats_config_proto_rawDesc,
            # 设置 NumEnums 为 0
            NumEnums:      0,
            # 设置 NumMessages 为 2
            NumMessages:   2,
            # 设置 NumExtensions 为 0
            NumExtensions: 0,
            # 设置 NumServices 为 0
            NumServices:   0,
        },
        # 设置 GoTypes 为预定义的 Go 类型
        GoTypes:           file_app_stats_config_proto_goTypes,
        # 设置 DependencyIndexes 为预定义的依赖索引
        DependencyIndexes: file_app_stats_config_proto_depIdxs,
        # 设置 MessageInfos 为预定义的消息类型信息
        MessageInfos:      file_app_stats_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_app_stats_config_proto
    File_app_stats_config_proto = out.File
    # 将 file_app_stats_config_proto_rawDesc 置为 nil
    file_app_stats_config_proto_rawDesc = nil
    # 将 file_app_stats_config_proto_goTypes 置为 nil
    file_app_stats_config_proto_goTypes = nil
    # 将 file_app_stats_config_proto_depIdxs 置为 nil
    file_app_stats_config_proto_depIdxs = nil
# 闭合前面的函数定义
```