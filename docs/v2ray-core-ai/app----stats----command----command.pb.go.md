# `v2ray-core\app\stats\command\command.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: app/stats/command/command.proto

package command

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个编译时断言，用于确保使用了足够新的 legacy proto 包版本。
const _ = proto.ProtoPackageIsVersion4

type GetStatsRequest struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    // 统计计数器的名称。
    Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
    // 是否重置计数器以获取其值。
    Reset_ bool `protobuf:"varint,2,opt,name=reset,proto3" json:"reset,omitempty"`
}

func (x *GetStatsRequest) Reset() {
    *x = GetStatsRequest{}  // 重置 GetStatsRequest 对象
    if protoimpl.UnsafeEnabled {
        mi := &file_app_stats_command_command_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *GetStatsRequest) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*GetStatsRequest) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *GetStatsRequest) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_command_command_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
}
    # 返回对象 x 的消息
    return mi.MessageOf(x)
// Deprecated: Use GetStatsRequest.ProtoReflect.Descriptor instead.
// 使用 GetStatsRequest.ProtoReflect.Descriptor 替代
func (*GetStatsRequest) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{0}
}

// 获取 GetStatsRequest 的名称
func (x *GetStatsRequest) GetName() string {
    if x != nil {
        return x.Name
    }
    return ""
}

// 获取 GetStatsRequest 的 Reset_ 属性
func (x *GetStatsRequest) GetReset_() bool {
    if x != nil {
        return x.Reset_
    }
    return false
}

// 定义 Stat 结构体
type Stat struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Name  string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
    Value int64  `protobuf:"varint,2,opt,name=value,proto3" json:"value,omitempty"`
}

// 重置 Stat 结构体
func (x *Stat) Reset() {
    *x = Stat{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_stats_command_command_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 Stat 结构体转换为字符串
func (x *Stat) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*Stat) ProtoMessage() {}

// 获取 Stat 结构体的反射信息
func (x *Stat) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_command_command_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Stat.ProtoReflect.Descriptor instead.
// 使用 Stat.ProtoReflect.Descriptor 替代
func (*Stat) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{1}
}

// 获取 Stat 结构体的名称
func (x *Stat) GetName() string {
    if x != nil {
        return x.Name
    }
    return ""
}

// 获取 Stat 结构体的值
func (x *Stat) GetValue() int64 {
    if x != nil {
        return x.Value
    }
    return 0
}

// 定义 GetStatsResponse 结构体
type GetStatsResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
    # 定义一个名为Stat的指针变量，用于存储protobuf编码的stat信息
    Stat *Stat `protobuf:"bytes,1,opt,name=stat,proto3" json:"stat,omitempty"`
}

// 重置 GetStatsResponse 对象
func (x *GetStatsResponse) Reset() {
    // 将 GetStatsResponse 对象重置为空对象
    *x = GetStatsResponse{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型
        mi := &file_app_stats_command_command_proto_msgTypes[2]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 GetStatsResponse 对象的字符串表示
func (x *GetStatsResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*GetStatsResponse) ProtoMessage() {}

// 返回 GetStatsResponse 对象的反射信息
func (x *GetStatsResponse) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_command_command_proto_msgTypes[2]
    // 如果启用了不安全操作并且对象不为空
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

// 弃用：使用 GetStatsResponse.ProtoReflect.Descriptor 替代
func (*GetStatsResponse) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{2}
}

// 获取 GetStatsResponse 对象的 Stat 属性
func (x *GetStatsResponse) GetStat() *Stat {
    if x != nil {
        return x.Stat
    }
    return nil
}

// 定义 QueryStatsRequest 结构体
type QueryStatsRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义 Pattern 字段
    Pattern string `protobuf:"bytes,1,opt,name=pattern,proto3" json:"pattern,omitempty"`
    // 定义 Reset_ 字段
    Reset_  bool   `protobuf:"varint,2,opt,name=reset,proto3" json:"reset,omitempty"`
}

