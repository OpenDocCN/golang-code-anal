# `v2ray-core\common\protocol\server_spec.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: common/protocol/server_spec.proto

package protocol

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    net "v2ray.com/core/common/net"  // 导入 net 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

type ServerEndpoint struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    Address *net.IPOrDomain `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`  // 地址字段
    Port    uint32          `protobuf:"varint,2,opt,name=port,proto3" json:"port,omitempty"`  // 端口字段
    User    []*User         `protobuf:"bytes,3,rep,name=user,proto3" json:"user,omitempty"`  // 用户字段
}

func (x *ServerEndpoint) Reset() {
    *x = ServerEndpoint{}  // 重置 ServerEndpoint 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_common_protocol_server_spec_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态信息
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *ServerEndpoint) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*ServerEndpoint) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *ServerEndpoint) ProtoReflect() protoreflect.Message {
    mi := &file_common_protocol_server_spec_proto_msgTypes[0]  // 获取消息类型信息
    # 检查是否启用了不安全的功能，并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil:
        # 获取 x 对应的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 如果消息状态中没有加载消息信息，则存储消息信息
        if ms.LoadMessageInfo() == nil:
            ms.StoreMessageInfo(mi)
        # 返回消息状态
        return ms
    # 返回 x 对应的消息信息
    return mi.MessageOf(x)
// Deprecated: Use ServerEndpoint.ProtoReflect.Descriptor instead.
// 此方法已废弃，建议使用 ServerEndpoint.ProtoReflect.Descriptor 方法
func (*ServerEndpoint) Descriptor() ([]byte, []int) {
    return file_common_protocol_server_spec_proto_rawDescGZIP(), []int{0}
}

// 获取服务器端点的地址信息
func (x *ServerEndpoint) GetAddress() *net.IPOrDomain {
    if x != nil {
        return x.Address
    }
    return nil
}

// 获取服务器端点的端口号
func (x *ServerEndpoint) GetPort() uint32 {
    if x != nil {
        return x.Port
    }
    return 0
}

// 获取服务器端点的用户信息列表
func (x *ServerEndpoint) GetUser() []*User {
    if x != nil {
        return x.User
    }
    return nil
}

// 以下是文件描述符相关的变量和原始描述符的字节流
var File_common_protocol_server_spec_proto protoreflect.FileDescriptor

var file_common_protocol_server_spec_proto_rawDesc = []byte{
    // 这里是文件的原始描述符的字节流，省略部分内容
}
    # 以十六进制表示的字节序列，可能是某种数据或配置信息
    0x01, 0x28, 0x0d, 0x52, 0x04, 0x70, 0x6f, 0x72, 0x74, 0x12, 0x34, 0x0a, 0x04, 0x75, 0x73, 0x65,
    0x72, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74,
    0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65, 0x72, 0x42,
    0x5f, 0x0a, 0x1e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x50, 0x01, 0x5a, 0x1e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
    0x6f, 0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x63, 0x6f, 0x6c, 0xaa, 0x02, 0x1a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65,
    0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c,
    0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保只执行一次初始化操作
var (
    file_common_protocol_server_spec_proto_rawDescOnce sync.Once
    file_common_protocol_server_spec_proto_rawDescData = file_common_protocol_server_spec_proto_rawDesc
)

// 使用 GZIP 压缩原始描述数据
func file_common_protocol_server_spec_proto_rawDescGZIP() []byte {
    file_common_protocol_server_spec_proto_rawDescOnce.Do(func() {
        file_common_protocol_server_spec_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_protocol_server_spec_proto_rawDescData)
    })
    return file_common_protocol_server_spec_proto_rawDescData
}

// 定义消息类型切片和 Go 类型切片
var file_common_protocol_server_spec_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_protocol_server_spec_proto_goTypes = []interface{}{
    (*ServerEndpoint)(nil), // 0: v2ray.core.common.protocol.ServerEndpoint
    (*net.IPOrDomain)(nil), // 1: v2ray.core.common.net.IPOrDomain
    (*User)(nil),           // 2: v2ray.core.common.protocol.User
}

// 定义依赖索引切片
var file_common_protocol_server_spec_proto_depIdxs = []int32{
    1, // 0: v2ray.core.common.protocol.ServerEndpoint.address:type_name -> v2ray.core.common.net.IPOrDomain
    2, // 1: v2ray.core.common.protocol.ServerEndpoint.user:type_name -> v2ray.core.common.protocol.User
    2, // [2:2] is the sub-list for method output_type
    2, // [2:2] is the sub-list for method input_type
    2, // [2:2] is the sub-list for extension type_name
    2, // [2:2] is the sub-list for extension extendee
    0, // [0:2] is the sub-list for field type_name
}

// 初始化函数，如果文件已经初始化，则直接返回
func init() { file_common_protocol_server_spec_proto_init() }
func file_common_protocol_server_spec_proto_init() {
    if File_common_protocol_server_spec_proto != nil {
        return
    }
    file_common_protocol_user_proto_init()
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_common_protocol_server_spec_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            # 根据不同的索引返回不同的状态
            switch v := v.(*ServerEndpoint); i:
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
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),  # 获取结构体 x 的包路径
            RawDescriptor: file_common_protocol_server_spec_proto_rawDesc,  # 使用原始描述符
            NumEnums:      0,  # 枚举数量为 0
            NumMessages:   1,  # 消息数量为 1
            NumExtensions: 0,  # 扩展数量为 0
            NumServices:   0,  # 服务数量为 0
        },
        GoTypes:           file_common_protocol_server_spec_proto_goTypes,  # 使用的 Go 类型
        DependencyIndexes: file_common_protocol_server_spec_proto_depIdxs,  # 依赖索引
        MessageInfos:      file_common_protocol_server_spec_proto_msgTypes,  # 消息信息
    }.Build()  # 构建类型
    
    # 将构建的结果赋值给 File_common_protocol_server_spec_proto
    File_common_protocol_server_spec_proto = out.File
    file_common_protocol_server_spec_proto_rawDesc = nil  # 清空原始描述符
    file_common_protocol_server_spec_proto_goTypes = nil  # 清空 Go 类型
    file_common_protocol_server_spec_proto_depIdxs = nil  # 清空依赖索引
# 闭合前面的函数定义
```