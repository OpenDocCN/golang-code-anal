# `v2ray-core\app\proxyman\command\command.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件路径: app/proxyman/command/command.proto

package command

import (
    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
    core "v2ray.com/core"  // 导入 core 包
    protocol "v2ray.com/core/common/protocol"  // 导入 protocol 包
    serial "v2ray.com/core/common/serial"  // 导入 serial 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码版本足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 版本足够新
)

// 这是一个编译时断言，用于确保正在使用足够新的旧版 proto 包。
const _ = proto.ProtoPackageIsVersion4

type AddUserOperation struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    User *protocol.User `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`  // 添加用户操作结构体
}

func (x *AddUserOperation) Reset() {
    *x = AddUserOperation{}  // 重置 AddUserOperation 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[0]  // 获取消息类型信息
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *AddUserOperation) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回 AddUserOperation 结构体的字符串表示
}

func (*AddUserOperation) ProtoMessage() {}  // AddUserOperation 结构体实现 ProtoMessage 接口

func (x *AddUserOperation) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[0]  // 获取消息类型信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)  // 存储消息信息
        }
        return ms
    }
    return mi.MessageOf(x)  // 返回消息类型信息
}
// Deprecated: Use AddUserOperation.ProtoReflect.Descriptor instead.
// 返回 AddUserOperation 的描述符
func (*AddUserOperation) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{0}
}

// 返回 AddUserOperation 中的 User 对象
func (x *AddUserOperation) GetUser() *protocol.User {
    if x != nil {
        return x.User
    }
    return nil
}

// RemoveUserOperation 结构体定义
type RemoveUserOperation struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Email string `protobuf:"bytes,1,opt,name=email,proto3" json:"email,omitempty"`
}

// 重置 RemoveUserOperation 结构体
func (x *RemoveUserOperation) Reset() {
    *x = RemoveUserOperation{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RemoveUserOperation 结构体的字符串表示
func (x *RemoveUserOperation) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 返回 RemoveUserOperation 结构体的 ProtoMessage
func (*RemoveUserOperation) ProtoMessage() {}

// 返回 RemoveUserOperation 结构体的 ProtoReflect
func (x *RemoveUserOperation) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use RemoveUserOperation.ProtoReflect.Descriptor instead.
// 返回 RemoveUserOperation 的描述符
func (*RemoveUserOperation) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{1}
}

// 返回 RemoveUserOperation 中的 Email 字段
func (x *RemoveUserOperation) GetEmail() string {
    if x != nil {
        return x.Email
    }
    return ""
}

// AddInboundRequest 结构体定义
type AddInboundRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Inbound *core.InboundHandlerConfig `protobuf:"bytes,1,opt,name=inbound,proto3" json:"inbound,omitempty"`
}

// 重置 AddInboundRequest 结构体
func (x *AddInboundRequest) Reset() {
    // ...
}
    # 创建一个空的AddInboundRequest结构体指针，并赋值给x
    *x = AddInboundRequest{}
    # 如果启用了不安全操作，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取file_app_proxyman_command_command_proto_msgTypes数组中索引为2的元素，并赋值给mi
        mi := &file_app_proxyman_command_command_proto_msgTypes[2]
        # 获取x指针的消息状态，并将消息信息mi存储到其中
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
// 将 AddInboundRequest 结构体转换为字符串形式
func (x *AddInboundRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*AddInboundRequest) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *AddInboundRequest) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use AddInboundRequest.ProtoReflect.Descriptor instead.
func (*AddInboundRequest) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{2}
}

// 获取 AddInboundRequest 结构体中的 Inbound 字段
func (x *AddInboundRequest) GetInbound() *core.InboundHandlerConfig {
    if x != nil {
        return x.Inbound
    }
    return nil
}

