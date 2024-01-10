# `v2ray-core\common\protocol\headers.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: common/protocol/headers.proto

// 定义 protocol 包
package protocol

// 导入必要的包
import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

// 确保生成的代码足够更新
const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码足够更新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保生成的代码足够更新
)

// 这是一个编译时断言，用于确保使用了足够更新的 legacy proto 包版本
const _ = proto.ProtoPackageIsVersion4

// 定义 SecurityType 枚举类型
type SecurityType int32

const (
    SecurityType_UNKNOWN           SecurityType = 0  // 未知安全类型
    SecurityType_LEGACY            SecurityType = 1  // 传统安全类型
    SecurityType_AUTO              SecurityType = 2  // 自动安全类型
    SecurityType_AES128_GCM        SecurityType = 3  // AES128_GCM 安全类型
    SecurityType_CHACHA20_POLY1305 SecurityType = 4  // CHACHA20_POLY1305 安全类型
    SecurityType_NONE              SecurityType = 5  // 无安全类型
)

// SecurityType 的枚举值映射
var (
    SecurityType_name = map[int32]string{  // 枚举值到名称的映射
        0: "UNKNOWN",
        1: "LEGACY",
        2: "AUTO",
        3: "AES128_GCM",
        4: "CHACHA20_POLY1305",
        5: "NONE",
    }
    SecurityType_value = map[string]int32{  // 名称到枚举值的映射
        "UNKNOWN":           0,
        "LEGACY":            1,
        "AUTO":              2,
        "AES128_GCM":        3,
        "CHACHA20_POLY1305": 4,
        "NONE":              5,
    }
)

// 返回 SecurityType 的枚举值
func (x SecurityType) Enum() *SecurityType {
    p := new(SecurityType)
    *p = x
    return p
}

// 返回 SecurityType 的字符串表示
func (x SecurityType) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

// 返回 SecurityType 的描述符
func (SecurityType) Descriptor() protoreflect.EnumDescriptor {
    # 返回 file_common_protocol_headers_proto_enumTypes 列表中第一个元素的 Descriptor 对象
    return file_common_protocol_headers_proto_enumTypes[0].Descriptor()
// 返回 SecurityType 的枚举类型
func (SecurityType) Type() protoreflect.EnumType {
    return &file_common_protocol_headers_proto_enumTypes[0]
}

// 返回 SecurityType 的枚举值
func (x SecurityType) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// 返回 SecurityType 的枚举描述，已废弃
func (SecurityType) EnumDescriptor() ([]byte, []int) {
    return file_common_protocol_headers_proto_rawDescGZIP(), []int{0}
}

// SecurityConfig 结构体定义
type SecurityConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Type SecurityType `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.common.protocol.SecurityType" json:"type,omitempty"`
}

