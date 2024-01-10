# `v2ray-core\proxy\shadowsocks\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/shadowsocks/config.proto

package shadowsocks

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
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type CipherType int32

const (
    CipherType_UNKNOWN           CipherType = 0
    CipherType_AES_128_CFB       CipherType = 1
    CipherType_AES_256_CFB       CipherType = 2
    CipherType_CHACHA20          CipherType = 3
    CipherType_CHACHA20_IETF     CipherType = 4
    CipherType_AES_128_GCM       CipherType = 5
    CipherType_AES_256_GCM       CipherType = 6
    CipherType_CHACHA20_POLY1305 CipherType = 7
    CipherType_NONE              CipherType = 8
)

// CipherType 的枚举值映射。
var (
    CipherType_name = map[int32]string{
        0: "UNKNOWN",
        1: "AES_128_CFB",
        2: "AES_256_CFB",
        3: "CHACHA20",
        4: "CHACHA20_IETF",
        5: "AES_128_GCM",
        6: "AES_256_GCM",
        7: "CHACHA20_POLY1305",
        8: "NONE",
    }
    # 定义一个映射，将字符串类型的加密算法名称映射到整数类型的值
    CipherType_value = map[string]int32{
        "UNKNOWN":           0,  # 未知加密算法对应数值为0
        "AES_128_CFB":       1,  # AES 128位 CFB模式加密对应数值为1
        "AES_256_CFB":       2,  # AES 256位 CFB模式加密对应数值为2
        "CHACHA20":          3,  # CHACHA20加密对应数值为3
        "CHACHA20_IETF":     4,  # CHACHA20 IETF加密对应数值为4
        "AES_128_GCM":       5,  # AES 128位 GCM模式加密对应数值为5
        "AES_256_GCM":       6,  # AES 256位 GCM模式加密对应数值为6
        "CHACHA20_POLY1305": 7,  # CHACHA20_POLY1305加密对应数值为7
        "NONE":              8,  # 无加密对应数值为8
    }
// Enum 返回 CipherType 的指针
func (x CipherType) Enum() *CipherType {
    // 创建一个新的 CipherType 指针
    p := new(CipherType)
    // 将 x 的值赋给 p
    *p = x
    // 返回 p
    return p
}

// String 返回 CipherType 的字符串表示
func (x CipherType) String() string {
    // 返回 x 的枚举字符串表示
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

// Descriptor 返回 CipherType 的枚举描述符
func (CipherType) Descriptor() protoreflect.EnumDescriptor {
    // 返回文件中第一个枚举类型的描述符
    return file_proxy_shadowsocks_config_proto_enumTypes[0].Descriptor()
}

// Type 返回 CipherType 的枚举类型
func (CipherType) Type() protoreflect.EnumType {
    // 返回文件中第一个枚举类型
    return &file_proxy_shadowsocks_config_proto_enumTypes[0]
}

// Number 返回 CipherType 的枚举数字
func (x CipherType) Number() protoreflect.EnumNumber {
    // 返回 x 的枚举数字
    return protoreflect.EnumNumber(x)
}

// EnumDescriptor 返回 CipherType 的枚举描述符
// Deprecated: 使用 CipherType.Descriptor 替代
func (CipherType) EnumDescriptor() ([]byte, []int) {
    // 返回经过 GZIP 压缩的文件原始描述符和枚举类型索引
    return file_proxy_shadowsocks_config_proto_rawDescGZIP(), []int{0}
}

// Account 是一个包含密码和加密类型的结构体
type Account struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Password   string     `protobuf:"bytes,1,opt,name=password,proto3" json:"password,omitempty"`
    CipherType CipherType `protobuf:"varint,2,opt,name=cipher_type,json=cipherType,proto3,enum=v2ray.core.proxy.shadowsocks.CipherType" json:"cipher_type,omitempty"`
}

// Reset 重置 Account 结构体
func (x *Account) Reset() {
    // 将 x 重置为一个空的 Account 结构体
    *x = Account{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_shadowsocks_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// String 返回 Account 结构体的字符串表示
func (x *Account) String() string {
    // 返回 Account 结构体的字符串表示
    return protoimpl.X.MessageStringOf(x)
}

// ProtoMessage 是一个空函数，用于实现 proto.Message 接口
func (*Account) ProtoMessage() {}

// ProtoReflect 返回 Account 结构体的反射信息
func (x *Account) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_shadowsocks_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Descriptor 返回 Account 结构体的描述符
// Deprecated: 使用 Account.ProtoReflect.Descriptor 替代
func (*Account) Descriptor() ([]byte, []int) {
    // 返回文件中第一个消息类型的描述符
    return file_proxy_shadowsocks_config_proto_rawDescGZIP(), []int{0}
}
    # 返回 file_proxy_shadowsocks_config_proto_rawDescGZIP() 函数的结果和空列表
    return file_proxy_shadowsocks_config_proto_rawDescGZIP(), []int{0}
// 获取账户密码
func (x *Account) GetPassword() string {
    // 如果账户不为空，则返回密码
    if x != nil {
        return x.Password
    }
    // 否则返回空字符串
    return ""
}

// 获取加密类型
func (x *Account) GetCipherType() CipherType {
    // 如果账户不为空，则返回加密类型
    if x != nil {
        return x.CipherType
    }
    // 否则返回未知的加密类型
    return CipherType_UNKNOWN
}

// 服务器配置结构体
type ServerConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // UdpEnabled 指定是否启用 Shadowsocks 的 UDP
    // 已弃用。使用 'network' 字段。
    //
    // 已弃用：不要使用。
    UdpEnabled bool           `protobuf:"varint,1,opt,name=udp_enabled,json=udpEnabled,proto3" json:"udp_enabled,omitempty"`
    User       *protocol.User `protobuf:"bytes,2,opt,name=user,proto3" json:"user,omitempty"`
    Network    []net.Network  `protobuf:"varint,3,rep,packed,name=network,proto3,enum=v2ray.core.common.net.Network" json:"network,omitempty"`
}

// 重置服务器配置
func (x *ServerConfig) Reset() {
    *x = ServerConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_shadowsocks_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回服务器配置的字符串表示
func (x *ServerConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*ServerConfig) ProtoMessage() {}

// 返回服务器配置的反射信息
func (x *ServerConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_shadowsocks_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 ServerConfig.ProtoReflect.Descriptor 代替。
func (*ServerConfig) Descriptor() ([]byte, []int) {
    return file_proxy_shadowsocks_config_proto_rawDescGZIP(), []int{1}
}

// 已弃用：不要使用。
func (x *ServerConfig) GetUdpEnabled() bool {
    if x != nil {
        return x.UdpEnabled
    }
    return false
}
// 从 ServerConfig 结构体中获取 User 对象的方法
func (x *ServerConfig) GetUser() *protocol.User {
    // 如果 ServerConfig 对象不为空，则返回其 User 对象
    if x != nil {
        return x.User
    }
    // 否则返回空值
    return nil
}

// 从 ServerConfig 结构体中获取 Network 切片的方法
func (x *ServerConfig) GetNetwork() []net.Network {
    // 如果 ServerConfig 对象不为空，则返回其 Network 切片
    if x != nil {
        return x.Network
    }
    // 否则返回空值
    return nil
}

// ClientConfig 结构体定义
type ClientConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Server []*protocol.ServerEndpoint `protobuf:"bytes,1,rep,name=server,proto3" json:"server,omitempty"`
}

// 重置 ClientConfig 对象的方法
func (x *ClientConfig) Reset() {
    *x = ClientConfig{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_shadowsocks_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ClientConfig 对象的字符串表示方法
func (x *ClientConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*ClientConfig) ProtoMessage() {}

// 返回 ClientConfig 对象的反射信息
func (x *ClientConfig) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_shadowsocks_config_proto_msgTypes[2]
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
    return file_proxy_shadowsocks_config_proto_rawDescGZIP(), []int{2}
}

// 从 ClientConfig 结构体中获取 Server 切片的方法
func (x *ClientConfig) GetServer() []*protocol.ServerEndpoint {
    // 如果 ClientConfig 对象不为空，则返回其 Server 切片
    if x != nil {
        return x.Server
    }
    // 否则返回空值
    return nil
}

// 定义 File_proxy_shadowsocks_config_proto 和 file_proxy_shadowsocks_config_proto_rawDesc 变量
var File_proxy_shadowsocks_config_proto protoreflect.FileDescriptor

var file_proxy_shadowsocks_config_proto_rawDesc = []byte{
    0x0a, 0x1e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x73, 0x68, 0x61, 0x64, 0x6f, 0x77, 0x73, 0x6f,
    0x63, 0x6b, 0x73, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x12, 0x1c, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f,
}
    # 以下是一系列十六进制数，可能是某种数据的编码
    0x78, 0x79, 0x2e, 0x73, 0x68, 0x61, 0x64, 0x6f, 0x77, 0x73, 0x6f, 0x63, 0x6b, 0x73, 0x1a, 0x18,
    0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x6e, 0x65, 0x74, 0x77, 0x6f,
    0x72, 0x6b, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x1a, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e,
    0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2f, 0x75, 0x73, 0x65, 0x72, 0x2e, 0x70,
    0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f,
    0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x5f, 0x73, 0x70, 0x65,
    0x63, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x70, 0x0a, 0x07, 0x41, 0x63, 0x63, 0x6f, 0x75,
    0x6e, 0x74, 0x12, 0x1a, 0x0a, 0x08, 0x70, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x18, 0x01,
    0x20, 0x01, 0x28, 0x09, 0x52, 0x08, 0x70, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x12, 0x49,
    0x0a, 0x0b, 0x63, 0x69, 0x70, 0x68, 0x65, 0x72, 0x5f, 0x74, 0x79, 0x70, 0x65, 0x18, 0x02, 0x20,
    0x01, 0x28, 0x0e, 0x32, 0x28, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x73, 0x68, 0x61, 0x64, 0x6f, 0x77, 0x73, 0x6f, 0x63,
    0x6b, 0x73, 0x2e, 0x43, 0x69, 0x70, 0x68, 0x65, 0x72, 0x54, 0x79, 0x70, 0x65, 0x52, 0x0a, 0x63,
    0x69, 0x70, 0x68, 0x65, 0x72, 0x54, 0x79, 0x70, 0x65, 0x22, 0xa3, 0x01, 0x0a, 0x0c, 0x53, 0x65,
    0x72, 0x76, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x23, 0x0a, 0x0b, 0x75, 0x64,
    0x70, 0x5f, 0x65, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x42,
    0x02, 0x18, 0x01, 0x52, 0x0a, 0x75, 0x64, 0x70, 0x45, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x12,
    0x34, 0x0a, 0x04, 0x75, 0x73, 0x65, 0x72, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x20, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f,
    0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52,
    # 以十六进制表示的字节码序列
    0x04, 0x75, 0x73, 0x65, 0x72, 0x12, 0x38, 0x0a, 0x07, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b,
    0x18, 0x03, 0x20, 0x03, 0x28, 0x0e, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e,
    0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x52, 0x07, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x22,
    0x52, 0x0a, 0x0c, 0x43, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
    0x42, 0x0a, 0x06, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32,
    0x2a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d,
    0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x72,
    0x76, 0x65, 0x72, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x06, 0x73, 0x65, 0x72,
    0x76, 0x65, 0x72, 0x2a, 0x9f, 0x01, 0x0a, 0x0a, 0x43, 0x69, 0x70, 0x68, 0x65, 0x72, 0x54, 0x79,
    0x70, 0x65, 0x12, 0x0b, 0x0a, 0x07, 0x55, 0x4e, 0x4b, 0x4e, 0x4f, 0x57, 0x4e, 0x10, 0x00, 0x12,
    0x0f, 0x0a, 0x0b, 0x41, 0x45, 0x53, 0x5f, 0x31, 0x32, 0x38, 0x5f, 0x43, 0x46, 0x42, 0x10, 0x01,
    0x12, 0x0f, 0x0a, 0x0b, 0x41, 0x45, 0x53, 0x5f, 0x32, 0x35, 0x36, 0x5f, 0x43, 0x46, 0x42, 0x10,
    0x02, 0x12, 0x0c, 0x0a, 0x08, 0x43, 0x48, 0x41, 0x43, 0x48, 0x41, 0x32, 0x30, 0x10, 0x03, 0x12,
    0x11, 0x0a, 0x0d, 0x43, 0x48, 0x41, 0x43, 0x48, 0x41, 0x32, 0x30, 0x5f, 0x49, 0x45, 0x54, 0x46,
    0x10, 0x04, 0x12, 0x0f, 0x0a, 0x0b, 0x41, 0x45, 0x53, 0x5f, 0x31, 0x32, 0x38, 0x5f, 0x47, 0x43,
    0x4d, 0x10, 0x05, 0x12, 0x0f, 0x0a, 0x0b, 0x41, 0x45, 0x53, 0x5f, 0x32, 0x35, 0x36, 0x5f, 0x47,
    0x43, 0x4d, 0x10, 0x06, 0x12, 0x15, 0x0a, 0x11, 0x43, 0x48, 0x41, 0x43, 0x48, 0x41, 0x32, 0x30,
    0x5f, 0x50, 0x4f, 0x4c, 0x59, 0x31, 0x33, 0x30, 0x35, 0x10, 0x07, 0x12, 0x08, 0x0a, 0x04, 0x4e,
    0x4f, 0x4e, 0x45, 0x10, 0x08, 0x42, 0x65, 0x0a, 0x20, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72,
    # 以十六进制表示的数据，可能是某种编码或者密文
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x73, 0x68,
    0x61, 0x64, 0x6f, 0x77, 0x73, 0x6f, 0x63, 0x6b, 0x73, 0x50, 0x01, 0x5a, 0x20, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72, 0x6f, 0x78,
    0x79, 0x2f, 0x73, 0x68, 0x61, 0x64, 0x6f, 0x77, 0x73, 0x6f, 0x63, 0x6b, 0x73, 0xaa, 0x02, 0x1c,
    0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79,
    0x2e, 0x53, 0x68, 0x61, 0x64, 0x6f, 0x77, 0x73, 0x6f, 0x63, 0x6b, 0x73, 0x62, 0x06, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保只执行一次初始化操作
var (
    file_proxy_shadowsocks_config_proto_rawDescOnce sync.Once
    file_proxy_shadowsocks_config_proto_rawDescData = file_proxy_shadowsocks_config_proto_rawDesc
)

// 对原始描述数据进行 GZIP 压缩
func file_proxy_shadowsocks_config_proto_rawDescGZIP() []byte {
    // 确保只执行一次 GZIP 压缩操作
    file_proxy_shadowsocks_config_proto_rawDescOnce.Do(func() {
        file_proxy_shadowsocks_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_shadowsocks_config_proto_rawDescData)
    })
    return file_proxy_shadowsocks_config_proto_rawDescData
}

// 定义枚举类型信息的切片
var file_proxy_shadowsocks_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
// 定义消息类型信息的切片
var file_proxy_shadowsocks_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
// 定义 Go 类型的切片
var file_proxy_shadowsocks_config_proto_goTypes = []interface{}{
    (CipherType)(0),                 // 0: v2ray.core.proxy.shadowsocks.CipherType
    (*Account)(nil),                 // 1: v2ray.core.proxy.shadowsocks.Account
    (*ServerConfig)(nil),            // 2: v2ray.core.proxy.shadowsocks.ServerConfig
    (*ClientConfig)(nil),            // 3: v2ray.core.proxy.shadowsocks.ClientConfig
    (*protocol.User)(nil),           // 4: v2ray.core.common.protocol.User
    (net.Network)(0),                // 5: v2ray.core.common.net.Network
    (*protocol.ServerEndpoint)(nil), // 6: v2ray.core.common.protocol.ServerEndpoint
}
// 定义依赖索引的切片
var file_proxy_shadowsocks_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.proxy.shadowsocks.Account.cipher_type:type_name -> v2ray.core.proxy.shadowsocks.CipherType
    4, // 1: v2ray.core.proxy.shadowsocks.ServerConfig.user:type_name -> v2ray.core.common.protocol.User
    5, // 2: v2ray.core.proxy.shadowsocks.ServerConfig.network:type_name -> v2ray.core.common.net.Network
    6, // 3: v2ray.core.proxy.shadowsocks.ClientConfig.server:type_name -> v2ray.core.common.protocol.ServerEndpoint
    4, // [4:4] is the sub-list for method output_type
    4, // [4:4] is the sub-list for method input_type
    4, // [4:4] is the sub-list for extension type_name
    4, // 表示从索引4开始，取到索引4，即取出索引为4的元素
    0, // 表示从索引0开始，取到索引4，即取出索引为0的元素到索引为3的元素
func init() { file_proxy_shadowsocks_config_proto_init() }
func file_proxy_shadowsocks_config_proto_init() {
    // 如果文件代理影子袜子配置协议不为空，则直接返回
    if File_proxy_shadowsocks_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则直接返回
    if !protoimpl.UnsafeEnabled {
        // 设置消息类型的导出器函数
        file_proxy_shadowsocks_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
        }
        file_proxy_shadowsocks_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
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
        }
        file_proxy_shadowsocks_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
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
        }
    }
    // 定义一个空结构体 x
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性，包括 GoPackagePath、RawDescriptor、NumEnums、NumMessages、NumExtensions 和 NumServices
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),  # 获取类型 x 的包路径
            RawDescriptor: file_proxy_shadowsocks_config_proto_rawDesc,  # 设置原始描述符
            NumEnums:      1,  # 设置枚举类型数量
            NumMessages:   3,  # 设置消息类型数量
            NumExtensions: 0,  # 设置扩展数量
            NumServices:   0,  # 设置服务数量
        },
        # 设置 GoTypes 属性
        GoTypes:           file_proxy_shadowsocks_config_proto_goTypes,
        # 设置 DependencyIndexes 属性
        DependencyIndexes: file_proxy_shadowsocks_config_proto_depIdxs,
        # 设置 EnumInfos 属性
        EnumInfos:         file_proxy_shadowsocks_config_proto_enumTypes,
        # 设置 MessageInfos 属性
        MessageInfos:      file_proxy_shadowsocks_config_proto_msgTypes,
    }.Build()  # 调用 Build 方法构建对象
    # 将构建好的 File 对象赋值给 File_proxy_shadowsocks_config_proto
    File_proxy_shadowsocks_config_proto = out.File
    # 将原始描述符设置为 nil
    file_proxy_shadowsocks_config_proto_rawDesc = nil
    # 将 GoTypes 设置为 nil
    file_proxy_shadowsocks_config_proto_goTypes = nil
    # 将 DependencyIndexes 设置为 nil
    file_proxy_shadowsocks_config_proto_depIdxs = nil
# 闭合前面的函数定义
```