// 重置 AddInboundResponse 结构体
func (x *AddInboundResponse) Reset() {
    *x = AddInboundResponse{}
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 AddInboundResponse 结构体转换为字符串形式
func (x *AddInboundResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*AddInboundResponse) ProtoMessage() {}

// 实现 ProtoReflect 接口的方法
func (x *AddInboundResponse) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[3]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use AddInboundResponse.ProtoReflect.Descriptor instead.
func (*AddInboundResponse) Descriptor() ([]byte, []int) {
    // 返回 AddInboundResponse 结构体的描述信息
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{3}
}
    # 返回 file_app_proxyman_command_command_proto_rawDescGZIP() 函数的结果和空的整数列表
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{3}
// 定义一个结构体类型 RemoveInboundRequest，包含消息状态、大小缓存和未知字段
type RemoveInboundRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义一个字符串类型的字段 Tag，使用protobuf标签指定其序号和其他属性
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
}

// 重置 RemoveInboundRequest 对象的状态
func (x *RemoveInboundRequest) Reset() {
    // 将 RemoveInboundRequest 对象重置为空对象
    *x = RemoveInboundRequest{}
    // 如果启用了不安全操作，获取消息类型并存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[4]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RemoveInboundRequest 对象的字符串表示形式
func (x *RemoveInboundRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*RemoveInboundRequest) ProtoMessage() {}

// 返回 RemoveInboundRequest 对象的反射信息
func (x *RemoveInboundRequest) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[4]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 已弃用：使用 RemoveInboundRequest.ProtoReflect.Descriptor 替代
func (*RemoveInboundRequest) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{4}
}