// 重置 SecurityConfig 结构体
func (x *SecurityConfig) Reset() {
    *x = SecurityConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_common_protocol_headers_proto_msgTypes[0]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SecurityConfig 结构体的字符串表示
func (x *SecurityConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SecurityConfig) ProtoMessage() {}

// 返回 SecurityConfig 结构体的反射信息
func (x *SecurityConfig) ProtoReflect() protoreflect.Message {
    mi := &file_common_protocol_headers_proto_msgTypes[0]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 返回 SecurityConfig 结构体的描述，已废弃
func (*SecurityConfig) Descriptor() ([]byte, []int) {
    return file_common_protocol_headers_proto_rawDescGZIP(), []int{0}
}

// 返回 SecurityConfig 结构体的 Type 字段值
func (x *SecurityConfig) GetType() SecurityType {
    if x != nil {
        return x.Type
    }
    return SecurityType_UNKNOWN
}

// 定义 File_common_protocol_headers_proto 变量
var File_common_protocol_headers_proto protoreflect.FileDescriptor

// 定义 file_common_protocol_headers_proto_rawDesc 变量
var file_common_protocol_headers_proto_rawDesc = []byte{
    0x0a, 0x1d, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    # 以十六进制表示的字节码序列，可能是某种数据或配置信息
    0x6c, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12,
    0x1a, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
    0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x22, 0x4e, 0x0a, 0x0e, 0x53,
    0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x3c, 0x0a,
    0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x28, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
    0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
    0x79, 0x54, 0x79, 0x70, 0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65, 0x2a, 0x62, 0x0a, 0x0c, 0x53,
    0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x54, 0x79, 0x70, 0x65, 0x12, 0x0b, 0x0a, 0x07, 0x55,
    0x4e, 0x4b, 0x4e, 0x4f, 0x57, 0x4e, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x4c, 0x45, 0x47, 0x41,
    0x43, 0x59, 0x10, 0x01, 0x12, 0x08, 0x0a, 0x04, 0x41, 0x55, 0x54, 0x4f, 0x10, 0x02, 0x12, 0x0e,
    0x0a, 0x0a, 0x41, 0x45, 0x53, 0x31, 0x32, 0x38, 0x5f, 0x47, 0x43, 0x4d, 0x10, 0x03, 0x12, 0x15,
    0x0a, 0x11, 0x43, 0x48, 0x41, 0x43, 0x48, 0x41, 0x32, 0x30, 0x5f, 0x50, 0x4f, 0x4c, 0x59, 0x31,
    0x33, 0x30, 0x35, 0x10, 0x04, 0x12, 0x08, 0x0a, 0x04, 0x4e, 0x4f, 0x4e, 0x45, 0x10, 0x05, 0x42,
    0x5f, 0x0a, 0x1e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
    0x6c, 0x50, 0x01, 0x5a, 0x1e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
    0x6f, 0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f,
    0x63, 0x6f, 0x6c, 0xaa, 0x02, 0x1a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65,
    0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c,
    # 这是一个十六进制数的列表，表示一些数据
// 定义变量，确保 file_common_protocol_headers_proto_rawDescData 只被初始化一次
var (
    file_common_protocol_headers_proto_rawDescOnce sync.Once
    file_common_protocol_headers_proto_rawDescData = file_common_protocol_headers_proto_rawDesc
)

// 使用 GZIP 压缩算法对 file_common_protocol_headers_proto_rawDescData 进行压缩
func file_common_protocol_headers_proto_rawDescGZIP() []byte {
    // 确保压缩操作只执行一次
    file_common_protocol_headers_proto_rawDescOnce.Do(func() {
        file_common_protocol_headers_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_protocol_headers_proto_rawDescData)
    })
    return file_common_protocol_headers_proto_rawDescData
}

// 定义枚举类型和消息类型的信息
var file_common_protocol_headers_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_common_protocol_headers_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_protocol_headers_proto_goTypes = []interface{}{
    (SecurityType)(0),      // 0: v2ray.core.common.protocol.SecurityType
    (*SecurityConfig)(nil), // 1: v2ray.core.common.protocol.SecurityConfig
}
var file_common_protocol_headers_proto_depIdxs = []int32{
    0, // 0: v2ray.core.common.protocol.SecurityConfig.type:type_name -> v2ray.core.common.protocol.SecurityType
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数
func init() { file_common_protocol_headers_proto_init() }
func file_common_protocol_headers_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_common_protocol_headers_proto != nil {
        return
    }
    // 如果不允许使用不安全的操作，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled {
        file_common_protocol_headers_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
            switch v := v.(*SecurityConfig); i {
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
        # 设置 File 属性，包括 GoPackagePath、RawDescriptor、NumEnums、NumMessages、NumExtensions、NumServices
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_common_protocol_headers_proto_rawDesc,
            NumEnums:      1,
            NumMessages:   1,
            NumExtensions: 0,
            NumServices:   0,
        },
        # 设置 GoTypes 属性
        GoTypes:           file_common_protocol_headers_proto_goTypes,
        # 设置 DependencyIndexes 属性
        DependencyIndexes: file_common_protocol_headers_proto_depIdxs,
        # 设置 EnumInfos 属性
        EnumInfos:         file_common_protocol_headers_proto_enumTypes,
        # 设置 MessageInfos 属性
        MessageInfos:      file_common_protocol_headers_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_common_protocol_headers_proto
    File_common_protocol_headers_proto = out.File
    # 将 file_common_protocol_headers_proto_rawDesc 置为 nil
    file_common_protocol_headers_proto_rawDesc = nil
    # 将 file_common_protocol_headers_proto_goTypes 置为 nil
    file_common_protocol_headers_proto_goTypes = nil
    # 将 file_common_protocol_headers_proto_depIdxs 置为 nil
    file_common_protocol_headers_proto_depIdxs = nil
# 闭合前面的函数定义
```