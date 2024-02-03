# `v2ray-core\transport\internet\headers\wechat\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: transport/internet/headers/wechat/config.proto

package wechat

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保此生成的代码足够更新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够更新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type VideoConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

func (x *VideoConfig) Reset() {
    *x = VideoConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_wechat_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

func (x *VideoConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*VideoConfig) ProtoMessage() {}

func (x *VideoConfig) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_wechat_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用: 使用 VideoConfig.ProtoReflect.Descriptor 代替。
func (*VideoConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_wechat_config_proto_rawDescGZIP(), []int{0}
}
// 定义变量 File_transport_internet_headers_wechat_config_proto，用于存储协议描述符
var File_transport_internet_headers_wechat_config_proto protoreflect.FileDescriptor

// 定义变量 file_transport_internet_headers_wechat_config_proto_rawDesc，存储原始协议描述符的字节流
var file_transport_internet_headers_wechat_config_proto_rawDesc = []byte{
    // 字节流内容
    0x0a, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2f, 0x77, 0x65, 0x63,
    0x68, 0x61, 0x74, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x12, 0x2c, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61,
    0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e,
    0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x77, 0x65, 0x63, 0x68, 0x61, 0x74, 0x22, 0x0d,
    0x0a, 0x0b, 0x56, 0x69, 0x64, 0x65, 0x6f, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x42, 0x95, 0x01,
    0x0a, 0x30, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72,
    0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x77, 0x65, 0x63, 0x68,
    0x61, 0x74, 0x50, 0x01, 0x5a, 0x30, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f,
    0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69,
    0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2f,
    0x77, 0x65, 0x63, 0x68, 0x61, 0x74, 0xaa, 0x02, 0x2c, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43,
    0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x57,
    0x65, 0x63, 0x68, 0x61, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

// 定义变量 file_transport_internet_headers_wechat_config_proto_rawDescOnce，用于确保原始协议描述符只被初始化一次
var (
    file_transport_internet_headers_wechat_config_proto_rawDescOnce sync.Once
}
    # 将变量file_transport_internet_headers_wechat_config_proto_rawDescData赋值为file_transport_internet_headers_wechat_config_proto_rawDesc的值
    file_transport_internet_headers_wechat_config_proto_rawDescData = file_transport_internet_headers_wechat_config_proto_rawDesc
# 返回经过 GZIP 压缩的文件 transport_internet_headers_wechat_config.proto 的原始描述数据
func file_transport_internet_headers_wechat_config_proto_rawDescGZIP() []byte {
    # 一次性执行函数，对原始描述数据进行 GZIP 压缩
    file_transport_internet_headers_wechat_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_headers_wechat_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_headers_wechat_config_proto_rawDescData)
    })
    # 返回经过 GZIP 压缩的原始描述数据
    return file_transport_internet_headers_wechat_config_proto_rawDescData
}

# 定义消息类型切片和 Go 类型切片
var file_transport_internet_headers_wechat_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_transport_internet_headers_wechat_config_proto_goTypes = []interface{}{
    (*VideoConfig)(nil), // 0: v2ray.core.transport.internet.headers.wechat.VideoConfig
}

# 定义依赖索引切片
var file_transport_internet_headers_wechat_config_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型名称的子列表
    0, // [0:0] 是扩展的扩展对象的子列表
    0, // [0:0] 是字段类型名称的子列表
}

# 初始化函数
func init() { file_transport_internet_headers_wechat_config_proto_init() }
func file_transport_internet_headers_wechat_config_proto_init() {
    # 如果文件已经初始化，则直接返回
    if File_transport_internet_headers_wechat_config_proto != nil {
        return
    }
    # 如果不允许使用不安全的操作，则设置消息类型的导出器
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_headers_wechat_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*VideoConfig); i {
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
    # 定义一个空结构体
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_transport_internet_headers_wechat_config_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 1
            NumMessages:   1,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为指定的 Go 类型
        GoTypes:           file_transport_internet_headers_wechat_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_transport_internet_headers_wechat_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_transport_internet_headers_wechat_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_headers_wechat_config_proto
    File_transport_internet_headers_wechat_config_proto = out.File
    # 将 file_transport_internet_headers_wechat_config_proto_rawDesc 设置为 nil
    file_transport_internet_headers_wechat_config_proto_rawDesc = nil
    # 将 file_transport_internet_headers_wechat_config_proto_goTypes 设置为 nil
    file_transport_internet_headers_wechat_config_proto_goTypes = nil
    # 将 file_transport_internet_headers_wechat_config_proto_depIdxs 设置为 nil
    file_transport_internet_headers_wechat_config_proto_depIdxs = nil
# 闭合前面的函数定义
```