// 获取 RemoveInboundRequest 对象的 Tag 字段值
func (x *RemoveInboundRequest) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 定义一个结构体类型 RemoveInboundResponse，包含消息状态、大小缓存和未知字段
type RemoveInboundResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置 RemoveInboundResponse 对象的状态
func (x *RemoveInboundResponse) Reset() {
    // 将 RemoveInboundResponse 对象重置为空对象
    *x = RemoveInboundResponse{}
    // 如果启用了不安全操作，获取消息类型并存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RemoveInboundResponse 对象的字符串表示形式
func (x *RemoveInboundResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*RemoveInboundResponse) ProtoMessage() {}
// 返回 RemoveInboundResponse 的反射信息
func (x *RemoveInboundResponse) ProtoReflect() protoreflect.Message {
    // 获取 RemoveInboundResponse 的消息类型信息
    mi := &file_app_proxyman_command_command_proto_msgTypes[5]
    // 如果启用了不安全操作并且 x 不为空
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

// Deprecated: 使用 RemoveInboundResponse.ProtoReflect.Descriptor 替代
func (*RemoveInboundResponse) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{5}
}

// 定义 AlterInboundRequest 结构体
type AlterInboundRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Tag       string               `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    Operation *serial.TypedMessage `protobuf:"bytes,2,opt,name=operation,proto3" json:"operation,omitempty"`
}

// 重置 AlterInboundRequest 结构体
func (x *AlterInboundRequest) Reset() {
    *x = AlterInboundRequest{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取 AlterInboundRequest 的消息类型信息
        mi := &file_app_proxyman_command_command_proto_msgTypes[6]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 AlterInboundRequest 结构体的字符串表示
func (x *AlterInboundRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*AlterInboundRequest) ProtoMessage() {}

// 返回 AlterInboundRequest 的反射信息
func (x *AlterInboundRequest) ProtoReflect() protoreflect.Message {
    // 获取 AlterInboundRequest 的消息类型信息
    mi := &file_app_proxyman_command_command_proto_msgTypes[6]
    // 如果启用了不安全操作并且 x 不为空
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

// Deprecated: 使用 AlterInboundRequest.ProtoReflect.Descriptor 替代
func (*AlterInboundRequest) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{6}
}
// 获取AlterInboundRequest的标签
func (x *AlterInboundRequest) GetTag() string {
    // 如果x不为空，则返回其标签
    if x != nil {
        return x.Tag
    }
    // 否则返回空字符串
    return ""
}

// 获取AlterInboundRequest的操作
func (x *AlterInboundRequest) GetOperation() *serial.TypedMessage {
    // 如果x不为空，则返回其操作
    if x != nil {
        return x.Operation
    }
    // 否则返回空
    return nil
}

// AlterInboundResponse结构体
type AlterInboundResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置AlterInboundResponse对象
func (x *AlterInboundResponse) Reset() {
    *x = AlterInboundResponse{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[7]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回AlterInboundResponse对象的字符串表示
func (x *AlterInboundResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现ProtoMessage接口
func (*AlterInboundResponse) ProtoMessage() {}

// 返回AlterInboundResponse对象的反射信息
func (x *AlterInboundResponse) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[7]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用AlterInboundResponse.ProtoReflect.Descriptor代替
func (*AlterInboundResponse) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{7}
}

// AddOutboundRequest结构体
type AddOutboundRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Outbound *core.OutboundHandlerConfig `protobuf:"bytes,1,opt,name=outbound,proto3" json:"outbound,omitempty"`
}

// 重置AddOutboundRequest对象
func (x *AddOutboundRequest) Reset() {
    *x = AddOutboundRequest{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[8]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}
// 返回 AddOutboundRequest 对象的字符串表示形式
func (x *AddOutboundRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*AddOutboundRequest) ProtoMessage() {}

// 返回 AddOutboundRequest 对象的反射信息
func (x *AddOutboundRequest) ProtoReflect() protoreflect.Message {
    // 获取 AddOutboundRequest 对象的消息类型信息
    mi := &file_app_proxyman_command_command_proto_msgTypes[8]
    // 如果启用了不安全的操作，并且对象不为空
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

// Deprecated: Use AddOutboundRequest.ProtoReflect.Descriptor instead.
// 返回 AddOutboundRequest 对象的描述符
func (*AddOutboundRequest) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{8}
}

// 返回 AddOutboundRequest 对象的 Outbound 属性
func (x *AddOutboundRequest) GetOutbound() *core.OutboundHandlerConfig {
    if x != nil {
        return x.Outbound
    }
    return nil
}

// 重置 AddOutboundResponse 对象
func (x *AddOutboundResponse) Reset() {
    *x = AddOutboundResponse{}
    // 如果启用了不安全的操作
    if protoimpl.UnsafeEnabled {
        // 获取 AddOutboundResponse 对象的消息类型信息
        mi := &file_app_proxyman_command_command_proto_msgTypes[9]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 AddOutboundResponse 对象的字符串表示形式
func (x *AddOutboundResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*AddOutboundResponse) ProtoMessage() {}

// 返回 AddOutboundResponse 对象的反射信息
func (x *AddOutboundResponse) ProtoReflect() protoreflect.Message {
    // 获取 AddOutboundResponse 对象的消息类型信息
    mi := &file_app_proxyman_command_command_proto_msgTypes[9]
    // 如果启用了不安全的操作，并且对象不为空
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

// Deprecated: Use AddOutboundResponse.ProtoReflect.Descriptor instead.
// 返回 AddOutboundResponse 对象的描述符
func (*AddOutboundResponse) Descriptor() ([]byte, []int) {
    # 返回 file_app_proxyman_command_command_proto_rawDescGZIP() 函数的结果和空的整数列表
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{9}
// 定义 RemoveOutboundRequest 结构体，包含状态、大小缓存和未知字段
type RemoveOutboundRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 定义 Tag 字段，类型为字符串，protobuf 标签为 bytes,1,opt,name=tag,proto3，json 标签为 tag,omitempty
    Tag string `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
}

