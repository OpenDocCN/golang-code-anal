# `v2ray-core\proxy\trojan\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/trojan/config.proto

package trojan

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    // 确保生成的代码足够新
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

type Account struct {
    state         protoimpl.MessageState  // 定义 Account 结构体的状态
    sizeCache     protoimpl.SizeCache  // 定义 Account 结构体的大小缓存
    unknownFields protoimpl.UnknownFields  // 定义 Account 结构体的未知字段

    Password string `protobuf:"bytes,1,opt,name=password,proto3" json:"password,omitempty"`  // 定义 Password 字段
}

func (x *Account) Reset() {
    *x = Account{}  // 重置 Account 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_trojan_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Account) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Account 结构体的字符串表示
}

func (*Account) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Account) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_trojan_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Account.ProtoReflect.Descriptor instead.
func (*Account) Descriptor() ([]byte, []int) {
    return file_proxy_trojan_config_proto_rawDescGZIP(), []int{0}  // 返回原始描述信息
}
// 获取账户密码的方法
func (x *Account) GetPassword() string {
    // 如果账户不为空，则返回密码
    if x != nil {
        return x.Password
    }
    // 否则返回空字符串
    return ""
}

// Fallback 结构体定义
type Fallback struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Alpn string `protobuf:"bytes,1,opt,name=alpn,proto3" json:"alpn,omitempty"`
    Path string `protobuf:"bytes,2,opt,name=path,proto3" json:"path,omitempty"`
    Type string `protobuf:"bytes,3,opt,name=type,proto3" json:"type,omitempty"`
    Dest string `protobuf:"bytes,4,opt,name=dest,proto3" json:"dest,omitempty"`
    Xver uint64 `protobuf:"varint,5,opt,name=xver,proto3" json:"xver,omitempty"`
}

