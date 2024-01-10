# `v2ray-core\proxy\vless\inbound\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/vless/inbound/config.proto

package inbound

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个在编译时断言的声明，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Fallback struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    Alpn string `protobuf:"bytes,1,opt,name=alpn,proto3" json:"alpn,omitempty"`  // Alpn 字段
    Path string `protobuf:"bytes,2,opt,name=path,proto3" json:"path,omitempty"`  // Path 字段
    Type string `protobuf:"bytes,3,opt,name=type,proto3" json:"type,omitempty"`  // Type 字段
    Dest string `protobuf:"bytes,4,opt,name=dest,proto3" json:"dest,omitempty"`  // Dest 字段
    Xver uint64 `protobuf:"varint,5,opt,name=xver,proto3" json:"xver,omitempty"`  // Xver 字段
}

func (x *Fallback) Reset() {
    *x = Fallback{}  // 重置 Fallback 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vless_inbound_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Fallback) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Fallback) ProtoMessage() {}  // Fallback 的 ProtoMessage 方法

func (x *Fallback) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vless_inbound_config_proto_msgTypes[0]  // 获取消息类型
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
// Deprecated: Use Fallback.ProtoReflect.Descriptor instead.
// 使用 Fallback.ProtoReflect.Descriptor 替代此方法（已废弃）
func (*Fallback) Descriptor() ([]byte, []int) {
    return file_proxy_vless_inbound_config_proto_rawDescGZIP(), []int{0}
}

// 获取 Alpn 字段的数值
func (x *Fallback) GetAlpn() string {
    if x != nil {
        return x.Alpn
    }
    return ""
}

// 获取 Path 字段的数值
func (x *Fallback) GetPath() string {
    if x != nil {
        return x.Path
    }
    return ""
}

// 获取 Type 字段的数值
func (x *Fallback) GetType() string {
    if x != nil {
        return x.Type
    }
    return ""
}

// 获取 Dest 字段的数值
func (x *Fallback) GetDest() string {
    if x != nil {
        return x.Dest
    }
    return ""
}

// 获取 Xver 字段的数值
func (x *Fallback) GetXver() uint64 {
    if x != nil {
        return x.Xver
    }
    return 0
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Clients []*protocol.User `protobuf:"bytes,1,rep,name=clients,proto3" json:"clients,omitempty"`
    // Decryption settings. Only applies to server side, and only accepts "none"
    // for now.
    Decryption string      `protobuf:"bytes,2,opt,name=decryption,proto3" json:"decryption,omitempty"`
    Fallbacks  []*Fallback `protobuf:"bytes,3,rep,name=fallbacks,proto3" json:"fallbacks,omitempty"`
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vless_inbound_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 Config 结构体转换为字符串
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vless_inbound_config_proto_msgTypes[1]
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
// 返回文件的原始描述和包含文件描述符的切片
func (*Config) Descriptor() ([]byte, []int) {
    return file_proxy_vless_inbound_config_proto_rawDescGZIP(), []int{1}
}

// 返回配置中的客户端列表
func (x *Config) GetClients() []*protocol.User {
    if x != nil {
        return x.Clients
    }
    return nil
}

// 返回配置中的解密方式
func (x *Config) GetDecryption() string {
    if x != nil {
        return x.Decryption
    }
    return ""
}

// 返回配置中的备用服务器列表
func (x *Config) GetFallbacks() []*Fallback {
    if x != nil {
        return x.Fallbacks
    }
    return nil
}

// 定义包含文件描述符的全局变量
var File_proxy_vless_inbound_config_proto protoreflect.FileDescriptor

// 包含原始描述的文件描述符的切片
var file_proxy_vless_inbound_config_proto_rawDesc = []byte{
    // 这里是一段十六进制数据，表示文件的原始描述
}
    # 定义一个十六进制数组
    0x01, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x3a, 0x0a, 0x07, 0x63, 0x6c,
    0x69, 0x65, 0x6e, 0x74, 0x73, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f,
    0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72,
    0x52, 0x07, 0x63, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x73, 0x12, 0x1e, 0x0a, 0x0a, 0x64, 0x65,
    0x63, 0x72, 0x79, 0x70, 0x74, 0x69, 0x6f, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52,
    0x0a, 0x64, 0x65, 0x63, 0x72, 0x79, 0x70, 0x74, 0x69, 0x6f, 0x6e, 0x12, 0x46, 0x0a, 0x09,
    0x66, 0x61, 0x6c, 0x6c, 0x62, 0x61, 0x63, 0x6b, 0x73, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b,
    0x32, 0x28, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70,
    0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6c, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f,
    0x75, 0x6e, 0x64, 0x2e, 0x46, 0x61, 0x6c, 0x6c, 0x62, 0x61, 0x63, 0x6b, 0x52, 0x09, 0x66,
    0x61, 0x6c, 0x6c, 0x62, 0x61, 0x63, 0x6b, 0x73, 0x42, 0x6b, 0x0a, 0x22, 0x63, 0x6f, 0x6d,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f,
    0x78, 0x79, 0x2e, 0x76, 0x6c, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e,
    0x64, 0x50, 0x01, 0x5a, 0x22, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f,
    0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x76, 0x6c, 0x65, 0x73,
    0x73, 0x2f, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0xaa, 0x02, 0x1e, 0x56, 0x32, 0x52, 0x61,
    0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x56, 0x6c,
    0x65, 0x73, 0x73, 0x2e, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x62, 0x06, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x33,
# 定义变量，用于确保文件描述数据只被初始化一次
var (
    file_proxy_vless_inbound_config_proto_rawDescOnce sync.Once
    file_proxy_vless_inbound_config_proto_rawDescData = file_proxy_vless_inbound_config_proto_rawDesc
)

# 定义函数，用于将文件描述数据进行 GZIP 压缩
func file_proxy_vless_inbound_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保文件描述数据只被压缩一次
    file_proxy_vless_inbound_config_proto_rawDescOnce.Do(func() {
        file_proxy_vless_inbound_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vless_inbound_config_proto_rawDescData)
    })
    # 返回压缩后的文件描述数据
    return file_proxy_vless_inbound_config_proto_rawDescData
}

