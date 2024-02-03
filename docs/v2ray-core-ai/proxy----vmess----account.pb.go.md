# `v2ray-core\proxy\vmess\account.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: proxy/vmess/account.proto

package vmess

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

const (
    // 确保生成的代码足够新。
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
    // 确保 runtime/protoimpl 足够新。
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// 这是一个在编译时断言的声明，用于确保使用足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Account struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    // 账户的 ID，以 UUID 的形式，例如 "66ad4540-b58c-4ad2-9926-ea63445a9b57"。
    Id string `protobuf:"bytes,1,opt,name=id,proto3" json:"id,omitempty"`  // ID 字段
    // 备用 ID 的数量。客户端和服务器必须共享相同的数量。
    AlterId uint32 `protobuf:"varint,2,opt,name=alter_id,json=alterId,proto3" json:"alter_id,omitempty"`  // AlterId 字段
    // 安全设置。仅适用于客户端端。
    SecuritySettings *protocol.SecurityConfig `protobuf:"bytes,3,opt,name=security_settings,json=securitySettings,proto3" json:"security_settings,omitempty"`  // 安全设置字段
    // 为此账户定义启用的测试
    TestsEnabled string `protobuf:"bytes,4,opt,name=tests_enabled,json=testsEnabled,proto3" json:"tests_enabled,omitempty"`  // TestsEnabled 字段
}

func (x *Account) Reset() {
    *x = Account{}  // 重置 Account 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vmess_account_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}
// 定义 Account 结构体的 String 方法，返回 Account 对象的字符串表示形式
func (x *Account) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 定义 Account 结构体的 ProtoMessage 方法
func (*Account) ProtoMessage() {}

// 定义 Account 结构体的 ProtoReflect 方法，返回 Message 对象
func (x *Account) ProtoReflect() protoreflect.Message {
    // 获取 Account 对象的消息类型信息
    mi := &file_proxy_vmess_account_proto_msgTypes[0]
    // 如果启用了不安全操作，并且 Account 对象不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取 Account 对象的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息状态中没有加载消息信息，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        // 返回消息状态
        return ms
    }
    // 返回消息对象
    return mi.MessageOf(x)
}

// Deprecated: Use Account.ProtoReflect.Descriptor instead.
// 定义 Account 结构体的 Descriptor 方法，返回原始描述符的 GZIP 压缩数据和偏移量
func (*Account) Descriptor() ([]byte, []int) {
    return file_proxy_vmess_account_proto_rawDescGZIP(), []int{0}
}

// 定义 Account 结构体的 GetId 方法，返回 Account 对象的 Id 属性
func (x *Account) GetId() string {
    if x != nil {
        return x.Id
    }
    return ""
}

// 定义 Account 结构体的 GetAlterId 方法，返回 Account 对象的 AlterId 属性
func (x *Account) GetAlterId() uint32 {
    if x != nil {
        return x.AlterId
    }
    return 0
}

// 定义 Account 结构体的 GetSecuritySettings 方法，返回 Account 对象的 SecuritySettings 属性
func (x *Account) GetSecuritySettings() *protocol.SecurityConfig {
    if x != nil {
        return x.SecuritySettings
    }
    return nil
}

// 定义 Account 结构体的 GetTestsEnabled 方法，返回 Account 对象的 TestsEnabled 属性
func (x *Account) GetTestsEnabled() string {
    if x != nil {
        return x.TestsEnabled
    }
    return ""
}

// 定义 File_proxy_vmess_account_proto 变量，表示文件描述符
var File_proxy_vmess_account_proto protoreflect.FileDescriptor

