# `v2ray-core\proxy\dns\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/dns/config.proto

package dns

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    net "v2ray.com/core/common/net"  // 导入 net 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用的是足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    // Server 是 DNS 服务器地址。如果指定，此地址将覆盖原始地址。
    Server *net.Endpoint `protobuf:"bytes,1,opt,name=server,proto3" json:"server,omitempty"`  // 服务器地址
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_dns_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Config) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_dns_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)  // 返回消息
}

// 已弃用: 使用 Config.ProtoReflect.Descriptor 代替。
# 返回使用 GZIP 压缩的原始文件描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_proxy_dns_config_proto_rawDescGZIP(), []int{0}
}

# 获取服务器端点配置信息
func (x *Config) GetServer() *net.Endpoint {
    if x != nil {
        return x.Server
    }
    return nil
}

# 定义文件描述符
var File_proxy_dns_config_proto protoreflect.FileDescriptor

# 原始文件描述符的 GZIP 压缩数据
var file_proxy_dns_config_proto_rawDesc = []byte{
    # 以下为 GZIP 压缩的原始文件描述符数据
}

# 一次性初始化原始文件描述符的 GZIP 压缩数据
var (
    file_proxy_dns_config_proto_rawDescOnce sync.Once
    file_proxy_dns_config_proto_rawDescData = file_proxy_dns_config_proto_rawDesc
)

# 返回使用 GZIP 压缩的原始文件描述符数据
func file_proxy_dns_config_proto_rawDescGZIP() []byte {
    # 使用Do方法确保只执行一次，将原始描述数据压缩为GZIP格式
    file_proxy_dns_config_proto_rawDescOnce.Do(func() {
        file_proxy_dns_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_dns_config_proto_rawDescData)
    })
    # 返回压缩后的原始描述数据
    return file_proxy_dns_config_proto_rawDescData
// 声明一个包级别的变量，用于存储消息类型信息
var file_proxy_dns_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// 声明一个包级别的变量，用于存储 Go 类型信息
var file_proxy_dns_config_proto_goTypes = []interface{}{
    (*Config)(nil),       // 0: v2ray.core.proxy.dns.Config
    (*net.Endpoint)(nil), // 1: v2ray.core.common.net.Endpoint
}
// 声明一个包级别的变量，用于存储依赖索引信息
var file_proxy_dns_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.proxy.dns.Config.server:type_name -> v2ray.core.common.net.Endpoint
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数
func init() { file_proxy_dns_config_proto_init() }
// 真正的初始化函数
func file_proxy_dns_config_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_proxy_dns_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_proxy_dns_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    // 创建一个类型构建器
    type x struct{}
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_proxy_dns_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_dns_config_proto_goTypes,
        DependencyIndexes: file_proxy_dns_config_proto_depIdxs,
        MessageInfos:      file_proxy_dns_config_proto_msgTypes,
    }.Build()
    // 将构建好的类型信息赋值给包级别的变量
    File_proxy_dns_config_proto = out.File
    // 清空原始描述和依赖索引
    file_proxy_dns_config_proto_rawDesc = nil
    file_proxy_dns_config_proto_goTypes = nil
    file_proxy_dns_config_proto_depIdxs = nil
}
```