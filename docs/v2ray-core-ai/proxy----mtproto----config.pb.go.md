# `v2ray-core\proxy\mtproto\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/mtproto/config.proto

package mtproto

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保正在使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Account struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Secret []byte `protobuf:"bytes,1,opt,name=secret,proto3" json:"secret,omitempty"`  // 定义 Account 结构体的 Secret 字段
}

func (x *Account) Reset() {
    *x = Account{}  // 重置 Account 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_mtproto_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Account) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Account 结构体的字符串表示形式
}

func (*Account) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Account) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_mtproto_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 Account.ProtoReflect.Descriptor 代替。
func (*Account) Descriptor() ([]byte, []int) {
    return file_proxy_mtproto_config_proto_rawDescGZIP(), []int{0}  // 返回原始描述符和索引
}
// GetSecret 返回账户的密钥
func (x *Account) GetSecret() []byte {
    // 如果账户不为空，则返回账户的密钥
    if x != nil {
        return x.Secret
    }
    // 否则返回空值
    return nil
}

// ServerConfig 服务器配置结构体
type ServerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // User 是允许连接到该入站的用户列表
    // 尽管这是一个重复字段，但目前只有第一个用户有效
    User []*protocol.User `protobuf:"bytes,1,rep,name=user,proto3" json:"user,omitempty"`
}

// Reset 重置服务器配置
func (x *ServerConfig) Reset() {
    *x = ServerConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_mtproto_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// String 返回服务器配置的字符串表示
func (x *ServerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// ProtoMessage 实现 ProtoMessage 接口
func (*ServerConfig) ProtoMessage() {}

// ProtoReflect 实现 ProtoReflect 接口
func (x *ServerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_mtproto_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use ServerConfig.ProtoReflect.Descriptor instead.
// Descriptor 返回服务器配置的描述符
func (*ServerConfig) Descriptor() ([]byte, []int) {
    return file_proxy_mtproto_config_proto_rawDescGZIP(), []int{1}
}

// GetUser 返回服务器配置中的用户列表
func (x *ServerConfig) GetUser() []*protocol.User {
    if x != nil {
        return x.User
    }
    return nil
}

// ClientConfig 客户端配置结构体
type ClientConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// Reset 重置客户端配置
func (x *ClientConfig) Reset() {
    *x = ClientConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_mtproto_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}
// 将 ClientConfig 结构体转换为字符串
func (x *ClientConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*ClientConfig) ProtoMessage() {}

// 返回 ClientConfig 结构体的反射信息
func (x *ClientConfig) ProtoReflect() protoreflect.Message {
    // 获取 ClientConfig 结构体的消息类型信息
    mi := &file_proxy_mtproto_config_proto_msgTypes[2]
    // 如果启用了不安全模式，并且结构体不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取结构体的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息状态中没有加载消息信息，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    // 返回消息类型信息
    return mi.MessageOf(x)
}

// 已废弃：使用 ClientConfig.ProtoReflect.Descriptor 替代
func (*ClientConfig) Descriptor() ([]byte, []int) {
    return file_proxy_mtproto_config_proto_rawDescGZIP(), []int{2}
}

// 定义 File_proxy_mtproto_config_proto 和 file_proxy_mtproto_config_proto_rawDesc 变量
var File_proxy_mtproto_config_proto protoreflect.FileDescriptor
var file_proxy_mtproto_config_proto_rawDesc = []byte{
    // 省略部分字节数据
};
    # 这是一个十六进制的数据序列，可能是某种配置或者编码的表示方式
    # 无法直接解释其作用，需要更多上下文才能理解
# 定义变量，用于确保 file_proxy_mtproto_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_mtproto_config_proto_rawDescOnce sync.Once
    file_proxy_mtproto_config_proto_rawDescData = file_proxy_mtproto_config_proto_rawDesc
)

# 定义函数，用于对 file_proxy_mtproto_config_proto_rawDescData 进行 GZIP 压缩
func file_proxy_mtproto_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_proxy_mtproto_config_proto_rawDescOnce.Do(func() {
        file_proxy_mtproto_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_mtproto_config_proto_rawDescData)
    })
    # 返回 GZIP 压缩后的数据
    return file_proxy_mtproto_config_proto_rawDescData
}

# 定义变量，存储消息类型和 Go 类型
var file_proxy_mtproto_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
var file_proxy_mtproto_config_proto_goTypes = []interface{}{
    (*Account)(nil),       // 0: v2ray.core.proxy.mtproto.Account
    (*ServerConfig)(nil),  // 1: v2ray.core.proxy.mtproto.ServerConfig
    (*ClientConfig)(nil),  // 2: v2ray.core.proxy.mtproto.ClientConfig
    (*protocol.User)(nil), // 3: v2ray.core.common.protocol.User
}

# 定义变量，存储依赖索引
var file_proxy_mtproto_config_proto_depIdxs = []int32{
    3, // 0: v2ray.core.proxy.mtproto.ServerConfig.user:type_name -> v2ray.core.common.protocol.User
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

# 初始化函数，用于初始化 file_proxy_mtproto_config_proto
func init() { file_proxy_mtproto_config_proto_init() }
func file_proxy_mtproto_config_proto_init() {
    # 如果 File_proxy_mtproto_config_proto 已经被初始化，则直接返回
    if File_proxy_mtproto_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_mtproto_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Account); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_mtproto_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*ServerConfig); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_mtproto_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
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
            RawDescriptor: file_proxy_mtproto_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   3,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_mtproto_config_proto_goTypes,
        DependencyIndexes: file_proxy_mtproto_config_proto_depIdxs,
        MessageInfos:      file_proxy_mtproto_config_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_proxy_mtproto_config_proto
    File_proxy_mtproto_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_proxy_mtproto_config_proto_rawDesc = nil
    file_proxy_mtproto_config_proto_goTypes = nil
    file_proxy_mtproto_config_proto_depIdxs = nil
# 闭合前面的函数定义
```