// 重置 RemoveOutboundRequest 结构体
func (x *RemoveOutboundRequest) Reset() {
    *x = RemoveOutboundRequest{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[10]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RemoveOutboundRequest 结构体的字符串表示
func (x *RemoveOutboundRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*RemoveOutboundRequest) ProtoMessage() {}

// 返回 RemoveOutboundRequest 结构体的反射信息
func (x *RemoveOutboundRequest) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[10]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use RemoveOutboundRequest.ProtoReflect.Descriptor instead.
// 返回 RemoveOutboundRequest 结构体的描述符
func (*RemoveOutboundRequest) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{10}
}

// 获取 RemoveOutboundRequest 结构体的 Tag 字段值
func (x *RemoveOutboundRequest) GetTag() string {
    if x != nil {
        return x.Tag
    }
    return ""
}

// 定义 RemoveOutboundResponse 结构体，包含状态、大小缓存和未知字段
type RemoveOutboundResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置 RemoveOutboundResponse 结构体
func (x *RemoveOutboundResponse) Reset() {
    *x = RemoveOutboundResponse{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[11]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RemoveOutboundResponse 结构体的字符串表示
func (x *RemoveOutboundResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*RemoveOutboundResponse) ProtoMessage() {}
func (x *RemoveOutboundResponse) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_proxyman_command_command_proto_msgTypes[11]
    // 如果启用了不安全操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        // 返回消息状态
        return ms
    }
    // 返回消息类型信息
    return mi.MessageOf(x)
}

// Deprecated: Use RemoveOutboundResponse.ProtoReflect.Descriptor instead.
func (*RemoveOutboundResponse) Descriptor() ([]byte, []int) {
    // 返回原始描述信息的 GZIP 压缩版本
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{11}
}

type AlterOutboundRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Tag       string               `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
    Operation *serial.TypedMessage `protobuf:"bytes,2,opt,name=operation,proto3" json:"operation,omitempty"`
}

func (x *AlterOutboundRequest) Reset() {
    // 重置 AlterOutboundRequest 结构体
    *x = AlterOutboundRequest{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_app_proxyman_command_command_proto_msgTypes[12]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

func (x *AlterOutboundRequest) String() string {
    // 返回消息的字符串表示形式
    return protoimpl.X.MessageStringOf(x)
}

func (*AlterOutboundRequest) ProtoMessage() {}

func (x *AlterOutboundRequest) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_app_proxyman_command_command_proto_msgTypes[12]
    // 如果启用了不安全操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空，则存储消息信息
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        // 返回消息状态
        return ms
    }
    // 返回消息类型信息
    return mi.MessageOf(x)
}

// Deprecated: Use AlterOutboundRequest.ProtoReflect.Descriptor instead.
func (*AlterOutboundRequest) Descriptor() ([]byte, []int) {
    // 返回原始描述信息的 GZIP 压缩版本
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{12}
}
// 获取AlterOutboundRequest结构体的Tag字段值
func (x *AlterOutboundRequest) GetTag() string {
    // 如果x不为空，则返回x的Tag字段值
    if x != nil {
        return x.Tag
    }
    // 如果x为空，则返回空字符串
    return ""
}

// 获取AlterOutboundRequest结构体的Operation字段值
func (x *AlterOutboundRequest) GetOperation() *serial.TypedMessage {
    // 如果x不为空，则返回x的Operation字段值
    if x != nil {
        return x.Operation
    }
    // 如果x为空，则返回空指针
    return nil
}

// 定义AlterOutboundResponse结构体
type AlterOutboundResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置AlterOutboundResponse结构体
func (x *AlterOutboundResponse) Reset() {
    *x = AlterOutboundResponse{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[13]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回AlterOutboundResponse结构体的字符串表示形式
func (x *AlterOutboundResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现ProtoMessage接口
func (*AlterOutboundResponse) ProtoMessage() {}

// 返回AlterOutboundResponse结构体的反射信息
func (x *AlterOutboundResponse) ProtoReflect() protoreflect.Message {
    mi := &file_app_proxyman_command_command_proto_msgTypes[13]
    // 如果启用了不安全操作并且x不为空，则存储消息信息并返回
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: 使用AlterOutboundResponse.ProtoReflect.Descriptor代替
func (*AlterOutboundResponse) Descriptor() ([]byte, []int) {
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{13}
}

// 定义Config结构体
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置Config结构体
func (x *Config) Reset() {
    *x = Config{}
    // 如果启用了不安全操作，则存储消息信息
    if protoimpl.UnsafeEnabled {
        mi := &file_app_proxyman_command_command_proto_msgTypes[14]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回Config结构体的字符串表示形式
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现ProtoMessage接口
func (*Config) ProtoMessage() {}

// 返回Config结构体的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    # 获取指向消息类型的指针
    mi := &file_app_proxyman_command_command_proto_msgTypes[14]
    # 如果启用了不安全操作并且 x 不为空
    if protoimpl.UnsafeEnabled && x != nil {
        # 获取 x 的消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 如果消息状态中的消息信息为空
        if ms.LoadMessageInfo() == nil {
            # 存储消息信息到消息状态中
            ms.StoreMessageInfo(mi)
        }
        # 返回消息状态
        return ms
    }
    # 返回消息类型的消息
    return mi.MessageOf(x)
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 使用 Config.ProtoReflect.Descriptor 替代
func (*Config) Descriptor() ([]byte, []int) {
    // 返回经过 GZIP 压缩的原始文件描述符
    return file_app_proxyman_command_command_proto_rawDescGZIP(), []int{14}
}

// 定义 File_app_proxyman_command_command_proto 变量
var File_app_proxyman_command_command_proto protoreflect.FileDescriptor

// 定义 file_app_proxyman_command_command_proto_rawDesc 变量
var file_app_proxyman_command_command_proto_rawDesc = []byte{
    // 以下为原始文件描述符的字节流
    # 创建一个十六进制数值列表
    0x28, 0x09, 0x52, 0x05, 0x65, 0x6d, 0x61, 0x69, 0x6c, 0x22, 0x4f, 0x0a, 0x11, 0x41, 0x64, 0x64,
    0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x3a,
    0x0a, 0x07, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32,
    0x20, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x49, 0x6e, 0x62,
    0x6f, 0x75, 0x6e, 0x64, 0x48, 0x61, 0x6e, 0x64, 0x6c, 0x65, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69,
    0x67, 0x52, 0x07, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x22, 0x14, 0x0a, 0x12, 0x41, 0x64,
    0x64, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65,
    # ...
    # 以十六进制表示的整数列表，可能是某种配置或参数的编码
    0x64, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74,
    0x12, 0x3d, 0x0a, 0x08, 0x6f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x18, 0x01, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x48, 0x61, 0x6e, 0x64, 0x6c, 0x65, 0x72, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x08, 0x6f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x22,
    0x15, 0x0a, 0x13, 0x41, 0x64, 0x64, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65,
    0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x29, 0x0a, 0x15, 0x52, 0x65, 0x6d, 0x6f, 0x76, 0x65,
    0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12,
    0x10, 0x0a, 0x03, 0x74, 0x61, 0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x74, 0x61,
    0x67, 0x22, 0x18, 0x0a, 0x16, 0x52, 0x65, 0x6d, 0x6f, 0x76, 0x65, 0x4f, 0x75, 0x74, 0x62, 0x6f,
    0x75, 0x6e, 0x64, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x6e, 0x0a, 0x14, 0x41,
    0x6c, 0x74, 0x65, 0x72, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75,
    0x65, 0x73, 0x74, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61, 0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09,
    0x52, 0x03, 0x74, 0x61, 0x67, 0x12, 0x44, 0x0a, 0x09, 0x6f, 0x70, 0x65, 0x72, 0x61, 0x74, 0x69,
    0x6f, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x73, 0x65, 0x72,
    0x69, 0x61, 0x6c, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x64, 0x4d, 0x65, 0x73, 0x73, 0x61, 0x67, 0x65,
    0x52, 0x09, 0x6f, 0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x22, 0x17, 0x0a, 0x15, 0x41,
    0x6c, 0x74, 0x65, 0x72, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x73, 0x70,
    0x6f, 0x6e, 0x73, 0x65, 0x22, 0x08, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x32, 0x90,
    # 创建一个十六进制数列表
    0x06, 0x0a, 0x0e, 0x48, 0x61, 0x6e, 0x64, 0x6c, 0x65, 0x72, 0x53, 0x65, 0x72, 0x76, 0x69, 0x63,
    0x65, 0x12, 0x77, 0x0a, 0x0a, 0x41, 0x64, 0x64, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12,
    0x32, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
    0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e,
    0x64, 0x2e, 0x41, 0x64, 0x64, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75,
    0x65, 0x73, 0x74, 0x1a, 0x33, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f,
    0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x41, 0x64, 0x64, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64,
    0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12, 0x80, 0x01, 0x0a, 0x0d, 0x52,
    0x65, 0x6d, 0x6f, 0x76, 0x65, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12, 0x35, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72,
    0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52,
    0x65, 0x6d, 0x6f, 0x76, 0x65, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75,
    0x65, 0x73, 0x74, 0x1a, 0x36, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f,
    0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x65, 0x6d, 0x6f, 0x76, 0x65, 0x49, 0x6e, 0x62, 0x6f,
    0x75, 0x6e, 0x64, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12, 0x7d, 0x0a,
    0x0c, 0x41, 0x6c, 0x74, 0x65, 0x72, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12, 0x34, 0x2e,
    0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70,
    0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e,
    # 以十六进制表示的整数列表，可能是某种配置或数据的编码
    0x41, 0x6c, 0x74, 0x65, 0x72, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75,
    0x65, 0x73, 0x74, 0x1a, 0x35, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f,
    0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x41, 0x6c, 0x74, 0x65, 0x72, 0x49, 0x6e, 0x62, 0x6f, 0x75,
    0x6e, 0x64, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12, 0x7a, 0x0a, 0x0b,
    0x41, 0x64, 0x64, 0x4f, 0x75, 74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12, 0x33, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f,
    0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x41, 0x64,
    0x64, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74,
    0x1a, 0x34, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70,
    0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61,
    0x6e, 0x64, 0x2e, 0x41, 0x64, 0x64, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65,
    0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12, 0x83, 0x01, 0x0a, 0x0e, 0x52, 0x65, 0x6d,
    0x6f, 0x76, 0x65, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x12, 0x36, 0x2e, 0x76, 0x32,
    0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f,
    0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x65,
    0x6d, 0x6f, 0x76, 0x65, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x71, 0x75,
    0x65, 0x73, 0x74, 0x1a, 0x37, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
    0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f,
    0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x65, 0x6d, 0x6f, 0x76, 0x65, 0x4f, 0x75, 0x74, 0x62,
    # 以十六进制表示的字节码序列
    0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x12, 0x80,
    0x01, 0x0a, 0x0d, 0x41, 0x6c, 0x74, 0x65, 0x72, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64,
    0x12, 0x35, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70,
    0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61,
    0x6e, 0x64, 0x2e, 0x41, 0x6c, 0x74, 0x65, 0x72, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64,
    0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x36, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
    0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61,
    0x6e, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x41, 0x6c, 0x74, 0x65, 0x72, 0x4f,
    0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22,
    0x00, 0x42, 0x6e, 0x0a, 0x23, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e,
    0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x23, 0x76, 0x32, 0x72, 0x61,
    0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x70,
    0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0xaa,
    0x02, 0x1f, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70,
    0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x6d, 0x61, 0x6e, 0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x61, 0x6e,
    0x64, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义变量 file_app_proxyman_command_command_proto_rawDescOnce，类型为 sync.Once，用于确保只执行一次
var (
    file_app_proxyman_command_command_proto_rawDescOnce sync.Once
    // 定义变量 file_app_proxyman_command_command_proto_rawDescData，类型为 file_app_proxyman_command_command_proto_rawDesc
    file_app_proxyman_command_command_proto_rawDescData = file_app_proxyman_command_command_proto_rawDesc
)

// 定义函数 file_app_proxyman_command_command_proto_rawDescGZIP，返回类型为 []byte
func file_app_proxyman_command_command_proto_rawDescGZIP() []byte {
    // 调用 sync.Once 的 Do 方法，确保只执行一次压缩操作
    file_app_proxyman_command_command_proto_rawDescOnce.Do(func() {
        // 使用 protoimpl.X.CompressGZIP 方法对 file_app_proxyman_command_command_proto_rawDescData 进行压缩
        file_app_proxyman_command_command_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_proxyman_command_command_proto_rawDescData)
    })
    // 返回压缩后的数据
    return file_app_proxyman_command_command_proto_rawDescData
}

// 定义变量 file_app_proxyman_command_command_proto_msgTypes，类型为 []protoimpl.MessageInfo，包含 15 个元素
var file_app_proxyman_command_command_proto_msgTypes = make([]protoimpl.MessageInfo, 15)
// 定义变量 file_app_proxyman_command_command_proto_goTypes，类型为 []interface{}，包含多种类型
var file_app_proxyman_command_command_proto_goTypes = []interface{}{
    (*AddUserOperation)(nil),           // 0: v2ray.core.app.proxyman.command.AddUserOperation
    (*RemoveUserOperation)(nil),        // 1: v2ray.core.app.proxyman.command.RemoveUserOperation
    (*AddInboundRequest)(nil),          // 2: v2ray.core.app.proxyman.command.AddInboundRequest
    (*AddInboundResponse)(nil),         // 3: v2ray.core.app.proxyman.command.AddInboundResponse
    (*RemoveInboundRequest)(nil),       // 4: v2ray.core.app.proxyman.command.RemoveInboundRequest
    (*RemoveInboundResponse)(nil),      // 5: v2ray.core.app.proxyman.command.RemoveInboundResponse
    (*AlterInboundRequest)(nil),        // 6: v2ray.core.app.proxyman.command.AlterInboundRequest
    (*AlterInboundResponse)(nil),       // 7: v2ray.core.app.proxyman.command.AlterInboundResponse
    (*AddOutboundRequest)(nil),         // 8: v2ray.core.app.proxyman.command.AddOutboundRequest
    (*AddOutboundResponse)(nil),        // 9: v2ray.core.app.proxyman.command.AddOutboundResponse
    (*RemoveOutboundRequest)(nil),      // 10: v2ray.core.app.proxyman.command.RemoveOutboundRequest
    (*RemoveOutboundResponse)(nil),     // 11: v2ray.core.app.proxyman.command.RemoveOutboundResponse
    (*AlterOutboundRequest)(nil),       // 12: v2ray.core.app.proxyman.command.AlterOutboundRequest
    (*AlterOutboundResponse)(nil),      // 13: 声明一个指向 AlterOutboundResponse 结构体的空指针
    (*Config)(nil),                     // 14: 声明一个指向 Config 结构体的空指针
    (*protocol.User)(nil),              // 15: 声明一个指向 User 结构体的空指针
    (*core.InboundHandlerConfig)(nil),  // 16: 声明一个指向 InboundHandlerConfig 结构体的空指针
    (*serial.TypedMessage)(nil),        // 17: 声明一个指向 TypedMessage 结构体的空指针
    (*core.OutboundHandlerConfig)(nil), // 18: 声明一个指向 OutboundHandlerConfig 结构体的空指针
// 定义一个包含 int32 类型元素的切片，用于存储依赖索引
var file_app_proxyman_command_command_proto_depIdxs = []int32{
    15, // 0: v2ray.core.app.proxyman.command.AddUserOperation.user:type_name -> v2ray.core.common.protocol.User
    16, // 1: v2ray.core.app.proxyman.command.AddInboundRequest.inbound:type_name -> v2ray.core.InboundHandlerConfig
    17, // 2: v2ray.core.app.proxyman.command.AlterInboundRequest.operation:type_name -> v2ray.core.common.serial.TypedMessage
    18, // 3: v2ray.core.app.proxyman.command.AddOutboundRequest.outbound:type_name -> v2ray.core.OutboundHandlerConfig
    17, // 4: v2ray.core.app.proxyman.command.AlterOutboundRequest.operation:type_name -> v2ray.core.common.serial.TypedMessage
    2,  // 5: v2ray.core.app.proxyman.command.HandlerService.AddInbound:input_type -> v2ray.core.app.proxyman.command.AddInboundRequest
    4,  // 6: v2ray.core.app.proxyman.command.HandlerService.RemoveInbound:input_type -> v2ray.core.app.proxyman.command.RemoveInboundRequest
    6,  // 7: v2ray.core.app.proxyman.command.HandlerService.AlterInbound:input_type -> v2ray.core.app.proxyman.command.AlterInboundRequest
    8,  // 8: v2ray.core.app.proxyman.command.HandlerService.AddOutbound:input_type -> v2ray.core.app.proxyman.command.AddOutboundRequest
    10, // 9: v2ray.core.app.proxyman.command.HandlerService.RemoveOutbound:input_type -> v2ray.core.app.proxyman.command.RemoveOutboundRequest
    12, // 10: v2ray.core.app.proxyman.command.HandlerService.AlterOutbound:input_type -> v2ray.core.app.proxyman.command.AlterOutboundRequest
    3,  // 11: v2ray.core.app.proxyman.command.HandlerService.AddInbound:output_type -> v2ray.core.app.proxyman.command.AddInboundResponse
    5,  // 12: v2ray.core.app.proxyman.command.HandlerService.RemoveInbound:output_type -> v2ray.core.app.proxyman.command.RemoveInboundResponse
    7,  // 13: v2ray.core.app.proxyman.command.HandlerService.AlterInbound:output_type -> v2ray.core.app.proxyman.command.AlterInboundResponse
}
    9,  // 14: v2ray.core.app.proxyman.command.HandlerService.AddOutbound:output_type -> v2ray.core.app.proxyman.command.AddOutboundResponse
    // 设置输出类型为 v2ray.core.app.proxyman.command.AddOutboundResponse，用于 v2ray.core.app.proxyman.command.HandlerService.AddOutbound 方法

    11, // 15: v2ray.core.app.proxyman.command.HandlerService.RemoveOutbound:output_type -> v2ray.core.app.proxyman.command.RemoveOutboundResponse
    // 设置输出类型为 v2ray.core.app.proxyman.command.RemoveOutboundResponse，用于 v2ray.core.app.proxyman.command.HandlerService.RemoveOutbound 方法

    13, // 16: v2ray.core.app.proxyman.command.HandlerService.AlterOutbound:output_type -> v2ray.core.app.proxyman.command.AlterOutboundResponse
    // 设置输出类型为 v2ray.core.app.proxyman.command.AlterOutboundResponse，用于 v2ray.core.app.proxyman.command.HandlerService.AlterOutbound 方法

    11, // [11:17] is the sub-list for method output_type
    // 设置方法输出类型的子列表起始位置

    5,  // [5:11] is the sub-list for method input_type
    // 设置方法输入类型的子列表起始位置

    5,  // [5:5] is the sub-list for extension type_name
    // 设置扩展类型名称的子列表起始位置

    5,  // [5:5] is the sub-list for extension extendee
    // 设置扩展扩展者的子列表起始位置

    0,  // [0:5] is the sub-list for field type_name
    // 设置字段类型名称的子列表起始位置
# 初始化函数，用于在包被导入时执行一些初始化操作
func init() { file_app_proxyman_command_command_proto_init() }

# 初始化命令协议文件
func file_app_proxyman_command_command_proto_init() {
    # 如果文件已经被初始化，则直接返回
    if File_app_proxyman_command_command_proto != nil {
        return
    }
    
    # 定义一个空结构体 x
    type x struct{}
    
    # 使用 protoimpl.TypeBuilder 构建类型
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            RawDescriptor: file_app_proxyman_command_command_proto_rawDesc,
            NumEnums:      0,
            NumMessages:   15,
            NumExtensions: 0,
            NumServices:   1,
        },
        GoTypes:           file_app_proxyman_command_command_proto_goTypes,
        DependencyIndexes: file_app_proxyman_command_command_proto_depIdxs,
        MessageInfos:      file_app_proxyman_command_command_proto_msgTypes,
    }.Build()
    
    # 将构建好的文件赋值给全局变量
    File_app_proxyman_command_command_proto = out.File
    file_app_proxyman_command_command_proto_rawDesc = nil
    file_app_proxyman_command_command_proto_goTypes = nil
    file_app_proxyman_command_command_proto_depIdxs = nil
}
```