// 重置 QueryStatsRequest 对象
func (x *QueryStatsRequest) Reset() {
    // 将 QueryStatsRequest 对象重置为空对象
    *x = QueryStatsRequest{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型
        mi := &file_app_stats_command_command_proto_msgTypes[3]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 QueryStatsRequest 对象的字符串表示
func (x *QueryStatsRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*QueryStatsRequest) ProtoMessage() {}

// 返回 QueryStatsRequest 对象的反射信息
func (x *QueryStatsRequest) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_command_command_proto_msgTypes[3]
    # 检查是否启用了不安全的特性，并且 x 不为空
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
// Deprecated: Use QueryStatsRequest.ProtoReflect.Descriptor instead.
// 使用 QueryStatsRequest.ProtoReflect.Descriptor 替代，已废弃
func (*QueryStatsRequest) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{3}
}

// 获取查询统计请求中的模式
func (x *QueryStatsRequest) GetPattern() string {
    if x != nil {
        return x.Pattern
    }
    return ""
}

// 获取查询统计请求中的重置标志
func (x *QueryStatsRequest) GetReset_() bool {
    if x != nil {
        return x.Reset_
    }
    return false
}

// 用于存储查询统计响应的结构体
type QueryStatsResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Stat []*Stat `protobuf:"bytes,1,rep,name=stat,proto3" json:"stat,omitempty"`
}

// 重置查询统计响应
func (x *QueryStatsResponse) Reset() {
    *x = QueryStatsResponse{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_stats_command_command_proto_msgTypes[4]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将查询统计响应转换为字符串
func (x *QueryStatsResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*QueryStatsResponse) ProtoMessage() {}

// 获取查询统计响应的反射信息
func (x *QueryStatsResponse) ProtoReflect() protoreflect.Message {
    mi := &file_app_stats_command_command_proto_msgTypes[4]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use QueryStatsResponse.ProtoReflect.Descriptor instead.
// 使用 QueryStatsResponse.ProtoReflect.Descriptor 替代，已废弃
func (*QueryStatsResponse) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{4}
}

// 获取查询统计响应中的统计数据
func (x *QueryStatsResponse) GetStat() []*Stat {
    if x != nil {
        return x.Stat
    }
    return nil
}

// 用于存储系统统计请求的结构体
type SysStatsRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置系统统计请求
func (x *SysStatsRequest) Reset() {
    *x = SysStatsRequest{}
}
    # 如果启用了不安全的功能，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_app_stats_command_command_proto_msgTypes[5]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 将 SysStatsRequest 结构体转换为字符串形式
func (x *SysStatsRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*SysStatsRequest) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *SysStatsRequest) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_stats_command_command_proto_msgTypes[5]
    // 如果启用了不安全的操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息未加载，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 SysStatsRequest.ProtoReflect.Descriptor 替代
func (*SysStatsRequest) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{5}
}

// SysStatsResponse 结构体定义
type SysStatsResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    NumGoroutine uint32 `protobuf:"varint,1,opt,name=NumGoroutine,proto3" json:"NumGoroutine,omitempty"`
    NumGC        uint32 `protobuf:"varint,2,opt,name=NumGC,proto3" json:"NumGC,omitempty"`
    Alloc        uint64 `protobuf:"varint,3,opt,name=Alloc,proto3" json:"Alloc,omitempty"`
    TotalAlloc   uint64 `protobuf:"varint,4,opt,name=TotalAlloc,proto3" json:"TotalAlloc,omitempty"`
    Sys          uint64 `protobuf:"varint,5,opt,name=Sys,proto3" json:"Sys,omitempty"`
    Mallocs      uint64 `protobuf:"varint,6,opt,name=Mallocs,proto3" json:"Mallocs,omitempty"`
    Frees        uint64 `protobuf:"varint,7,opt,name=Frees,proto3" json:"Frees,omitempty"`
    LiveObjects  uint64 `protobuf:"varint,8,opt,name=LiveObjects,proto3" json:"LiveObjects,omitempty"`
    PauseTotalNs uint64 `protobuf:"varint,9,opt,name=PauseTotalNs,proto3" json:"PauseTotalNs,omitempty"`
    Uptime       uint32 `protobuf:"varint,10,opt,name=Uptime,proto3" json:"Uptime,omitempty"`
}

