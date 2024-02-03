# `v2ray-core\proxy\dokodemo\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本：
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件：proxy/dokodemo/config.proto

package dokodemo

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    net "v2ray.com/core/common/net"  // 导入 net 包
)

const (
    // 确保此生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保正在使用足够新的旧版 proto 包。
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    Address *net.IPOrDomain `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`  // 地址字段
    Port    uint32          `protobuf:"varint,2,opt,name=port,proto3" json:"port,omitempty"`  // 端口字段
    // Dokodemo 接受的网络列表。
    // 已弃用。使用 networks。
    //
    // 已弃用：请勿使用。
    NetworkList *net.NetworkList `protobuf:"bytes,3,opt,name=network_list,json=networkList,proto3" json:"network_list,omitempty"`  // 网络列表字段
    // Dokodemo 接受的网络列表。
    Networks []net.Network `protobuf:"varint,7,rep,packed,name=networks,proto3,enum=v2ray.core.common.net.Network" json:"networks,omitempty"`  // 网络列表字段
    // 已弃用：请勿使用。
    Timeout        uint32 `protobuf:"varint,4,opt,name=timeout,proto3" json:"timeout,omitempty"`  // 超时字段
    FollowRedirect bool   `protobuf:"varint,5,opt,name=follow_redirect,json=followRedirect,proto3" json:"follow_redirect,omitempty"`  // 是否跟随重定向字段
    # 定义一个名为UserLevel的字段，类型为uint32，使用protobuf标记为可选字段，字段编号为6，JSON标记为user_level，如果字段为空则不序列化
    UserLevel      uint32 `protobuf:"varint,6,opt,name=user_level,json=userLevel,proto3" json:"user_level,omitempty"`
// 重置 Config 对象，将其内容置空
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则获取消息类型并存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_dokodemo_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Config 对象的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 返回 Config 对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_dokodemo_config_proto_msgTypes[0]
    // 如果启用了不安全操作并且 Config 对象不为空，则获取消息状态并存储消息信息
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
    return file_proxy_dokodemo_config_proto_rawDescGZIP(), []int{0}
}

// 获取 Config 对象的 Address 字段
func (x *Config) GetAddress() *net.IPOrDomain {
    if x != nil {
        return x.Address
    }
    return nil
}

// 获取 Config 对象的 Port 字段
func (x *Config) GetPort() uint32 {
    if x != nil {
        return x.Port
    }
    return 0
}

// Deprecated: 不要使用
func (x *Config) GetNetworkList() *net.NetworkList {
    if x != nil {
        return x.NetworkList
    }
    return nil
}

// 获取 Config 对象的 Networks 字段
func (x *Config) GetNetworks() []net.Network {
    if x != nil {
        return x.Networks
    }
    return nil
}

// Deprecated: 不要使用
func (x *Config) GetTimeout() uint32 {
    if x != nil {
        return x.Timeout
    }
    return 0
}

// 获取 Config 对象的 FollowRedirect 字段
func (x *Config) GetFollowRedirect() bool {
    if x != nil {
        return x.FollowRedirect
    }
    return false
}

// 获取 Config 对象的 UserLevel 字段
func (x *Config) GetUserLevel() uint32 {
    if x != nil {
        return x.UserLevel
    }
    return 0
}

// File_proxy_dokodemo_config_proto 的文件描述符
var File_proxy_dokodemo_config_proto protoreflect.FileDescriptor