// 重置 Fallback 结构体
func (x *Fallback) Reset() {
    *x = Fallback{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_trojan_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Fallback 结构体的字符串表示
func (x *Fallback) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Fallback) ProtoMessage() {}

// 返回 Fallback 结构体的反射信息
func (x *Fallback) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_trojan_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 Fallback.ProtoReflect.Descriptor 替代
func (*Fallback) Descriptor() ([]byte, []int) {
    return file_proxy_trojan_config_proto_rawDescGZIP(), []int{1}
}

// 获取 Alpn 字段的值
func (x *Fallback) GetAlpn() string {
    if x != nil {
        return x.Alpn
    }
    return ""
}

// 获取 Path 字段的值
func (x *Fallback) GetPath() string {
    if x != nil {
        return x.Path
    }
    return ""
}

// 获取 Type 字段的值
func (x *Fallback) GetType() string {
    if x != nil {
        return x.Type
    }
    return ""
}

// 获取 Dest 字段的值
func (x *Fallback) GetDest() string {
    if x != nil {
        return x.Dest
    }
    return ""
}

// 获取 Xver 字段的值
func (x *Fallback) GetXver() uint64 {
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的 Xver 属性
        return x.Xver
    # 否则返回 0
    return 0
// 定义 ClientConfig 结构体，包含状态、大小缓存和未知字段
type ClientConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 服务器端点列表
    Server []*protocol.ServerEndpoint `protobuf:"bytes,1,rep,name=server,proto3" json:"server,omitempty"`
}

// 重置 ClientConfig 对象
func (x *ClientConfig) Reset() {
    *x = ClientConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_trojan_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ClientConfig 对象的字符串表示
func (x *ClientConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ClientConfig) ProtoMessage() {}

// 返回 ClientConfig 对象的反射信息
func (x *ClientConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_trojan_config_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use ClientConfig.ProtoReflect.Descriptor instead.
// 返回 ClientConfig 对象的描述符
func (*ClientConfig) Descriptor() ([]byte, []int) {
    return file_proxy_trojan_config_proto_rawDescGZIP(), []int{2}
}

// 获取 ClientConfig 对象的服务器端点列表
func (x *ClientConfig) GetServer() []*protocol.ServerEndpoint {
    if x != nil {
        return x.Server
    }
    return nil
}

// 定义 ServerConfig 结构体，包含状态、大小缓存和未知字段
type ServerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 用户列表
    Users     []*protocol.User `protobuf:"bytes,1,rep,name=users,proto3" json:"users,omitempty"`
    // 回退列表
    Fallbacks []*Fallback      `protobuf:"bytes,3,rep,name=fallbacks,proto3" json:"fallbacks,omitempty"`
}

// 重置 ServerConfig 对象
func (x *ServerConfig) Reset() {
    *x = ServerConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_trojan_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ServerConfig 对象的字符串表示
func (x *ServerConfig) String() string {
    # 返回给定对象的字符串表示形式
    return protoimpl.X.MessageStringOf(x)
// 定义 ServerConfig 结构体的 ProtoMessage 方法
func (*ServerConfig) ProtoMessage() {}

// 定义 ServerConfig 结构体的 ProtoReflect 方法
func (x *ServerConfig) ProtoReflect() protoreflect.Message {
    // 获取 ServerConfig 对应的消息类型信息
    mi := &file_proxy_trojan_config_proto_msgTypes[3]
    // 如果启用了不安全操作，并且 x 不为空
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

// Deprecated: Use ServerConfig.ProtoReflect.Descriptor instead.
// 定义 ServerConfig 结构体的 Descriptor 方法
func (*ServerConfig) Descriptor() ([]byte, []int) {
    return file_proxy_trojan_config_proto_rawDescGZIP(), []int{3}
}

// 定义 ServerConfig 结构体的 GetUsers 方法
func (x *ServerConfig) GetUsers() []*protocol.User {
    // 如果 x 不为空，则返回 Users 字段
    if x != nil {
        return x.Users
    }
    return nil
}

// 定义 ServerConfig 结构体的 GetFallbacks 方法
func (x *ServerConfig) GetFallbacks() []*Fallback {
    // 如果 x 不为空，则返回 Fallbacks 字段
    if x != nil {
        return x.Fallbacks
    }
    return nil
}

// 定义 File_proxy_trojan_config_proto 变量
var File_proxy_trojan_config_proto protoreflect.FileDescriptor

// 定义 file_proxy_trojan_config_proto_rawDesc 变量
var file_proxy_trojan_config_proto_rawDesc = []byte{
    // 以下为二进制数据，用于描述文件结构
    // ...
}
    # 定义一组十六进制数，可能是某种配置或数据的表示方式
    0x6c, 0x6c, 0x62, 0x61, 0x63, 0x6b, 0x12, 0x12, 0x0a, 0x04, 0x61, 0x6c, 0x70, 0x6e, 0x18, 0x01,
    0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x61, 0x6c, 0x70, 0x6e, 0x12, 0x12, 0x0a, 0x04, 0x70, 0x61,
    0x74, 0x68, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x70, 0x61, 0x74, 0x68, 0x12, 0x12,
    0x0a, 0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x74, 0x79,
    0x70, 0x65, 0x12, 0x12, 0x0a, 0x04, 0x64, 0x65, 0x73, 0x74, 0x18, 0x04, 0x20, 0x01, 0x28, 0x09,
    0x52, 0x04, 0x64, 0x65, 0x73, 0x74, 0x12, 0x12, 0x0a, 0x04, 0x78, 0x76, 0x65, 0x72, 0x18, 0x05,
    0x20, 0x01, 0x28, 0x04, 0x52, 0x04, 0x78, 0x76, 0x65, 0x72, 0x22, 0x52, 0x0a, 0x0c, 0x43, 0x6c,
    0x69, 0x65, 0x6e, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x42, 0x0a, 0x06, 0x73, 0x65,
    0x72, 0x76, 0x65, 0x72, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x2a, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70,
    0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x45, 0x6e,
    0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x06, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x22, 0x87,
    0x01, 0x0a, 0x0c, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x36, 0x0a, 0x05, 0x75, 0x73, 0x65, 0x72, 0x73, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
    0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72,
    0x52, 0x05, 0x75, 0x73, 0x65, 0x72, 0x73, 0x12, 0x3f, 0x0a, 0x09, 0x66, 0x61, 0x6c, 0x6c, 0x62,
    0x61, 0x63, 0x6b, 0x73, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x74, 0x72,
    0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x46, 0x61, 0x6c, 0x6c, 0x62, 0x61, 0x63, 0x6b, 0x52, 0x09, 0x66,
    # 以十六进制表示的数据，可能是某种编码或者密文
    0x61, 0x6c, 0x6c, 0x62, 0x61, 0x63, 0x6b, 0x73, 0x42, 0x56, 0x0a, 0x1b, 0x63, 0x6f, 0x6d, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61,
    0x6e, 0x50, 0x01, 0x5a, 0x1b, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
    0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e,
    0xaa, 0x02, 0x17, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72,
    0x6f, 0x78, 0x79, 0x2e, 0x54, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x33,
# 定义变量，确保 file_proxy_trojan_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_trojan_config_proto_rawDescOnce sync.Once
    file_proxy_trojan_config_proto_rawDescData = file_proxy_trojan_config_proto_rawDesc
)

# 定义函数，用于对 file_proxy_trojan_config_proto_rawDescData 进行 GZIP 压缩
func file_proxy_trojan_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_proxy_trojan_config_proto_rawDescOnce.Do(func() {
        file_proxy_trojan_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_trojan_config_proto_rawDescData)
    })
    # 返回 GZIP 压缩后的数据
    return file_proxy_trojan_config_proto_rawDescData
}

# 定义变量，存储消息类型
var file_proxy_trojan_config_proto_msgTypes = make([]protoimpl.MessageInfo, 4)
# 定义变量，存储 Go 类型
var file_proxy_trojan_config_proto_goTypes = []interface{}{
    (*Account)(nil),                 // 0: v2ray.core.proxy.trojan.Account
    (*Fallback)(nil),                // 1: v2ray.core.proxy.trojan.Fallback
    (*ClientConfig)(nil),            // 2: v2ray.core.proxy.trojan.ClientConfig
    (*ServerConfig)(nil),            // 3: v2ray.core.proxy.trojan.ServerConfig
    (*protocol.ServerEndpoint)(nil), // 4: v2ray.core.common.protocol.ServerEndpoint
    (*protocol.User)(nil),           // 5: v2ray.core.common.protocol.User
}
# 定义变量，存储依赖索引
var file_proxy_trojan_config_proto_depIdxs = []int32{
    4, // 0: v2ray.core.proxy.trojan.ClientConfig.server:type_name -> v2ray.core.common.protocol.ServerEndpoint
    5, // 1: v2ray.core.proxy.trojan.ServerConfig.users:type_name -> v2ray.core.common.protocol.User
    1, // 2: v2ray.core.proxy.trojan.ServerConfig.fallbacks:type_name -> v2ray.core.proxy.trojan.Fallback
    3, // [3:3] is the sub-list for method output_type
    3, // [3:3] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

# 初始化函数，确保文件未被初始化
func init() { file_proxy_trojan_config_proto_init() }
func file_proxy_trojan_config_proto_init() {
    if File_proxy_trojan_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_trojan_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            # 根据不同的消息类型返回相应的字段
            switch v := v.(*Account); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
        file_proxy_trojan_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            # 根据不同的消息类型返回相应的字段
            switch v := v.(*Fallback); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
        file_proxy_trojan_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
            # 根据不同的消息类型返回相应的字段
            switch v := v.(*ClientConfig); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
        file_proxy_trojan_config_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{}:
            # 根据不同的消息类型返回相应的字段
            switch v := v.(*ServerConfig); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
    # 定义一个空结构体 x
    type x struct{}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为 file_proxy_trojan_config_proto_rawDesc
            RawDescriptor: file_proxy_trojan_config_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 4
            NumMessages:   4,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为 file_proxy_trojan_config_proto_goTypes
        GoTypes:           file_proxy_trojan_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为 file_proxy_trojan_config_proto_depIdxs
        DependencyIndexes: file_proxy_trojan_config_proto_depIdxs,
        # 设置 MessageInfos 属性为 file_proxy_trojan_config_proto_msgTypes
        MessageInfos:      file_proxy_trojan_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_proxy_trojan_config_proto
    File_proxy_trojan_config_proto = out.File
    # 将 file_proxy_trojan_config_proto_rawDesc 设置为 nil
    file_proxy_trojan_config_proto_rawDesc = nil
    # 将 file_proxy_trojan_config_proto_goTypes 设置为 nil
    file_proxy_trojan_config_proto_goTypes = nil
    # 将 file_proxy_trojan_config_proto_depIdxs 设置为 nil
    file_proxy_trojan_config_proto_depIdxs = nil
# 闭合前面的函数定义
```