// 重置 SysStatsResponse 结构体
func (x *SysStatsResponse) Reset() {
    *x = SysStatsResponse{}
}
    # 如果启用了不安全的功能，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_app_stats_command_command_proto_msgTypes[6]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 将 SysStatsResponse 结构体转换为字符串
func (x *SysStatsResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SysStatsResponse) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *SysStatsResponse) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_stats_command_command_proto_msgTypes[6]
    // 如果启用了不安全的操作，并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息未加载，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use SysStatsResponse.ProtoReflect.Descriptor instead.
// 获取消息的描述符
func (*SysStatsResponse) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{6}
}

// 获取 NumGoroutine 字段的值
func (x *SysStatsResponse) GetNumGoroutine() uint32 {
    if x != nil {
        return x.NumGoroutine
    }
    return 0
}

// 获取 NumGC 字段的值
func (x *SysStatsResponse) GetNumGC() uint32 {
    if x != nil {
        return x.NumGC
    }
    return 0
}

// 获取 Alloc 字段的值
func (x *SysStatsResponse) GetAlloc() uint64 {
    if x != nil {
        return x.Alloc
    }
    return 0
}

// 获取 TotalAlloc 字段的值
func (x *SysStatsResponse) GetTotalAlloc() uint64 {
    if x != nil {
        return x.TotalAlloc
    }
    return 0
}

// 获取 Sys 字段的值
func (x *SysStatsResponse) GetSys() uint64 {
    if x != nil {
        return x.Sys
    }
    return 0
}

// 获取 Mallocs 字段的值
func (x *SysStatsResponse) GetMallocs() uint64 {
    if x != nil {
        return x.Mallocs
    }
    return 0
}

// 获取 Frees 字段的值
func (x *SysStatsResponse) GetFrees() uint64 {
    if x != nil {
        return x.Frees
    }
    return 0
}

// 获取 LiveObjects 字段的值
func (x *SysStatsResponse) GetLiveObjects() uint64 {
    if x != nil {
        return x.LiveObjects
    }
    return 0
}

// 获取 PauseTotalNs 字段的值
func (x *SysStatsResponse) GetPauseTotalNs() uint64 {
    if x != nil {
        return x.PauseTotalNs
    }
    return 0
}

// 获取 Uptime 字段的值
func (x *SysStatsResponse) GetUptime() uint32 {
    if x != nil {
        return x.Uptime
    }
    return 0
}

// Config 结构体
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    # 定义一个变量 unknownFields，类型为 protoimpl.UnknownFields
// 重置 Config 对象，将其内容置为空
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_app_stats_command_command_proto_msgTypes[7]
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
    mi := &file_app_stats_command_command_proto_msgTypes[7]
    // 如果启用了不安全操作并且 Config 对象不为空，则将消息状态存储到消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    return file_app_stats_command_command_proto_rawDescGZIP(), []int{7}
}

// 定义 FileDescriptor 变量
var File_app_stats_command_command_proto protoreflect.FileDescriptor

