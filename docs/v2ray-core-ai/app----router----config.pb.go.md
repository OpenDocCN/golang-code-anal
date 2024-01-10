# `v2ray-core\app\router\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/router/config.proto

package router

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

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// 域名值的类型。
type Domain_Type int32

const (
    // 该值直接使用。
    Domain_Plain Domain_Type = 0
    // 该值作为正则表达式使用。
    Domain_Regex Domain_Type = 1
    // 该值是根域名。
    Domain_Domain Domain_Type = 2
    // 该值是域名。
    Domain_Full Domain_Type = 3
)

// Domain_Type 的枚举值映射。
var (
    Domain_Type_name = map[int32]string{
        0: "Plain",
        1: "Regex",
        2: "Domain",
        3: "Full",
    }
    Domain_Type_value = map[string]int32{
        "Plain":  0,
        "Regex":  1,
        "Domain": 2,
        "Full":   3,
    }
)

func (x Domain_Type) Enum() *Domain_Type {
    p := new(Domain_Type)
    *p = x
    return p
}

func (x Domain_Type) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Domain_Type) Descriptor() protoreflect.EnumDescriptor {
    return file_app_router_config_proto_enumTypes[0].Descriptor()
}

func (Domain_Type) Type() protoreflect.EnumType {
    return &file_app_router_config_proto_enumTypes[0]
}
// 返回枚举类型的数值
func (x Domain_Type) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: 使用 Domain_Type.Descriptor 替代
func (Domain_Type) EnumDescriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{0, 0}
}

// 配置域策略
type Config_DomainStrategy int32

const (
    // 使用原始域名
    Config_AsIs Config_DomainStrategy = 0
    // 总是解析域名对应的 IP
    Config_UseIp Config_DomainStrategy = 1
    // 如果域名不匹配任何规则，则解析为 IP
    Config_IpIfNonMatch Config_DomainStrategy = 2
    // 如果任何规则要求 IP 匹配，则解析为 IP
    Config_IpOnDemand Config_DomainStrategy = 3
)

// Config_DomainStrategy 的枚举值映射
var (
    Config_DomainStrategy_name = map[int32]string{
        0: "AsIs",
        1: "UseIp",
        2: "IpIfNonMatch",
        3: "IpOnDemand",
    }
    Config_DomainStrategy_value = map[string]int32{
        "AsIs":         0,
        "UseIp":        1,
        "IpIfNonMatch": 2,
        "IpOnDemand":   3,
    }
)

// 返回 Config_DomainStrategy 的枚举指针
func (x Config_DomainStrategy) Enum() *Config_DomainStrategy {
    p := new(Config_DomainStrategy)
    *p = x
    return p
}

// 返回 Config_DomainStrategy 的字符串表示
func (x Config_DomainStrategy) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

// 返回 Config_DomainStrategy 的枚举描述
func (Config_DomainStrategy) Descriptor() protoreflect.EnumDescriptor {
    return file_app_router_config_proto_enumTypes[1].Descriptor()
}

// 返回 Config_DomainStrategy 的枚举类型
func (Config_DomainStrategy) Type() protoreflect.EnumType {
    return &file_app_router_config_proto_enumTypes[1]
}

// 返回枚举类型的数值
func (x Config_DomainStrategy) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// Deprecated: 使用 Config_DomainStrategy.Descriptor 替代
func (Config_DomainStrategy) EnumDescriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{8, 0}
}

// 用于路由决策的域名
type Domain struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    // 未知字段，类型为 protoimpl.UnknownFields
    unknownFields protoimpl.UnknownFields

    // 域名匹配类型
    Type Domain_Type `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.router.Domain_Type" json:"type,omitempty"`
    // 域名数值
    Value string `protobuf:"bytes,2,opt,name=value,proto3" json:"value,omitempty"`
    // 此域名的属性，可用于过滤
    Attribute []*Domain_Attribute `protobuf:"bytes,3,rep,name=attribute,proto3" json:"attribute,omitempty"`
// 重置 Domain 结构体的值为默认值
func (x *Domain) Reset() {
    *x = Domain{}
    // 如果启用了不安全操作，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Domain 结构体的字符串表示形式
func (x *Domain) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Domain) ProtoMessage() {}

// 返回 Domain 结构体的反射信息
func (x *Domain) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[0]
    // 如果启用了不安全操作并且 x 不为 nil，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Domain.ProtoReflect.Descriptor instead.
// 返回 Domain 结构体的描述符
func (*Domain) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{0}
}

// 返回 Domain 结构体的 Type 值
func (x *Domain) GetType() Domain_Type {
    if x != nil {
        return x.Type
    }
    return Domain_Plain
}

// 返回 Domain 结构体的 Value 值
func (x *Domain) GetValue() string {
    if x != nil {
        return x.Value
    }
    return ""
}

// 返回 Domain 结构体的 Attribute 切片
func (x *Domain) GetAttribute() []*Domain_Attribute {
    if x != nil {
        return x.Attribute
    }
    return nil
}

// IP for routing decision, in CIDR form.
// CIDR 结构体表示路由决策中的 IP 地址
type CIDR struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // IP address, should be either 4 or 16 bytes.
    Ip []byte `protobuf:"bytes,1,opt,name=ip,proto3" json:"ip,omitempty"`
    // Number of leading ones in the network mask.
    Prefix uint32 `protobuf:"varint,2,opt,name=prefix,proto3" json:"prefix,omitempty"`
}

// 重置 CIDR 结构体的值为默认值
func (x *CIDR) Reset() {
    *x = CIDR{}
    // 如果启用了不安全操作，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 CIDR 结构体的字符串表示形式
func (x *CIDR) String() string {
    return protoimpl.X.MessageStringOf(x)
}
// 实现 ProtoMessage 接口的方法
func (*CIDR) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *CIDR) ProtoReflect() protoreflect.Message {
    // 获取 CIDR 对应的消息类型信息
    mi := &file_app_router_config_proto_msgTypes[1]
    // 如果启用了不安全的特性并且 x 不为空
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

// Deprecated: Use CIDR.ProtoReflect.Descriptor instead.
// 获取 CIDR 的描述符
func (*CIDR) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{1}
}

// 获取 CIDR 的 IP 地址
func (x *CIDR) GetIp() []byte {
    if x != nil {
        return x.Ip
    }
    return nil
}

// 获取 CIDR 的前缀
func (x *CIDR) GetPrefix() uint32 {
    if x != nil {
        return x.Prefix
    }
    return 0
}

