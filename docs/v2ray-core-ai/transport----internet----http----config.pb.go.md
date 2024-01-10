# `v2ray-core\transport\internet\http\config.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/http/config.proto

package http

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    // 确保生成的代码足够新
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个编译时断言，用于确保使用足够新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

type Config struct {
    state         protoimpl.MessageState  // Config 对象的状态
    sizeCache     protoimpl.SizeCache  // Config 对象的大小缓存
    unknownFields protoimpl.UnknownFields  // Config 对象的未知字段

    Host []string `protobuf:"bytes,1,rep,name=host,proto3" json:"host,omitempty"`  // 主机列表
    Path string   `protobuf:"bytes,2,opt,name=path,proto3" json:"path,omitempty"`  // 路径
}

func (x *Config) Reset() {
    *x = Config{}  // 重置 Config 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_http_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Config 对象的字符串表示
}

func (*Config) ProtoMessage() {}  // Config 对象的 ProtoMessage 方法

func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_http_config_proto_msgTypes[0]  // 获取消息类型
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 废弃: 使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    # 返回 file_transport_internet_http_config_proto_rawDescGZIP() 函数的结果和空的整型数组
    return file_transport_internet_http_config_proto_rawDescGZIP(), []int{0}
// 获取配置中的主机列表
func (x *Config) GetHost() []string {
    // 如果配置不为空，则返回主机列表
    if x != nil {
        return x.Host
    }
    // 如果配置为空，则返回空列表
    return nil
}

// 获取配置中的路径
func (x *Config) GetPath() string {
    // 如果配置不为空，则返回路径
    if x != nil {
        return x.Path
    }
    // 如果配置为空，则返回空字符串
    return ""
}

// 定义协议缓冲文件描述符
var File_transport_internet_http_config_proto protoreflect.FileDescriptor

// 定义协议缓冲文件描述符的原始字节
var file_transport_internet_http_config_proto_rawDesc = []byte{
    // 这里是一段十六进制的字节流，表示协议缓冲文件描述符的原始内容
}

var (
    # 创建一个sync.Once对象，确保file_transport_internet_http_config_proto_rawDescData只被初始化一次
    file_transport_internet_http_config_proto_rawDescOnce sync.Once
    # 将file_transport_internet_http_config_proto_rawDescData初始化为file_transport_internet_http_config_proto_rawDesc的值
    file_transport_internet_http_config_proto_rawDescData = file_transport_internet_http_config_proto_rawDesc
// 返回经 GZIP 压缩的文件描述数据
func file_transport_internet_http_config_proto_rawDescGZIP() []byte {
    // 一次性执行 GZIP 压缩文件描述数据
    file_transport_internet_http_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_http_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_http_config_proto_rawDescData)
    })
    // 返回压缩后的文件描述数据
    return file_transport_internet_http_config_proto_rawDescData
}

// 消息类型信息数组
var file_transport_internet_http_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
// Go 语言类型数组
var file_transport_internet_http_config_proto_goTypes = []interface{}{
    (*Config)(nil), // 0: v2ray.core.transport.internet.http.Config
}
// 依赖索引数组
var file_transport_internet_http_config_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型的子列表
}

// 初始化函数
func init() { file_transport_internet_http_config_proto_init() }
func file_transport_internet_http_config_proto_init() {
    // 如果文件描述不为空，则返回
    if File_transport_internet_http_config_proto != nil {
        return
    }
    // 如果不允许使用不安全操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_transport_internet_http_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
    // 定义一个空结构体
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_transport_internet_http_config_proto_rawDesc,
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
        GoTypes:           file_transport_internet_http_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_transport_internet_http_config_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_transport_internet_http_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_transport_internet_http_config_proto
    File_transport_internet_http_config_proto = out.File
    # 将 file_transport_internet_http_config_proto_rawDesc 设置为 nil
    file_transport_internet_http_config_proto_rawDesc = nil
    # 将 file_transport_internet_http_config_proto_goTypes 设置为 nil
    file_transport_internet_http_config_proto_goTypes = nil
    # 将 file_transport_internet_http_config_proto_depIdxs 设置为 nil
    file_transport_internet_http_config_proto_depIdxs = nil
# 闭合前面的函数定义
```