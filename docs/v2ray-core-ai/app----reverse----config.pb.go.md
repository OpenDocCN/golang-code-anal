# `v2ray-core\app\reverse\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: app/reverse/config.proto

package reverse

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

type Control_State int32

const (
    Control_ACTIVE Control_State = 0
    Control_DRAIN  Control_State = 1
)

// Control_State 的枚举值映射。
var (
    Control_State_name = map[int32]string{
        0: "ACTIVE",
        1: "DRAIN",
    }
    Control_State_value = map[string]int32{
        "ACTIVE": 0,
        "DRAIN":  1,
    }
)

func (x Control_State) Enum() *Control_State {
    p := new(Control_State)
    *p = x
    return p
}

func (x Control_State) String() string {
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Control_State) Descriptor() protoreflect.EnumDescriptor {
    return file_app_reverse_config_proto_enumTypes[0].Descriptor()
}

func (Control_State) Type() protoreflect.EnumType {
    return &file_app_reverse_config_proto_enumTypes[0]
}

func (x Control_State) Number() protoreflect.EnumNumber {
    return protoreflect.EnumNumber(x)
}

// 已弃用: 使用 Control_State.Descriptor 代替。
func (Control_State) EnumDescriptor() ([]byte, []int) {
    return file_app_reverse_config_proto_rawDescGZIP(), []int{0, 0}
}

type Control struct {
    state         protoimpl.MessageState
    # sizeCache用于存储消息大小的缓存
    sizeCache     protoimpl.SizeCache
    # unknownFields用于存储未知字段
    unknownFields protoimpl.UnknownFields

    # State表示控制状态，使用protobuf标记为varint类型，字段序号为1，可选字段，字段名为state，枚举类型为v2ray.core.app.reverse.Control_State
    State  Control_State `protobuf:"varint,1,opt,name=state,proto3,enum=v2ray.core.app.reverse.Control_State" json:"state,omitempty"`
    # Random表示随机字节序列，使用protobuf标记为bytes类型，字段序号为99，可选字段，字段名为random
    Random []byte        `protobuf:"bytes,99,opt,name=random,proto3" json:"random,omitempty"`
}

// 重置控制对象的状态为初始状态
func (x *Control) Reset() {
    // 将控制对象重置为空对象
    *x = Control{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_reverse_config_proto_msgTypes[0]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回控制对象的字符串表示
func (x *Control) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*Control) ProtoMessage() {}

// 返回控制对象的反射信息
func (x *Control) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_reverse_config_proto_msgTypes[0]
    // 如果启用了不安全操作并且控制对象不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 Control.ProtoReflect.Descriptor 代替
func (*Control) Descriptor() ([]byte, []int) {
    return file_app_reverse_config_proto_rawDescGZIP(), []int{0}
}

// 获取控制对象的状态
func (x *Control) GetState() Control_State {
    if x != nil {
        return x.State
    }
    return Control_ACTIVE
}

// 获取控制对象的随机数据
func (x *Control) GetRandom() []byte {
    if x != nil {
        return x.Random
    }
    return nil
}

// 桥接配置对象
type BridgeConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Tag    string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    Domain string `protobuf:"bytes,2,opt,name=domain,proto3" json:"domain,omitempty"`
}

