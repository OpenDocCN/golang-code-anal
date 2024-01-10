# `v2ray-core\common\log\log.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: common/log/log.proto

package log

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

// 这是一个编译时断言，用于确保使用足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type Severity int32

const (
    Severity_Unknown Severity = 0
    Severity_Error   Severity = 1
    Severity_Warning Severity = 2
    Severity_Info    Severity = 3
    Severity_Debug   Severity = 4
)

// Severity 的枚举值映射。
var (
    Severity_name = map[int32]string{
        0: "Unknown",
        1: "Error",
        2: "Warning",
        3: "Info",
        4: "Debug",
    }
    Severity_value = map[string]int32{
        "Unknown": 0,
        "Error":   1,
        "Warning": 2,
        "Info":    3,
        "Debug":   4,
    }
)

func (x Severity) Enum() *Severity {
    p := new(Severity)
    *p = x
    return p
}

func (x Severity) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Severity) Descriptor() protoreflect.EnumDescriptor {
    return file_common_log_log_proto_enumTypes[0].Descriptor()
}

func (Severity) Type() protoreflect.EnumType {
    return &file_common_log_log_proto_enumTypes[0]
}

func (x Severity) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// 已弃用: 使用 Severity.Descriptor 代替。
func (Severity) EnumDescriptor() ([]byte, []int) {
    # 返回 file_common_log_log_proto_rawDescGZIP() 函数的结果和空的整数切片
    return file_common_log_log_proto_rawDescGZIP(), []int{0}
# 定义变量 File_common_log_log_proto 为 protoreflect.FileDescriptor 类型
var File_common_log_log_proto protoreflect.FileDescriptor

# 定义变量 file_common_log_log_proto_rawDesc 为包含字节的切片
var file_common_log_log_proto_rawDesc = []byte{
    # 以下为字节数据，表示文件描述符的内容
    0x0a, 0x14, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6c, 0x6f, 0x67, 0x2f, 0x6c, 0x6f, 0x67,
    0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
    0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6c, 0x6f, 0x67, 0x2a, 0x44, 0x0a,
    0x08, 0x53, 0x65, 0x76, 0x65, 0x72, 0x69, 0x74, 0x79, 0x12, 0x0b, 0x0a, 0x07, 0x55, 0x6e, 0x6b,
    0x6e, 0x6f, 0x77, 0x6e, 0x10, 0x00, 0x12, 0x09, 0x0a, 0x05, 0x45, 0x72, 0x72, 0x6f, 0x72, 0x10,
    0x01, 0x12, 0x0b, 0x0a, 0x07, 0x57, 0x61, 0x72, 0x6e, 0x69, 0x6e, 0x67, 0x10, 0x02, 0x12, 0x08,
    0x0a, 0x04, 0x49, 0x6e, 0x66, 0x6f, 0x10, 0x03, 0x12, 0x09, 0x0a, 0x05, 0x44, 0x65, 0x62, 0x75,
    0x67, 0x10, 0x04, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6c, 0x6f, 0x67,
    0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
    0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6c, 0x6f, 0x67, 0xaa, 0x02, 0x15,
    0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f,
    0x6e, 0x2e, 0x4c, 0x6f, 0x67, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

# 定义变量 file_common_log_log_proto_rawDescOnce 为 sync.Once 类型
var (
    file_common_log_log_proto_rawDescOnce sync.Once
    # 定义变量 file_common_log_log_proto_rawDescData 为 file_common_log_log_proto_rawDesc 的副本
    file_common_log_log_proto_rawDescData = file_common_log_log_proto_rawDesc
)

# 定义函数 file_common_log_log_proto_rawDescGZIP，返回值为字节切片
func file_common_log_log_proto_rawDescGZIP() []byte {
    # 调用 sync.Once 的 Do 方法，确保函数只会被执行一次
    file_common_log_log_proto_rawDescOnce.Do(func() {
        # 使用 protoimpl.X.CompressGZIP 方法对 file_common_log_log_proto_rawDescData 进行压缩
        file_common_log_log_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_log_log_proto_rawDescData)
    })
    # 返回压缩后的数据
    return file_common_log_log_proto_rawDescData
}

# 定义变量 file_common_log_log_proto_enumTypes 为 protoimpl.EnumInfo 类型的切片
var file_common_log_log_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
# 定义变量 file_common_log_log_proto_goTypes 为包含接口类型的切片
var file_common_log_log_proto_goTypes = []interface{}{
    (Severity)(0), // 0: v2ray.core.common.log.Severity
}
# 定义一个空的 int32 数组，用于存储方法输出类型、输入类型、扩展类型、扩展对象和字段类型的索引
var file_common_log_log_proto_depIdxs = []int32{
    0, // [0:0] 是方法输出类型的子列表
    0, // [0:0] 是方法输入类型的子列表
    0, // [0:0] 是扩展类型名称的子列表
    0, // [0:0] 是扩展对象的子列表
    0, // [0:0] 是字段类型名称的子列表
}

# 初始化函数
func init() { file_common_log_log_proto_init() }

# 初始化 common_log_log.proto 文件
func file_common_log_log_proto_init() {
    # 如果 File_common_log_log_proto 不为空，则直接返回
    if File_common_log_log_proto != nil {
        return
    }
    # 定义一个空结构体 x
    type x struct{}
    # 使用 protoimpl.TypeBuilder 构建文件描述符
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_common_log_log_proto_rawDesc,
            NumEnums:      1,
            NumMessages:   0,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_common_log_log_proto_goTypes,
        DependencyIndexes: file_common_log_log_proto_depIdxs,
        EnumInfos:         file_common_log_log_proto_enumTypes,
    }.Build()
    # 将构建好的文件描述符赋值给 File_common_log_log_proto
    File_common_log_log_proto = out.File
    # 将原始文件描述符和 Go 类型清空
    file_common_log_log_proto_rawDesc = nil
    file_common_log_log_proto_goTypes = nil
    file_common_log_log_proto_depIdxs = nil
}
```