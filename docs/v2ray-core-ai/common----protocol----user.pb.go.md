# `v2ray-core\common\protocol\user.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: common/protocol/user.proto

// 定义 protocol 包
package protocol

// 导入必要的包
import (
    proto "github.com/golang/protobuf/proto"  // 导入 protobuf 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

// 确保生成的代码足够更新
const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码足够更新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保生成的代码足够更新
)

// 这是一个编译时断言，用于确保使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4  // 确保使用足够更新的 legacy proto 包版本。

// User 是所有协议的通用用户。
type User struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    Level uint32 `protobuf:"varint,1,opt,name=level,proto3" json:"level,omitempty"`  // 用户级别
    Email string `protobuf:"bytes,2,opt,name=email,proto3" json:"email,omitempty"`  // 用户邮箱
    // 协议特定的账户信息。必须是其中一个代理的账户 proto。
    Account *serial.TypedMessage `protobuf:"bytes,3,opt,name=account,proto3" json:"account,omitempty"`  // 账户信息
}

// 重置 User 对象
func (x *User) Reset() {
    *x = User{}
    if protoimpl.UnsafeEnabled {
        mi := &file_common_protocol_user_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 User 对象的字符串表示
func (x *User) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*User) ProtoMessage() {}

// 返回 User 对象的反射信息
func (x *User) ProtoReflect() protoreflect.Message {
    mi := &file_common_protocol_user_proto_msgTypes[0]
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
// Deprecated: Use User.ProtoReflect.Descriptor instead.
// 使用 User.ProtoReflect.Descriptor 替代
func (*User) Descriptor() ([]byte, []int) {
    return file_common_protocol_user_proto_rawDescGZIP(), []int{0}
}

// 获取用户等级
func (x *User) GetLevel() uint32 {
    if x != nil {
        return x.Level
    }
    return 0
}

// 获取用户邮箱
func (x *User) GetEmail() string {
    if x != nil {
        return x.Email
    }
    return ""
}

// 获取用户账户信息
func (x *User) GetAccount() *serial.TypedMessage {
    if x != nil {
        return x.Account
    }
    return nil
}

// 用户协议文件描述符
var File_common_protocol_user_proto protoreflect.FileDescriptor

// 用户协议文件原始描述符
var file_common_protocol_user_proto_rawDesc = []byte{
    // 省略部分内容
}
    # 这是一串十六进制数字，可能是某种编码或者加密后的数据
    # 无法确定具体含义，需要根据上下文或者其他信息进行解析
// 定义变量，确保 file_common_protocol_user_proto_rawDescData 只被初始化一次
var (
    file_common_protocol_user_proto_rawDescOnce sync.Once
    file_common_protocol_user_proto_rawDescData = file_common_protocol_user_proto_rawDesc
)

// 使用 GZIP 压缩文件的原始描述数据
func file_common_protocol_user_proto_rawDescGZIP() []byte {
    // 确保只有一次压缩操作
    file_common_protocol_user_proto_rawDescOnce.Do(func() {
        file_common_protocol_user_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_protocol_user_proto_rawDescData)
    })
    return file_common_protocol_user_proto_rawDescData
}

// 定义消息类型和 Go 类型
var file_common_protocol_user_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_protocol_user_proto_goTypes = []interface{}{
    (*User)(nil),                // 0: v2ray.core.common.protocol.User
    (*serial.TypedMessage)(nil), // 1: v2ray.core.common.serial.TypedMessage
}

// 定义依赖索引
var file_common_protocol_user_proto_depIdxs = []int32{
    1, // 0: v2ray.core.common.protocol.User.account:type_name -> v2ray.core.common.serial.TypedMessage
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数
func init() { file_common_protocol_user_proto_init() }
func file_common_protocol_user_proto_init() {
    // 如果文件已经初始化，则直接返回
    if File_common_protocol_user_proto != nil {
        return
    }
    // 如果未启用不安全操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_common_protocol_user_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*User); i {
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
            RawDescriptor: file_common_protocol_user_proto_rawDesc,
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
        GoTypes:           file_common_protocol_user_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_common_protocol_user_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_common_protocol_user_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_common_protocol_user_proto
    File_common_protocol_user_proto = out.File
    # 将 file_common_protocol_user_proto_rawDesc 设置为 nil
    file_common_protocol_user_proto_rawDesc = nil
    # 将 file_common_protocol_user_proto_goTypes 设置为 nil
    file_common_protocol_user_proto_goTypes = nil
    # 将 file_common_protocol_user_proto_depIdxs 设置为 nil
    file_common_protocol_user_proto_depIdxs = nil
# 闭合前面的函数定义
```