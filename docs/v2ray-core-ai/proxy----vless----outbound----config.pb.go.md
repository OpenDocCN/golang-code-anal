# `v2ray-core\proxy\vless\outbound\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/vless/outbound/config.proto

package outbound

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    // 确保此生成的代码足够新。
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

    Vnext []*protocol.ServerEndpoint `protobuf:"bytes,1,rep,name=vnext,proto3" json:"vnext,omitempty"`  // Vnext 字段，类型为 protocol.ServerEndpoint 的切片
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vless_outbound_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Config 结构体的字符串表示形式
}

func (*Config) ProtoMessage() {}  // Config 结构体实现 ProtoMessage 接口

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vless_outbound_config_proto_msgTypes[0]  // 获取消息类型
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
    # 返回 file_proxy_vless_outbound_config_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_proxy_vless_outbound_config_proto_rawDescGZIP(), []int{0}
// 获取配置文件中的 Vnext 字段，返回 ServerEndpoint 结构体的切片
func (x *Config) GetVnext() []*protocol.ServerEndpoint {
    // 如果配置文件不为空，则返回 Vnext 字段
    if x != nil {
        return x.Vnext
    }
    // 如果配置文件为空，则返回空值
    return nil
}

// 定义文件描述符变量
var File_proxy_vless_outbound_config_proto protoreflect.FileDescriptor

// 定义原始文件描述符的字节切片
var file_proxy_vless_outbound_config_proto_rawDesc = []byte{
    // ...（省略部分内容）
};
    # 这些十六进制数字可能是表示字符串的编码，但需要进一步处理才能确定其具体含义
// 定义一个变量，用于确保 file_proxy_vless_outbound_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_vless_outbound_config_proto_rawDescOnce sync.Once
    file_proxy_vless_outbound_config_proto_rawDescData = file_proxy_vless_outbound_config_proto_rawDesc
)

// 定义一个函数，用于将 file_proxy_vless_outbound_config_proto_rawDescData 压缩为 GZIP 格式的字节数组
func file_proxy_vless_outbound_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保只有第一次调用时才会执行压缩操作
    file_proxy_vless_outbound_config_proto_rawDescOnce.Do(func() {
        file_proxy_vless_outbound_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vless_outbound_config_proto_rawDescData)
    })
    return file_proxy_vless_outbound_config_proto_rawDescData
}

// 定义消息类型的切片
var file_proxy_vless_outbound_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// 定义 Go 类型的切片
var file_proxy_vless_outbound_config_proto_goTypes = []interface{}{
    (*Config)(nil),                  // 0: v2ray.core.proxy.vless.outbound.Config
    (*protocol.ServerEndpoint)(nil), // 1: v2ray.core.common.protocol.ServerEndpoint
}
// 定义依赖索引的切片
var file_proxy_vless_outbound_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.proxy.vless.outbound.Config.vnext:type_name -> v2ray.core.common.protocol.ServerEndpoint
    1, // [1:1] 是输出类型的子列表
    1, // [1:1] 是输入类型的子列表
    1, // [1:1] 是扩展类型的子列表
    1, // [1:1] 是扩展的扩展对象
    0, // [0:1] 是字段类型的子列表
}

// 初始化函数，用于初始化文件
func init() { file_proxy_vless_outbound_config_proto_init() }
func file_proxy_vless_outbound_config_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_proxy_vless_outbound_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出器
    if !protoimpl.UnsafeEnabled {
        file_proxy_vless_outbound_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    // 定义一个空结构体
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_proxy_vless_outbound_config_proto_rawDesc,
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
        GoTypes:           file_proxy_vless_outbound_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_proxy_vless_outbound_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_proxy_vless_outbound_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_proxy_vless_outbound_config_proto
    File_proxy_vless_outbound_config_proto = out.File
    # 将 file_proxy_vless_outbound_config_proto_rawDesc 置为 nil
    file_proxy_vless_outbound_config_proto_rawDesc = nil
    # 将 file_proxy_vless_outbound_config_proto_goTypes 置为 nil
    file_proxy_vless_outbound_config_proto_goTypes = nil
    # 将 file_proxy_vless_outbound_config_proto_depIdxs 置为 nil
    file_proxy_vless_outbound_config_proto_depIdxs = nil
# 闭合前面的函数定义
```