// 定义 GeoIP 结构体
type GeoIP struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    CountryCode string  `protobuf:"bytes,1,opt,name=country_code,json=countryCode,proto3" json:"country_code,omitempty"`
    Cidr        []*CIDR `protobuf:"bytes,2,rep,name=cidr,proto3" json:"cidr,omitempty"`
}

// 重置 GeoIP 结构体
func (x *GeoIP) Reset() {
    *x = GeoIP{}
    // 如果启用了不安全的特性
    if protoimpl.UnsafeEnabled {
        // 获取 GeoIP 对应的消息类型信息
        mi := &file_app_router_config_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 获取 GeoIP 结构体的字符串表示
func (x *GeoIP) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*GeoIP) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *GeoIP) ProtoReflect() protoreflect.Message {
    // 获取 GeoIP 对应的消息类型信息
    mi := &file_app_router_config_proto_msgTypes[2]
    // 如果启用了不安全的特性并且 x 不为空
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

// Deprecated: Use GeoIP.ProtoReflect.Descriptor instead.
// 获取 GeoIP 的描述符
func (*GeoIP) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{2}
}
// 获取 GeoIP 结构体中的 CountryCode 字段值
func (x *GeoIP) GetCountryCode() string {
    // 如果 GeoIP 结构体不为空，则返回其 CountryCode 字段值
    if x != nil {
        return x.CountryCode
    }
    // 如果 GeoIP 结构体为空，则返回空字符串
    return ""
}

// 获取 GeoIP 结构体中的 Cidr 字段值
func (x *GeoIP) GetCidr() []*CIDR {
    // 如果 GeoIP 结构体不为空，则返回其 Cidr 字段值
    if x != nil {
        return x.Cidr
    }
    // 如果 GeoIP 结构体为空，则返回空数组
    return nil
}

// GeoIPList 结构体定义
type GeoIPList struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Entry []*GeoIP `protobuf:"bytes,1,rep,name=entry,proto3" json:"entry,omitempty"`
}

// 重置 GeoIPList 结构体
func (x *GeoIPList) Reset() {
    *x = GeoIPList{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 GeoIPList 结构体的字符串表示形式
func (x *GeoIPList) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*GeoIPList) ProtoMessage() {}

// 返回 GeoIPList 结构体的反射信息
func (x *GeoIPList) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[3]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 GeoIPList.ProtoReflect.Descriptor 替代
func (*GeoIPList) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{3}
}

// 获取 GeoIPList 结构体中的 Entry 字段值
func (x *GeoIPList) GetEntry() []*GeoIP {
    // 如果 GeoIPList 结构体不为空，则返回其 Entry 字段值
    if x != nil {
        return x.Entry
    }
    // 如果 GeoIPList 结构体为空，则返回空数组
    return nil
}

// GeoSite 结构体定义
type GeoSite struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    CountryCode string    `protobuf:"bytes,1,opt,name=country_code,json=countryCode,proto3" json:"country_code,omitempty"`
    Domain      []*Domain `protobuf:"bytes,2,rep,name=domain,proto3" json:"domain,omitempty"`
}

// 重置 GeoSite 结构体
func (x *GeoSite) Reset() {
    *x = GeoSite{}
}
    # 如果启用了不安全的功能，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_app_router_config_proto_msgTypes[4]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 定义 GeoSite 结构体的 String 方法，返回其字符串表示形式
func (x *GeoSite) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 定义 GeoSite 结构体的 ProtoMessage 方法
func (*GeoSite) ProtoMessage() {}

// 定义 GeoSite 结构体的 ProtoReflect 方法，返回消息的反射类型
func (x *GeoSite) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[4]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use GeoSite.ProtoReflect.Descriptor instead.
// 定义 GeoSite 结构体的 Descriptor 方法，返回消息的描述符
func (*GeoSite) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{4}
}

// 定义 GeoSite 结构体的 GetCountryCode 方法，返回其国家代码
func (x *GeoSite) GetCountryCode() string {
    if x != nil {
        return x.CountryCode
    }
    return ""
}

// 定义 GeoSite 结构体的 GetDomain 方法，返回其域名列表
func (x *GeoSite) GetDomain() []*Domain {
    if x != nil {
        return x.Domain
    }
    return nil
}

// 定义 GeoSiteList 结构体，包含消息状态、大小缓存和未知字段
type GeoSiteList struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Entry []*GeoSite `protobuf:"bytes,1,rep,name=entry,proto3" json:"entry,omitempty"`
}

