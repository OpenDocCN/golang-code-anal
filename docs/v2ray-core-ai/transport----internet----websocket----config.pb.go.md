# `v2ray-core\transport\internet\websocket\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/websocket/config.proto

package websocket

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

// 这是一个编译时断言，用于确保使用足够更新的旧版 proto 包。
const _ = proto.ProtoPackageIsVersion4

type Header struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    Key   string `protobuf:"bytes,1,opt,name=key,proto3" json:"key,omitempty"`  // Key 字段
    Value string `protobuf:"bytes,2,opt,name=value,proto3" json:"value,omitempty"`  // Value 字段
}

func (x *Header) Reset() {
    *x = Header{}  // 重置 Header 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_websocket_config_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Header) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Header) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Header) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_websocket_config_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 Header.ProtoReflect.Descriptor 代替。
func (*Header) Descriptor() ([]byte, []int) {
    # 返回 file_transport_internet_websocket_config_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_transport_internet_websocket_config_proto_rawDescGZIP(), []int{0}
// 获取 Header 结构体的 Key 字段值
func (x *Header) GetKey() string {
    // 如果 x 不为 nil，则返回其 Key 字段值
    if x != nil {
        return x.Key
    }
    // 否则返回空字符串
    return ""
}

// 获取 Header 结构体的 Value 字段值
func (x *Header) GetValue() string {
    // 如果 x 不为 nil，则返回其 Value 字段值
    if x != nil {
        return x.Value
    }
    // 否则返回空字符串
    return ""
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // WebSocket 服务的 URL 路径。空值表示根路径(/)。
    Path                string    `protobuf:"bytes,2,opt,name=path,proto3" json:"path,omitempty"`
    // Header 结构体的切片
    Header              []*Header `protobuf:"bytes,3,rep,name=header,proto3" json:"header,omitempty"`
    // 是否接受代理协议
    AcceptProxyProtocol bool      `protobuf:"varint,4,opt,name=accept_proxy_protocol,json=acceptProxyProtocol,proto3" json:"accept_proxy_protocol,omitempty"`
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_websocket_config_proto_msgTypes[1]
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
    mi := &file_transport_internet_websocket_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 返回 Config 结构体的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_websocket_config_proto_rawDescGZIP(), []int{1}
}

// 获取 Config 结构体的 Path 字段值
func (x *Config) GetPath() string {
    // 如果 x 不为 nil，则返回其 Path 字段值
    if x != nil {
        return x.Path
    }
    // 否则返回空字符串
    return ""
}

// 获取 Config 结构体的 Header 字段值
func (x *Config) GetHeader() []*Header {
    // 如果 x 不为 nil，则返回其 Header 字段值
    if x != nil {
        return x.Header
    }
    // 否则返回空切片
    return nil
}

// 获取 Config 结构体的 AcceptProxyProtocol 字段值
func (x *Config) GetAcceptProxyProtocol() bool {
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的 AcceptProxyProtocol 属性
        return x.AcceptProxyProtocol
    # 如果 x 为空，返回 false
    return false
// 定义变量 File_transport_internet_websocket_config_proto，用于存储协议描述符
var File_transport_internet_websocket_config_proto protoreflect.FileDescriptor

// 定义变量 file_transport_internet_websocket_config_proto_rawDesc，存储协议描述符的原始字节数据
var file_transport_internet_websocket_config_proto_rawDesc = []byte{
    // 协议描述符的原始字节数据
    0x0a, 0x29, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
    0x72, 0x6e, 0x65, 0x74, 0x2f, 0x77, 0x65, 0x62, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0x2f, 0x63,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x27, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72,
    0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x77, 0x65, 0x62, 0x73, 0x6f,
    0x63, 0x6b, 0x65, 0x74, 0x22, 0x30, 0x0a, 0x06, 0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x12, 0x10,
    0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79,
    0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52,
    0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22, 0x9f, 0x01, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69,
    0x67, 0x12, 0x12, 0x0a, 0x04, 0x70, 0x61, 0x74, 0x68, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52,
    0x04, 0x70, 0x61, 0x74, 0x68, 0x12, 0x47, 0x0a, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x18,
    0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x2f, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74,
    0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x77, 0x65, 0x62, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0x2e,
    0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x52, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x12, 0x32,
    0x0a, 0x15, 0x61, 0x63, 0x63, 0x65, 0x70, 0x74, 0x5f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x5f, 0x70,
    0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x04, 0x20, 0x01, 0x28, 0x08, 0x52, 0x13, 0x61,
    0x63, 0x63, 0x65, 0x70, 0x74, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63,
    // 更多的协议描述符的原始字节数据
    # 这是一个十六进制数的列表，可能是用于某种数据加密或编码的参数
    0x6f, 0x6c, 0x4a, 0x04, 0x08, 0x01, 0x10, 0x02, 0x42, 0x86, 0x01, 0x0a, 0x2b, 0x63, 0x6f, 0x6d,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e,
    0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x77,
    0x65, 0x62, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0x50, 0x01, 0x5a, 0x2b, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73,
    0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x77, 0x65,
    0x62, 0x73, 0x6f, 0x63, 0x6b, 0x65, 0x74, 0xaa, 0x02, 0x27, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e,
    0x43, 0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49,
    0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x57, 0x65, 0x62, 0x73, 0x6f, 0x63, 0x6b, 0x65,
    0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
# 定义变量 file_transport_internet_websocket_config_proto_rawDescOnce，用于确保 file_transport_internet_websocket_config_proto_rawDescData 只被初始化一次
var (
    file_transport_internet_websocket_config_proto_rawDescOnce sync.Once
    file_transport_internet_websocket_config_proto_rawDescData = file_transport_internet_websocket_config_proto_rawDesc
)

# 定义函数 file_transport_internet_websocket_config_proto_rawDescGZIP，用于对原始描述数据进行 GZIP 压缩
func file_transport_internet_websocket_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_transport_internet_websocket_config_proto_rawDescOnce.Do(func() {
        # 对原始描述数据进行 GZIP 压缩，并赋值给 file_transport_internet_websocket_config_proto_rawDescData
        file_transport_internet_websocket_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_websocket_config_proto_rawDescData)
    })
    # 返回 GZIP 压缩后的数据
    return file_transport_internet_websocket_config_proto_rawDescData
}

# 定义变量 file_transport_internet_websocket_config_proto_msgTypes，存储消息类型信息
var file_transport_internet_websocket_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
# 定义变量 file_transport_internet_websocket_config_proto_goTypes，存储 Go 语言类型信息
var file_transport_internet_websocket_config_proto_goTypes = []interface{}{
    (*Header)(nil), // 0: v2ray.core.transport.internet.websocket.Header
    (*Config)(nil), // 1: v2ray.core.transport.internet.websocket.Config
}
# 定义变量 file_transport_internet_websocket_config_proto_depIdxs，存储依赖索引信息
var file_transport_internet_websocket_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.transport.internet.websocket.Config.header:type_name -> v2ray.core.transport.internet.websocket.Header
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

# 初始化函数，用于初始化 WebSocket 配置协议
func init() { file_transport_internet_websocket_config_proto_init() }
func file_transport_internet_websocket_config_proto_init() {
    # 如果文件已经被初始化，则直接返回，避免重复初始化
    if File_transport_internet_websocket_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_transport_internet_websocket_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*Header); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_transport_internet_websocket_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
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
            RawDescriptor: file_transport_internet_websocket_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_transport_internet_websocket_config_proto_goTypes,
        DependencyIndexes: file_transport_internet_websocket_config_proto_depIdxs,
        MessageInfos:      file_transport_internet_websocket_config_proto_msgTypes,
    }.Build()
    # 设置全局变量 File_transport_internet_websocket_config_proto 为构建的类型
    File_transport_internet_websocket_config_proto = out.File
    # 重置原始描述和 Go 类型
    file_transport_internet_websocket_config_proto_rawDesc = nil
    file_transport_internet_websocket_config_proto_goTypes = nil
    file_transport_internet_websocket_config_proto_depIdxs = nil
# 闭合前面的函数定义
```