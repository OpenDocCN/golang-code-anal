# `v2ray-core\app\dns\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: app/dns/config.proto

package dns

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    router "v2ray.com/core/app/router"  // 导入 router 包
    net "v2ray.com/core/common/net"  // 导入 net 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type DomainMatchingType int32  // 定义 DomainMatchingType 类型

const (
    DomainMatchingType_Full      DomainMatchingType = 0  // 定义 DomainMatchingType_Full 枚举值
    DomainMatchingType_Subdomain DomainMatchingType = 1  // 定义 DomainMatchingType_Subdomain 枚举值
    DomainMatchingType_Keyword   DomainMatchingType = 2  // 定义 DomainMatchingType_Keyword 枚举值
    DomainMatchingType_Regex     DomainMatchingType = 3  // 定义 DomainMatchingType_Regex 枚举值
)

// DomainMatchingType 的枚举值映射
var (
    DomainMatchingType_name = map[int32]string{  // 枚举值名称映射
        0: "Full",
        1: "Subdomain",
        2: "Keyword",
        3: "Regex",
    }
    DomainMatchingType_value = map[string]int32{  // 枚举值映射
        "Full":      0,
        "Subdomain": 1,
        "Keyword":   2,
        "Regex":     3,
    }
)

func (x DomainMatchingType) Enum() *DomainMatchingType {  // 定义 DomainMatchingType 的 Enum 方法
    p := new(DomainMatchingType)
    *p = x
    return p
}

func (x DomainMatchingType) String() string {  // 定义 DomainMatchingType 的 String 方法
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (DomainMatchingType) Descriptor() protoreflect.EnumDescriptor {  // 定义 DomainMatchingType 的 Descriptor 方法
    return file_app_dns_config_proto_enumTypes[0].Descriptor()
}

func (DomainMatchingType) Type() protoreflect.EnumType {  // 定义 DomainMatchingType 的 Type 方法
    return &file_app_dns_config_proto_enumTypes[0]
}
// 返回枚举类型的数值
func (x DomainMatchingType) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: Use DomainMatchingType.Descriptor instead.
// 返回枚举类型的描述符
func (DomainMatchingType) EnumDescriptor() ([]byte, []int) {
    return file_app_dns_config_proto_rawDescGZIP(), []int{0}
}

// NameServer 结构体定义
type NameServer struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Address           *net.Endpoint                `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
    PrioritizedDomain []*NameServer_PriorityDomain `protobuf:"bytes,2,rep,name=prioritized_domain,json=prioritizedDomain,proto3" json:"prioritized_domain,omitempty"`
    Geoip             []*router.GeoIP              `protobuf:"bytes,3,rep,name=geoip,proto3" json:"geoip,omitempty"`
    OriginalRules     []*NameServer_OriginalRule   `protobuf:"bytes,4,rep,name=original_rules,json=originalRules,proto3" json:"original_rules,omitempty"`
}

// 重置 NameServer 结构体
func (x *NameServer) Reset() {
    *x = NameServer{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_dns_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 NameServer 结构体的字符串表示
func (x *NameServer) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*NameServer) ProtoMessage() {}

// 返回 NameServer 结构体的反射信息
func (x *NameServer) ProtoReflect() protoreflect.Message {
    mi := &file_app_dns_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use NameServer.ProtoReflect.Descriptor instead.
// 返回 NameServer 结构体的描述符
func (*NameServer) Descriptor() ([]byte, []int) {
    return file_app_dns_config_proto_rawDescGZIP(), []int{0}
}

// 返回 NameServer 结构体的 Address 字段
func (x *NameServer) GetAddress() *net.Endpoint {
    if x != nil {
        return x.Address
    }
}
    # 返回空值
    return nil
# 获取优先级域名列表
func (x *NameServer) GetPrioritizedDomain() []*NameServer_PriorityDomain {
    # 如果 NameServer 对象不为空，则返回其优先级域名列表
    if x != nil {
        return x.PrioritizedDomain
    }
    # 如果 NameServer 对象为空，则返回空值
    return nil
}

# 获取地理位置信息列表
func (x *NameServer) GetGeoip() []*router.GeoIP {
    # 如果 NameServer 对象不为空，则返回其地理位置信息列表
    if x != nil {
        return x.Geoip
    }
    # 如果 NameServer 对象为空，则返回空值
    return nil
}

# 获取原始规则列表
func (x *NameServer) GetOriginalRules() []*NameServer_OriginalRule {
    # 如果 NameServer 对象不为空，则返回其原始规则列表
    if x != nil {
        return x.OriginalRules
    }
    # 如果 NameServer 对象为空，则返回空值
    return nil
}

# Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    # DNS 使用的名称服务器列表，目前仅支持传统的 UDP 服务器。特殊值 'localhost' 可以设置为域地址，以使用本地系统上的 DNS。
    #
    # 已弃用：请勿使用。
    NameServers []*net.Endpoint `protobuf:"bytes,1,rep,name=NameServers,proto3" json:"NameServers,omitempty"`
    # DNS 客户端使用的名称服务器列表。
    NameServer []*NameServer `protobuf:"bytes,5,rep,name=name_server,json=nameServer,proto3" json:"name_server,omitempty"`
    # 静态主机。域名到 IP 的映射。
    #
    # 已弃用：请勿使用。
    Hosts map[string]*net.IPOrDomain `protobuf:"bytes,2,rep,name=Hosts,proto3" json:"Hosts,omitempty" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
    # EDNS 客户端子网的客户端 IP。必须是 4 字节（IPv4）或 16 字节（IPv6）。
    ClientIp    []byte                `protobuf:"bytes,3,opt,name=client_ip,json=clientIp,proto3" json:"client_ip,omitempty"`
    StaticHosts []*Config_HostMapping `protobuf:"bytes,4,rep,name=static_hosts,json=staticHosts,proto3" json:"static_hosts,omitempty"`
    # DNS 客户端的入站标签。
    Tag string `protobuf:"bytes,6,opt,name=tag,proto3" json:"tag,omitempty"`
}