// 重置 GeoSiteList 结构体
func (x *GeoSiteList) Reset() {
    *x = GeoSiteList{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 定义 GeoSiteList 结构体的 String 方法，返回其字符串表示形式
func (x *GeoSiteList) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 定义 GeoSiteList 结构体的 ProtoMessage 方法
func (*GeoSiteList) ProtoMessage() {}

// 定义 GeoSiteList 结构体的 ProtoReflect 方法，返回消息的反射类型
func (x *GeoSiteList) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[5]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use GeoSiteList.ProtoReflect.Descriptor instead.
// 定义 GeoSiteList 结构体的 Descriptor 方法，返回消息的描述符
func (*GeoSiteList) Descriptor() ([]byte, []int) {
    # 返回 file_app_router_config_proto_rawDescGZIP() 函数的结果和空的整数列表
    return file_app_router_config_proto_rawDescGZIP(), []int{5}
}
// GetEntry 方法返回 GeoSiteList 结构体中的 Entry 切片
func (x *GeoSiteList) GetEntry() []*GeoSite {
    // 如果 x 不为 nil，则返回 x 中的 Entry 切片
    if x != nil {
        return x.Entry
    }
    // 如果 x 为 nil，则返回 nil
    return nil
}

// RoutingRule 结构体定义
type RoutingRule struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // TargetTag 可以是 Tag 或 BalancingTag
    //    *RoutingRule_Tag
    //    *RoutingRule_BalancingTag
    TargetTag isRoutingRule_TargetTag `protobuf_oneof:"target_tag"`
    // 匹配目标域名的域名列表
    Domain []*Domain `protobuf:"bytes,2,rep,name=domain,proto3" json:"domain,omitempty"`
    // 匹配目标 IP 地址的 CIDR 列表。已弃用。请使用下面的 geoip。
    //
    // 已弃用：请勿使用。
    Cidr []*CIDR `protobuf:"bytes,3,rep,name=cidr,proto3" json:"cidr,omitempty"`
    // 匹配目标 IP 地址的 GeoIP 列表。如果存在此条目，则上面的 cidr 将不起作用。具有相同国家代码的 GeoIP 字段应包含完全相同的内容。它们将在运行时合并。对于自定义的 GeoIP，请将国家代码留空。
    Geoip []*GeoIP `protobuf:"bytes,10,rep,name=geoip,proto3" json:"geoip,omitempty"`
    // 端口范围 [from, to]。如果目标端口在此范围内，则此规则生效。已弃用。请使用 port_list。
    //
    // 已弃用：请勿使用。
    PortRange *net.PortRange `protobuf:"bytes,4,opt,name=port_range,json=portRange,proto3" json:"port_range,omitempty"`
    // 端口列表
    PortList *net.PortList `protobuf:"bytes,14,opt,name=port_list,json=portList,proto3" json:"port_list,omitempty"`
    // 网络列表。已弃用。请使用 networks。
    //
    // 已弃用：请勿使用。
    NetworkList *net.NetworkList `protobuf:"bytes,5,opt,name=network_list,json=networkList,proto3" json:"network_list,omitempty"`
    // 匹配的网络列表
}
    # 网络配置列表，包含网络类型和参数
    Networks []net.Network `protobuf:"varint,13,rep,packed,name=networks,proto3,enum=v2ray.core.common.net.Network" json:"networks,omitempty"`
    # 源 IP 地址匹配的 CIDR 列表
    #
    # 已废弃：请勿使用
    SourceCidr []*CIDR `protobuf:"bytes,6,rep,name=source_cidr,json=sourceCidr,proto3" json:"source_cidr,omitempty"`
    # 源 IP 地址匹配的 GeoIP 列表。如果存在此条目，则上面的 source_cidr 将不起作用。
    SourceGeoip []*GeoIP `protobuf:"bytes,11,rep,name=source_geoip,json=sourceGeoip,proto3" json:"source_geoip,omitempty"`
    # 源端口匹配的端口列表
    SourcePortList *net.PortList `protobuf:"bytes,16,opt,name=source_port_list,json=sourcePortList,proto3" json:"source_port_list,omitempty"`
    # 用户邮箱列表
    UserEmail []string `protobuf:"bytes,7,rep,name=user_email,json=userEmail,proto3" json:"user_email,omitempty"`
    # 入站标签列表
    InboundTag []string `protobuf:"bytes,8,rep,name=inbound_tag,json=inboundTag,proto3" json:"inbound_tag,omitempty"`
    # 协议列表
    Protocol []string `protobuf:"bytes,9,rep,name=protocol,proto3" json:"protocol,omitempty"`
    # 属性
    Attributes string `protobuf:"bytes,15,opt,name=attributes,proto3" json:"attributes,omitempty"`
// 重置 RoutingRule 对象，将其重新初始化为空对象
func (x *RoutingRule) Reset() {
    *x = RoutingRule{}
    // 如果启用了不安全操作，则获取消息类型并存储消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[6]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RoutingRule 对象的字符串表示形式
func (x *RoutingRule) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*RoutingRule) ProtoMessage() {}

// 返回 RoutingRule 对象的反射信息
func (x *RoutingRule) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[6]
    // 如果启用了不安全操作并且对象不为空，则获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 RoutingRule.ProtoReflect.Descriptor 替代
func (*RoutingRule) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{6}
}

// 获取 RoutingRule 对象的目标标签
func (m *RoutingRule) GetTargetTag() isRoutingRule_TargetTag {
    if m != nil {
        return m.TargetTag
    }
    return nil
}

// 获取 RoutingRule 对象的标签
func (x *RoutingRule) GetTag() string {
    if x, ok := x.GetTargetTag().(*RoutingRule_Tag); ok {
        return x.Tag
    }
    return ""
}

// 获取 RoutingRule 对象的负载均衡标签
func (x *RoutingRule) GetBalancingTag() string {
    if x, ok := x.GetTargetTag().(*RoutingRule_BalancingTag); ok {
        return x.BalancingTag
    }
    return ""
}

// 获取 RoutingRule 对象的域名列表
func (x *RoutingRule) GetDomain() []*Domain {
    if x != nil {
        return x.Domain
    }
    return nil
}

// 弃用：不要使用
func (x *RoutingRule) GetCidr() []*CIDR {
    if x != nil {
        return x.Cidr
    }
    return nil
}

// 获取 RoutingRule 对象的地理位置信息列表
func (x *RoutingRule) GetGeoip() []*GeoIP {
    if x != nil {
        return x.Geoip
    }
    return nil
}

// 弃用：不要使用
func (x *RoutingRule) GetPortRange() *net.PortRange {
    if x != nil {
        return x.PortRange
    }
    return nil
}

// 获取 RoutingRule 对象的端口列表
func (x *RoutingRule) GetPortList() *net.PortList {
    if x != nil {
        return x.PortList
    }
    return nil
}
// Deprecated: Do not use.
// 获取路由规则的网络列表
func (x *RoutingRule) GetNetworkList() *net.NetworkList {
    if x != nil {
        return x.NetworkList
    }
    return nil
}

// 获取路由规则的网络
func (x *RoutingRule) GetNetworks() []net.Network {
    if x != nil {
        return x.Networks
    }
    return nil
}

// Deprecated: Do not use.
// 获取路由规则的源 CIDR
func (x *RoutingRule) GetSourceCidr() []*CIDR {
    if x != nil {
        return x.SourceCidr
    }
    return nil
}

// 获取路由规则的源 GeoIP
func (x *RoutingRule) GetSourceGeoip() []*GeoIP {
    if x != nil {
        return x.SourceGeoip
    }
    return nil
}

// 获取路由规则的源端口列表
func (x *RoutingRule) GetSourcePortList() *net.PortList {
    if x != nil {
        return x.SourcePortList
    }
    return nil
}

// 获取路由规则的用户邮箱
func (x *RoutingRule) GetUserEmail() []string {
    if x != nil {
        return x.UserEmail
    }
    return nil
}

// 获取路由规则的入站标签
func (x *RoutingRule) GetInboundTag() []string {
    if x != nil {
        return x.InboundTag
    }
    return nil
}

// 获取路由规则的协议
func (x *RoutingRule) GetProtocol() []string {
    if x != nil {
        return x.Protocol
    }
    return nil
}

// 获取路由规则的属性
func (x *RoutingRule) GetAttributes() string {
    if x != nil {
        return x.Attributes
    }
    return ""
}

// 路由规则的目标标签接口
type isRoutingRule_TargetTag interface {
    isRoutingRule_TargetTag()
}

// 路由规则的标签
type RoutingRule_Tag struct {
    // 指向的出站标签
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3,oneof"`
}

// 路由规则的负载均衡标签
type RoutingRule_BalancingTag struct {
    // 路由负载均衡标签
    BalancingTag string `protobuf:"bytes,12,opt,name=balancing_tag,json=balancingTag,proto3,oneof"`
}

