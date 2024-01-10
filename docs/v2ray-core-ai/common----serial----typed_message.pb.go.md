# `v2ray-core\common\serial\typed_message.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: common/serial/typed_message.proto

package serial

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

// 这是一个编译时断言，用于确保使用足够更新的 legacy proto 包。
const _ = proto.ProtoPackageIsVersion4

// TypedMessage 是一个序列化的 proto 消息以及其类型名称。
type TypedMessage struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    // 消息类型的名称，从 protobuf API 中检索。
    Type string `protobuf:"bytes,1,opt,name=type,proto3" json:"type,omitempty"`
    // 序列化的 proto 消息。
    Value []byte `protobuf:"bytes,2,opt,name=value,proto3" json:"value,omitempty"`
}

func (x *TypedMessage) Reset() {
    *x = TypedMessage{}  // 重置 TypedMessage 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_common_serial_typed_message_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *TypedMessage) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*TypedMessage) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *TypedMessage) ProtoReflect() protoreflect.Message {
    mi := &file_common_serial_typed_message_proto_msgTypes[0]  // 获取消息类型信息
    // 返回消息的反射信息
    return mi
}
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
// Deprecated: Use TypedMessage.ProtoReflect.Descriptor instead.
// 使用 TypedMessage.ProtoReflect.Descriptor 替代
func (*TypedMessage) Descriptor() ([]byte, []int) {
    return file_common_serial_typed_message_proto_rawDescGZIP(), []int{0}
}

// 获取 TypedMessage 的类型
func (x *TypedMessage) GetType() string {
    if x != nil {
        return x.Type
    }
    return ""
}

// 获取 TypedMessage 的值
func (x *TypedMessage) GetValue() []byte {
    if x != nil {
        return x.Value
    }
    return nil
}

// 定义文件描述符
var File_common_serial_typed_message_proto protoreflect.FileDescriptor

// 原始文件描述符的字节流
var file_common_serial_typed_message_proto_rawDesc = []byte{
    // 省略部分内容
}

// 保证 file_common_serial_typed_message_proto_rawDescOnce 只执行一次
var (
    file_common_serial_typed_message_proto_rawDescOnce sync.Once
}
    # 将变量file_common_serial_typed_message_proto_rawDescData赋值为file_common_serial_typed_message_proto_rawDesc的值
    file_common_serial_typed_message_proto_rawDescData = file_common_serial_typed_message_proto_rawDesc
# 返回经过 GZIP 压缩的 file_common_serial_typed_message_proto_rawDescData
func file_common_serial_typed_message_proto_rawDescGZIP() []byte {
    # 一次性执行函数，用于对 file_common_serial_typed_message_proto_rawDescData 进行 GZIP 压缩
    file_common_serial_typed_message_proto_rawDescOnce.Do(func() {
        file_common_serial_typed_message_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_serial_typed_message_proto_rawDescData)
    })
    # 返回经过 GZIP 压缩的 file_common_serial_typed_message_proto_rawDescData
    return file_common_serial_typed_message_proto_rawDescData
}

# 定义消息类型的信息数组
var file_common_serial_typed_message_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
# 定义 Go 语言类型的接口数组
var file_common_serial_typed_message_proto_goTypes = []interface{}{
    (*TypedMessage)(nil), // 0: v2ray.core.common.serial.TypedMessage
}
# 定义依赖索引数组
var file_common_serial_typed_message_proto_depIdxs = []int32{
    0, // [0:0] 是输出类型的子列表
    0, // [0:0] 是输入类型的子列表
    0, // [0:0] 是扩展类型的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型的子列表
}

# 初始化函数
func init() { file_common_serial_typed_message_proto_init() }
# 初始化 file_common_serial_typed_message_proto
func file_common_serial_typed_message_proto_init() {
    # 如果 File_common_serial_typed_message_proto 不为空，则返回
    if File_common_serial_typed_message_proto != nil {
        return
    }
    # 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_common_serial_typed_message_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*TypedMessage); i {
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
    # 定义一个空结构体 x
    type x struct{}
}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_common_serial_typed_message_proto_rawDesc,
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
        GoTypes:           file_common_serial_typed_message_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_common_serial_typed_message_proto_depIdxs,
        # 设置 MessageInfos 属性为指定的消息信息
        MessageInfos:      file_common_serial_typed_message_proto_msgTypes,
    }.Build()
    # 设置 File_common_serial_typed_message_proto 为 out.File
    File_common_serial_typed_message_proto = out.File
    # 将原始描述符设置为 nil
    file_common_serial_typed_message_proto_rawDesc = nil
    # 将 Go 类型设置为 nil
    file_common_serial_typed_message_proto_goTypes = nil
    # 将依赖索引设置为 nil
    file_common_serial_typed_message_proto_depIdxs = nil
# 闭合前面的函数定义
```