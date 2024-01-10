# `v2ray-core\proxy\socks\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/socks/config.proto

package socks

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    net "v2ray.com/core/common/net"  // 导入 net 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    // 确保此生成的代码足够更新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够更新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保正在使用足够更新的旧 proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// AuthType 是 Socks 代理的认证类型。
type AuthType int32

const (
    // NO_AUTH 用于匿名认证。
    AuthType_NO_AUTH AuthType = 0
    // PASSWORD 用于用户名/密码认证。
    AuthType_PASSWORD AuthType = 1
)

// AuthType 的枚举值映射。
var (
    AuthType_name = map[int32]string{
        0: "NO_AUTH",
        1: "PASSWORD",
    }
    AuthType_value = map[string]int32{
        "NO_AUTH":  0,
        "PASSWORD": 1,
    }
)

func (x AuthType) Enum() *AuthType {
    p := new(AuthType)
    *p = x
    return p
}

func (x AuthType) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (AuthType) Descriptor() protoreflect.EnumDescriptor {
    return file_proxy_socks_config_proto_enumTypes[0].Descriptor()
}

func (AuthType) Type() protoreflect.EnumType {
    return &file_proxy_socks_config_proto_enumTypes[0]
}

func (x AuthType) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// 已弃用: 使用 AuthType.Descriptor 代替。
// 返回代理配置文件的原始描述信息，并使用 GZIP 压缩
func (AuthType) EnumDescriptor() ([]byte, []int) {
    return file_proxy_socks_config_proto_rawDescGZIP(), []int{0}
}

// Account 表示一个 Socks 账户
type Account struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Username string `protobuf:"bytes,1,opt,name=username,proto3" json:"username,omitempty"`  // 用户名
    Password string `protobuf:"bytes,2,opt,name=password,proto3" json:"password,omitempty"`  // 密码
}

