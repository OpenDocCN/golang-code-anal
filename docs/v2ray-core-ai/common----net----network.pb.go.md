# `v2ray-core\common\net\network.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: common/net/network.proto

package net

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Network int32

const (
    Network_Unknown Network = 0  // 枚举值 Unknown
    // 已弃用: 请勿使用。
    Network_RawTCP Network = 1  // 枚举值 RawTCP
    Network_TCP    Network = 2  // 枚举值 TCP
    Network_UDP    Network = 3  // 枚举值 UDP
)

// 枚举值的映射关系。
var (
    Network_name = map[int32]string{
        0: "Unknown",
        1: "RawTCP",
        2: "TCP",
        3: "UDP",
    }
    Network_value = map[string]int32{
        "Unknown": 0,
        "RawTCP":  1,
        "TCP":     2,
        "UDP":     3,
    }
)

func (x Network) Enum() *Network {
    p := new(Network)
    *p = x
    return p
}

func (x Network) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Network) Descriptor() protoreflect.EnumDescriptor {
    return file_common_net_network_proto_enumTypes[0].Descriptor()
}

func (Network) Type() protoreflect.EnumType {
    return &file_common_net_network_proto_enumTypes[0]
}

func (x Network) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// 已弃用: 使用 Network.Descriptor 代替。
func (Network) EnumDescriptor() ([]byte, []int) {
    return file_common_net_network_proto_rawDescGZIP(), []int{0}
}
// NetworkList 是 Network 的列表
type NetworkList struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Network []Network `protobuf:"varint,1,rep,packed,name=network,proto3,enum=v2ray.core.common.net.Network" json:"network,omitempty"`
}

// 重置 NetworkList 对象
func (x *NetworkList) Reset() {
    *x = NetworkList{}
    if protoimpl.UnsafeEnabled {
        mi := &file_common_net_network_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 NetworkList 对象的字符串表示
func (x *NetworkList) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*NetworkList) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *NetworkList) ProtoReflect() protoreflect.Message {
    mi := &file_common_net_network_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用 NetworkList.ProtoReflect.Descriptor 替代
func (*NetworkList) Descriptor() ([]byte, []int) {
    return file_common_net_network_proto_rawDescGZIP(), []int{0}
}

// 获取 NetworkList 对象的 Network 列表
func (x *NetworkList) GetNetwork() []Network {
    if x != nil {
        return x.Network
    }
    return nil
}

var File_common_net_network_proto protoreflect.FileDescriptor

var file_common_net_network_proto_rawDesc = []byte{
    0x0a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x6e, 0x65, 0x74,
    0x77, 0x6f, 0x72, 0x6b, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65,
    0x74, 0x22, 0x47, 0x0a, 0x0b, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x4c, 0x69, 0x73, 0x74,
    0x12, 0x38, 0x0a, 0x07, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x18, 0x01, 0x20, 0x03, 0x28,
    # 以十六进制表示的字节序列，可能是某种数据或配置信息
    0x0e, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63,
    0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72,
    0x6b, 0x52, 0x07, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x2a, 0x38, 0x0a, 0x07, 0x4e, 0x65,
    0x74, 0x77, 0x6f, 0x72, 0x6b, 0x12, 0x0b, 0x0a, 0x07, 0x55, 0x6e, 0x6b, 0x6e, 0x6f, 0x77, 0x6e,
    0x10, 0x00, 0x12, 0x0e, 0x0a, 0x06, 0x52, 0x61, 0x77, 0x54, 0x43, 0x50, 0x10, 0x01, 0x1a, 0x02,
    0x08, 0x01, 0x12, 0x07, 0x0a, 0x03, 0x54, 0x43, 0x50, 0x10, 0x02, 0x12, 0x07, 0x0a, 0x03, 0x55,
    0x44, 0x50, 0x10, 0x03, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65,
    0x74, 0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
    0x6f, 0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0xaa, 0x02,
    0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f, 0x6d, 0x6d,
    0x6f, 0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保 file_common_net_network_proto_rawDescData 只被初始化一次
var (
    file_common_net_network_proto_rawDescOnce sync.Once
    file_common_net_network_proto_rawDescData = file_common_net_network_proto_rawDesc
)

// 使用 GZIP 压缩算法对 file_common_net_network_proto_rawDescData 进行压缩
func file_common_net_network_proto_rawDescGZIP() []byte {
    // 确保压缩操作只执行一次
    file_common_net_network_proto_rawDescOnce.Do(func() {
        file_common_net_network_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_net_network_proto_rawDescData)
    })
    return file_common_net_network_proto_rawDescData
}

// 定义枚举类型和消息类型的信息
var file_common_net_network_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_common_net_network_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_net_network_proto_goTypes = []interface{}{
    (Network)(0),        // 0: v2ray.core.common.net.Network
    (*NetworkList)(nil), // 1: v2ray.core.common.net.NetworkList
}
var file_common_net_network_proto_depIdxs = []int32{
    0, // 0: v2ray.core.common.net.NetworkList.network:type_name -> v2ray.core.common.net.Network
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数，确保文件未被初始化过
func init() { file_common_net_network_proto_init() }
func file_common_net_network_proto_init() {
    if File_common_net_network_proto != nil {
        return
    }
    if !protoimpl.UnsafeEnabled {
        // 设置消息类型的导出器
        file_common_net_network_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*NetworkList); i {
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
        # 设置 File 属性，包括 GoPackagePath、RawDescriptor、NumEnums、NumMessages、NumExtensions、NumServices
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 为 file_common_net_network_proto_rawDesc
            RawDescriptor: file_common_net_network_proto_rawDesc,
            # 设置 NumEnums 为 1
            NumEnums:      1,
            # 设置 NumMessages 为 1
            NumMessages:   1,
            # 设置 NumExtensions 为 0
            NumExtensions: 0,
            # 设置 NumServices 为 0
            NumServices:   0,
        },
        # 设置 GoTypes 为 file_common_net_network_proto_goTypes
        GoTypes:           file_common_net_network_proto_goTypes,
        # 设置 DependencyIndexes 为 file_common_net_network_proto_depIdxs
        DependencyIndexes: file_common_net_network_proto_depIdxs,
        # 设置 EnumInfos 为 file_common_net_network_proto_enumTypes
        EnumInfos:         file_common_net_network_proto_enumTypes,
        # 设置 MessageInfos 为 file_common_net_network_proto_msgTypes
        MessageInfos:      file_common_net_network_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_common_net_network_proto
    File_common_net_network_proto = out.File
    # 将 file_common_net_network_proto_rawDesc 置为 nil
    file_common_net_network_proto_rawDesc = nil
    # 将 file_common_net_network_proto_goTypes 置为 nil
    file_common_net_network_proto_goTypes = nil
    # 将 file_common_net_network_proto_depIdxs 置为 nil
    file_common_net_network_proto_depIdxs = nil
# 闭合前面的函数定义
```