# 重置 Config 结构体
func (x *Config) Reset() {
    # 将 Config 结构体重置为空值
    *x = Config{}
}
    # 如果启用了不安全的功能，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_app_dns_config_proto_msgTypes[1]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 返回配置对象的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回配置对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_dns_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_dns_config_proto_rawDescGZIP(), []int{1}
}

// Deprecated: 不要使用
func (x *Config) GetNameServers() []*net.Endpoint {
    if x != nil {
        return x.NameServers
    }
    return nil
}

// 返回配置对象的 NameServer 字段
func (x *Config) GetNameServer() []*NameServer {
    if x != nil {
        return x.NameServer
    }
    return nil
}

// Deprecated: 不要使用
func (x *Config) GetHosts() map[string]*net.IPOrDomain {
    if x != nil {
        return x.Hosts
    }
    return nil
}

// 返回配置对象的 ClientIp 字段
func (x *Config) GetClientIp() []byte {
    if x != nil {
        return x.ClientIp
    }
    return nil
}

// 返回配置对象的 StaticHosts 字段
func (x *Config) GetStaticHosts() []*Config_HostMapping {
    if x != nil {
        return x.StaticHosts
    }
    return nil
}

// 返回配置对象的 Tag 字段
func (x *Config) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 重置 NameServer_PriorityDomain 对象
type NameServer_PriorityDomain struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Type   DomainMatchingType `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.dns.DomainMatchingType" json:"type,omitempty"`
    Domain string             `protobuf:"bytes,2,opt,name=domain,proto3" json:"domain,omitempty"`
}