// 重置 Account 对象
func (x *Account) Reset() {
    *x = Account{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_socks_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Account 对象的字符串表示
func (x *Account) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Account) ProtoMessage() {}

// 返回 Account 对象的反射信息
func (x *Account) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_socks_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Account.ProtoReflect.Descriptor instead.
// 返回 Account 对象的描述信息
func (*Account) Descriptor() ([]byte, []int) {
    return file_proxy_socks_config_proto_rawDescGZIP(), []int{0}
}

// 获取 Account 对象的用户名
func (x *Account) GetUsername() string {
    if x != nil {
        return x.Username
    }
    return ""
}

// 获取 Account 对象的密码
func (x *Account) GetPassword() string {
    if x != nil {
        return x.Password
    }
    return ""
}

// ServerConfig 是 Socks 服务器的 protobuf 配置
type ServerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    AuthType   AuthType          `protobuf:"varint,1,opt,name=auth_type,json=authType,proto3,enum=v2ray.core.proxy.socks.AuthType" json:"auth_type,omitempty"`  // 认证类型
    # 用于存储账户信息的映射，键为字符串类型，值为字符串类型
    Accounts   map[string]string `protobuf:"bytes,2,rep,name=accounts,proto3" json:"accounts,omitempty" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
    # 用于存储 IP 地址或域名的对象指针
    Address    *net.IPOrDomain   `protobuf:"bytes,3,opt,name=address,proto3" json:"address,omitempty"`
    # 用于表示是否启用 UDP 的布尔类型变量
    UdpEnabled bool              `protobuf:"varint,4,opt,name=udp_enabled,json=udpEnabled,proto3" json:"udp_enabled,omitempty"`
    # 已废弃的字段，不建议使用
    Timeout   uint32 `protobuf:"varint,5,opt,name=timeout,proto3" json:"timeout,omitempty"`
    # 用户级别的整数值
    UserLevel uint32 `protobuf:"varint,6,opt,name=user_level,json=userLevel,proto3" json:"user_level,omitempty"`
// 重置 ServerConfig 对象为默认值
func (x *ServerConfig) Reset() {
    *x = ServerConfig{}
    // 如果启用了不安全操作，存储消息类型信息
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_socks_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ServerConfig 对象的字符串表示
func (x *ServerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ServerConfig) ProtoMessage() {}

// 返回 ServerConfig 对象的反射信息
func (x *ServerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_socks_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用 ServerConfig.ProtoReflect.Descriptor 替代
func (*ServerConfig) Descriptor() ([]byte, []int) {
    return file_proxy_socks_config_proto_rawDescGZIP(), []int{1}
}

// 返回 ServerConfig 对象的 AuthType 属性
func (x *ServerConfig) GetAuthType() AuthType {
    if x != nil {
        return x.AuthType
    }
    return AuthType_NO_AUTH
}

// 返回 ServerConfig 对象的 Accounts 属性
func (x *ServerConfig) GetAccounts() map[string]string {
    if x != nil {
        return x.Accounts
    }
    return nil
}

// 返回 ServerConfig 对象的 Address 属性
func (x *ServerConfig) GetAddress() *net.IPOrDomain {
    if x != nil {
        return x.Address
    }
    return nil
}

// 返回 ServerConfig 对象的 UdpEnabled 属性
func (x *ServerConfig) GetUdpEnabled() bool {
    if x != nil {
        return x.UdpEnabled
    }
    return false
}

// Deprecated: 不要使用
// 返回 ServerConfig 对象的 Timeout 属性
func (x *ServerConfig) GetTimeout() uint32 {
    if x != nil {
        return x.Timeout
    }
    return 0
}

// 返回 ServerConfig 对象的 UserLevel 属性
func (x *ServerConfig) GetUserLevel() uint32 {
    if x != nil {
        return x.UserLevel
    }
    return 0
}

// ClientConfig 是 Socks 客户端的 protobuf 配置
type ClientConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Sever 是 Socks 服务器地址列表
    # 定义一个名为Server的指针数组，用于存储protocol.ServerEndpoint类型的数据
    Server []*protocol.ServerEndpoint `protobuf:"bytes,1,rep,name=server,proto3" json:"server,omitempty"`
// 重置 ClientConfig 对象
func (x *ClientConfig) Reset() {
    // 将 ClientConfig 对象重置为空对象
    *x = ClientConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型
        mi := &file_proxy_socks_config_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
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
    // 获取消息类型
    mi := &file_proxy_socks_config_proto_msgTypes[2]
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

// 已弃用：使用 ClientConfig.ProtoReflect.Descriptor 替代
func (*ClientConfig) Descriptor() ([]byte, []int) {
    return file_proxy_socks_config_proto_rawDescGZIP(), []int{2}
}

// 获取 ClientConfig 对象的 Server 字段
func (x *ClientConfig) GetServer() []*protocol.ServerEndpoint {
    // 如果对象不为空，则返回 Server 字段
    if x != nil {
        return x.Server
    }
    return nil
}

// 定义 File_proxy_socks_config_proto 和 file_proxy_socks_config_proto_rawDesc
var File_proxy_socks_config_proto protoreflect.FileDescriptor

var file_proxy_socks_config_proto_rawDesc = []byte{
    // 以下为字节数据，省略注释
    # 创建一个十六进制数值列表
    0x65, 0x72, 0x6e, 0x61, 0x6d, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x08, 0x75, 0x73,
    0x65, 0x72, 0x6e, 0x61, 0x6d, 0x65, 0x12, 0x1a, 0x0a, 0x08, 0x70, 0x61, 0x73, 0x73, 0x77, 0x6f,
    0x72, 0x64, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x08, 0x70, 0x61, 0x73, 0x73, 0x77, 0x6f,
    0x72, 0x64, 0x22, 0xf5, 0x02, 0x0a, 0x0c, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x43, 0x6f, 0x6e,
    0x66, 0x69, 0x67, 0x12, 0x3d, 0x0a, 0x09, 0x61, 0x75, 0x74, 0x68, 0x5f, 0x74, 0x79, 0x70, 0x65,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x73, 0x6f, 0x63, 0x6b, 0x73, 0x2e,
    0x41, 0x75, 0x74, 0x68, 0x54, 0x79, 0x70, 0x65, 0x52, 0x08, 0x61, 0x75, 0x74, 0x68, 0x54, 0x79,
    0x70, 0x65, 0x12, 0x4e, 0x0a, 0x08, 0x61, 0x63, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x73, 0x18, 0x02,
    0x20, 0x03, 0x28, 0x0b, 0x32, 0x32, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x73, 0x6f, 0x63, 0x6b, 0x73, 0x2e, 0x53, 0x65,
    0x72, 0x76, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x41, 0x63, 0x63, 0x6f, 0x75,
    0x6e, 0x74, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x52, 0x08, 0x61, 0x63, 0x63, 0x6f, 0x75, 0x6e,
    0x74, 0x73, 0x12, 0x3b, 0x0a, 0x07, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x18, 0x03, 0x20,
    0x01, 0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x49, 0x50, 0x4f, 0x72,
    0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x07, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x12,
    0x1f, 0x0a, 0x0b, 0x75, 0x64, 0x70, 0x5f, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x18, 0x04,
    0x20, 0x01, 0x28, 0x08, 0x52, 0x0a, 0x75, 0x64, 0x70, 0x45, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64,
    0x12, 0x1c, 0x0a, 0x07, 0x74, 0x69, 0x6d, 0x65, 0x6f, 0x75, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28,
    # 以十六进制表示的字节码序列
    0x0d, 0x42, 0x02, 0x18, 0x01, 0x52, 0x07, 0x74, 0x69, 0x6d, 0x65, 0x6f, 0x75, 0x74, 0x12, 0x1d,
    0x0a, 0x0a, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x6c, 0x65, 0x76, 0x65, 0x6c, 0x18, 0x06, 0x20, 0x01,
    0x28, 0x0d, 0x52, 0x09, 0x75, 0x73, 0x65, 0x72, 0x4c, 0x65, 0x76, 0x65, 0x6c, 0x1a, 0x3b, 0x0a,
    0x0d, 0x41, 0x63, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x12, 0x10,
    0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79,
    0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52,
    0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x3a, 0x02, 0x38, 0x01, 0x22, 0x52, 0x0a, 0x0c, 0x43, 0x6c,
    0x69, 0x65, 0x6e, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x42, 0x0a, 0x06, 0x73, 0x65,
    0x72, 0x76, 0x65, 0x72, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x2a, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70,
    0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x45, 0x6e,
    0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x06, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x2a, 0x25,
    0x0a, 0x08, 0x41, 0x75, 0x74, 0x68, 0x54, 0x79, 0x70, 0x65, 0x12, 0x0b, 0x0a, 0x07, 0x4e, 0x4f,
    0x5f, 0x41, 0x55, 0x54, 0x48, 0x10, 0x00, 0x12, 0x0c, 0x0a, 0x08, 0x50, 0x41, 0x53, 0x53, 0x57,
    0x4f, 0x52, 0x44, 0x10, 0x01, 0x42, 0x53, 0x0a, 0x1a, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x73, 0x6f,
    0x63, 0x6b, 0x73, 0x50, 0x01, 0x5a, 0x1a, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d,
    0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x73, 0x6f, 0x63, 0x6b,
    0x73, 0xaa, 0x02, 0x16, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50,
    0x72, 0x6f, 0x78, 0x79, 0x2e, 0x53, 0x6f, 0x63, 0x6b, 0x73, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    # 这是十六进制数，表示为0x6f和0x33
// 定义一个全局变量，用于确保 file_proxy_socks_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_socks_config_proto_rawDescOnce sync.Once
    file_proxy_socks_config_proto_rawDescData = file_proxy_socks_config_proto_rawDesc
)

// 定义一个函数，用于对 file_proxy_socks_config_proto_rawDescData 进行 GZIP 压缩
func file_proxy_socks_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_proxy_socks_config_proto_rawDescOnce.Do(func() {
        file_proxy_socks_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_socks_config_proto_rawDescData)
    })
    return file_proxy_socks_config_proto_rawDescData
}

// 定义枚举类型的信息数组
var file_proxy_socks_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
// 定义消息类型的信息数组
var file_proxy_socks_config_proto_msgTypes = make([]protoimpl.MessageInfo, 4)
// 定义 Go 语言类型的数组
var file_proxy_socks_config_proto_goTypes = []interface{}{
    (AuthType)(0),                   // 0: v2ray.core.proxy.socks.AuthType
    (*Account)(nil),                 // 1: v2ray.core.proxy.socks.Account
    (*ServerConfig)(nil),            // 2: v2ray.core.proxy.socks.ServerConfig
    (*ClientConfig)(nil),            // 3: v2ray.core.proxy.socks.ClientConfig
    nil,                             // 4: v2ray.core.proxy.socks.ServerConfig.AccountsEntry
    (*net.IPOrDomain)(nil),          // 5: v2ray.core.common.net.IPOrDomain
    (*protocol.ServerEndpoint)(nil), // 6: v2ray.core.common.protocol.ServerEndpoint
}
// 定义依赖索引的数组
var file_proxy_socks_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.proxy.socks.ServerConfig.auth_type:type_name -> v2ray.core.proxy.socks.AuthType
    4, // 1: v2ray.core.proxy.socks.ServerConfig.accounts:type_name -> v2ray.core.proxy.socks.ServerConfig.AccountsEntry
    5, // 2: v2ray.core.proxy.socks.ServerConfig.address:type_name -> v2ray.core.common.net.IPOrDomain
    6, // 3: v2ray.core.proxy.socks.ClientConfig.server:type_name -> v2ray.core.common.protocol.ServerEndpoint
    4, // [4:4] 是输出类型的子列表
    4, // [4:4] 是输入类型的子列表
    4, // [4:4] 是扩展类型的子列表
    4, // [4:4] 是扩展对象的子列表
    0, // [0:4] 是字段类型的子列表
}
func init() { file_proxy_socks_config_proto_init() }  // 初始化函数，调用 file_proxy_socks_config_proto_init() 函数
func file_proxy_socks_config_proto_init() {  // 定义 file_proxy_socks_config_proto_init() 函数
    if File_proxy_socks_config_proto != nil {  // 如果 File_proxy_socks_config_proto 不为空，则返回
        return
    }
    if !protoimpl.UnsafeEnabled {  // 如果 protoimpl.UnsafeEnabled 为假
        file_proxy_socks_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {  // 设置消息类型 0 的导出函数
            switch v := v.(*Account); i {  // 判断消息类型
            case 0:  // 如果为 0
                return &v.state  // 返回状态
            case 1:  // 如果为 1
                return &v.sizeCache  // 返回大小缓存
            case 2:  // 如果为 2
                return &v.unknownFields  // 返回未知字段
            default:  // 默认情况
                return nil  // 返回空
            }
        }
        file_proxy_socks_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {  // 设置消息类型 1 的导出函数
            switch v := v.(*ServerConfig); i {  // 判断消息类型
            case 0:  // 如果为 0
                return &v.state  // 返回状态
            case 1:  // 如果为 1
                return &v.sizeCache  // 返回大小缓存
            case 2:  // 如果为 2
                return &v.unknownFields  // 返回未知字段
            default:  // 默认情况
                return nil  // 返回空
            }
        }
        file_proxy_socks_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {  // 设置消息类型 2 的导出函数
            switch v := v.(*ClientConfig); i {  // 判断消息类型
            case 0:  // 如果为 0
                return &v.state  // 返回状态
            case 1:  // 如果为 1
                return &v.sizeCache  // 返回大小缓存
            case 2:  // 如果为 2
                return &v.unknownFields  // 返回未知字段
            default:  // 默认情况
                return nil  // 返回空
            }
        }
    }
    type x struct{}  // 定义空结构体 x
    out := protoimpl.TypeBuilder{  // 创建 protoimpl.TypeBuilder 结构体
        File: protoimpl.DescBuilder{  // 设置文件描述
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),  // 设置 Go 包路径
            RawDescriptor: file_proxy_socks_config_proto_rawDesc,  // 设置原始描述
            NumEnums:      1,  // 设置枚举数量
            NumMessages:   4,  // 设置消息数量
            NumExtensions: 0,  // 设置扩展数量
            NumServices:   0,  // 设置服务数量
        },
        GoTypes:           file_proxy_socks_config_proto_goTypes,  // 设置 Go 类型
        DependencyIndexes: file_proxy_socks_config_proto_depIdxs,  // 设置依赖索引
        EnumInfos:         file_proxy_socks_config_proto_enumTypes,  // 设置枚举信息
        MessageInfos:      file_proxy_socks_config_proto_msgTypes,  // 设置消息信息
    }.Build()  // 构建
}
    # 将 out.File 赋值给 File_proxy_socks_config_proto
    File_proxy_socks_config_proto = out.File
    # 将 file_proxy_socks_config_proto_rawDesc 初始化为 nil
    file_proxy_socks_config_proto_rawDesc = nil
    # 将 file_proxy_socks_config_proto_goTypes 初始化为 nil
    file_proxy_socks_config_proto_goTypes = nil
    # 将 file_proxy_socks_config_proto_depIdxs 初始化为 nil
    file_proxy_socks_config_proto_depIdxs = nil
# 闭合前面的函数定义
```