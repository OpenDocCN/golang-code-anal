# `v2ray-core\transport\internet\domainsocket\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/domainsocket/config.proto

package domainsocket

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

// 这是一个在编译时断言的声明，用于确保使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    // 域套接字的路径。这会覆盖上游调用者的 IP/Port 参数。
    Path string `protobuf:"bytes,1,opt,name=path,proto3" json:"path,omitempty"`
    // Abstract 指定是否使用抽象命名空间。
    // 传统上，Unix 域套接字是基于文件系统的。抽象域套接字可以在不获取文件锁的情况下使用。
    Abstract bool `protobuf:"varint,2,opt,name=abstract,proto3" json:"abstract,omitempty"`
    // 一些应用程序，例如 haproxy，在使用抽象 UDS 时，使用 sockaddr_un.sun_path 的完整长度来进行 connect(2) 或 bind(2)。
    Padding             bool `protobuf:"varint,3,opt,name=padding,proto3" json:"padding,omitempty"`
    AcceptProxyProtocol bool `protobuf:"varint,4,opt,name=acceptProxyProtocol,proto3" json:"acceptProxyProtocol,omitempty"`
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
}
    # 如果启用了不安全的功能，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_transport_internet_domainsocket_config_proto_msgTypes[0]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
}
// 返回配置的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回配置的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_domainsocket_config_proto_msgTypes[0]
    // 如果启用了不安全的操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息未加载
        if ms.LoadMessageInfo() == nil {
            // 存储消息信息
            ms.StoreMessageInfo(mi)
        }
        // 返回消息状态
        return ms
    }
    // 返回消息类型信息
    return mi.MessageOf(x)
}

// 已弃用：使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_domainsocket_config_proto_rawDescGZIP(), []int{0}
}

// 获取配置的路径
func (x *Config) GetPath() string {
    if x != nil {
        return x.Path
    }
    return ""
}

// 获取配置的抽象状态
func (x *Config) GetAbstract() bool {
    if x != nil {
        return x.Abstract
    }
    return false
}

// 获取配置的填充状态
func (x *Config) GetPadding() bool {
    if x != nil {
        return x.Padding
    }
    return false
}

// 获取配置的接受代理协议状态
func (x *Config) GetAcceptProxyProtocol() bool {
    if x != nil {
        return x.AcceptProxyProtocol
    }
    return false
}

// 文件描述符
var File_transport_internet_domainsocket_config_proto protoreflect.FileDescriptor

// 原始描述信息
var file_transport_internet_domainsocket_config_proto_rawDesc = []byte{
    0x0a, 0x2c, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x73, 0x6f, 0x63, 0x6b, 0x65,
    0x74, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x2a,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73,
    0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x64, 0x6f,
    0x6d, 0x61, 0x69, 0x6e, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0x22, 0x84, 0x01, 0x0a, 0x06, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x12, 0x0a, 0x04, 0x70, 0x61, 0x74, 0x68, 0x18, 0x01, 0x20,
    # 以十六进制表示的整数列表，可能是某种数据的编码
    0x01, 0x28, 0x09, 0x52, 0x04, 0x70, 0x61, 0x74, 0x68, 0x12, 0x1a, 0x0a, 0x08, 0x61, 0x62, 0x73,
    0x74, 0x72, 0x61, 0x63, 0x74, 0x18, 0x02, 0x20, 0x01, 0x28, 0x08, 0x52, 0x08, 0x61, 0x62, 0x73,
    0x74, 0x72, 0x61, 0x63, 0x74, 0x12, 0x18, 0x0a, 0x07, 0x70, 0x61, 0x64, 0x64, 0x69, 0x6e, 0x67,
    0x18, 0x03, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x70, 0x61, 0x64, 0x64, 0x69, 0x6e, 0x67, 0x12,
    0x30, 0x0a, 0x13, 0x61, 0x63, 0x63, 0x65, 0x70, 0x74, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x50, 0x72,
    0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x04, 0x20, 0x01, 0x28, 0x08, 0x52, 0x13, 0x61, 0x63,
    0x63, 0x65, 0x70, 0x74, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x42, 0x8f, 0x01, 0x0a, 0x2e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69,
    0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x73, 0x6f,
    0x63, 0x6b, 0x65, 0x74, 0x50, 0x01, 0x5a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74,
    0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e,
    0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0xaa, 0x02, 0x2a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43,
    0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x53, 0x6f, 0x63,
    0x6b, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义一个同步锁，确保只有一个goroutine执行file_transport_internet_domainsocket_config_proto_rawDescOnce.Do函数
var (
    file_transport_internet_domainsocket_config_proto_rawDescOnce sync.Once
    // 定义原始描述数据的变量，使用sync.Once确保只初始化一次
    file_transport_internet_domainsocket_config_proto_rawDescData = file_transport_internet_domainsocket_config_proto_rawDesc
)

// 使用GZIP压缩原始描述数据并返回压缩后的字节流
func file_transport_internet_domainsocket_config_proto_rawDescGZIP() []byte {
    // 使用sync.Once确保只执行一次GZIP压缩操作
    file_transport_internet_domainsocket_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_domainsocket_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_domainsocket_config_proto_rawDescData)
    })
    // 返回压缩后的字节流
    return file_transport_internet_domainsocket_config_proto_rawDescData
}

// 定义消息类型的切片
var file_transport_internet_domainsocket_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// 定义Go类型的切片
var file_transport_internet_domainsocket_config_proto_goTypes = []interface{}{
    (*Config)(nil), // 0: v2ray.core.transport.internet.domainsocket.Config
}
// 定义依赖索引的切片
var file_transport_internet_domainsocket_config_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型的子列表
}

// 初始化函数，用于初始化配置文件
func init() { file_transport_internet_domainsocket_config_proto_init() }
func file_transport_internet_domainsocket_config_proto_init() {
    // 如果配置文件不为空，则直接返回
    if File_transport_internet_domainsocket_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_domainsocket_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    // 定义一个空结构体x
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_transport_internet_domainsocket_config_proto_rawDesc,
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
        GoTypes:           file_transport_internet_domainsocket_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_transport_internet_domainsocket_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_transport_internet_domainsocket_config_proto_msgTypes,
    }.Build()
    # 将 File_transport_internet_domainsocket_config_proto 设置为 out.File
    File_transport_internet_domainsocket_config_proto = out.File
    # 将 file_transport_internet_domainsocket_config_proto_rawDesc 设置为 nil
    file_transport_internet_domainsocket_config_proto_rawDesc = nil
    # 将 file_transport_internet_domainsocket_config_proto_goTypes 设置为 nil
    file_transport_internet_domainsocket_config_proto_goTypes = nil
    # 将 file_transport_internet_domainsocket_config_proto_depIdxs 设置为 nil
    file_transport_internet_domainsocket_config_proto_depIdxs = nil
# 闭合前面的函数定义
```