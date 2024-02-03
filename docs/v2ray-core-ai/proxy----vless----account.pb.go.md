# `v2ray-core\proxy\vless\account.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: proxy/vless/account.proto

package vless

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

// 这是一个编译时断言，用于确保正在使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Account struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    // 账户的 ID，以 UUID 的形式，例如 "66ad4540-b58c-4ad2-9926-ea63445a9b57"。
    Id string `protobuf:"bytes,1,opt,name=id,proto3" json:"id,omitempty"`  // ID 字段
    // 流量设置。可能是 "xtls-rprx-origin"。
    Flow string `protobuf:"bytes,2,opt,name=flow,proto3" json:"flow,omitempty"`  // Flow 字段
    // 加密设置。仅适用于客户端端，目前仅接受 "none"。
    Encryption string `protobuf:"bytes,3,opt,name=encryption,proto3" json:"encryption,omitempty"`  // Encryption 字段
}

func (x *Account) Reset() {
    *x = Account{}  // 重置 Account 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_proxy_vless_account_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Account) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 Account 对象的字符串表示形式
}

func (*Account) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Account) ProtoReflect() protoreflect.Message {
    mi := &file_proxy_vless_account_proto_msgTypes[0]  // 获取消息类型信息
    // 返回 Account 对象的反射信息
    return mi
}
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
// Deprecated: Use Account.ProtoReflect.Descriptor instead.
// 此方法已废弃，建议使用 Account.ProtoReflect.Descriptor 方法
func (*Account) Descriptor() ([]byte, []int) {
    // 返回文件的原始描述信息和索引
    return file_proxy_vless_account_proto_rawDescGZIP(), []int{0}
}

// 获取账户ID
func (x *Account) GetId() string {
    if x != nil {
        return x.Id
    }
    return ""
}

// 获取账户流量
func (x *Account) GetFlow() string {
    if x != nil {
        return x.Flow
    }
    return ""
}

// 获取账户加密方式
func (x *Account) GetEncryption() string {
    if x != nil {
        return x.Encryption
    }
    return ""
}

// 定义文件描述符
var File_proxy_vless_account_proto protoreflect.FileDescriptor

// 原始描述信息的压缩数据
var file_proxy_vless_account_proto_rawDesc = []byte{
    // 压缩后的原始描述信息数据
}
// 定义变量，确保 file_proxy_vless_account_proto_rawDescData 只被初始化一次
var (
    file_proxy_vless_account_proto_rawDescOnce sync.Once
    file_proxy_vless_account_proto_rawDescData = file_proxy_vless_account_proto_rawDesc
)

// 使用 GZIP 压缩文件内容
func file_proxy_vless_account_proto_rawDescGZIP() []byte {
    // 确保文件内容只被压缩一次
    file_proxy_vless_account_proto_rawDescOnce.Do(func() {
        file_proxy_vless_account_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vless_account_proto_rawDescData)
    })
    return file_proxy_vless_account_proto_rawDescData
}

// 定义消息类型和 Go 类型
var file_proxy_vless_account_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_proxy_vless_account_proto_goTypes = []interface{}{
    (*Account)(nil), // 0: v2ray.core.proxy.vless.Account
}

// 定义依赖索引
var file_proxy_vless_account_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展的扩展对象
    0, // [0:0] 是字段类型的子列表
}

// 初始化函数
func init() { file_proxy_vless_account_proto_init() }
func file_proxy_vless_account_proto_init() {
    // 如果文件不为空，则返回
    if File_proxy_vless_account_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出器
    if !protoimpl.UnsafeEnabled {
        file_proxy_vless_account_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
            RawDescriptor: file_proxy_vless_account_proto_rawDesc,
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
        GoTypes:           file_proxy_vless_account_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_proxy_vless_account_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_proxy_vless_account_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_proxy_vless_account_proto
    File_proxy_vless_account_proto = out.File
    # 将 file_proxy_vless_account_proto_rawDesc 置为 nil
    file_proxy_vless_account_proto_rawDesc = nil
    # 将 file_proxy_vless_account_proto_goTypes 置为 nil
    file_proxy_vless_account_proto_goTypes = nil
    # 将 file_proxy_vless_account_proto_depIdxs 置为 nil
    file_proxy_vless_account_proto_depIdxs = nil
# 闭合前面的函数定义
```