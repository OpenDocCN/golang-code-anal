# `v2ray-core\transport\internet\xtls\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/xtls/config.proto

package xtls

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

type Certificate_Usage int32

const (
    Certificate_ENCIPHERMENT     Certificate_Usage = 0
    Certificate_AUTHORITY_VERIFY Certificate_Usage = 1
    Certificate_AUTHORITY_ISSUE  Certificate_Usage = 2
)

// 证书用途的枚举值映射。
var (
    Certificate_Usage_name = map[int32]string{
        0: "ENCIPHERMENT",
        1: "AUTHORITY_VERIFY",
        2: "AUTHORITY_ISSUE",
    }
    Certificate_Usage_value = map[string]int32{
        "ENCIPHERMENT":     0,
        "AUTHORITY_VERIFY": 1,
        "AUTHORITY_ISSUE":  2,
    }
)

func (x Certificate_Usage) Enum() *Certificate_Usage {
    p := new(Certificate_Usage)
    *p = x
    return p
}

func (x Certificate_Usage) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Certificate_Usage) Descriptor() protoreflect.EnumDescriptor {
    return file_transport_internet_xtls_config_proto_enumTypes[0].Descriptor()
}

func (Certificate_Usage) Type() protoreflect.EnumType {
    return &file_transport_internet_xtls_config_proto_enumTypes[0]
}

func (x Certificate_Usage) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}
// Deprecated: Use Certificate_Usage.Descriptor instead.
// 返回枚举类型的描述符
func (Certificate_Usage) EnumDescriptor() ([]byte, []int) {
    return file_transport_internet_xtls_config_proto_rawDescGZIP(), []int{0, 0}
}

// Certificate 结构体定义
type Certificate struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // XTLS 证书的 x509 格式
    Certificate []byte `protobuf:"bytes,1,opt,name=Certificate,proto3" json:"Certificate,omitempty"`
    // XTLS 密钥的 x509 格式
    Key   []byte            `protobuf:"bytes,2,opt,name=Key,proto3" json:"Key,omitempty"`
    // 证书的用途
    Usage Certificate_Usage `protobuf:"varint,3,opt,name=usage,proto3,enum=v2ray.core.transport.internet.xtls.Certificate_Usage" json:"usage,omitempty"`
}

