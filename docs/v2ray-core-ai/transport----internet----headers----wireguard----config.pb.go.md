# `v2ray-core\transport\internet\headers\wireguard\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/headers/wireguard/config.proto

package wireguard

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

type WireguardConfig struct {
    state         protoimpl.MessageState  // 定义 WireguardConfig 结构体的状态
    sizeCache     protoimpl.SizeCache  // 定义 WireguardConfig 结构体的大小缓存
    unknownFields protoimpl.UnknownFields  // 定义 WireguardConfig 结构体的未知字段
}

func (x *WireguardConfig) Reset() {
    *x = WireguardConfig{}  // 重置 WireguardConfig 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_wireguard_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *WireguardConfig) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 WireguardConfig 结构体的字符串表示
}

func (*WireguardConfig) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *WireguardConfig) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_wireguard_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 WireguardConfig.ProtoReflect.Descriptor 代替。
func (*WireguardConfig) Descriptor() ([]byte, []int) {
    # 返回 file_transport_internet_headers_wireguard_config_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_transport_internet_headers_wireguard_config_proto_rawDescGZIP(), []int{0}
// 定义变量 File_transport_internet_headers_wireguard_config_proto，用于存储协议描述符
var File_transport_internet_headers_wireguard_config_proto protoreflect.FileDescriptor

// 定义变量 file_transport_internet_headers_wireguard_config_proto_rawDesc，存储协议描述符的原始字节数据
var file_transport_internet_headers_wireguard_config_proto_rawDesc = []byte{
    // 协议描述符的原始字节数据
    0x0a, 0x31, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2f, 0x77, 0x69, 0x72,
    0x65, 0x67, 0x75, 0x61, 0x72, 0x64, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x12, 0x2f, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e,
    0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x77, 0x69, 0x72, 0x65, 0x67,
    0x75, 0x61, 0x72, 0x64, 0x22, 0x11, 0x0a, 0x0f, 0x57, 0x69, 0x72, 0x65, 0x67, 0x75, 0x61, 0x72,
    0x64, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x42, 0x9e, 0x01, 0x0a, 0x33, 0x63, 0x6f, 0x6d, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73,
    0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65,
    0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x77, 0x69, 0x72, 0x65, 0x67, 0x75, 0x61, 0x72, 0x64, 0x50,
    0x01, 0x5a, 0x33, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72,
    0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2f, 0x77, 0x69, 0x72,
    0x65, 0x67, 0x75, 0x61, 0x72, 0x64, 0xaa, 0x02, 0x2f, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43,
    0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x57,
    0x69, 0x72, 0x65, 0x67, 0x75, 0x61, 0x72, 0x64, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

var (
    # 创建一个sync.Once对象，用于确保某个操作只执行一次
    file_transport_internet_headers_wireguard_config_proto_rawDescOnce sync.Once
    # 将file_transport_internet_headers_wireguard_config_proto_rawDesc的值赋给file_transport_internet_headers_wireguard_config_proto_rawDescData
    file_transport_internet_headers_wireguard_config_proto_rawDescData = file_transport_internet_headers_wireguard_config_proto_rawDesc
// 返回 Wireguard 配置文件的原始描述数据的 GZIP 压缩版本
func file_transport_internet_headers_wireguard_config_proto_rawDescGZIP() []byte {
    // 一次性执行函数，用于对原始描述数据进行 GZIP 压缩
    file_transport_internet_headers_wireguard_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_headers_wireguard_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_headers_wireguard_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的原始描述数据
    return file_transport_internet_headers_wireguard_config_proto_rawDescData
}

// 定义消息类型的信息数组
var file_transport_internet_headers_wireguard_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// 定义 Go 类型的接口数组
var file_transport_internet_headers_wireguard_config_proto_goTypes = []interface{}{
    (*WireguardConfig)(nil), // 0: v2ray.core.transport.internet.headers.wireguard.WireguardConfig
}
// 定义依赖索引数组
var file_transport_internet_headers_wireguard_config_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型名的子列表
    0, // [0:0] 是扩展的扩展对象的子列表
    0, // [0:0] 是字段类型名的子列表
}

// 初始化函数
func init() { file_transport_internet_headers_wireguard_config_proto_init() }
func file_transport_internet_headers_wireguard_config_proto_init() {
    // 如果文件已经初始化，则直接返回
    if File_transport_internet_headers_wireguard_config_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_headers_wireguard_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*WireguardConfig); i {
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
    // 定义一个空结构体
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 结构体实例
    out := protoimpl.TypeBuilder{
        # 设置 File 字段，包括 GoPackagePath、RawDescriptor、NumEnums、NumMessages、NumExtensions、NumServices
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_transport_internet_headers_wireguard_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        # 设置 GoTypes 字段
        GoTypes:           file_transport_internet_headers_wireguard_config_proto_goTypes,
        # 设置 DependencyIndexes 字段
        DependencyIndexes: file_transport_internet_headers_wireguard_config_proto_depIdxs,
        # 设置 MessageInfos 字段
        MessageInfos:      file_transport_internet_headers_wireguard_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_headers_wireguard_config_proto
    File_transport_internet_headers_wireguard_config_proto = out.File
    # 将 file_transport_internet_headers_wireguard_config_proto_rawDesc 置为 nil
    file_transport_internet_headers_wireguard_config_proto_rawDesc = nil
    # 将 file_transport_internet_headers_wireguard_config_proto_goTypes 置为 nil
    file_transport_internet_headers_wireguard_config_proto_goTypes = nil
    # 将 file_transport_internet_headers_wireguard_config_proto_depIdxs 置为 nil
    file_transport_internet_headers_wireguard_config_proto_depIdxs = nil
# 闭合前面的函数定义
```