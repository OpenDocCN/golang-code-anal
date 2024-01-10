# `v2ray-core\proxy\blackhole\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/blackhole/config.proto

package blackhole

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

const (
    // 确保此生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用的是足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type NoneResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

func (x *NoneResponse) Reset() {
    *x = NoneResponse{}
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_blackhole_config_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

func (x *NoneResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*NoneResponse) ProtoMessage() {}

func (x *NoneResponse) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_blackhole_config_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 NoneResponse.ProtoReflect.Descriptor 代替。
func (*NoneResponse) Descriptor() ([]byte, []int) {
    return file_proxy_blackhole_config_proto_rawDescGZIP(), []int{0}
}

type HTTPResponse struct {
    # 定义变量 state，用于存储 protoimpl.MessageState 对象
    state         protoimpl.MessageState
    # 定义变量 sizeCache，用于存储 protoimpl.SizeCache 对象
    sizeCache     protoimpl.SizeCache
    # 定义变量 unknownFields，用于存储 protoimpl.UnknownFields 对象
    unknownFields protoimpl.UnknownFields
// 重置 HTTPResponse 对象，将其重置为空对象
func (x *HTTPResponse) Reset() {
    *x = HTTPResponse{}
    // 如果启用了不安全操作，则将消息信息存储到指针 x 的消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_blackhole_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 HTTPResponse 对象的字符串表示形式
func (x *HTTPResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*HTTPResponse) ProtoMessage() {}

// 返回 HTTPResponse 对象的反射信息
func (x *HTTPResponse) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_blackhole_config_proto_msgTypes[1]
    // 如果启用了不安全操作并且 x 不为空，则将消息信息存储到指针 x 的消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 HTTPResponse.ProtoReflect.Descriptor 替代
func (*HTTPResponse) Descriptor() ([]byte, []int) {
    return file_proxy_blackhole_config_proto_rawDescGZIP(), []int{1}
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Response *serial.TypedMessage `protobuf:"bytes,1,opt,name=response,proto3" json:"response,omitempty"`
}

// 重置 Config 对象，将其重置为空对象
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则将消息信息存储到指针 x 的消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_blackhole_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Config 对象的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*Config) ProtoMessage() {}

// 返回 Config 对象的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_blackhole_config_proto_msgTypes[2]
    // 如果启用了不安全操作并且 x 不为空，则将消息信息存储到指针 x 的消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 Config.ProtoReflect.Descriptor 替代
# 返回配置文件的原始描述和一个整数切片
func (*Config) Descriptor() ([]byte, []int) {
    return file_proxy_blackhole_config_proto_rawDescGZIP(), []int{2}
}

# 返回响应消息的类型化消息
func (x *Config) GetResponse() *serial.TypedMessage {
    if x != nil {
        return x.Response
    }
    return nil
}

# 定义文件描述符
var File_proxy_blackhole_config_proto protoreflect.FileDescriptor

# 定义原始描述的字节切片
var file_proxy_blackhole_config_proto_rawDesc = []byte{
    # 以下为字节码，用于描述文件结构
}
    # 创建一个十六进制数的序列，可能是用于某种特定的编码或者数据表示方式
    0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x62, 0x6c, 0x61, 0x63, 0x6b, 0x68, 0x6f, 0x6c, 0x65, 0xaa,
    0x02, 0x1a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f,
    0x78, 0x79, 0x2e, 0x42, 0x6c, 0x61, 0x63, 0x6b, 0x68, 0x6f, 0x6c, 0x65, 0x62, 0x06, 0x70, 0x72,
    0x6f, 0x74, 0x6f, 0x33,
# 定义变量 file_proxy_blackhole_config_proto_rawDescOnce，用于确保 file_proxy_blackhole_config_proto_rawDescData 只被初始化一次
var (
    file_proxy_blackhole_config_proto_rawDescOnce sync.Once
    file_proxy_blackhole_config_proto_rawDescData = file_proxy_blackhole_config_proto_rawDesc
)

# 定义函数 file_proxy_blackhole_config_proto_rawDescGZIP，用于将 file_proxy_blackhole_config_proto_rawDescData 压缩为 GZIP 格式的字节数组
func file_proxy_blackhole_config_proto_rawDescGZIP() []byte {
    # 使用 sync.Once 确保只有一个 goroutine 执行压缩操作
    file_proxy_blackhole_config_proto_rawDescOnce.Do(func() {
        # 调用 protoimpl.X.CompressGZIP 方法对 file_proxy_blackhole_config_proto_rawDescData 进行 GZIP 压缩
        file_proxy_blackhole_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_blackhole_config_proto_rawDescData)
    })
    # 返回 GZIP 压缩后的字节数组
    return file_proxy_blackhole_config_proto_rawDescData
}

# 定义变量 file_proxy_blackhole_config_proto_msgTypes，存储消息类型信息
var file_proxy_blackhole_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
# 定义变量 file_proxy_blackhole_config_proto_goTypes，存储 Go 类型信息
var file_proxy_blackhole_config_proto_goTypes = []interface{}{
    (*NoneResponse)(nil),        // 0: v2ray.core.proxy.blackhole.NoneResponse
    (*HTTPResponse)(nil),        // 1: v2ray.core.proxy.blackhole.HTTPResponse
    (*Config)(nil),              // 2: v2ray.core.proxy.blackhole.Config
    (*serial.TypedMessage)(nil), // 3: v2ray.core.common.serial.TypedMessage
}
# 定义变量 file_proxy_blackhole_config_proto_depIdxs，存储依赖索引信息
var file_proxy_blackhole_config_proto_depIdxs = []int32{
    3, // 0: v2ray.core.proxy.blackhole.Config.response:type_name -> v2ray.core.common.serial.TypedMessage
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

# 初始化函数，用于初始化文件代理黑洞配置
func init() { file_proxy_blackhole_config_proto_init() }
# 初始化文件代理黑洞配置的函数，用于确保只初始化一次
func file_proxy_blackhole_config_proto_init() {
    # 如果 File_proxy_blackhole_config_proto 不为空，则直接返回，避免重复初始化
    if File_proxy_blackhole_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_proxy_blackhole_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*NoneResponse); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_blackhole_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*HTTPResponse); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_proxy_blackhole_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
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
            RawDescriptor: file_proxy_blackhole_config_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   3,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_proxy_blackhole_config_proto_goTypes,
        DependencyIndexes: file_proxy_blackhole_config_proto_depIdxs,
        MessageInfos:      file_proxy_blackhole_config_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_proxy_blackhole_config_proto
    File_proxy_blackhole_config_proto = out.File
    # 将原始描述和 Go 类型设置为 nil
    file_proxy_blackhole_config_proto_rawDesc = nil
    file_proxy_blackhole_config_proto_goTypes = nil
    file_proxy_blackhole_config_proto_depIdxs = nil
# 闭合前面的函数定义
```