// 重置 Certificate 结构体
func (x *Certificate) Reset() {
    *x = Certificate{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_xtls_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Certificate 结构体的字符串表示
func (x *Certificate) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Certificate) ProtoMessage() {}

// 返回 Certificate 结构体的反射信息
func (x *Certificate) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_xtls_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Certificate.ProtoReflect.Descriptor instead.
// 返回结构体的描述符
func (*Certificate) Descriptor() ([]byte, []int) {
    return file_transport_internet_xtls_config_proto_rawDescGZIP(), []int{0}
}

// 获取 Certificate 结构体中的 Certificate 字段
func (x *Certificate) GetCertificate() []byte {
    if x != nil {
        return x.Certificate
    }
    return nil
}

// 获取 Certificate 结构体中的 Key 字段
func (x *Certificate) GetKey() []byte {
    if x != nil {
        return x.Key
    }
    return nil
}

// 获取 Certificate 结构体中的 Usage 字段
func (x *Certificate) GetUsage() Certificate_Usage {
    # 如果 x 不为空，则返回 x 的 Usage 属性
    if x != nil:
        return x.Usage
    # 如果 x 为空，则返回 Certificate_ENCIPHERMENT
    return Certificate_ENCIPHERMENT
// Config 结构体定义，包含了状态、大小缓存和未知字段
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 是否允许使用自签名证书
    AllowInsecure bool `protobuf:"varint,1,opt,name=allow_insecure,json=allowInsecure,proto3" json:"allow_insecure,omitempty"`
    // 是否允许使用不安全的密码套件
    AllowInsecureCiphers bool `protobuf:"varint,5,opt,name=allow_insecure_ciphers,json=allowInsecureCiphers,proto3" json:"allow_insecure_ciphers,omitempty"`
    // 服务器上要提供的证书列表
    Certificate []*Certificate `protobuf:"bytes,2,rep,name=certificate,proto3" json:"certificate,omitempty"`
    // 覆盖服务器名称
    ServerName string `protobuf:"bytes,3,opt,name=server_name,json=serverName,proto3" json:"server_name,omitempty"`
    // ALPN 值的字符串列表
    NextProtocol []string `protobuf:"bytes,4,rep,name=next_protocol,json=nextProtocol,proto3" json:"next_protocol,omitempty"`
    // 是否禁用会话（票据）恢复
    DisableSessionResumption bool `protobuf:"varint,6,opt,name=disable_session_resumption,json=disableSessionResumption,proto3" json:"disable_session_resumption,omitempty"`
    // 如果为 true，则不会加载系统上的根证书进行验证
    DisableSystemRoot bool `protobuf:"varint,7,opt,name=disable_system_root,json=disableSystemRoot,proto3" json:"disable_system_root,omitempty"`
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_xtls_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Config 结构体的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回 Config 结构体的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_xtls_config_proto_msgTypes[1]
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
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 返回配置的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_xtls_config_proto_rawDescGZIP(), []int{1}
}

// 返回是否允许不安全连接
func (x *Config) GetAllowInsecure() bool {
    if x != nil {
        return x.AllowInsecure
    }
    return false
}

// 返回是否允许使用不安全密码
func (x *Config) GetAllowInsecureCiphers() bool {
    if x != nil {
        return x.AllowInsecureCiphers
    }
    return false
}

// 返回证书列表
func (x *Config) GetCertificate() []*Certificate {
    if x != nil {
        return x.Certificate
    }
    return nil
}

// 返回服务器名称
func (x *Config) GetServerName() string {
    if x != nil {
        return x.ServerName
    }
    return ""
}

// 返回下一个协议列表
func (x *Config) GetNextProtocol() []string {
    if x != nil {
        return x.NextProtocol
    }
    return nil
}

// 返回是否禁用会话恢复
func (x *Config) GetDisableSessionResumption() bool {
    if x != nil {
        return x.DisableSessionResumption
    }
    return false
}

// 返回是否禁用系统根证书
func (x *Config) GetDisableSystemRoot() bool {
    if x != nil {
        return x.DisableSystemRoot
    }
    return false
}

// 定义文件描述符
var File_transport_internet_xtls_config_proto protoreflect.FileDescriptor

// 定义原始描述符的字节流
var file_transport_internet_xtls_config_proto_rawDesc = []byte{
    // 这里是一段很长的字节流，省略部分内容
}
    # 创建一个十六进制数列表
    0x4b, 0x65, 0x79, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0c, 0x52, 0x03, 0x4b, 0x65, 0x79, 0x12, 0x4b,
    0x0a, 0x05, 0x75, 0x73, 0x61, 0x67, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x35, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73,
    0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x78, 0x74,
    0x6c, 0x73, 0x2e, 0x43, 0x65, 0x72, 0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x2e, 0x55,
    0x73, 0x61, 0x67, 0x65, 0x52, 0x05, 0x75, 0x73, 0x61, 0x67, 0x65, 0x22, 0x44, 0x0a, 0x05, 0x55,
    0x73, 0x61, 0x67, 0x65, 0x12, 0x10, 0x0a, 0x0c, 0x45, 0x4e, 0x43, 0x49, 0x50, 0x48, 0x45, 0x52,
    0x4d, 0x45, 0x4e, 0x54, 0x10, 0x00, 0x12, 0x14, 0x0a, 0x10, 0x41, 0x55, 0x54, 0x48, 0x4f, 0x52,
    0x49, 0x54, 0x59, 0x5f, 0x56, 0x45, 0x52, 0x49, 0x46, 0x59, 0x10, 0x01, 0x12, 0x13, 0x0a, 0x0f,
    0x41, 0x55, 0x54, 0x48, 0x4f, 0x52, 0x49, 0x54, 0x59, 0x5f, 0x49, 0x53, 0x53, 0x55, 0x45, 0x10,
    0x02, 0x22, 0xec, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x25, 0x0a, 0x0e,
    0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x5f, 0x69, 0x6e, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x18, 0x01,
    0x20, 0x01, 0x28, 0x08, 0x52, 0x0d, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x49, 0x6e, 0x73, 0x65, 0x63,
    0x75, 0x72, 0x65, 0x12, 0x34, 0x0a, 0x16, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x5f, 0x69, 0x6e, 0x73,
    0x65, 0x63, 0x75, 0x72, 0x65, 0x5f, 0x63, 0x69, 0x70, 0x68, 0x65, 0x72, 0x73, 0x18, 0x05, 0x20,
    0x01, 0x28, 0x08, 0x52, 0x14, 0x61, 0x6c, 0x6c, 0x6f, 0x77, 0x49, 0x6e, 0x73, 0x65, 0x63, 0x75,
    0x72, 0x65, 0x43, 0x69, 0x70, 0x68, 0x65, 0x72, 0x73, 0x12, 0x51, 0x0a, 0x0b, 0x63, 0x65, 0x72,
    0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x2f,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e,
    0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x78,
    # 这是一个十六进制的字节流，可能是某种数据的编码表示
    0x74, 0x6c, 0x73, 0x2e, 0x43, 0x65, 0x72, 0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x52,
    0x0b, 0x63, 0x65, 0x72, 0x74, 0x69, 0x66, 0x69, 0x63, 0x61, 0x74, 0x65, 0x12, 0x1f, 0x0a, 0x0b,
    0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x5f, 0x6e, 0x61, 0x6d, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28,
    0x09, 0x52, 0x0a, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x4e, 0x61, 0x6d, 0x65, 0x12, 0x23, 0x0a,
    0x0d, 0x6e, 0x65, 0x78, 0x74, 0x5f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x04,
    0x20, 0x03, 0x28, 0x09, 0x52, 0x0c, 0x6e, 0x65, 0x78, 0x74, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63,
    0x6f, 0x6c, 0x12, 0x3c, 0x0a, 0x1a, 0x64, 0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x5f, 0x73, 0x65,
    0x73, 0x73, 0x69, 0x6f, 0x6e, 0x5f, 0x72, 0x65, 0x73, 0x75, 0x6d, 0x70, 0x74, 0x69, 0x6f, 0x6e,
    0x18, 0x06, 0x20, 0x01, 0x28, 0x08, 0x52, 0x18, 0x64, 0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x53,
    0x65, 0x73, 0x73, 0x69, 0x6f, 0x6e, 0x52, 0x65, 0x73, 0x75, 0x6d, 0x70, 0x74, 0x69, 0x6f, 0x6e,
    0x12, 0x2e, 0x0a, 0x13, 0x64, 0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x5f, 0x73, 0x79, 0x73, 0x74,
    0x65, 0x6d, 0x5f, 0x72, 0x6f, 0x6f, 0x74, 0x18, 0x07, 0x20, 0x01, 0x28, 0x08, 0x52, 0x11, 0x64,
    0x69, 0x73, 0x61, 0x62, 0x6c, 0x65, 0x53, 0x79, 0x73, 0x74, 0x65, 0x6d, 0x52, 0x6f, 0x6f, 0x74,
    # 这里可能是一些编码的数据，需要进一步处理才能理解其含义
    # 这些是十六进制数，可能是表示某种数据或者编码
    # 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x58, 0x74, 0x6c, 0x73, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义一个同步锁，确保只有一个goroutine执行初始化操作
var (
    file_transport_internet_xtls_config_proto_rawDescOnce sync.Once
    // 定义一个变量用于存储原始描述数据
    file_transport_internet_xtls_config_proto_rawDescData = file_transport_internet_xtls_config_proto_rawDesc
)

// 定义一个函数，用于对原始描述数据进行GZIP压缩
func file_transport_internet_xtls_config_proto_rawDescGZIP() []byte {
    // 使用sync.Once确保只有一个goroutine执行压缩操作
    file_transport_internet_xtls_config_proto_rawDescOnce.Do(func() {
        // 调用protoimpl.X.CompressGZIP对原始描述数据进行压缩
        file_transport_internet_xtls_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_xtls_config_proto_rawDescData)
    })
    // 返回压缩后的描述数据
    return file_transport_internet_xtls_config_proto_rawDescData
}