// 定义 file_proxy_vmess_account_proto_rawDesc 变量，存储原始描述符的 GZIP 压缩数据
var file_proxy_vmess_account_proto_rawDesc = []byte{
    // 压缩数据
}
    # 以十六进制表示的字节码序列，可能是某种协议的数据包
    0x0a, 0x08, 0x61, 0x6c, 0x74, 0x65, 0x72, 0x5f, 0x69, 0x64, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d,
    0x52, 0x07, 0x61, 0x6c, 0x74, 0x65, 0x72, 0x49, 0x64, 0x12, 0x57, 0x0a, 0x11, 0x73, 0x65, 0x63,
    0x75, 0x72, 0x69, 0x74, 0x79, 0x5f, 0x73, 0x65, 0x74, 0x74, 0x69, 0x6e, 0x67, 0x73, 0x18, 0x03,
    0x20, 0x01, 0x28, 0x0b, 0x32, 0x2a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x2e, 0x53, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67,
    0x52, 0x10, 0x73, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x53, 0x65, 0x74, 0x74, 0x69, 0x6e,
    0x67, 0x73, 0x12, 0x23, 0x0a, 0x0d, 0x74, 0x65, 0x73, 0x74, 0x73, 0x5f, 0x65, 0x6e, 0x61, 0x62,
    0x6c, 0x65, 0x64, 0x18, 0x04, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0c, 0x74, 0x65, 0x73, 0x74, 0x73,
    0x45, 0x6e, 0x61, 0x62, 0x6c, 0x65, 0x64, 0x42, 0x53, 0x0a, 0x1a, 0x63, 0x6f, 0x6d, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e,
    0x76, 0x6d, 0x65, 0x73, 0x73, 0xaa, 0x02, 0x16, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f,
    0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x56, 0x6d, 0x65, 0x73, 0x73, 0x62, 0x06,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义了一个全局变量，用于确保 file_proxy_vmess_account_proto_rawDescData 只被初始化一次
var (
    file_proxy_vmess_account_proto_rawDescOnce sync.Once
    file_proxy_vmess_account_proto_rawDescData = file_proxy_vmess_account_proto_rawDesc
)

// 定义了一个函数，用于对 file_proxy_vmess_account_proto_rawDescData 进行 GZIP 压缩
func file_proxy_vmess_account_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保 GZIP 压缩只执行一次
    file_proxy_vmess_account_proto_rawDescOnce.Do(func() {
        file_proxy_vmess_account_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vmess_account_proto_rawDescData)
    })
    return file_proxy_vmess_account_proto_rawDescData
}

// 定义了两个全局变量，用于存储消息类型和 Go 类型
var file_proxy_vmess_account_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_proxy_vmess_account_proto_goTypes = []interface{}{
    (*Account)(nil),                 // 0: v2ray.core.proxy.vmess.Account
    (*protocol.SecurityConfig)(nil), // 1: v2ray.core.common.protocol.SecurityConfig
}

// 定义了一个全局变量，用于存储依赖索引
var file_proxy_vmess_account_proto_depIdxs = []int32{
    1, // 0: v2ray.core.proxy.vmess.Account.security_settings:type_name -> v2ray.core.common.protocol.SecurityConfig
    1, // [1:1] 是输出类型的子列表
    1, // [1:1] 是输入类型的子列表
    1, // [1:1] 是扩展类型的子列表
    1, // [1:1] 是扩展的扩展对象的子列表
    0, // [0:1] 是字段类型的子列表
}

// 初始化函数，用于初始化 file_proxy_vmess_account_proto
func init() { file_proxy_vmess_account_proto_init() }
func file_proxy_vmess_account_proto_init() {
    // 如果 File_proxy_vmess_account_proto 不为空，则直接返回
    if File_proxy_vmess_account_proto != nil {
        return
    }
    // 如果 protoimpl.UnsafeEnabled 为 false，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_proxy_vmess_account_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*Account); i {
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
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为 file_proxy_vmess_account_proto_rawDesc
            RawDescriptor: file_proxy_vmess_account_proto_rawDesc,
            # 设置 NumEnums 属性为 0
            NumEnums:      0,
            # 设置 NumMessages 属性为 1
            NumMessages:   1,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为 file_proxy_vmess_account_proto_goTypes
        GoTypes:           file_proxy_vmess_account_proto_goTypes,
        # 设置 DependencyIndexes 属性为 file_proxy_vmess_account_proto_depIdxs
        DependencyIndexes: file_proxy_vmess_account_proto_depIdxs,
        # 设置 MessageInfos 属性为 file_proxy_vmess_account_proto_msgTypes
        MessageInfos:      file_proxy_vmess_account_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_proxy_vmess_account_proto
    File_proxy_vmess_account_proto = out.File
    # 将 file_proxy_vmess_account_proto_rawDesc 置为 nil
    file_proxy_vmess_account_proto_rawDesc = nil
    # 将 file_proxy_vmess_account_proto_goTypes 置为 nil
    file_proxy_vmess_account_proto_goTypes = nil
    # 将 file_proxy_vmess_account_proto_depIdxs 置为 nil
    file_proxy_vmess_account_proto_depIdxs = nil
# 闭合前面的函数定义
```