// 重置桥接配置对象的状态为初始状态
func (x *BridgeConfig) Reset() {
    // 将桥接配置对象重置为空对象
    *x = BridgeConfig{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_reverse_config_proto_msgTypes[1]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回桥接配置对象的字符串表示
func (x *BridgeConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*BridgeConfig) ProtoMessage() {}

// 返回桥接配置对象的反射信息
func (x *BridgeConfig) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_reverse_config_proto_msgTypes[1]
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
// Deprecated: Use BridgeConfig.ProtoReflect.Descriptor instead.
// 使用 BridgeConfig.ProtoReflect.Descriptor 替代
func (*BridgeConfig) Descriptor() ([]byte, []int) {
    return file_app_reverse_config_proto_rawDescGZIP(), []int{1}
}

// 获取 BridgeConfig 结构体中的 Tag 字段值
func (x *BridgeConfig) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取 BridgeConfig 结构体中的 Domain 字段值
func (x *BridgeConfig) GetDomain() string {
    if x != nil {
        return x.Domain
    }
    return ""
}

// 定义 PortalConfig 结构体
type PortalConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Tag    string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    Domain string `protobuf:"bytes,2,opt,name=domain,proto3" json:"domain,omitempty"`
}

// 重置 PortalConfig 结构体
func (x *PortalConfig) Reset() {
    *x = PortalConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_reverse_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 PortalConfig 结构体转换为字符串
func (x *PortalConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*PortalConfig) ProtoMessage() {}

// 获取 PortalConfig 结构体的反射信息
func (x *PortalConfig) ProtoReflect() protoreflect.Message {
    mi := &file_app_reverse_config_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use PortalConfig.ProtoReflect.Descriptor instead.
// 使用 PortalConfig.ProtoReflect.Descriptor 替代
func (*PortalConfig) Descriptor() ([]byte, []int) {
    return file_app_reverse_config_proto_rawDescGZIP(), []int{2}
}

// 获取 PortalConfig 结构体中的 Tag 字段值
func (x *PortalConfig) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 获取 PortalConfig 结构体中的 Domain 字段值
func (x *PortalConfig) GetDomain() string {
    if x != nil {
        return x.Domain
    }
    return ""
}

// 定义 Config 结构体
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
    # 定义一个指针数组，用于存储 BridgeConfig 结构体的数据，protobuf 标签指定了序列化和反序列化的规则，json 标签指定了 JSON 序列化和反序列化的规则
    BridgeConfig []*BridgeConfig `protobuf:"bytes,1,rep,name=bridge_config,json=bridgeConfig,proto3" json:"bridge_config,omitempty"`
    # 定义一个指针数组，用于存储 PortalConfig 结构体的数据，protobuf 标签指定了序列化和反序列化的规则，json 标签指定了 JSON 序列化和反序列化的规则
    PortalConfig []*PortalConfig `protobuf:"bytes,2,rep,name=portal_config,json=portalConfig,proto3" json:"portal_config,omitempty"`
// 重置 Config 对象，将其值设为默认值
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，获取消息类型并存储消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_app_reverse_config_proto_msgTypes[3]
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
    mi := &file_app_reverse_config_proto_msgTypes[3]
    // 如果启用了不安全操作并且 Config 对象不为空，获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用方法：使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_reverse_config_proto_rawDescGZIP(), []int{3}
}

// 获取 Config 对象的 BridgeConfig 字段
func (x *Config) GetBridgeConfig() []*BridgeConfig {
    if x != nil {
        return x.BridgeConfig
    }
    return nil
}

// 获取 Config 对象的 PortalConfig 字段
func (x *Config) GetPortalConfig() []*PortalConfig {
    if x != nil {
        return x.PortalConfig
    }
    return nil
}

// 定义 File_app_reverse_config_proto 的变量
var File_app_reverse_config_proto protoreflect.FileDescriptor

// 定义 file_app_reverse_config_proto_rawDesc 的变量
var file_app_reverse_config_proto_rawDesc = []byte{
    // 这里是一大段字节数据，表示文件描述符的原始数据
}
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x65 的整数
    0x65, 
    # 创建一个十六进制数值为 0x52 的整数
    0x52, 
    # 创建一个十六进制数值为 0x05 的整数
    0x05, 
    # 创建一个十六进制数值为 0x73 的整数
    0x73, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x65 的整数
    0x65, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x16 的整数
    0x16, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x06 的整数
    0x06, 
    # 创建一个十六进制数值为 0x72 的整数
    0x72, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61,
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x64 的整数
    0x64, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6d 的整数
    0x6d, 
    # 创建一个十六进制数值为 0x18 的整数
    0x18, 
    # 创建一个十六进制数值为 0x63 的整数
    0x63, 
    # 创建一个十六进制数值为 0x20 的整数
    0x20, 
    # 创建一个十六进制数值为 0x01 的整数
    0x01, 
    # 创建一个十六进制数值为 0x28 的整数
    0x28, 
    # 创建一个十六进制数值为 0x0c 的整数
    0x0c, 
    # 创建一个十六进制数值为 0x52 的整数
    0x52, 
    # 创建一个十六进制数值为 0x06 的整数
    0x06, 
    # 创建一个十六进制数值为 0x72 的整数
    0x72, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x64 的整数
    0x64,
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6d 的整数
    0x6d, 
    # 创建一个十六进制数值为 0x22 的整数
    0x22, 
    # 创建一个十六进制数值为 0x1e 的整数
    0x1e, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x05 的整数
    0x05, 
    # 创建一个十六进制数值为 0x53 的整数
    0x53, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x65 的整数
    0x65, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x06 的整数
    0x06, 
    # 创建一个十六进制���值为 0x41 的整数
    0x41, 
    # 创建一个十六进制数值为 0x43 的整数
    0x43, 
    # 创建一个十六进制数值为 0x54 的整数
    0x54, 
    # 创建一个十六进制数值为 0x49 的整数
    0x49, 
    # 创建一个十六进制数值为 0x56 的整数
    0x56, 
    # 创建一个十六进制数值为 0x45 的整数
    0x45, 
    # 创建一个十六进制数值为 0x10 的整数
    0x10, 
    # 创建一个十六进制数值为 0x00 的整数
    0x00, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x09 的整数
    0x09, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x05 的整数
    0x05, 
    # 创建一个十六进制数值为 0x44 的整数
    0x44, 
    # 创建一个十六进制数值为 0x52 的整数
    0x52, 
    # 创建一个十六进制数值为 0x41 的整数
    0x41, 
    # 创建一个十六进制数值为 0x49 的整数
    0x49, 
    # 创建一个十六进制数值为 0x4e 的整数
    0x4e, 
    # 创建一个十六进制数值为 0x10 的整数
    0x10, 
    # 创建一个十六进制数值为 0x01 的整数
    0x01, 
    # 创建一个十六进制数值为 0x22 的整数
    0x22, 
    # 创建一个十六进制数值为 0x38 的整数
    0x38, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x0c 的整数
    0x0c, 
    # 创建一个十六进制数值为 0x42 的整数
    0x42, 
    # 创建一个十六进制数值为 0x72 的整数
    0x72, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x64 的整数
    0x64, 
    # 创建一个十六进制数值为 0x67 的整数
    0x67, 
    # 创建一个十六进制数值为 0x65 的整数
    0x65, 
    # 创建一个十六进制数值为 0x43 的整数
    0x43, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x66 的整数
    0x66, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x67 的整数
    0x67, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x10 的整数
    0x10, 
    # 创建一个十六进制数值为 0x00 的整数
    0x00, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x09 的整数
    0x09, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x05 的整数
    0x05, 
    # 创建一个十六进制数值为 0x44 的整数
    0x44, 
    # 创建一个十六进制数值为 0x52 的整数
    0x52, 
    # 创建一个十六进制数值为 0x41 的整数
    0x41, 
    # 创建一个十六进制数值为 0x49 的整数
    0x49, 
    # 创建一个十六进制数值为 0x4e 的整数
    0x4e, 
    # 创建一个十六进制数值为 0x10 的整数
    0x10, 
    # 创建一个十六进制数值为 0x01 的整数
    0x01, 
    # 创建一个十六进制数值为 0x22 的整数
    0x22, 
    # 创建一个十六进制数值为 0x38 的整数
    0x38, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x0c 的整数
    0x0c, 
    # 创建一个十六进制数值为 0x50 的整数
    0x50, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x72 的整数
    0x72, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c, 
    # 创建一个十六进制数值为 0x43 的整数
    0x43, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x66 的整数
    0x66, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x67 的整数
    0x67, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x10 的整数
    0x10, 
    # 创建一个十六进制数值为 0x00 的整数
    0x00, 
    # 创建一个十六进制数值为 0x12 ��整数
    0x12, 
    # 创建一个十六进制数值为 0x16 的整数
    0x16, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x06 的整数
    0x06, 
    # 创建一个十六进制数值为 0x64 的整数
    0x64, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6d 的整数
    0x6d, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x18 的整数
    0x18, 
    # 创建一个十六进制数值为 0x02 的整数
    0x02, 
    # 创建一个十六进制数值为 0x20 的整数
    0x20, 
    # 创建一个十六进制数值为 0x01 的整数
    0x01, 
    # 创建一个十六进制数值为 0x28 的整数
    0x28, 
    # 创建一个十六进制数值为 0x09 的整数
    0x09, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x05 的整数
    0x05, 
    # 创建一个十六进制数值为 0x64 的整数
    0x64, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6d 的整数
    0x6d, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x22 的整数
    0x22, 
    # 创建一个十六进制数值为 0x38 的整数
    0x38, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x0c 的整数
    0x0c, 
    # 创建一个十六进制数值为 0x50 的整数
    0x50, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x72 的整数
    0x72, 
    # 创建一个十六进制数值为 0x74 的整数
    0x74, 
    # 创建一个十六进制数值为 0x61 的整数
    0x61, 
    # 创建一个十六进制数值为 0x6c 的整数
    0x6c, 
    # 创建一个十六进制数值为 0x43 的整数
    0x43, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x66 的整数
    0x66, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x67 的整数
    0x67, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x16 的整数
    0x16, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建一个十六进制数值为 0x06 的整数
    0x06, 
    # 创建一个十六进制数值为 0x43 的整数
    0x43, 
    # 创建一个十六进制数值为 0x6f 的整数
    0x6f, 
    # 创建一个十六进制数值为 0x6e 的整数
    0x6e, 
    # 创建一个十六进制数值为 0x66 的整数
    0x66, 
    # 创建一个十六进制数值为 0x69 的整数
    0x69, 
    # 创建一个十六进制数值为 0x67 的整数
    0x67, 
    # 创建一个十六进制数值为 0x12 的整数
    0x12, 
    # 创建一个十六进制数值为 0x49 的整数
    0x49, 
    # 创建一个十六进制数值为 0x0a 的整数
    0x0a, 
    # 创建
    # 这是一个十六进制数的列表，可能是用来表示某种数据或者配置信息
    0x74, 0x61, 0x6c, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x0c, 0x70, 0x6f, 0x72, 0x74, 0x61,
    0x6c, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x42, 0x57, 0x0a, 0x1c, 0x63, 0x6f, 0x6d, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e,
    0x72, 0x65, 0x76, 0x65, 0x72, 0x73, 0x65, 0x50, 0x01, 0x5a, 0x1a, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x65,
    0x76, 0x65, 0xaa, 0x02, 0x18, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
    0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x52, 0x65, 0x76, 0x65, 0x72, 0x73, 0x65, 0x62, 0x06, 0x70,
    0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量，确保 file_app_reverse_config_proto_rawDescData 只被初始化一次
var (
    file_app_reverse_config_proto_rawDescOnce sync.Once
    file_app_reverse_config_proto_rawDescData = file_app_reverse_config_proto_rawDesc
)

// 使用 GZIP 压缩文件的原始描述数据
func file_app_reverse_config_proto_rawDescGZIP() []byte {
    // 确保只有一个 goroutine 执行 GZIP 压缩操作
    file_app_reverse_config_proto_rawDescOnce.Do(func() {
        file_app_reverse_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_reverse_config_proto_rawDescData)
    })
    return file_app_reverse_config_proto_rawDescData
}

// 定义枚举类型和消息类型的切片
var file_app_reverse_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_app_reverse_config_proto_msgTypes = make([]protoimpl.MessageInfo, 4)
var file_app_reverse_config_proto_goTypes = []interface{}{
    (Control_State)(0),   // 0: v2ray.core.app.reverse.Control.State
    (*Control)(nil),      // 1: v2ray.core.app.reverse.Control
    (*BridgeConfig)(nil), // 2: v2ray.core.app.reverse.BridgeConfig
    (*PortalConfig)(nil), // 3: v2ray.core.app.reverse.PortalConfig
    (*Config)(nil),       // 4: v2ray.core.app.reverse.Config
}

// 定义依赖索引的切片
var file_app_reverse_config_proto_depIdxs = []int32{
    0, // 0: v2ray.core.app.reverse.Control.state:type_name -> v2ray.core.app.reverse.Control.State
    2, // 1: v2ray.core.app.reverse.Config.bridge_config:type_name -> v2ray.core.app.reverse.BridgeConfig
    3, // 2: v2ray.core.app.reverse.Config.portal_config:type_name -> v2ray.core.app.reverse.PortalConfig
    3, // [3:3] is the sub-list for method output_type
    3, // [3:3] is the sub-list for method input_type
    3, // [3:3] is the sub-list for extension type_name
    3, // [3:3] is the sub-list for extension extendee
    0, // [0:3] is the sub-list for field type_name
}

// 初始化函数
func init() { file_app_reverse_config_proto_init() }
func file_app_reverse_config_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_app_reverse_config_proto != nil {
        return
    }
}
    # 如果未启用 protoimpl.UnsafeEnabled，则设置消息类型的导出函数
    if !protoimpl.UnsafeEnabled:
        file_app_reverse_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{}:
            # 对 Control 类型的消息设置导出函数
            switch v := v.(*Control); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        # 对 BridgeConfig 类型的消息设置导出函数
        file_app_reverse_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*BridgeConfig); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        # 对 PortalConfig 类型的消息设置导出函数
        file_app_reverse_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{}:
            switch v := v.(*PortalConfig); i {
            case 0:
                return &v.state
            case 1:
                return &v.sizeCache
            case 2:
                return &v.unknownFields
            default:
                return nil
            }
        # 对 Config 类型的消息设置导出函数
        file_app_reverse_config_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{}:
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
    # 定义一个空结构体 x
    type x struct{}
    # 创建一个 protoimpl.TypeBuilder 对象
    out := protoimpl.TypeBuilder{
        # 设置 File 属性
        File: protoimpl.DescBuilder{
            # 设置 GoPackagePath 属性为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            # 设置 RawDescriptor 属性为指定的原始描述符
            RawDescriptor: file_app_reverse_config_proto_rawDesc,
            # 设置 NumEnums 属性为 1
            NumEnums:      1,
            # 设置 NumMessages 属性为 4
            NumMessages:   4,
            # 设置 NumExtensions 属性为 0
            NumExtensions: 0,
            # 设置 NumServices 属性为 0
            NumServices:   0,
        },
        # 设置 GoTypes 属性为指定的 Go 类型
        GoTypes:           file_app_reverse_config_proto_goTypes,
        # 设置 DependencyIndexes 属性为指定的依赖索引
        DependencyIndexes: file_app_reverse_config_proto_depIdxs,
        # 设置 EnumInfos 属性为指定的枚举类型信息
        EnumInfos:         file_app_reverse_config_proto_enumTypes,
        # 设置 MessageInfos 属性为指定的消息类型信息
        MessageInfos:      file_app_reverse_config_proto_msgTypes,
    }.Build()
    # 将 out.File 赋值给 File_app_reverse_config_proto
    File_app_reverse_config_proto = out.File
    # 将 file_app_reverse_config_proto_rawDesc 设置为 nil
    file_app_reverse_config_proto_rawDesc = nil
    # 将 file_app_reverse_config_proto_goTypes 设置为 nil
    file_app_reverse_config_proto_goTypes = nil
    # 将 file_app_reverse_config_proto_depIdxs 设置为 nil
    file_app_reverse_config_proto_depIdxs = nil
# 闭合前面的函数定义
```