// 定义枚举类型信息的切片
var file_transport_internet_xtls_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
// 定义消息类型信息的切片
var file_transport_internet_xtls_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
// 定义Go类型的切片
var file_transport_internet_xtls_config_proto_goTypes = []interface{}{
    (Certificate_Usage)(0), // 0: v2ray.core.transport.internet.xtls.Certificate.Usage
    (*Certificate)(nil),    // 1: v2ray.core.transport.internet.xtls.Certificate
    (*Config)(nil),         // 2: v2ray.core.transport.internet.xtls.Config
}
// 定义依赖索引的切片
var file_transport_internet_xtls_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.transport.internet.xtls.Certificate.usage:type_name -> v2ray.core.transport.internet.xtls.Certificate.Usage
    1, // 1: v2ray.core.transport.internet.xtls.Config.certificate:type_name -> v2ray.core.transport.internet.xtls.Certificate
    2, // [2:2] is the sub-list for method output_type
    2, // [2:2] is the sub-list for method input_type
    2, // [2:2] is the sub-list for extension type_name
    2, // [2:2] is the sub-list for extension extendee
    0, // [0:2] is the sub-list for field type_name
}

// 初始化函数
func init() { file_transport_internet_xtls_config_proto_init() }
func file_transport_internet_xtls_config_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_transport_internet_xtls_config_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_transport_internet_xtls_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Certificate); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_transport_internet_xtls_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
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
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_transport_internet_xtls_config_proto_rawDesc,
            NumEnums:      1,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_transport_internet_xtls_config_proto_goTypes,
        DependencyIndexes: file_transport_internet_xtls_config_proto_depIdxs,
        EnumInfos:         file_transport_internet_xtls_config_proto_enumTypes,
        MessageInfos:      file_transport_internet_xtls_config_proto_msgTypes,
    }.Build()
    # 设置 File_transport_internet_xtls_config_proto 为构建的类型
    File_transport_internet_xtls_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_transport_internet_xtls_config_proto_rawDesc = nil
    file_transport_internet_xtls_config_proto_goTypes = nil
    file_transport_internet_xtls_config_proto_depIdxs = nil
# 闭合前面的函数定义
```