// 重置 NameServer_PriorityDomain 对象
func (x *NameServer_PriorityDomain) Reset() {
    *x = NameServer_PriorityDomain{}
}
    # 如果启用了不安全的功能，则执行以下操作
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_app_dns_config_proto_msgTypes[2]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 将 NameServer_PriorityDomain 结构体转换为字符串
func (x *NameServer_PriorityDomain) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*NameServer_PriorityDomain) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *NameServer_PriorityDomain) ProtoReflect() protoreflect.Message {
    mi := &file_app_dns_config_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use NameServer_PriorityDomain.ProtoReflect.Descriptor instead.
// 获取 NameServer_PriorityDomain 的描述符
func (*NameServer_PriorityDomain) Descriptor() ([]byte, []int) {
    return file_app_dns_config_proto_rawDescGZIP(), []int{0, 0}
}

// 获取 NameServer_PriorityDomain 的匹配类型
func (x *NameServer_PriorityDomain) GetType() DomainMatchingType {
    if x != nil {
        return x.Type
    }
    return DomainMatchingType_Full
}

// 获取 NameServer_PriorityDomain 的域名
func (x *NameServer_PriorityDomain) GetDomain() string {
    if x != nil {
        return x.Domain
    }
    return ""
}

// 重置 NameServer_OriginalRule 结构体
func (x *NameServer_OriginalRule) Reset() {
    *x = NameServer_OriginalRule{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_dns_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 NameServer_OriginalRule 结构体转换为字符串
func (x *NameServer_OriginalRule) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*NameServer_OriginalRule) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *NameServer_OriginalRule) ProtoReflect() protoreflect.Message {
    mi := &file_app_dns_config_proto_msgTypes[3]
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
// Deprecated: Use NameServer_OriginalRule.ProtoReflect.Descriptor instead.
// 使用 NameServer_OriginalRule.ProtoReflect.Descriptor 替代
func (*NameServer_OriginalRule) Descriptor() ([]byte, []int) {
    return file_app_dns_config_proto_rawDescGZIP(), []int{0, 1}
}

// 获取规则
func (x *NameServer_OriginalRule) GetRule() string {
    if x != nil {
        return x.Rule
    }
    return ""
}

// 获取大小
func (x *NameServer_OriginalRule) GetSize() uint32 {
    if x != nil {
        return x.Size
    }
    return 0
}

// 主机映射配置
type Config_HostMapping struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Type   DomainMatchingType `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.dns.DomainMatchingType" json:"type,omitempty"`
    Domain string             `protobuf:"bytes,2,opt,name=domain,proto3" json:"domain,omitempty"`
    Ip     [][]byte           `protobuf:"bytes,3,rep,name=ip,proto3" json:"ip,omitempty"`
    // ProxiedDomain indicates the mapped domain has the same IP address on this
    // domain. V2Ray will use this domain for IP queries. This field is only
    // effective if ip is empty.
    ProxiedDomain string `protobuf:"bytes,4,opt,name=proxied_domain,json=proxiedDomain,proto3" json:"proxied_domain,omitempty"`
}

// 重置配置
func (x *Config_HostMapping) Reset() {
    *x = Config_HostMapping{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_dns_config_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 转换为字符串
func (x *Config_HostMapping) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config_HostMapping) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *Config_HostMapping) ProtoReflect() protoreflect.Message {
    mi := &file_app_dns_config_proto_msgTypes[5]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
}
    # 返回对象 x 的消息
    return mi.MessageOf(x)
// Deprecated: Use Config_HostMapping.ProtoReflect.Descriptor instead.
// 返回文件描述符的原始字节和路径
func (*Config_HostMapping) Descriptor() ([]byte, []int) {
    return file_app_dns_config_proto_rawDescGZIP(), []int{1, 1}
}

// 返回主机映射的类型
func (x *Config_HostMapping) GetType() DomainMatchingType {
    if x != nil {
        return x.Type
    }
    return DomainMatchingType_Full
}

// 返回主机映射的域名
func (x *Config_HostMapping) GetDomain() string {
    if x != nil {
        return x.Domain
    }
    return ""
}

// 返回主机映射的 IP 地址
func (x *Config_HostMapping) GetIp() [][]byte {
    if x != nil {
        return x.Ip
    }
    return nil
}

// 返回代理的域名
func (x *Config_HostMapping) GetProxiedDomain() string {
    if x != nil {
        return x.ProxiedDomain
    }
    return ""
}

// 定义文件描述符
var File_app_dns_config_proto protoreflect.FileDescriptor

// 定义文件描述符的原始字节
var file_app_dns_config_proto_rawDesc = []byte{
    // 这里是一段很长的原始字节数据，省略部分内容
}
    # 创建一个十六进制数值列表，可能是某种数据的编码
    0x6e, 0x65, 0x74, 0x2e, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x07, 0x61, 0x64,
    0x64, 0x72, 0x65, 0x73, 0x73, 0x12, 0x5c, 0x0a, 0x12, 0x70, 0x72, 0x69, 0x6f, 0x72, 0x69, 0x74,
    0x69, 0x7a, 0x65, 0x64, 0x5f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x03, 0x28,
    0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61,
    0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65,
    0x72, 0x2e, 0x50, 0x72, 0x69, 0x6f, 0x72, 0x69, 0x74, 0x79, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e,
    0x52, 0x11, 0x70, 0x72, 0x69, 0x6f, 0x72, 0x69, 0x74, 0x69, 0x7a, 0x65, 0x64, 0x44, 0x6f, 0x6d,
    0x61, 0x69, 0x6e, 0x12, 0x32, 0x0a, 0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x18, 0x03, 0x20, 0x03,
    0x28, 0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50,
    0x52, 0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x12, 0x52, 0x0a, 0x0e, 0x6f, 0x72, 0x69, 0x67, 0x69,
    0x6e, 0x61, 0x6c, 0x5f, 0x72, 0x75, 0x6c, 0x65, 0x73, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0b, 0x32,
    0x2b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
    0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x2e,
    0x4f, 0x72, 0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x52, 0x75, 0x6c, 0x65, 0x52, 0x0d, 0x6f, 0x72,
    0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x52, 0x75, 0x6c, 0x65, 0x73, 0x1a, 0x64, 0x0a, 0x0e, 0x50,
    0x72, 0x69, 0x6f, 0x72, 0x69, 0x74, 0x79, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x3a, 0x0a,
    0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x26, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73,
    0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4d, 0x61, 0x74, 0x63, 0x68, 0x69, 0x6e, 0x67, 0x54,
    # 创建一个十六进制数列表，用于表示一些特定的数值
    0x79, 0x70, 0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65, 0x12, 0x16, 0x0a, 0x06, 0x64, 0x6f, 0x6d,
    0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69,
    0x6e, 0x1a, 0x36, 0x0a, 0x0c, 0x4f, 0x72, 0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x52, 0x75, 0x6c,
    0x65, 0x12, 0x12, 0x0a, 0x04, 0x72, 0x75, 0x6c, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52,
    0x04, 0x72, 0x75, 0x6c, 0x65, 0x12, 0x12, 0x0a, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x18, 0x02, 0x20,
    0x01, 0x28, 0x0d, 0x52, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x22, 0xc3, 0x04, 0x0a, 0x06, 0x43, 0x6f,
    0x6e, 0x66, 0x69, 0x67, 0x12, 0x45, 0x0a, 0x0b, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76,
    0x65, 0x72, 0x73, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1f, 0x2e, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65,
    0x74, 0x2e, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0b,
    0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x12, 0x3f, 0x0a, 0x0b, 0x6e, 0x61,
    0x6d, 0x65, 0x5f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x18, 0x05, 0x20, 0x03, 0x28, 0x0b, 0x32,
    0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
    0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x52,
    0x0a, 0x6e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x12, 0x3f, 0x0a, 0x05, 0x48,
    0x6f, 0x73, 0x74, 0x73, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x25, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e,
    0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x48, 0x6f, 0x73, 0x74, 0x73, 0x45, 0x6e, 0x74, 0x72,
    0x79, 0x42, 0x02, 0x18, 0x01, 0x52, 0x05, 0x48, 0x6f, 0x73, 0x74, 0x73, 0x12, 0x1b, 0x0a, 0x09,
    0x63, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x5f, 0x69, 0x70, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0c,
    # 以十六进制表示的字节码，可能是某种数据或配置信息
    0x52, 0x08, 0x63, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x49, 0x70, 0x12, 0x49, 0x0a, 0x0c, 0x73, 0x74,
    0x61, 0x74, 0x69, 0x63, 0x5f, 0x68, 0x6f, 0x73, 0x74, 0x73, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0b,
    0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70,
    0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x48, 0x6f, 0x73,
    0x74, 0x4d, 0x61, 0x70, 0x70, 0x69, 0x6e, 0x67, 0x52, 0x0b, 0x73, 0x74, 0x61, 0x74, 0x69, 0x63,
    0x48, 0x6f, 0x73, 0x74, 0x73, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61, 0x67, 0x18, 0x06, 0x20, 0x01,
    0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x1a, 0x5b, 0x0a, 0x0a, 0x48, 0x6f, 0x73, 0x74, 0x73,
    0x45, 0x6e, 0x74, 0x72, 0x79, 0x12, 0x10, 0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01,
    0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12, 0x37, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65,
    0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x49,
    0x50, 0x4f, 0x72, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65,
    0x3a, 0x02, 0x38, 0x01, 0x1a, 0x98, 0x01, 0x0a, 0x0b, 0x48, 0x6f, 0x73, 0x74, 0x4d, 0x61, 0x70,
    0x70, 0x69, 0x6e, 0x67, 0x12, 0x3a, 0x0a, 0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01,
    0x28, 0x0e, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4d, 0x61,
    0x74, 0x63, 0x68, 0x69, 0x6e, 0x67, 0x54, 0x79, 0x70, 0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65,
    0x12, 0x16, 0x0a, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09,
    0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e,
    # 这里是一段十六进制表示的字节码，可能是某种数据或配置信息
    # 以十六进制表示的数据，可能是某种编码或者密钥
    0x69, 0x65, 0x64, 0x5f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x04, 0x20, 0x01, 0x28, 0x09,
    0x52, 0x0d, 0x70, 0x72, 0x6f, 0x78, 0x69, 0x65, 0x64, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x2a,
    0x45, 0x0a, 0x12, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4d, 0x61, 0x74, 0x63, 0x68, 0x69, 0x6e,
    0x67, 0x54, 0x79, 0x70, 0x65, 0x12, 0x08, 0x0a, 0x04, 0x46, 0x75, 0x6c, 0x6c, 0x10, 0x00, 0x12,
    0x0d, 0x0a, 0x09, 0x53, 0x75, 0x62, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x10, 0x01, 0x12, 0x0b,
    0x0a, 0x07, 0x4b, 0x65, 0x79, 0x77, 0x6f, 0x72, 0x64, 0x10, 0x02, 0x12, 0x09, 0x0a, 0x05, 0x52,
    0x65, 0x67, 0x65, 0x78, 0x10, 0x03, 0x42, 0x47, 0x0a, 0x16, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73,
    0x50, 0x01, 0x5a, 0x16, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
    0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x64, 0x6e, 0x73, 0xaa, 0x02, 0x12, 0x56, 0x32, 0x52,
    0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x44, 0x6e, 0x73, 0x62,
    0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量 file_app_dns_config_proto_rawDescOnce，类型为 sync.Once，用于确保 file_app_dns_config_proto_rawDescData 只被初始化一次
var (
    file_app_dns_config_proto_rawDescOnce sync.Once
    file_app_dns_config_proto_rawDescData = file_app_dns_config_proto_rawDesc
)

// 定义函数 file_app_dns_config_proto_rawDescGZIP，返回类型为 []byte，用于对 file_app_dns_config_proto_rawDescData 进行 GZIP 压缩
func file_app_dns_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_app_dns_config_proto_rawDescOnce.Do(func() {
        // 对 file_app_dns_config_proto_rawDescData 进行 GZIP 压缩
        file_app_dns_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_dns_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的数据
    return file_app_dns_config_proto_rawDescData
}

// 定义变量 file_app_dns_config_proto_enumTypes，类型为 []protoimpl.EnumInfo，用于存储枚举类型信息
var file_app_dns_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
// 定义变量 file_app_dns_config_proto_msgTypes，类型为 []protoimpl.MessageInfo，用于存储消息类型信息
var file_app_dns_config_proto_msgTypes = make([]protoimpl.MessageInfo, 6)
// 定义变量 file_app_dns_config_proto_goTypes，类型为 []interface{}，用于存储 Go 类型信息
var file_app_dns_config_proto_goTypes = []interface{}{
    (DomainMatchingType)(0),           // 0: v2ray.core.app.dns.DomainMatchingType
    (*NameServer)(nil),                // 1: v2ray.core.app.dns.NameServer
    (*Config)(nil),                    // 2: v2ray.core.app.dns.Config
    (*NameServer_PriorityDomain)(nil), // 3: v2ray.core.app.dns.NameServer.PriorityDomain
    (*NameServer_OriginalRule)(nil),   // 4: v2ray.core.app.dns.NameServer.OriginalRule
    nil,                               // 5: v2ray.core.app.dns.Config.HostsEntry
    (*Config_HostMapping)(nil),        // 6: v2ray.core.app.dns.Config.HostMapping
    (*net.Endpoint)(nil),              // 7: v2ray.core.common.net.Endpoint
    (*router.GeoIP)(nil),              // 8: v2ray.core.app.router.GeoIP
    (*net.IPOrDomain)(nil),            // 9: v2ray.core.common.net.IPOrDomain
}
// 定义变量 file_app_dns_config_proto_depIdxs，类型为 []int32，用于存储依赖索引信息
var file_app_dns_config_proto_depIdxs = []int32{
    7,  // 0: v2ray.core.app.dns.NameServer.address:type_name -> v2ray.core.common.net.Endpoint
    3,  // 1: v2ray.core.app.dns.NameServer.prioritized_domain:type_name -> v2ray.core.app.dns.NameServer.PriorityDomain
    8,  // 2: v2ray.core.app.dns.NameServer.geoip:type_name -> v2ray.core.app.router.GeoIP
    4,  // 3: v2ray.core.app.dns.NameServer.original_rules:type_name -> v2ray.core.app.dns.NameServer.OriginalRule
    7,  // 4: v2ray.core.app.dns.Config.NameServers:type_name -> v2ray.core.common.net.Endpoint
    1,  // 5: v2ray.core.app.dns.Config.name_server:type_name -> v2ray.core.app.dns.NameServer
    // 第5个元素表示 v2ray.core.app.dns.Config.name_server 的类型为 v2ray.core.app.dns.NameServer

    5,  // 6: v2ray.core.app.dns.Config.Hosts:type_name -> v2ray.core.app.dns.Config.HostsEntry
    // 第6个元素表示 v2ray.core.app.dns.Config.Hosts 的类型为 v2ray.core.app.dns.Config.HostsEntry

    6,  // 7: v2ray.core.app.dns.Config.static_hosts:type_name -> v2ray.core.app.dns.Config.HostMapping
    // 第7个元素表示 v2ray.core.app.dns.Config.static_hosts 的类型为 v2ray.core.app.dns.Config.HostMapping

    0,  // 8: v2ray.core.app.dns.NameServer.PriorityDomain.type:type_name -> v2ray.core.app.dns.DomainMatchingType
    // 第8个元素表示 v2ray.core.app.dns.NameServer.PriorityDomain.type 的类型为 v2ray.core.app.dns.DomainMatchingType

    9,  // 9: v2ray.core.app.dns.Config.HostsEntry.value:type_name -> v2ray.core.common.net.IPOrDomain
    // 第9个元素表示 v2ray.core.app.dns.Config.HostsEntry.value 的类型为 v2ray.core.common.net.IPOrDomain

    0,  // 10: v2ray.core.app.dns.Config.HostMapping.type:type_name -> v2ray.core.app.dns.DomainMatchingType
    // 第10个元素表示 v2ray.core.app.dns.Config.HostMapping.type 的类型为 v2ray.core.app.dns.DomainMatchingType

    11, // [11:11] is the sub-list for method output_type
    // 第11个元素表示 [11:11] 是 output_type 方法的子列表

    11, // [11:11] is the sub-list for method input_type
    // 第11个元素表示 [11:11] 是 input_type 方法的子列表

    11, // [11:11] is the sub-list for extension type_name
    // 第11个元素表示 [11:11] 是 type_name 扩展的子列表

    11, // [11:11] is the sub-list for extension extendee
    // 第11个元素表示 [11:11] 是 extendee 扩展的子列表

    0,  // [0:11] is the sub-list for field type_name
    // 第0个元素表示 [0:11] 是 type_name 字段的子列表
# 初始化函数，用于在包被导入时执行
func init() { file_app_dns_config_proto_init() }
# 实际的初始化函数，用于初始化文件的 DNS 配置
func file_app_dns_config_proto_init() {
    # 如果文件的 DNS 配置已经被初始化，则直接返回
    if File_app_dns_config_proto != nil {
        return
    }
    # 如果 protoimpl.UnsafeEnabled 为假，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        # 设置 NameServer 消息类型的导出函数
        file_app_dns_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            # 根据索引 i 选择返回相应的字段
            switch v := v.(*NameServer); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
        # 设置 Config 消息类型的导出函数
        file_app_dns_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            # 根据索引 i 选择返回相应的字段
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
        # 设置 NameServer_PriorityDomain 消息类型的导出函数
        file_app_dns_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
            # 根据索引 i 选择返回相应的字段
            switch v := v.(*NameServer_PriorityDomain); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
        # 设置 NameServer_OriginalRule 消息类型的导出函数
        file_app_dns_config_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{}:
            # 根据索引 i 选择返回相应的字段
            switch v := v.(*NameServer_OriginalRule); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
        }
        # 设置 Config_HostMapping 消息类型的导出函数
        file_app_dns_config_proto_msgTypes[5].Exporter = func(v interface{}, i int) interface{}:
            # 根据索引 i 选择返回相应的字段
            switch v := v.(*Config_HostMapping); i {
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
    # 定义一个结构体 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建一个类型
    out := protoimpl.TypeBuilder{
        # 使用 protoimpl.DescBuilder 构建一个描述
        File: protoimpl.DescBuilder{
            # 设置 Go 包路径为结构体 x 的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置原始描述为指定的原始描述
            RawDescriptor: file_app_dns_config_proto_rawDesc,
            # 设置枚举类型数量为 1
            NumEnums:      1,
            # 设置消息类型数量为 6
            NumMessages:   6,
            # 设置扩展数量为 0
            NumExtensions: 0,
            # 设置服务数量为 0
            NumServices:   0,
        },
        # 设置 Go 类型为指定的 Go 类型
        GoTypes:           file_app_dns_config_proto_goTypes,
        # 设置依赖索引为指定的依赖索引
        DependencyIndexes: file_app_dns_config_proto_depIdxs,
        # 设置枚举信息为指定的枚举类型
        EnumInfos:         file_app_dns_config_proto_enumTypes,
        # 设置消息信息为指定的消息类型
        MessageInfos:      file_app_dns_config_proto_msgTypes,
    }.Build()
    # 将构建的文件赋值给 File_app_dns_config_proto
    File_app_dns_config_proto = out.File
    # 将原始描述设置为 nil
    file_app_dns_config_proto_rawDesc = nil
    # 将 Go 类型设置为 nil
    file_app_dns_config_proto_goTypes = nil
    # 将依赖索引设置为 nil
    file_app_dns_config_proto_depIdxs = nil
# 闭合前面的函数定义
```