# `v2ray-core\common\net\address.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: common/net/address.proto

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

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// 网络主机的地址。可以是 IP 地址或域地址。
type IPOrDomain struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    // 可分配给 Address 的类型:
    //    *IPOrDomain_Ip
    //    *IPOrDomain_Domain
    Address isIPOrDomain_Address `protobuf_oneof:"address"`  // isIPOrDomain_Address 接口
}

func (x *IPOrDomain) Reset() {
    *x = IPOrDomain{}  // 重置 IPOrDomain 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_common_net_address_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *IPOrDomain) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*IPOrDomain) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *IPOrDomain) ProtoReflect() protoreflect.Message {
    mi := &file_common_net_address_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)  // 返回消息对象
}
// Deprecated: Use IPOrDomain.ProtoReflect.Descriptor instead.
// 返回文件描述符的原始字节和路径
func (*IPOrDomain) Descriptor() ([]byte, []int) {
    return file_common_net_address_proto_rawDescGZIP(), []int{0}
}

// 返回地址的接口类型
func (m *IPOrDomain) GetAddress() isIPOrDomain_Address {
    if m != nil {
        return m.Address
    }
    return nil
}

// 返回 IP 地址
func (x *IPOrDomain) GetIp() []byte {
    if x, ok := x.GetAddress().(*IPOrDomain_Ip); ok {
        return x.Ip
    }
    return nil
}

// 返回域名
func (x *IPOrDomain) GetDomain() string {
    if x, ok := x.GetAddress().(*IPOrDomain_Domain); ok {
        return x.Domain
    }
    return ""
}

// 定义 IPOrDomain_Address 的接口类型
type isIPOrDomain_Address interface {
    isIPOrDomain_Address()
}

// 定义 IPOrDomain_Ip 结构体，包含 IP 地址
type IPOrDomain_Ip struct {
    // IP address. Must by either 4 or 16 bytes.
    Ip []byte `protobuf:"bytes,1,opt,name=ip,proto3,oneof"`
}

// 定义 IPOrDomain_Domain 结构体，包含域名
type IPOrDomain_Domain struct {
    // Domain address.
    Domain string `protobuf:"bytes,2,opt,name=domain,proto3,oneof"`
}

// 实现 IPOrDomain_Ip 结构体的接口方法
func (*IPOrDomain_Ip) isIPOrDomain_Address() {}

// 实现 IPOrDomain_Domain 结构体的接口方法
func (*IPOrDomain_Domain) isIPOrDomain_Address() {}

// 定义全局变量 File_common_net_address_proto
var File_common_net_address_proto protoreflect.FileDescriptor

// 定义全局变量 file_common_net_address_proto_rawDesc，包含文件描述符的原始字节
var file_common_net_address_proto_rawDesc = []byte{
    // 省略部分内容
}
    # 这是一系列十六进制数，可能是某种编码或者数据表示方式，但无法确定具体含义
    # 无法确定这些数字的作用，需要更多上下文才能解释其含义
// 定义一个变量，用于确保 file_common_net_address_proto_rawDescData 只被初始化一次
var (
    file_common_net_address_proto_rawDescOnce sync.Once
    file_common_net_address_proto_rawDescData = file_common_net_address_proto_rawDesc
)

// 定义一个函数，用于对 file_common_net_address_proto_rawDescData 进行 GZIP 压缩
func file_common_net_address_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_common_net_address_proto_rawDescOnce.Do(func() {
        file_common_net_address_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_net_address_proto_rawDescData)
    })
    return file_common_net_address_proto_rawDescData
}

// 定义消息类型的切片和 Go 类型的切片
var file_common_net_address_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_net_address_proto_goTypes = []interface{}{
    (*IPOrDomain)(nil), // 0: v2ray.core.common.net.IPOrDomain
}

// 定义依赖索引的切片
var file_common_net_address_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型的子列表
}

// 初始化函数
func init() { file_common_net_address_proto_init() }
func file_common_net_address_proto_init() {
    // 如果 File_common_net_address_proto 不为空，则直接返回
    if File_common_net_address_proto != nil {
        return
    }
    // 如果 protoimpl.UnsafeEnabled 未启用，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_common_net_address_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*IPOrDomain); i {
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
    // 设置消息类型的 OneofWrappers
    file_common_net_address_proto_msgTypes[0].OneofWrappers = []interface{}{
        (*IPOrDomain_Ip)(nil),
        (*IPOrDomain_Domain)(nil),
    }
    // 定义一个空结构体 x
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为 file_common_net_address_proto_rawDesc
            RawDescriptor: file_common_net_address_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 1
            NumMessages:   1,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为 file_common_net_address_proto_goTypes
        GoTypes:           file_common_net_address_proto_goTypes,
        # 设置 DependencyIndexes 属性为 file_common_net_address_proto_depIdxs
        DependencyIndexes: file_common_net_address_proto_depIdxs,
        # 设置 MessageInfos 属性为 file_common_net_address_proto_msgTypes
        MessageInfos:      file_common_net_address_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_common_net_address_proto
    File_common_net_address_proto = out.File
    # 将 file_common_net_address_proto_rawDesc 设置为 nil
    file_common_net_address_proto_rawDesc = nil
    # 将 file_common_net_address_proto_goTypes 设置为 nil
    file_common_net_address_proto_goTypes = nil
    # 将 file_common_net_address_proto_depIdxs 设置为 nil
    file_common_net_address_proto_depIdxs = nil
# 闭合前面的函数定义
```