// 实现路由规则的目标标签接口
func (*RoutingRule_Tag) isRoutingRule_TargetTag() {}

// 实现路由规则的负载均衡标签接口
func (*RoutingRule_BalancingTag) isRoutingRule_TargetTag() {}

// 负载均衡规则
type BalancingRule struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Tag              string   `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
}
    # 定义一个名为OutboundSelector的字符串数组，用于存储多个出站选择器
    OutboundSelector []string `protobuf:"bytes,2,rep,name=outbound_selector,json=outboundSelector,proto3" json:"outbound_selector,omitempty"`
// 重置 BalancingRule 对象
func (x *BalancingRule) Reset() {
    // 将 BalancingRule 对象重置为空对象
    *x = BalancingRule{}
    // 如果启用了不安全操作，则获取消息类型并存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[7]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 BalancingRule 对象的字符串表示
func (x *BalancingRule) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*BalancingRule) ProtoMessage() {}

// 返回 BalancingRule 对象的反射信息
func (x *BalancingRule) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[7]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use BalancingRule.ProtoReflect.Descriptor instead.
// 返回 BalancingRule 对象的描述符
func (*BalancingRule) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{7}
}

// 获取 BalancingRule 对象的标签
func (x *BalancingRule) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取 BalancingRule 对象的出站选择器
func (x *BalancingRule) GetOutboundSelector() []string {
    if x != nil {
        return x.OutboundSelector
    }
    return nil
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    DomainStrategy Config_DomainStrategy `protobuf:"varint,1,opt,name=domain_strategy,json=domainStrategy,proto3,enum=v2ray.core.app.router.Config_DomainStrategy" json:"domain_strategy,omitempty"`
    Rule           []*RoutingRule        `protobuf:"bytes,2,rep,name=rule,proto3" json:"rule,omitempty"`
    BalancingRule  []*BalancingRule      `protobuf:"bytes,3,rep,name=balancing_rule,json=balancingRule,proto3" json:"balancing_rule,omitempty"`
}

// 重置 Config 对象
func (x *Config) Reset() {
    // 将 Config 对象重置为空对象
    *x = Config{}
    // 如果启用了不安全操作，则获取消息类型并存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[8]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}
// 返回配置对象的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回配置对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[8]
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
    return file_app_router_config_proto_rawDescGZIP(), []int{8}
}

// 获取配置对象的域策略
func (x *Config) GetDomainStrategy() Config_DomainStrategy {
    if x != nil {
        return x.DomainStrategy
    }
    return Config_AsIs
}

// 获取配置对象的路由规则
func (x *Config) GetRule() []*RoutingRule {
    if x != nil {
        return x.Rule
    }
    return nil
}

// 获取配置对象的负载均衡规则
func (x *Config) GetBalancingRule() []*BalancingRule {
    if x != nil {
        return x.BalancingRule
    }
    return nil
}

// 域属性结构体
type Domain_Attribute struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Key string `protobuf:"bytes,1,opt,name=key,proto3" json:"key,omitempty"`
    // 可赋值给 TypedValue 的类型:
    //    *Domain_Attribute_BoolValue
    //    *Domain_Attribute_IntValue
    TypedValue isDomain_Attribute_TypedValue `protobuf_oneof:"typed_value"`
}

// 重置域属性对象
func (x *Domain_Attribute) Reset() {
    *x = Domain_Attribute{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_router_config_proto_msgTypes[9]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回域属性对象的字符串表示形式
func (x *Domain_Attribute) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Domain_Attribute) ProtoMessage() {}

// 返回域属性对象的反射信息
func (x *Domain_Attribute) ProtoReflect() protoreflect.Message {
    mi := &file_app_router_config_proto_msgTypes[9]
}
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
// Deprecated: Use Domain_Attribute.ProtoReflect.Descriptor instead.
// 使用 Domain_Attribute.ProtoReflect.Descriptor 替代
func (*Domain_Attribute) Descriptor() ([]byte, []int) {
    return file_app_router_config_proto_rawDescGZIP(), []int{0, 0}
}

// 获取属性的键值
func (x *Domain_Attribute) GetKey() string {
    if x != nil {
        return x.Key
    }
    return ""
}

// 获取属性的类型化数值
func (m *Domain_Attribute) GetTypedValue() isDomain_Attribute_TypedValue {
    if m != nil {
        return m.TypedValue
    }
    return nil
}

// 获取属性的布尔值
func (x *Domain_Attribute) GetBoolValue() bool {
    if x, ok := x.GetTypedValue().(*Domain_Attribute_BoolValue); ok {
        return x.BoolValue
    }
    return false
}

// 获取属性的整数值
func (x *Domain_Attribute) GetIntValue() int64 {
    if x, ok := x.GetTypedValue().(*Domain_Attribute_IntValue); ok {
        return x.IntValue
    }
    return 0
}

// 属性类型化数值接口
type isDomain_Attribute_TypedValue interface {
    isDomain_Attribute_TypedValue()
}

// 属性布尔值结构体
type Domain_Attribute_BoolValue struct {
    BoolValue bool `protobuf:"varint,2,opt,name=bool_value,json=boolValue,proto3,oneof"`
}

// 属性整数值结构体
type Domain_Attribute_IntValue struct {
    IntValue int64 `protobuf:"varint,3,opt,name=int_value,json=intValue,proto3,oneof"`
}

// 实现属性布尔值类型化数值接口
func (*Domain_Attribute_BoolValue) isDomain_Attribute_TypedValue() {}

// 实现属性整数值类型化数值接口
func (*Domain_Attribute_IntValue) isDomain_Attribute_TypedValue() {}

// 文件描述符
var File_app_router_config_proto protoreflect.FileDescriptor

// 原始文件描述符的字节流
var file_app_router_config_proto_rawDesc = []byte{
    0x0a, 0x17, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2f, 0x63, 0x6f, 0x6e,
    0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
    0x1a, 0x15, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x70, 0x6f, 0x72,
    0x74, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f,
    0x6e, 0x65, 0x74, 0x2f, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x2e, 0x70, 0x72, 0x6f, 0x74,
}
    # 创建一个十六进制数值列表
    0x6f, 0x22, 0xbf, 0x02, 0x0a, 0x06, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x36, 0x0a, 0x04,
    # 创建一个类型为 int32 的字段，值为 1
    0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x22, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74,
    0x65, 0x72, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x52, 0x04,
    # 创建一个类型为 string 的字段，值为 "type"
    0x74, 0x79, 0x70, 0x65, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01,
    0x28, 0x09, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x12, 0x45, 0x0a, 0x09, 0x61, 0x74, 0x74, 0x72,
    0x69, 0x62, 0x75, 0x74, 0x65, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x27, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65,
    0x72, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x2e, 0x41, 0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74,
    0x65, 0x52, 0x09, 0x61, 0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x1a, 0x6c, 0x0a, 0x09, 0x41,
    0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x12, 0x10, 0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01,
    0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12, 0x1f, 0x0a, 0x0a, 0x62, 0x6f, 0x6f, 0x6c,
    0x5f, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x08, 0x48, 0x00, 0x52, 0x09, 0x62,
    0x6f, 0x6f, 0x6c, 0x56, 0x61, 0x6c, 0x75, 0x65, 0x12, 0x1d, 0x0a, 0x09, 0x69, 0x6e, 0x74, 0x5f, 0x76,
    0x61, 0x6c, 0x75, 0x65, 0x18, 0x03, 0x20, 0x01, 0x28, 0x03, 0x48, 0x00, 0x52, 0x08, 0x69, 0x6e, 0x74,
    0x56, 0x61, 0x6c, 0x75, 0x65, 0x42, 0x0d, 0x0a, 0x0b, 0x74, 0x79, 0x70, 0x65, 0x64, 0x5f, 0x76, 0x61,
    0x6c, 0x75, 0x65, 0x22, 0x32, 0x0a, 0x04, 0x54, 0x79, 0x70, 0x65, 0x12, 0x09, 0x0a, 0x05, 0x50, 0x6c,
    0x61, 0x69, 0x6e, 0x10, 0x00, 0x12, 0x09, 0x0a, 0x05, 0x52, 0x65, 0x67, 0x65, 0x78, 0x10, 0x01, 0x12,
    0x0a, 0x0a, 0x06, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x10, 0x02, 0x12, 0x08, 0x0a, 0x04, 0x46, 0x75,
    0x6c,
    # 创建一个十六进制数值为0x6c的整数
    0x6c, 
    # 创建一个十六进制数值为0x10的整数
    0x10, 
    # 创建一个十六进制数值为0x03的整数
    0x03, 
    # 创建一个十六进制数值为0x22的整数
    0x22, 
    # 创建一个十六进制数值为0x2e的整数
    0x2e, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x04的整数
    0x04, 
    # 创建一个十六进制数值为0x43的整数
    0x43, 
    # 创建一个十六进制数值为0x49的整数
    0x49, 
    # 创建一个十六进制数值为0x44的整数
    0x44, 
    # 创建一个十六进制数值为0x12的整数
    0x12, 
    # 创建一个十六进制数值为0x0e的整数
    0x0e, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x02的整数
    0x02, 
    # 创建一个十六进制数值为0x69的整数
    0x69,
    # 创建一个十六进制数值为0x70的整数
    0x70, 
    # 创建一个十六进制数值为0x18的整数
    0x18, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x20的整数
    0x20, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x28的整数
    0x28, 
    # 创建一个十六进制数值为0x0c的整数
    0x0c, 
    # 创建一个十六进制数值为0x52的整数
    0x52, 
    # 创建一个十六进制数值为0x02的整数
    0x02, 
    # 创建一个十六进制数值为0x69的整数
    0x69, 
    # 创建一个十六进制数值为0x70的整数
    0x70, 
    # 创建一个十六进制数值为0x12的整数
    0x12, 
    # 创建一个十六进制数值为0x16的整数
    0x16, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x06的整数
    0x06, 
    # 创建一个十六进制数值为0x70的整数
    0x70,
    # 创建一个十六进制数值为0x72的整数
    0x72, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x66的整数
    0x66, 
    # 创建一个十六进制数值为0x69的整数
    0x69, 
    # 创建一个十六进制数值为0x78的整数
    0x78, 
    # 创建一个十六进制数值为0x18的整数
    0x18, 
    # 创建一个十六进制数值为0x02的整数
    0x02, 
    # 创建一个十六进制数值为0x20的整数
    0x20, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x28的整数
    0x28, 
    # 创建一个十六进制数值为0x0d的整数
    0x0d, 
    # 创建一个十六进制数值为0x52的整数
    0x52, 
    # 创建一个十六进制数值为0x06的整数
    0x06, 
    # 创建一个十六进制数值为0x70的整数
    0x70, 
    # 创建一个十六进制数值为0x72的整数
    0x72, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x66的整数
    0x66, 
    # 创建一个十六进制数值为0x69的整数
    0x69, 
    # 创建一个十六进制数值为0x78的整数
    0x78, 
    # 创建一个十六进制数值为0x22的整数
    0x22, 
    # 创建一个十六进制数值为0x5b的整数
    0x5b, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x05的整数
    0x05, 
    # 创建一个十六进制数值为0x47的整数
    0x47, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x49的整数
    0x49, 
    # 创建一个十六进制数值为0x50的整数
    0x50, 
    # 创建一个十六进制数值为0x12的整数
    0x12, 
    # 创建一个十六进制数值为0x21的整数
    0x21, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x0c的整数
    0x0c, 
    # 创建一个十六进制数值为0x63的整数
    0x63, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x75的整数
    0x75, 
    # 创建一个十六进制数值为0x6e的整数
    0x6e, 
    # 创建一个十六进制数值为0x74的整数
    0x74, 
    # 创建一个十六进制数值为0x72的整数
    0x72, 
    # 创建一个十六进制数值为0x79的整数
    0x79, 
    # 创建一个十六进制数值为0x5f的整数
    0x5f, 
    # 创建一个十六进制数值为0x63的整数
    0x63, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x64的整数
    0x64, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x18的整数
    0x18, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x20的整数
    0x20, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x28的整数
    0x28, 
    # 创建一个十六进制数值为0x09的整数
    0x09, 
    # 创建一个十六进制数值为0x47的整数
    0x47, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x49的整数
    0x49, 
    # 创建一个十六进制数值为0x50的整数
    0x50, 
    # 创建一个十六进制数值为0x12的整数
    0x12, 
    # 创建一个十六进制数值为0x32的整数
    0x32, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x05的整数
    0x05, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x6e的整数
    0x6e, 
    # 创建一个十六进制数值为0x74的整数
    0x74, 
    # 创建一个十六进制数值为0x72的整数
    0x72, 
    # 创建一个十六进制数值为0x79的整数
    0x79, 
    # 创建一个十六进制数值为0x18的整数
    0x18, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x20的整数
    0x20, 
    # 创建一个十六进制数值为0x03的整数
    0x03, 
    # 创建一个十六进制数值为0x28的整数
    0x28, 
    # 创建一个十六进制数值为0x0b的整数
    0x0b, 
    # 创建一个十六进制数值为0x47的整数
    0x47, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x49的整数
    0x49, 
    # 创建一个十六进制数值为0x50的整数
    0x50, 
    # 创建一个十六进制数值为0x12的整数
    0x12, 
    # 创建一个十六进制数值为0x21的整数
    0x21, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x0c的整数
    0x0c, 
    # 创建一个十六进制数值为0x63的整数
    0x63, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x75的整数
    0x75, 
    # 创建一个十六进制数值为0x6e的整数
    0x6e, 
    # 创建一个十六进制数值为0x74的整数
    0x74, 
    # 创建一个十六进制数值为0x72的整数
    0x72, 
    # 创建一个十六进制数值为0x79的整数
    0x79, 
    # 创建一个十六进制数值为0x5f的整数
    0x5f, 
    # 创建一个十六进制数值为0x63的整数
    0x63, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x64的整数
    0x64, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x18的整数
    0x18, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x20的整数
    0x20, 
    # 创建一个十六进制数值为0x01的整数
    0x01, 
    # 创建一个十六进制数值为0x28的整数
    0x28, 
    # 创建一个十六进制数值为0x09的整数
    0x09, 
    # 创建一个十六进制数值为0x47的整数
    0x47, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x49的整数
    0x49, 
    # 创建一个十六进制数值为0x50的整数
    0x50, 
    # 创建一个十六进制数值为0x12的整数
    0x12, 
    # 创建一个十六进制数值为0x35的整数
    0x35, 
    # 创建一个十六进制数值为0x0a的整数
    0x0a, 
    # 创建一个十六进制数值为0x06的整数
    0x06, 
    # 创建一个十六进制数值为0x64的整数
    0x64, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x6d的整数
    0x6d, 
    # 创建一个十六进制数值为0x61的整数
    0x61, 
    # 创建一个十六进制数值为0x69的整数
    0x69, 
    # 创建一个十六进制数值为0x6e的整数
    0x6e, 
    # 创建一个十六进制数值为0x18的整数
    0x18, 
    # 创建一个十六进制数值为0x02的整数
    0x02, 
    # 创建一个十六进制数值为0x20的整数
    0x20, 
    # 创建一个十六进制数值为0x03的整数
    0x03, 
    # 创建一个十六进制数值为0x28的整数
    0x28, 
    # 创建一个十六进制数值为0x0b的整数
    0x0b, 
    # 创建一个十六进制数值为0x47的整数
    0x47, 
    # 创建一个十六进制数值为0x65的整数
    0x65, 
    # 创建一个十六进制数值为0x6f的整数
    0x6f, 
    # 创建一个十六进制数值为0x53的整数
    0x53, 
    # 创建一个十六进制数值为0x69的整数
    0x69, 
    # 创建一个十六进制数值为0x74的整数
    0x74,
    # 创建一个十六进制数值列表
    0x65, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x34, 0x0a, 0x05, 0x65, 0x6e, 0x74, 0x72, 0x79, 0x18, 0x01,
    0x20, 0x03, 0x28, 0x0b, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f,
    0x53, 0x69, 0x74, 0x65, 0x52, 0x05, 0x65, 0x6e, 0x74, 0x72, 0x79, 0x22, 0xca, 0x06, 0x0a, 0x0b,
    0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x12, 0x12, 0x0a, 0x03, 0x74,
    0x61, 0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x48, 0x00, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12,
    0x25, 0x0a, 0x0d, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x5f, 0x74, 0x61, 0x67,
    0x18, 0x0c, 0x20, 0x01, 0x28, 0x09, 0x48, 0x00, 0x52, 0x0c, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63,
    0x69, 0x6e, 0x67, 0x54, 0x61, 0x67, 0x12, 0x35, 0x0a, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e,
    0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x44,
    0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x33, 0x0a,
    0x04, 0x63, 0x69, 0x64, 0x72, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1b, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75,
    0x74, 0x65, 0x72, 0x2e, 0x43, 0x49, 0x44, 0x52, 0x42, 0x02, 0x18, 0x01, 0x52, 0x04, 0x63, 0x69,
    0x64, 0x72, 0x12, 0x32, 0x0a, 0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x18, 0x0a, 0x20, 0x03, 0x28,
    0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61,
    0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x52,
    0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x12, 0x43, 0x0a, 0x0a, 0x70, 0x6f, 0x72, 0x74, 0x5f, 0x72,
    0x61, 0x6e, 0x67, 0x65, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72,
    # 创建一个十六进制数组，表示一些数据
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e,
    0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65, 0x42, 0x02, 0x18, 0x01,
    # 创建一个新的对象
    52, 0x09, 0x70, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65, 0x12, 0x3c, 0x0a, 0x09, 0x70,
    # 设置属性值
    0x6f, 0x72, 0x74, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x0e, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1f,
    # 设置属性值
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
    0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x52,
    # 设置属性值
    0x08, 0x70, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x49, 0x0a, 0x0c, 0x6e, 0x65, 0x74,
    # 设置属性值
    0x77, 0x6f, 0x72, 0x6b, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28, 0x0b, 0x32,
    # 设置属性值
    0x22, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d,
    0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x4c,
    0x69, 0x73, 0x74, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0b, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b,
    0x4c, 0x69, 0x73, 0x74, 0x12, 0x3a, 0x0a, 0x08, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x73,
    # 设置属性值
    0x18, 0x0d, 0x20, 0x03, 0x28, 0x0e, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e,
    0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x52, 0x08, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x73,
    # 设置属性值
    0x12, 0x40, 0x0a, 0x0b, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x5f, 0x63, 0x69, 0x64, 0x72, 0x18,
    0x06, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x43, 0x49,
    0x44, 0x52, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0a, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x43, 0x69,
    0x64, 0x72, 0x12, 0x3f, 0x0a, 0x0c, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x5f, 0x67, 0x65, 0x6f,
    # 以十六进制表示的整数列表，可能是某种数据或配置信息
    0x69, 0x70, 0x18, 0x0b, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
    0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x52, 0x0b, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x47, 0x65,
    0x6f, 0x69, 0x70, 0x12, 0x49, 0x0a, 0x10, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x5f, 0x70, 0x6f,
    0x72, 0x74, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x10, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1f, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f,
    0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x52, 0x0e,
    0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x1d,
    0x0a, 0x0a, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x65, 0x6d, 0x61, 0x69, 0x6c, 0x18, 0x07, 0x20, 0x03,
    0x28, 0x09, 0x52, 0x09, 0x75, 0x73, 0x65, 0x72, 0x45, 0x6d, 0x61, 0x69, 0x6c, 0x12, 0x1f, 0x0a,
    0x0b, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x5f, 0x74, 0x61, 0x67, 0x18, 0x08, 0x20, 0x03,
    0x28, 0x09, 0x52, 0x0a, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x54, 0x61, 0x67, 0x12, 0x1a,
    0x0a, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x09, 0x20, 0x03, 0x28, 0x09,
    0x52, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x1e, 0x0a, 0x0a, 0x61, 0x74,
    0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x18, 0x0f, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0a,
    0x61, 0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x42, 0x0c, 0x0a, 0x0a, 0x74, 0x61,
    0x72, 0x67, 0x65, 0x74, 0x5f, 0x74, 0x61, 0x67, 0x22, 0x4e, 0x0a, 0x0d, 0x42, 0x61, 0x6c, 0x61,
    0x6e, 0x63, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61, 0x67,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12, 0x2b, 0x0a, 0x11, 0x6f,
    0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x5f, 0x73, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72,
    # 创建一个十六进制数值列表
    0x18, 0x02, 0x20, 0x03, 0x28, 0x09, 0x52, 0x10, 0x6f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64,
    0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x22, 0xad, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e,
    0x66, 0x69, 0x67, 0x12, 0x55, 0x0a, 0x0f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x5f, 0x73, 0x74,
    0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x2c, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
    0x75, 0x74, 0x65, 0x72, 0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x44, 0x6f, 0x6d, 0x61,
    0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x52, 0x0e, 0x64, 0x6f, 0x6d, 0x61,
    0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x36, 0x0a, 0x04, 0x72, 0x75,
    0x6c, 0x65, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x22, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
    0x2e, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x52, 0x04, 0x72, 0x75,
    0x6c, 0x65, 0x12, 0x4b, 0x0a, 0x0e, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x5f,
    0x72, 0x75, 0x6c, 0x65, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x24, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74,
    0x65, 0x72, 0x2e, 0x42, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65,
    0x52, 0x0d, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x22,
    0x47, 0x0a, 0x0e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67,
    0x79, 0x12, 0x08, 0x0a, 0x04, 0x41, 0x73, 0x49, 0x73, 0x10, 0x00, 0x12, 0x09, 0x0a, 0x05, 0x55,
    0x73, 0x65, 0x49, 0x70, 0x10, 0x01, 0x12, 0x10, 0x0a, 0x0c, 0x49, 0x70, 0x49, 0x66, 0x4e, 0x6f,
    0x6e, 0x4d, 0x61, 0x74, 0x63, 0x68, 0x10, 0x02, 0x12, 0x0e, 0x0a, 0x0a, 0x49, 0x70, 0x4f, 0x6e,
    # 这是一个十六进制的数据序列，可能是用于某种数据传输或存储的编码格式
    # 以下是具体的十六进制数据：
    0x44, 0x65, 0x6d, 0x61, 0x6e, 0x64, 0x10, 0x03, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72,
    0x6f, 0x75, 0x74, 0x65, 0x72, 0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74,
    0x65, 0x72, 0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
    0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x33,
// 定义变量 file_app_router_config_proto_rawDescOnce，类型为 sync.Once，用于确保 file_app_router_config_proto_rawDescData 只被初始化一次
var (
    file_app_router_config_proto_rawDescOnce sync.Once
    file_app_router_config_proto_rawDescData = file_app_router_config_proto_rawDesc
)

// 定义函数 file_app_router_config_proto_rawDescGZIP，返回类型为 []byte，用于对 file_app_router_config_proto_rawDescData 进行 GZIP 压缩
func file_app_router_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_app_router_config_proto_rawDescOnce.Do(func() {
        file_app_router_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_router_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的数据
    return file_app_router_config_proto_rawDescData
}

// 定义变量 file_app_router_config_proto_enumTypes，类型为 []protoimpl.EnumInfo，用于存储枚举类型信息
var file_app_router_config_proto_enumTypes = make([]protoimpl.EnumInfo, 2)
// 定义变量 file_app_router_config_proto_msgTypes，类型为 []protoimpl.MessageInfo，用于存储消息类型信息
var file_app_router_config_proto_msgTypes = make([]protoimpl.MessageInfo, 10)
// 定义变量 file_app_router_config_proto_goTypes，类型为 []interface{}，用于存储 Go 类型信息
var file_app_router_config_proto_goTypes = []interface{}{
    (Domain_Type)(0),           // 0: v2ray.core.app.router.Domain.Type
    (Config_DomainStrategy)(0), // 1: v2ray.core.app.router.Config.DomainStrategy
    (*Domain)(nil),             // 2: v2ray.core.app.router.Domain
    (*CIDR)(nil),               // 3: v2ray.core.app.router.CIDR
    (*GeoIP)(nil),              // 4: v2ray.core.app.router.GeoIP
    (*GeoIPList)(nil),          // 5: v2ray.core.app.router.GeoIPList
    (*GeoSite)(nil),            // 6: v2ray.core.app.router.GeoSite
    (*GeoSiteList)(nil),        // 7: v2ray.core.app.router.GeoSiteList
    (*RoutingRule)(nil),        // 8: v2ray.core.app.router.RoutingRule
    (*BalancingRule)(nil),      // 9: v2ray.core.app.router.BalancingRule
    (*Config)(nil),             // 10: v2ray.core.app.router.Config
    (*Domain_Attribute)(nil),   // 11: v2ray.core.app.router.Domain.Attribute
    (*net.PortRange)(nil),      // 12: v2ray.core.common.net.PortRange
    (*net.PortList)(nil),       // 13: v2ray.core.common.net.PortList
    (*net.NetworkList)(nil),    // 14: v2ray.core.common.net.NetworkList
    (net.Network)(0),           // 15: v2ray.core.common.net.Network
}
// 定义变量 file_app_router_config_proto_depIdxs，类型为 []int32，用于存储依赖索引信息
var file_app_router_config_proto_depIdxs = []int32{
    0,  // 0: v2ray.core.app.router.Domain.type:type_name -> v2ray.core.app.router.Domain.Type
    11, // 1: v2ray.core.app.router.Domain.attribute:type_name -> v2ray.core.app.router.Domain.Attribute
    // 定义编号为1的映射关系，将 v2ray.core.app.router.Domain.attribute 的类型名映射为 v2ray.core.app.router.Domain.Attribute

    3,  // 2: v2ray.core.app.router.GeoIP.cidr:type_name -> v2ray.core.app.router.CIDR
    // 定义编号为2的映射关系，将 v2ray.core.app.router.GeoIP.cidr 的类型名映射为 v2ray.core.app.router.CIDR

    4,  // 3: v2ray.core.app.router.GeoIPList.entry:type_name -> v2ray.core.app.router.GeoIP
    // 定义编号为3的映射关系，将 v2ray.core.app.router.GeoIPList.entry 的类型名映射为 v2ray.core.app.router.GeoIP

    2,  // 4: v2ray.core.app.router.GeoSite.domain:type_name -> v2ray.core.app.router.Domain
    // 定义编号为4的映射关系，将 v2ray.core.app.router.GeoSite.domain 的类型名映射为 v2ray.core.app.router.Domain

    6,  // 5: v2ray.core.app.router.GeoSiteList.entry:type_name -> v2ray.core.app.router.GeoSite
    // 定义编号为5的映射关系，将 v2ray.core.app.router.GeoSiteList.entry 的类型名映射为 v2ray.core.app.router.GeoSite

    2,  // 6: v2ray.core.app.router.RoutingRule.domain:type_name -> v2ray.core.app.router.Domain
    // 定义编号为6的映射关系，将 v2ray.core.app.router.RoutingRule.domain 的类型名映射为 v2ray.core.app.router.Domain

    3,  // 7: v2ray.core.app.router.RoutingRule.cidr:type_name -> v2ray.core.app.router.CIDR
    // 定义编号为7的映射关系，将 v2ray.core.app.router.RoutingRule.cidr 的类型名映射为 v2ray.core.app.router.CIDR

    4,  // 8: v2ray.core.app.router.RoutingRule.geoip:type_name -> v2ray.core.app.router.GeoIP
    // 定义编号为8的映射关系，将 v2ray.core.app.router.RoutingRule.geoip 的类型名映射为 v2ray.core.app.router.GeoIP

    12, // 9: v2ray.core.app.router.RoutingRule.port_range:type_name -> v2ray.core.common.net.PortRange
    // 定义编号为9的映射关系，将 v2ray.core.app.router.RoutingRule.port_range 的类型名映射为 v2ray.core.common.net.PortRange

    13, // 10: v2ray.core.app.router.RoutingRule.port_list:type_name -> v2ray.core.common.net.PortList
    // 定义编号为10的映射关系，将 v2ray.core.app.router.RoutingRule.port_list 的类型名映射为 v2ray.core.common.net.PortList

    14, // 11: v2ray.core.app.router.RoutingRule.network_list:type_name -> v2ray.core.common.net.NetworkList
    // 定义编号为11的映射关系，将 v2ray.core.app.router.RoutingRule.network_list 的类型名映射为 v2ray.core.common.net.NetworkList

    15, // 12: v2ray.core.app.router.RoutingRule.networks:type_name -> v2ray.core.common.net.Network
    // 定义编号为12的映射关系，将 v2ray.core.app.router.RoutingRule.networks 的类型名映射为 v2ray.core.common.net.Network

    3,  // 13: v2ray.core.app.router.RoutingRule.source_cidr:type_name -> v2ray.core.app.router.CIDR
    // 定义编号为13的映射关系，将 v2ray.core.app.router.RoutingRule.source_cidr 的类型名映射为 v2ray.core.app.router.CIDR

    4,  // 14: v2ray.core.app.router.RoutingRule.source_geoip:type_name -> v2ray.core.app.router.GeoIP
    // 定义编号为14的映射关系，将 v2ray.core.app.router.RoutingRule.source_geoip 的类型名映射为 v2ray.core.app.router.GeoIP

    13, // 15: v2ray.core.app.router.RoutingRule.source_port_list:type_name -> v2ray.core.common.net.PortList
    // 定义编号为15的映射关系，将 v2ray.core.app.router.RoutingRule.source_port_list 的类型名映射为 v2ray.core.common.net.PortList

    1,  // 16: v2ray.core.app.router.Config.domain_strategy:type_name -> v2ray.core.app.router.Config.DomainStrategy
    // 定义编号为16的映射关系，将 v2ray.core.app.router.Config.domain_strategy 的类型名映射为 v2ray.core.app.router.Config.DomainStrategy

    8,  // 17: v2ray.core.app.router.Config.rule:type_name -> v2ray.core.app.router.RoutingRule
    // 定义编号为17的映射关系，将 v2ray.core.app.router.Config.rule 的类型名映射为 v2ray.core.app.router.RoutingRule

    9,  // 18: v2ray.core.app.router.Config.balancing_rule:type_name -> v2ray.core.app.router.BalancingRule
    // 定义编号为18的映射关系，将 v2ray.core.app.router.Config.balancing_rule 的类型名映射为 v2ray.core.app.router.BalancingRule

    19, // [19:19] is the sub-list for method output_type
    // 定义编号为19的映射关系，用于方法的输出类型

    19, // [19:19] is the sub-list for method input_type
    // 定义编号为19的映射关系，用于方法的输入类型

    19, // [19:19] is the sub-list for extension type_name
    // 定义编号为19的映射关系，用于扩展类型名
    // 19, // [19:19] is the sub-list for extension extendee
    // 0,  // [0:19] is the sub-list for field type_name
func init() { file_app_router_config_proto_init() }
func file_app_router_config_proto_init() {
    // 如果 File_app_router_config_proto 不为空，则直接返回，不进行初始化操作
    if File_app_router_config_proto != nil {
        return
    }
    // 设置消息类型6的 OneofWrappers 属性为指定的接口数组
    file_app_router_config_proto_msgTypes[6].OneofWrappers = []interface{}{
        (*RoutingRule_Tag)(nil),
        (*RoutingRule_BalancingTag)(nil),
    }
    // 设置消息类型9的 OneofWrappers 属性为指定的接口数组
    file_app_router_config_proto_msgTypes[9].OneofWrappers = []interface{}{
        (*Domain_Attribute_BoolValue)(nil),
        (*Domain_Attribute_IntValue)(nil),
    }
    // 创建一个空结构体 x
    type x struct{}
    // 使用 protoimpl.TypeBuilder 构建类型描述符
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_app_router_config_proto_rawDesc,
            NumEnums:      2,
            NumMessages:   10,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_app_router_config_proto_goTypes,
        DependencyIndexes: file_app_router_config_proto_depIdxs,
        EnumInfos:         file_app_router_config_proto_enumTypes,
        MessageInfos:      file_app_router_config_proto_msgTypes,
    }.Build()
    // 将构建好的类型描述符赋值给 File_app_router_config_proto
    File_app_router_config_proto = out.File
    // 将原始描述符和类型信息清空，释放内存
    file_app_router_config_proto_rawDesc = nil
    file_app_router_config_proto_goTypes = nil
    file_app_router_config_proto_depIdxs = nil
}
```