# 定义变量，存储消息类型信息
var file_proxy_vless_inbound_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
# 定义变量，存储 Go 类型信息
var file_proxy_vless_inbound_config_proto_goTypes = []interface{}{
    (*Fallback)(nil),      // 0: v2ray.core.proxy.vless.inbound.Fallback
    (*Config)(nil),        // 1: v2ray.core.proxy.vless.inbound.Config
    (*protocol.User)(nil), // 2: v2ray.core.common.protocol.User
}
# 定义变量，存储依赖索引信息
var file_proxy_vless_inbound_config_proto_depIdxs = []int32{
    2, // 0: v2ray.core.proxy.vless.inbound.Config.clients:type_name -> v2ray.core.common.protocol.User
    0, // 1: v2ray.core.proxy.vless.inbound.Config.fallbacks:type_name -> v2ray.core.proxy.vless.inbound.Fallback
    2, // [2:2] is the sub-list for method output_type
    2, // [2:2] is the sub-list for method input_type
    2, // [2:2] is the sub-list for extension type_name
    2, // [2:2] is the sub-list for extension extendee
    0, // [0:2] is the sub-list for field type_name
}

# 初始化函数，用于确保文件描述数据只被初始化一次
func init() { file_proxy_vless_inbound_config_proto_init() }
func file_proxy_vless_inbound_config_proto_init() {
    # 如果文件描述数据已经被初始化，则直接返回
    if File_proxy_vless_inbound_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_vless_inbound_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Fallback); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_vless_inbound_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
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
    out := protoimpl.TypeBuilder:
        File: protoimpl.DescBuilder:
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_proxy_vless_inbound_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        GoTypes:           file_proxy_vless_inbound_config_proto_goTypes,
        DependencyIndexes: file_proxy_vless_inbound_config_proto_depIdxs,
        MessageInfos:      file_proxy_vless_inbound_config_proto_msgTypes,
    # 将构建的类型赋值给 File_proxy_vless_inbound_config_proto
    File_proxy_vless_inbound_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_proxy_vless_inbound_config_proto_rawDesc = nil
    file_proxy_vless_inbound_config_proto_goTypes = nil
    # 将依赖索引设置为 nil
    file_proxy_vless_inbound_config_proto_depIdxs = nil
# 闭合前面的函数定义
```