// 定义原始描述信息的字节切片
var file_app_stats_command_command_proto_rawDesc = []byte{
    // 这里是一段二进制数据，表示文件描述信息
}
    # 创建一个十六进制数值列表
    0x28, 0x09, 0x52, 0x04, 0x6e, 0x61, 0x6d, 0x65, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75,
    0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x03, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22, 0x4a,
    0x0a, 0x10, 0x47, 0x65, 0x74, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e,
    0x73, 0x65, 0x12, 0x36, 0x0a, 0x04, 0x73, 0x74, 0x61, 0x74, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x22, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70,
    0x70, 0x2e, 0x73, 0x74, 0x61, 0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e,
    0x53, 0x74, 0x61, 0x74, 0x52, 0x04, 0x73, 0x74, 0x61, 0x74, 0x22, 0x43, 0x0a, 0x11, 0x51, 0x75,
    0x65, 0x72, 0x79, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12,
    0x18, 0a, 0x07, 0x70, 0x61, 0x74, 0x74, 0x65, 0x72, 0x6e, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09,
    0x52, 0x07, 0x70, 0x61, 0x74, 0x74, 0x65, 0x72, 0x6e, 0x12, 0x14, 0a, 0x05, 0x72, 0x65, 0x73,
    0x65, 0x74, 0x18, 0x02, 0x20, 0x01, 0x28, 0x08, 0x52, 0x05, 0x72, 0x65, 0x73, 0x65, 0x74, 0x22,
    0x4c, 0x0a, 0x12, 0x51, 0x75, 0x65, 0x72, 0x79, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x73,
    0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x36, 0x0a, 0x04, 0x73, 0x74, 0x61, 0x74, 0x18, 0x01, 0x20,
    0x03, 0x28, 0x0b, 0x32, 0x22, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x61, 0x70, 0x70, 0x2e, 0x73, 0x74, 0x61, 0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61,
    0x6e, 0x64, 0x2e, 0x53, 0x74, 0x61, 0x74, 0x52, 0x04, 0x73, 0x74, 0x61, 0x74, 0x22, 0x11, 0x0a,
    0x0f, 0x53, 0x79, 0x73, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74,
    # 创建一个十六进制数值列表，表示一些数据
    0x47, 0x6f, 0x72, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x65, 0x12, 0x14, 0x0a, 0x05, 0x4e, 0x75, 0x6d,
    0x47, 0x43, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05, 0x4e, 0x75, 0x6d, 0x47, 0x43, 0x12,
    0x14, 0x0a, 0x05, 0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x18, 0x03, 0x20, 0x01, 0x28, 0x04, 0x52, 0x05,
    0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x12, 0x1e, 0x0a, 0x0a, 0x54, 0x6f, 0x74, 0x61, 0x6c, 0x41, 0x6c,
    0x6c, 0x6f, 0x63, 0x18, 0x04, 0x20, 0x01, 0x28, 0x04, 0x52, 0x0a, 0x54, 0x6f, 0x74, 0x61, 0x6c,
    0x41, 0x6c, 0x6c, 0x6f, 0x63, 0x12, 0x10, 0a, 0x03, 0x53, 0x79, 0x73, 0x18, 0x05, 0x20, 0x01, 0x28,
    0x04, 0x52, 0x03, 0x53, 0x79, 0x73, 0x12, 0x18, 0x0a, 0x07, 0x4d, 0x61, 0x6c, 0x6c, 0x6f, 0x63, 0x73,
    0x18, 0x06, 0x20, 0x01, 0x28, 0x04, 0x52, 0x07, 0x4d, 0x61, 0x6c, 0x6c, 0x6f, 0x63, 0x73, 0x12, 0x14,
    0x0a, 0x05, 0x46, 0x72, 0x65, 0x65, 0x73, 0x18, 0x07, 0x20, 0x01, 0x28, 0x04, 0x52, 0x05, 0x46, 0x72,
    0x65, 0x65, 0x73, 0x12, 0x20, 0x0a, 0x0b, 0x4c, 0x69, 0x76, 0x65, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
    0x73, 0x18, 0x08, 0x20, 0x01, 0x28, 0x04, 0x52, 0x0b, 0x4c, 0x69, 0x76, 0x65, 0x4f, 0x62, 0x6a, 0x65,
    0x63, 0x74, 0x73, 0x12, 0x22, 0x0a, 0x0c, 0x50, 0x61, 0x75, 0x73, 0x65, 0x54, 0x6f, 0x74, 0x61, 0x6c,
    0x4e, 0x73, 0x18, 0x09, 0x20, 0x01, 0x28, 0x04, 0x52, 0x0c, 0x50, 0x61, 0x75, 0x73, 0x65, 0x54, 0x6f,
    0x74, 0x61, 0x6c, 0x4e, 0x73, 0x12, 0x16, 0x0a, 0x06, 0x55, 0x70, 0x74, 0x69, 0x6d, 0x65, 0x18, 0x0a,
    0x20, 0x01, 0x28, 0x0d, 0x52, 0x06, 0x55, 0x70, 0x74, 0x69, 0x6d, 0x65, 0x22, 0x08, 0x0a, 0x06, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x32, 0xde, 0x02, 0x0a, 0x0c, 0x53, 0x74, 0x61, 0x74, 0x73, 0x53, 0x65,
    0x72, 0x76, 0x69, 0x63, 0x65, 0x12, 0x6b, 0x0a, 0x08, 0x47, 0x65, 0x74, 0x53, 0x74, 0x61, 0x74, 0x73,
    # 以上是一些十六进制数值列表，表示一些数据
    # 创建一个十六进制数列表，可能是某种数据的编码
    0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x2e, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x73, 0x74, 0x61,
    0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x47, 0x65, 0x74, 0x53, 0x74,
    0x61, 0x74, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12, 0x71, 0x0a,
    0x0a, 0x51, 0x75, 0x65, 0x72, 0x79, 0x53, 0x74, 0x61, 0x74, 0x73, 0x12, 0x2f, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x73, 0x74, 0x61,
    0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x51, 0x75, 0x65, 0x72, 0x79,
    0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12,
    0x6e, 0x0a, 0x0b, 0x47, 0x65, 0x74, 0x53, 0x79, 0x73, 0x53, 0x74, 0x61, 0x74, 0x73, 0x12, 0x2d,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e,
    0x73, 0x74, 0x61, 0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x53, 0x79,
    0x73, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x2e, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x73,
    0x74, 0x61, 0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x53, 0x79, 0x73,
    0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x42,
    0x65, 0x0a, 0x20, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
    0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x73, 0x74, 0x61, 0x74, 0x73, 0x2e, 0x63, 0x6f, 0x6d,
    # 这是一串十六进制数字，可能是某种编码或者数据表示方式，需要进一步解析才能理解其含义
