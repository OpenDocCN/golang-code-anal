# `v2ray-core\common\net\destination.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: common/net/destination.proto

package net

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

// 这是一个编译时断言，用于确保使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// 网络连接的端点。
type Endpoint struct {
    state         protoimpl.MessageState  // 状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    Network Network     `protobuf:"varint,1,opt,name=network,proto3,enum=v2ray.core.common.net.Network" json:"network,omitempty"`  // 网络类型
    Address *IPOrDomain `protobuf:"bytes,2,opt,name=address,proto3" json:"address,omitempty"`  // IP 或域名
    Port    uint32      `protobuf:"varint,3,opt,name=port,proto3" json:"port,omitempty"`  // 端口
}

func (x *Endpoint) Reset() {
    *x = Endpoint{}  // 重置 Endpoint 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_common_net_destination_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Endpoint) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Endpoint) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Endpoint) ProtoReflect() protoreflect.Message {
    mi := &file_common_net_destination_proto_msgTypes[0]  // 获取消息类型信息
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
// Deprecated: Use Endpoint.ProtoReflect.Descriptor instead.
// 描述符方法，返回文件描述符的原始字节和路径
func (*Endpoint) Descriptor() ([]byte, []int) {
    return file_common_net_destination_proto_rawDescGZIP(), []int{0}
}

// 获取网络类型
func (x *Endpoint) GetNetwork() Network {
    if x != nil {
        return x.Network
    }
    return Network_Unknown
}

// 获取地址
func (x *Endpoint) GetAddress() *IPOrDomain {
    if x != nil {
        return x.Address
    }
    return nil
}

// 获取端口号
func (x *Endpoint) GetPort() uint32 {
    if x != nil {
        return x.Port
    }
    return 0
}

// 文件描述符
var File_common_net_destination_proto protoreflect.FileDescriptor

// 原始文件描述符的字节
var file_common_net_destination_proto_rawDesc = []byte{
    // 这里是一段很长的原始文件描述符的字节，省略部分内容
}
    # 以十六进制表示的字节码序列
    0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x49, 0x50, 0x4f, 0x72, 0x44, 0x6f,
    0x6d, 0x61, 0x69, 0x6e, 0x52, 0x07, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x12, 0x12, 0x0a,
    0x04, 0x70, 0x6f, 0x72, 0x74, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x04, 0x70, 0x6f, 0x72,
    0x74, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x50, 0x01,
    0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65,
    0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0xaa, 0x02, 0x15, 0x56, 0x32,
    0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
    0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义一个全局变量，用于确保 file_common_net_destination_proto_rawDescData 只被初始化一次
var (
    file_common_net_destination_proto_rawDescOnce sync.Once
    file_common_net_destination_proto_rawDescData = file_common_net_destination_proto_rawDesc
)

// 定义一个函数，用于对 file_common_net_destination_proto_rawDescData 进行 GZIP 压缩
func file_common_net_destination_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_common_net_destination_proto_rawDescOnce.Do(func() {
        file_common_net_destination_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_net_destination_proto_rawDescData)
    })
    return file_common_net_destination_proto_rawDescData
}

// 定义消息类型的切片和 Go 类型的切片
var file_common_net_destination_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_net_destination_proto_goTypes = []interface{}{
    (*Endpoint)(nil),   // 0: v2ray.core.common.net.Endpoint
    (Network)(0),       // 1: v2ray.core.common.net.Network
    (*IPOrDomain)(nil), // 2: v2ray.core.common.net.IPOrDomain
}

// 定义依赖索引的切片
var file_common_net_destination_proto_depIdxs = []int32{
    1, // 0: v2ray.core.common.net.Endpoint.network:type_name -> v2ray.core.common.net.Network
    2, // 1: v2ray.core.common.net.Endpoint.address:type_name -> v2ray.core.common.net.IPOrDomain
    2, // [2:2] is the sub-list for method output_type
    2, // [2:2] is the sub-list for method input_type
    2, // [2:2] is the sub-list for extension type_name
    2, // [2:2] is the sub-list for extension extendee
    0, // [0:2] is the sub-list for field type_name
}

// 初始化函数，用于确保文件未被初始化
func init() { file_common_net_destination_proto_init() }
func file_common_net_destination_proto_init() {
    if File_common_net_destination_proto != nil {
        return
    }
    file_common_net_network_proto_init()
    file_common_net_address_proto_init()
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_common_net_destination_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Endpoint); i {
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
            RawDescriptor: file_common_net_destination_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_common_net_destination_proto_goTypes,
        DependencyIndexes: file_common_net_destination_proto_depIdxs,
        MessageInfos:      file_common_net_destination_proto_msgTypes,
    }.Build()

    # 设置 File_common_net_destination_proto 为构建的类型
    File_common_net_destination_proto = out.File

    # 重置原始描述、Go 类型和依赖索引
    file_common_net_destination_proto_rawDesc = nil
    file_common_net_destination_proto_goTypes = nil
    file_common_net_destination_proto_depIdxs = nil
# 闭合前面的函数定义
```