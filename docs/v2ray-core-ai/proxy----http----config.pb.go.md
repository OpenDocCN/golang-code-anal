# `v2ray-core\proxy\http\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/http/config.proto

package http

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

type Account struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    Username string `protobuf:"bytes,1,opt,name=username,proto3" json:"username,omitempty"`  // 用户名字段
    Password string `protobuf:"bytes,2,opt,name=password,proto3" json:"password,omitempty"`  // 密码字段
}

func (x *Account) Reset() {
    *x = Account{}  // 重置 Account 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_http_config_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Account) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Account 对象的字符串表示形式
}

func (*Account) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Account) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_http_config_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用: 使用 Account.ProtoReflect.Descriptor 代替。
// 返回 HTTP 配置文件的原始描述信息和索引
func (*Account) Descriptor() ([]byte, []int) {
    return file_proxy_http_config_proto_rawDescGZIP(), []int{0}
}

// 获取账户的用户名
func (x *Account) GetUsername() string {
    if x != nil {
        return x.Username
    }
    return ""
}

// 获取账户的密码
func (x *Account) GetPassword() string {
    if x != nil {
        return x.Password
    }
    return ""
}

// HTTP 代理服务器的配置
type ServerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 已废弃：不要使用
    Timeout          uint32            `protobuf:"varint,1,opt,name=timeout,proto3" json:"timeout,omitempty"`
    Accounts         map[string]string `protobuf:"bytes,2,rep,name=accounts,proto3" json:"accounts,omitempty" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
    AllowTransparent bool              `protobuf:"varint,3,opt,name=allow_transparent,json=allowTransparent,proto3" json:"allow_transparent,omitempty"`
    UserLevel        uint32            `protobuf:"varint,4,opt,name=user_level,json=userLevel,proto3" json:"user_level,omitempty"`
}

// 重置 ServerConfig 对象
func (x *ServerConfig) Reset() {
    *x = ServerConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_http_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ServerConfig 对象的字符串表示
func (x *ServerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*ServerConfig) ProtoMessage() {}

// 返回 ServerConfig 对象的反射信息
func (x *ServerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_http_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已废弃：使用 ServerConfig.ProtoReflect.Descriptor 替代
func (*ServerConfig) Descriptor() ([]byte, []int) {
    # 返回 file_proxy_http_config_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_proxy_http_config_proto_rawDescGZIP(), []int{1}
// Deprecated: Do not use.
// 获取超时时间，如果对象不为空则返回超时时间，否则返回0
func (x *ServerConfig) GetTimeout() uint32 {
    if x != nil {
        return x.Timeout
    }
    return 0
}

// 获取账户信息，如果对象不为空则返回账户信息，否则返回nil
func (x *ServerConfig) GetAccounts() map[string]string {
    if x != nil {
        return x.Accounts
    }
    return nil
}

// 获取是否允许透明代理，如果对象不为空则返回是否允许透明代理，否则返回false
func (x *ServerConfig) GetAllowTransparent() bool {
    if x != nil {
        return x.AllowTransparent
    }
    return false
}

// 获取用户级别，如果对象不为空则返回用户级别，否则返回0
func (x *ServerConfig) GetUserLevel() uint32 {
    if x != nil {
        return x.UserLevel
    }
    return 0
}

// ClientConfig 是 HTTP 代理客户端的 protobuf 配置
type ClientConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Server 是 HTTP 服务器地址的列表
    Server []*protocol.ServerEndpoint `protobuf:"bytes,1,rep,name=server,proto3" json:"server,omitempty"`
}

// 重置 ClientConfig 对象
func (x *ClientConfig) Reset() {
    *x = ClientConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_http_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ClientConfig 对象的字符串表示
func (x *ClientConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*ClientConfig) ProtoMessage() {}

// 返回 ClientConfig 对象的反射信息
func (x *ClientConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_http_config_proto_msgTypes[2]
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
    return file_proxy_http_config_proto_rawDescGZIP(), []int{2}
}

// 获取服务器列表，如果对象不为空则返回服务器列表，否则返回nil
func (x *ClientConfig) GetServer() []*protocol.ServerEndpoint {
    if x != nil {
        return x.Server
    }
    return nil
}

// File_proxy_http_config_proto 是 protobuf 文件描述符
var File_proxy_http_config_proto protoreflect.FileDescriptor
# 定义一个名为 file_proxy_http_config_proto_rawDesc 的变量，存储的是字节流
var file_proxy_http_config_proto_rawDesc = []byte{
    # 以下是字节流的十六进制表示，表示一个特定的协议的原始描述信息
    0x0a, 0x17, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x68, 0x74, 0x74, 0x70, 0x2f, 0x63, 0x6f, 0x6e,
    0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x68, 0x74, 0x74, 0x70,
    0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x2f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x5f, 0x73, 0x70, 0x65, 0x63, 0x2e, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x22, 0x41, 0x0a, 0x07, 0x41, 0x63, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x12, 0x1a,
    0x0a, 0x08, 0x75, 0x73, 0x65, 0x72, 0x6e, 0x61, 0x6d, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09,
    0x52, 0x08, 0x75, 0x73, 0x65, 0x72, 0x6e, 0x61, 0x6d, 0x65, 0x12, 0x1a, 0x0a, 0x08, 0x70, 0x61,
    0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x08, 0x70, 0x61,
    0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x22, 0x84, 0x02, 0x0a, 0x0c, 0x53, 0x65, 0x72, 0x76, 0x65,
    0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x1c, 0x0a, 0x07, 0x74, 0x69, 0x6d, 0x65, 0x6f,
    0x75, 0x74, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x42, 0x02, 0x18, 0x01, 0x52, 0x07, 0x74, 0x69,
    0x6d, 0x65, 0x6f, 0x75, 0x74, 0x12, 0x4d, 0x0a, 0x08, 0x61, 0x63, 0x63, 0x6f, 0x75, 0x6e, 0x74,
    0x73, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x31, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x2e,
    0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x41, 0x63, 0x63,
    0x6f, 0x75, 0x6e, 0x74, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x52, 0x08, 0x61, 0x63, 0x63, 0x6f,
    0x75, 0x6e, 0x74, 0x73, 0x12, 0x2b, 0x0a, 0x11, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x5f, 0x74, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x61, 0x72, 0x65, 0x6e, 0x74, 0x18, 0x03, 0x20, 0x01, 0x28, 0x08, 0x52,
    # 以下是一个十六进制数的列表，可能是某种配置或数据的表示
    0x10, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x61, 0x72, 0x65, 0x6e,
    0x74, 0x12, 0x1d, 0x0a, 0x0a, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x6c, 0x65, 0x76, 0x65, 0x6c, 0x18,
    0x04, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x09, 0x75, 0x73, 0x65, 0x72, 0x4c, 0x65, 0x76, 0x65, 0x6c,
    0x1a, 0x3b, 0x0a, 0x0d, 0x41, 0x63, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x73, 0x45, 0x6e, 0x74, 0x72,
    0x79, 0x12, 0x10, 0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03,
    0x6b, 0x65, 0x79, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01,
    0x28, 0x09, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x3a, 0x02, 0x38, 0x01, 0x22, 0x52, 0x0a,
    0x0c, 0x43, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x42, 0x0a,
    0x06, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x2a, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f,
    0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x72, 0x76, 0x65,
    0x72, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x06, 0x73, 0x65, 0x72, 0x76, 0x65,
    0x72, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x50, 0x01,
    0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65,
    0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x68, 0x74, 0x74, 0x70, 0xaa, 0x02, 0x15, 0x56, 0x32,
    0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x48,
    0x74, 0x74, 0x70, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
# 定义变量 file_proxy_http_config_proto_rawDescOnce，用于确保 file_proxy_http_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_http_config_proto_rawDescOnce sync.Once
    file_proxy_http_config_proto_rawDescData = file_proxy_http_config_proto_rawDesc
)

# 定义函数 file_proxy_http_config_proto_rawDescGZIP，用于获取经过 GZIP 压缩的文件描述数据
func file_proxy_http_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保 file_proxy_http_config_proto_rawDescData 只被压缩一次
    file_proxy_http_config_proto_rawDescOnce.Do(func() {
        file_proxy_http_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_http_config_proto_rawDescData)
    })
    # 返回经过 GZIP 压缩的文件描述数据
    return file_proxy_http_config_proto_rawDescData
}

# 定义变量 file_proxy_http_config_proto_msgTypes，存储消息类型信息
var file_proxy_http_config_proto_msgTypes = make([]protoimpl.MessageInfo, 4)
# 定义变量 file_proxy_http_config_proto_goTypes，存储 Go 语言类型信息
var file_proxy_http_config_proto_goTypes = []interface{}{
    (*Account)(nil),                 // 0: v2ray.core.proxy.http.Account
    (*ServerConfig)(nil),            // 1: v2ray.core.proxy.http.ServerConfig
    (*ClientConfig)(nil),            // 2: v2ray.core.proxy.http.ClientConfig
    nil,                             // 3: v2ray.core.proxy.http.ServerConfig.AccountsEntry
    (*protocol.ServerEndpoint)(nil), // 4: v2ray.core.common.protocol.ServerEndpoint
}
# 定义变量 file_proxy_http_config_proto_depIdxs，存储依赖索引信息
var file_proxy_http_config_proto_depIdxs = []int32{
    3, // 0: v2ray.core.proxy.http.ServerConfig.accounts:type_name -> v2ray.core.proxy.http.ServerConfig.AccountsEntry
    4, // 1: v2ray.core.proxy.http.ClientConfig.server:type_name -> v2ray.core.common.protocol.ServerEndpoint
    2, // [2:2] is the sub-list for method output_type
    2, // [2:2] is the sub-list for method input_type
    2, // [2:2] is the sub-list for extension type_name
    2, // [2:2] is the sub-list for extension extendee
    0, // [0:2] is the sub-list for field type_name
}

# 初始化函数，用于初始化文件描述信息
func init() { file_proxy_http_config_proto_init() }
func file_proxy_http_config_proto_init() {
    # 如果 File_proxy_http_config_proto 不为空，则直接返回，避免重复初始化
    if File_proxy_http_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_http_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Account); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_http_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*ServerConfig); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_http_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*ClientConfig); i:
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
            RawDescriptor: file_proxy_http_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   4,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_http_config_proto_goTypes,
        DependencyIndexes: file_proxy_http_config_proto_depIdxs,
        MessageInfos:      file_proxy_http_config_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_proxy_http_config_proto
    File_proxy_http_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_proxy_http_config_proto_rawDesc = nil
    file_proxy_http_config_proto_goTypes = nil
    file_proxy_http_config_proto_depIdxs = nil
# 闭合前面的函数定义
```