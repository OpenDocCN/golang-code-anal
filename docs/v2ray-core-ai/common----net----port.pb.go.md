# `v2ray-core\common\net\port.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: common/net/port.proto

package net

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

// 这是一个编译时断言，用于确保使用了足够更新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

// PortRange 表示一个端口范围。
type PortRange struct {
    state         protoimpl.MessageState  // protoimpl.MessageState 对象
    sizeCache     protoimpl.SizeCache  // protoimpl.SizeCache 对象
    unknownFields protoimpl.UnknownFields  // protoimpl.UnknownFields 对象

    // 起始端口。
    From uint32 `protobuf:"varint,1,opt,name=From,proto3" json:"From,omitempty"`
    // 结束端口（包含在内）。
    To uint32 `protobuf:"varint,2,opt,name=To,proto3" json:"To,omitempty"`
}

func (x *PortRange) Reset() {
    *x = PortRange{}  // 重置 PortRange 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_common_net_port_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *PortRange) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 PortRange 对象的字符串表示
}

func (*PortRange) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *PortRange) ProtoReflect() protoreflect.Message {
    mi := &file_common_net_port_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)  // 返回消息类型信息
}
// Deprecated: Use PortRange.ProtoReflect.Descriptor instead.
// 返回 PortRange 的描述符
func (*PortRange) Descriptor() ([]byte, []int) {
    return file_common_net_port_proto_rawDescGZIP(), []int{0}
}

// 返回 PortRange 结构体中的 From 字段值
func (x *PortRange) GetFrom() uint32 {
    if x != nil {
        return x.From
    }
    return 0
}

// 返回 PortRange 结构体中的 To 字段值
func (x *PortRange) GetTo() uint32 {
    if x != nil {
        return x.To
    }
    return 0
}

// PortList 是端口列表的结构体
type PortList struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Range 是 PortRange 结构体的切片
    Range []*PortRange `protobuf:"bytes,1,rep,name=range,proto3" json:"range,omitempty"`
}

// 重置 PortList 结构体
func (x *PortList) Reset() {
    *x = PortList{}
    if protoimpl.UnsafeEnabled {
        mi := &file_common_net_port_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 PortList 结构体的字符串表示
func (x *PortList) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*PortList) ProtoMessage() {}

// 返回 PortList 结构体的反射信息
func (x *PortList) ProtoReflect() protoreflect.Message {
    mi := &file_common_net_port_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use PortList.ProtoReflect.Descriptor instead.
// 返回 PortList 的描述符
func (*PortList) Descriptor() ([]byte, []int) {
    return file_common_net_port_proto_rawDescGZIP(), []int{1}
}

// 返回 PortList 结构体中 Range 字段的值
func (x *PortList) GetRange() []*PortRange {
    if x != nil {
        return x.Range
    }
    return nil
}

// File_common_net_port_proto 是端口协议的文件描述符
var File_common_net_port_proto protoreflect.FileDescriptor

// file_common_net_port_proto_rawDesc 是端口协议的原始描述符
var file_common_net_port_proto_rawDesc = []byte{
    0x0a, 0x15, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x70, 0x6f, 0x72,
    0x74, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
}
    # 创建一个十六进制数的列表
    0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x22, 0x2f,
    0x0a, 0x09, 0x50, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65, 0x12, 0x12, 0x0a, 0x04, 0x46,
    0x72, 0x6f, 0x6d, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x04, 0x46, 0x72, 0x6f, 0x6d, 0x12,
    0x0e, 0x0a, 0x02, 0x54, 0x6f, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x02, 0x54, 0x6f, 0x22,
    0x42, 0x0a, 0x08, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x36, 0x0a, 0x05, 0x72,
    0x61, 0x6e, 0x67, 0x65, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72,
    0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e,
    0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65, 0x52, 0x05, 0x72, 0x61,
    0x6e, 0x67, 0x65, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74,
    0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
    0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0xaa, 0x02, 0x15,
    0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f,
    0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保只执行一次初始化操作
var (
    file_common_net_port_proto_rawDescOnce sync.Once
    file_common_net_port_proto_rawDescData = file_common_net_port_proto_rawDesc
)

// 使用 GZIP 压缩原始描述数据
func file_common_net_port_proto_rawDescGZIP() []byte {
    file_common_net_port_proto_rawDescOnce.Do(func() {
        file_common_net_port_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_net_port_proto_rawDescData)
    })
    return file_common_net_port_proto_rawDescData
}

// 定义消息类型和 Go 类型
var file_common_net_port_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
var file_common_net_port_proto_goTypes = []interface{}{
    (*PortRange)(nil), // 0: v2ray.core.common.net.PortRange
    (*PortList)(nil),  // 1: v2ray.core.common.net.PortList
}

// 定义依赖索引
var file_common_net_port_proto_depIdxs = []int32{
    0, // 0: v2ray.core.common.net.PortList.range:type_name -> v2ray.core.common.net.PortRange
    1, // [1:1] is the sub-list for method output_type
    1, // [1:1] is the sub-list for method input_type
    1, // [1:1] is the sub-list for extension type_name
    1, // [1:1] is the sub-list for extension extendee
    0, // [0:1] is the sub-list for field type_name
}

// 初始化函数
func init() { file_common_net_port_proto_init() }
func file_common_net_port_proto_init() {
    // 如果文件已经初始化，则直接返回
    if File_common_net_port_proto != nil {
        return
    }
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_common_net_port_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*PortRange); i:
                case 0:
                    return &v.state
                case 1:
                    return &v.sizeCache
                case 2:
                    return &v.unknownFields
                default:
                    return nil
        file_common_net_port_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*PortList); i:
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
            RawDescriptor: file_common_net_port_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   2,
            NumExtensions: 0,
            NumServices:   0,
        },
        GoTypes:           file_common_net_port_proto_goTypes,
        DependencyIndexes: file_common_net_port_proto_depIdxs,
        MessageInfos:      file_common_net_port_proto_msgTypes,
    }.Build()
    # 将构建的类型赋值给 File_common_net_port_proto
    File_common_net_port_proto = out.File
    # 清空原始描述和 Go 类型
    file_common_net_port_proto_rawDesc = nil
    file_common_net_port_proto_goTypes = nil
    file_common_net_port_proto_depIdxs = nil
# 闭合前面的函数定义
```