// 定义变量，确保 file_app_stats_command_command_proto_rawDescData 只被初始化一次
var (
    file_app_stats_command_command_proto_rawDescOnce sync.Once
    file_app_stats_command_command_proto_rawDescData = file_app_stats_command_command_proto_rawDesc
)

// 使用 GZIP 压缩文件内容
func file_app_stats_command_command_proto_rawDescGZIP() []byte {
    // 确保文件内容只被压缩一次
    file_app_stats_command_command_proto_rawDescOnce.Do(func() {
        file_app_stats_command_command_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_stats_command_command_proto_rawDescData)
    })
    return file_app_stats_command_command_proto_rawDescData
}

// 定义消息类型
var file_app_stats_command_command_proto_msgTypes = make([]protoimpl.MessageInfo, 8)
// 定义 Go 类型
var file_app_stats_command_command_proto_goTypes = []interface{}{
    (*GetStatsRequest)(nil),    // 0: v2ray.core.app.stats.command.GetStatsRequest
    (*Stat)(nil),               // 1: v2ray.core.app.stats.command.Stat
    (*GetStatsResponse)(nil),   // 2: v2ray.core.app.stats.command.GetStatsResponse
    (*QueryStatsRequest)(nil),  // 3: v2ray.core.app.stats.command.QueryStatsRequest
    (*QueryStatsResponse)(nil), // 4: v2ray.core.app.stats.command.QueryStatsResponse
    (*SysStatsRequest)(nil),    // 5: v2ray.core.app.stats.command.SysStatsRequest
    (*SysStatsResponse)(nil),   // 6: v2ray.core.app.stats.command.SysStatsResponse
    (*Config)(nil),             // 7: v2ray.core.app.stats.command.Config
}
// 定义依赖索引
var file_app_stats_command_command_proto_depIdxs = []int32{
    1, // 0: v2ray.core.app.stats.command.GetStatsResponse.stat:type_name -> v2ray.core.app.stats.command.Stat
    1, // 1: v2ray.core.app.stats.command.QueryStatsResponse.stat:type_name -> v2ray.core.app.stats.command.Stat
    0, // 2: v2ray.core.app.stats.command.StatsService.GetStats:input_type -> v2ray.core.app.stats.command.GetStatsRequest
    3, // 3: v2ray.core.app.stats.command.StatsService.QueryStats:input_type -> v2ray.core.app.stats.command.QueryStatsRequest
    5, // 4: v2ray.core.app.stats.command.StatsService.GetSysStats:input_type -> v2ray.core.app.stats.command.SysStatsRequest
    2, // 5: v2ray.core.app.stats.command.StatsService.GetStats:output_type -> v2ray.core.app.stats.command.GetStatsResponse
    // 第5个元素，表示 v2ray.core.app.stats.command.StatsService.GetStats 方法的输出类型为 v2ray.core.app.stats.command.GetStatsResponse

    4, // 6: v2ray.core.app.stats.command.StatsService.QueryStats:output_type -> v2ray.core.app.stats.command.QueryStatsResponse
    // 第6个元素，表示 v2ray.core.app.stats.command.StatsService.QueryStats 方法的输出类型为 v2ray.core.app.stats.command.QueryStatsResponse

    6, // 7: v2ray.core.app.stats.command.StatsService.GetSysStats:output_type -> v2ray.core.app.stats.command.SysStatsResponse
    // 第7个元素，表示 v2ray.core.app.stats.command.StatsService.GetSysStats 方法的输出类型为 v2ray.core.app.stats.command.SysStatsResponse

    5, // [5:8] is the sub-list for method output_type
    // 第5到第8个元素是方法输出类型的子列表

    2, // [2:5] is the sub-list for method input_type
    // 第2到第5个元素是方法输入类型的子列表

    2, // [2:2] is the sub-list for extension type_name
    // 第2到第2个元素是扩展类型名的子列表

    2, // [2:2] is the sub-list for extension extendee
    // 第2到第2个元素是扩展扩展者的子列表

    0, // [0:2] is the sub-list for field type_name
    // 第0到第2个元素是字段类型名的子列表
func init() { file_app_stats_command_command_proto_init() }
func file_app_stats_command_command_proto_init() {
    // 如果 File_app_stats_command_command_proto 不为空，则直接返回，不进行初始化
    if File_app_stats_command_command_proto != nil {
        return
    }
    // 定义一个空结构体 x
    type x struct{}
    // 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            // 设置 Go 包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            // 设置原始描述
            RawDescriptor: file_app_stats_command_command_proto_rawDesc,
            // 设置枚举数量为 0
            NumEnums:      0,
            // 设置消息数量为 8
            NumMessages:   8,
            // 设置扩展数量为 0
            NumExtensions: 0,
            // 设置服务数量为 1
            NumServices:   1,
        },
        // 设置 Go 类型
        GoTypes:           file_app_stats_command_command_proto_goTypes,
        // 设置依赖索引
        DependencyIndexes: file_app_stats_command_command_proto_depIdxs,
        // 设置消息信息
        MessageInfos:      file_app_stats_command_command_proto_msgTypes,
    }.Build()
    // 将构建的类型赋值给 File_app_stats_command_command_proto
    File_app_stats_command_command_proto = out.File
    // 将原始描述设置为空
    file_app_stats_command_command_proto_rawDesc = nil
    // 将 Go 类型设置为空
    file_app_stats_command_command_proto_goTypes = nil
    // 将依赖索引设置为空
    file_app_stats_command_command_proto_depIdxs = nil
}
```