// File_proxy_dokodemo_config_proto_rawDesc 的原始描述
var file_proxy_dokodemo_config_proto_rawDesc = []byte{
    0x0a, 0x1b, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x64, 0x6f, 0x6b, 0x6f, 0x64, 0x65, 0x6d, 0x6f,
}
    # 创建一个十六进制数列表，表示一些数据
    0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x19, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e,
    0x64, 0x6f, 0x6b, 0x6f, 0x64, 0x65, 0x6d, 0x6f, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e,
    0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x2e, 0x70, 0x72, 0x6f,
    0x74, 0x6f, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x6e,
    0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0xc6, 0x02, 0x0a,
    0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x3b, 0x0a, 0x07, 0x61, 0x64, 0x64, 0x72, 0x65,
    0x73, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74,
    0x2e, 0x49, 0x50, 0x4f, 0x72, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x07, 0x61, 0x64, 0x64,
    0x72, 0x65, 0x73, 0x73, 0x12, 0x12, 0x0a, 0x04, 0x70, 0x6f, 0x72, 0x74, 0x18, 0x02, 0x20, 0x01,
    0x28, 0x0d, 0x52, 0x04, 0x70, 0x6f, 0x72, 0x74, 0x12, 0x49, 0x0a, 0x0c, 0x6e, 0x65, 0x74, 0x77,
    0x6f, 0x72, 0x6b, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x22,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
    0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x4c, 0x69,
    0x73, 0x74, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0b, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x4c,
    0x69, 0x73, 0x74, 0x12, 0x3a, 0x0a, 0x08, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x73, 0x18,
    0x07, 0x20, 0x03, 0x28, 0x0e, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e, 0x65,
    0x74, 0x77, 0x6f, 0x72, 0x6b, 0x52, 0x08, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x73, 0x12,
    # 定义一个十六进制数列表
    0x1c, 0x0a, 0x07, 0x74, 0x69, 0x6d, 0x65, 0x6f, 0x75, 0x74, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0d,
    0x42, 0x02, 0x18, 0x01, 0x52, 0x07, 0x74, 0x69, 0x6d, 0x65, 0x6f, 0x75, 0x74, 0x12, 0x27, 0x0a,
    0x0f, 0x66, 0x6f, 0x6c, 0x6c, 0x6f, 0x77, 0x5f, 0x72, 0x65, 0x64, 0x69, 0x72, 0x65, 0x63, 0x74,
    0x18, 0x05, 0x20, 0x01, 0x28, 0x08, 0x52, 0x0e, 0x66, 0x6f, 0x6c, 0x6c, 0x6f, 0x77, 0x52, 0x65,
    0x64, 0x69, 0x72, 0x65, 0x63, 0x74, 0x12, 0x1d, 0x0a, 0x0a, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x6c,
    0x65, 0x76, 0x65, 0x6c, 0x18, 0x06, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x09, 0x75, 0x73, 0x65, 0x72,
    0x4c, 0x65, 0x76, 0x65, 0x6c, 0x42, 0x5c, 0x0a, 0x1d, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x64, 0x6f,
    0x6b, 0x6f, 0x64, 0x65, 0x6d, 0x6f, 0x50, 0x01, 0x5a, 0x1d, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x64,
    0x6f, 0x6b, 0x6f, 0x64, 0x65, 0x6d, 0xaa, 0x02, 0x19, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43,
    0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x44, 0x6f, 0x6b, 0x6f, 0x64, 0x65,
    0x6d, 0x6f, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义一个变量，用于确保 file_proxy_dokodemo_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_dokodemo_config_proto_rawDescOnce sync.Once
    file_proxy_dokodemo_config_proto_rawDescData = file_proxy_dokodemo_config_proto_rawDesc
)

// 定义一个函数，用于对 file_proxy_dokodemo_config_proto_rawDescData 进行 GZIP 压缩
func file_proxy_dokodemo_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_proxy_dokodemo_config_proto_rawDescOnce.Do(func() {
        file_proxy_dokodemo_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_dokodemo_config_proto_rawDescData)
    })
    return file_proxy_dokodemo_config_proto_rawDescData
}

// 定义消息类型的切片
var file_proxy_dokodemo_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// 定义 Go 类型的切片
var file_proxy_dokodemo_config_proto_goTypes = []interface{}{
    (*Config)(nil),          // 0: v2ray.core.proxy.dokodemo.Config
    (*net.IPOrDomain)(nil),  // 1: v2ray.core.common.net.IPOrDomain
    (*net.NetworkList)(nil), // 2: v2ray.core.common.net.NetworkList
    (net.Network)(0),        // 3: v2ray.core.common.net.Network
}
// 定义依赖索引的切片
var file_proxy_dokodemo_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.proxy.dokodemo.Config.address:type_name -> v2ray.core.common.net.IPOrDomain
    2, // 1: v2ray.core.proxy.dokodemo.Config.network_list:type_name -> v2ray.core.common.net.NetworkList
    3, // 2: v2ray.core.proxy.dokodemo.Config.networks:type_name -> v2ray.core.common.net.Network
    3, // [3:3] is the sub-list for method output_type
    3, // [3:3] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

// 初始化函数
func init() { file_proxy_dokodemo_config_proto_init() }
// 初始化 file_proxy_dokodemo_config_proto
func file_proxy_dokodemo_config_proto_init() {
    // 如果 File_proxy_dokodemo_config_proto 不为空，则直接返回
    if File_proxy_dokodemo_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置 file_proxy_dokodemo_config_proto_msgTypes[0].Exporter 函数
    if !protoimpl.UnsafeEnabled {
        file_proxy_dokodemo_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    # 定义一个空结构体 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_proxy_dokodemo_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_dokodemo_config_proto_goTypes,
        DependencyIndexes: file_proxy_dokodemo_config_proto_depIdxs,
        MessageInfos:      file_proxy_dokodemo_config_proto_msgTypes,
    }.Build()
    # 将构建的结果赋值给 File_proxy_dokodemo_config_proto
    File_proxy_dokodemo_config_proto = out.File
    # 清空 file_proxy_dokodemo_config_proto_rawDesc
    file_proxy_dokodemo_config_proto_rawDesc = nil
    # 清空 file_proxy_dokodemo_config_proto_goTypes
    file_proxy_dokodemo_config_proto_goTypes = nil
    # 清空 file_proxy_dokodemo_config_proto_depIdxs
    file_proxy_dokodemo_config_proto_depIdxs = nil
# 闭合前面的函数定义
```