# `trojan-go\api\service\api.pb.go`

```
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.27.1
//     protoc        v3.17.3
// 源文件: api.proto

package service

import (
    protoreflect "google.golang.org/protobuf/reflect/protoreflect"  // 导入 protoreflect 包
    protoimpl "google.golang.org/protobuf/runtime/protoimpl"  // 导入 protoimpl 包
    reflect "reflect"  // 导入 reflect 包
    sync "sync"  // 导入 sync 包
)

const (
    _ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)  // 确保生成的代码足够新
    _ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)  // 确保 runtime/protoimpl 足够新
)

type SetUsersRequest_Operation int32  // 定义 SetUsersRequest_Operation 类型为 int32

const (
    SetUsersRequest_Add    SetUsersRequest_Operation = 0  // 设置 SetUsersRequest_Add 的值为 0
    SetUsersRequest_Delete SetUsersRequest_Operation = 1  // 设置 SetUsersRequest_Delete 的值为 1
    SetUsersRequest_Modify SetUsersRequest_Operation = 2  // 设置 SetUsersRequest_Modify 的值为 2
)

// SetUsersRequest_Operation 的枚举值映射
var (
    SetUsersRequest_Operation_name = map[int32]string{  // 枚举值到名称的映射
        0: "Add",
        1: "Delete",
        2: "Modify",
    }
    SetUsersRequest_Operation_value = map[string]int32{  // 名称到枚举值的映射
        "Add":    0,
        "Delete": 1,
        "Modify": 2,
    }
)

func (x SetUsersRequest_Operation) Enum() *SetUsersRequest_Operation {  // 返回 SetUsersRequest_Operation 的枚举值
    p := new(SetUsersRequest_Operation)
    *p = x
    return p
}

func (x SetUsersRequest_Operation) String() string {  // 返回 SetUsersRequest_Operation 的字符串表示
    return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (SetUsersRequest_Operation) Descriptor() protoreflect.EnumDescriptor {  // 返回 SetUsersRequest_Operation 的枚举描述符
    return file_api_proto_enumTypes[0].Descriptor()
}

func (SetUsersRequest_Operation) Type() protoreflect.EnumType {  // 返回 SetUsersRequest_Operation 的枚举类型
    return &file_api_proto_enumTypes[0]
}

func (x SetUsersRequest_Operation) Number() protoreflect.EnumNumber {  // 返回 SetUsersRequest_Operation 的枚举值
    return protoreflect.EnumNumber(x)
}

// Deprecated: Use SetUsersRequest_Operation.Descriptor instead.
func (SetUsersRequest_Operation) EnumDescriptor() ([]byte, []int) {  // 返回 SetUsersRequest_Operation 的枚举描述符
    return file_api_proto_rawDescGZIP(), []int{10, 0}
}

type Traffic struct {  // Traffic 结构体
    # 定义状态变量
    state         protoimpl.MessageState
    # 定义消息大小缓存变量
    sizeCache     protoimpl.SizeCache
    # 定义未知字段变量
    unknownFields protoimpl.UnknownFields

    # 定义上传流量变量，使用 varint 编码，字段编号为 1，可选字段，JSON 名为 upload_traffic
    UploadTraffic   uint64 `protobuf:"varint,1,opt,name=upload_traffic,json=uploadTraffic,proto3" json:"upload_traffic,omitempty"`
    # 定义下载流量变量，使用 varint 编码，字段编号为 2，可选字段，JSON 名为 download_traffic
    DownloadTraffic uint64 `protobuf:"varint,2,opt,name=download_traffic,json=downloadTraffic,proto3" json:"download_traffic,omitempty"`
}

// 重置 Traffic 对象
func (x *Traffic) Reset() {
    // 将 Traffic 对象重置为空对象
    *x = Traffic{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_api_proto_msgTypes[0]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Traffic 对象的字符串表示
func (x *Traffic) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Traffic) ProtoMessage() {}

// 返回 Traffic 对象的反射信息
func (x *Traffic) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_api_proto_msgTypes[0]
    // 如果启用了不安全操作并且 Traffic 对象不为空
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

// 已弃用：使用 Traffic.ProtoReflect.Descriptor 替代
func (*Traffic) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{0}
}

// 获取上传流量
func (x *Traffic) GetUploadTraffic() uint64 {
    // 如果 Traffic 对象不为空，则返回上传流量
    if x != nil {
        return x.UploadTraffic
    }
    return 0
}

// 获取下载流量
func (x *Traffic) GetDownloadTraffic() uint64 {
    // 如果 Traffic 对象不为空，则返回下载流量
    if x != nil {
        return x.DownloadTraffic
    }
    return 0
}

// Speed 结构体
type Speed struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 上传速度
    UploadSpeed   uint64 `protobuf:"varint,1,opt,name=upload_speed,json=uploadSpeed,proto3" json:"upload_speed,omitempty"`
    // 下载速度
    DownloadSpeed uint64 `protobuf:"varint,2,opt,name=download_speed,json=downloadSpeed,proto3" json:"download_speed,omitempty"`
}

// 重置 Speed 对象
func (x *Speed) Reset() {
    // 将 Speed 对象重置为空对象
    *x = Speed{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_api_proto_msgTypes[1]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Speed 对象的字符串表示
func (x *Speed) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Speed) ProtoMessage() {}

// 返回 Speed 对象的反射信息
func (x *Speed) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_api_proto_msgTypes[1]
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
// Deprecated: Use Speed.ProtoReflect.Descriptor instead.
func (*Speed) Descriptor() ([]byte, []int) {
    // 返回文件_api_proto_rawDescGZIP的内容和索引为1的元素
    return file_api_proto_rawDescGZIP(), []int{1}
}

func (x *Speed) GetUploadSpeed() uint64 {
    // 如果x不为空，则返回x的上传速度，否则返回0
    if x != nil {
        return x.UploadSpeed
    }
    return 0
}

func (x *Speed) GetDownloadSpeed() uint64 {
    // 如果x不为空，则返回x的下载速度，否则返回0
    if x != nil {
        return x.DownloadSpeed
    }
    return 0
}

type User struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Password string `protobuf:"bytes,1,opt,name=password,proto3" json:"password,omitempty"`
    Hash     string `protobuf:"bytes,2,opt,name=hash,proto3" json:"hash,omitempty"`
}

func (x *User) Reset() {
    // 重置User对象
    *x = User{}
    if protoimpl.UnsafeEnabled {
        mi := &file_api_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

func (x *User) String() string {
    // 返回User对象的字符串表示形式
    return protoimpl.X.MessageStringOf(x)
}

func (*User) ProtoMessage() {}

func (x *User) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[2]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use User.ProtoReflect.Descriptor instead.
func (*User) Descriptor() ([]byte, []int) {
    // 返回文件_api_proto_rawDescGZIP的内容和索引为2的元素
    return file_api_proto_rawDescGZIP(), []int{2}
}

func (x *User) GetPassword() string {
    // 如果x不为空，则返回x的密码，否则返回空字符串
    if x != nil {
        return x.Password
    }
    return ""
}

func (x *User) GetHash() string {
    // 如果x不为空，则返回x的哈希值，否则返回空字符串
    if x != nil {
        return x.Hash
    }
    return ""
}

type UserStatus struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    User         *User    `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
    # TrafficTotal 表示总流量，使用 Traffic 类型，protobuf 标签指定了字段的类型和名称
    TrafficTotal *Traffic `protobuf:"bytes,2,opt,name=traffic_total,json=trafficTotal,proto3" json:"traffic_total,omitempty"`
    # SpeedCurrent 表示当前速度，使用 Speed 类型，protobuf 标签指定了字段的类型和名称
    SpeedCurrent *Speed   `protobuf:"bytes,3,opt,name=speed_current,json=speedCurrent,proto3" json:"speed_current,omitempty"`
    # SpeedLimit 表示速度限制，使用 Speed 类型，protobuf 标签指定了字段的类型和名称
    SpeedLimit   *Speed   `protobuf:"bytes,4,opt,name=speed_limit,json=speedLimit,proto3" json:"speed_limit,omitempty"`
    # IpCurrent 表示当前 IP 数量，使用 int32 类型，protobuf 标签指定了字段的类型和名称
    IpCurrent    int32    `protobuf:"varint,5,opt,name=ip_current,json=ipCurrent,proto3" json:"ip_current,omitempty"`
    # IpLimit 表示 IP 数量限制，使用 int32 类型，protobuf 标签指定了字段的类型和名称
    IpLimit      int32    `protobuf:"varint,6,opt,name=ip_limit,json=ipLimit,proto3" json:"ip_limit,omitempty"`
// 重置 UserStatus 对象
func (x *UserStatus) Reset() {
    // 将 UserStatus 对象重置为空对象
    *x = UserStatus{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_api_proto_msgTypes[3]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 UserStatus 对象的字符串表示
func (x *UserStatus) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*UserStatus) ProtoMessage() {}

// 返回 UserStatus 对象的反射信息
func (x *UserStatus) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_api_proto_msgTypes[3]
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

// 已弃用：使用 UserStatus.ProtoReflect.Descriptor 替代
func (*UserStatus) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{3}
}

// 返回 UserStatus 对象的 User 字段
func (x *UserStatus) GetUser() *User {
    if x != nil {
        return x.User
    }
    return nil
}

// 返回 UserStatus 对象的 TrafficTotal 字段
func (x *UserStatus) GetTrafficTotal() *Traffic {
    if x != nil {
        return x.TrafficTotal
    }
    return nil
}

// 返回 UserStatus 对象的 SpeedCurrent 字段
func (x *UserStatus) GetSpeedCurrent() *Speed {
    if x != nil {
        return x.SpeedCurrent
    }
    return nil
}

// 返回 UserStatus 对象的 SpeedLimit 字段
func (x *UserStatus) GetSpeedLimit() *Speed {
    if x != nil {
        return x.SpeedLimit
    }
    return nil
}

// 返回 UserStatus 对象的 IpCurrent 字段
func (x *UserStatus) GetIpCurrent() int32 {
    if x != nil {
        return x.IpCurrent
    }
    return 0
}

// 返回 UserStatus 对象的 IpLimit 字段
func (x *UserStatus) GetIpLimit() int32 {
    if x != nil {
        return x.IpLimit
    }
    return 0
}

// 定义 GetTrafficRequest 结构体
type GetTrafficRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    User *User `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
}

// 重置 GetTrafficRequest 对象
func (x *GetTrafficRequest) Reset() {
    // 将 GetTrafficRequest 对象重置为空对象
    *x = GetTrafficRequest{}
    # 如果启用了不安全的功能，则执行以下代码块
    if protoimpl.UnsafeEnabled:
        # 获取消息类型的信息
        mi := &file_api_proto_msgTypes[4]
        # 获取消息的状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        # 存储消息的信息
        ms.StoreMessageInfo(mi)
// 将 GetTrafficRequest 结构体转换为字符串
func (x *GetTrafficRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*GetTrafficRequest) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *GetTrafficRequest) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[4]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use GetTrafficRequest.ProtoReflect.Descriptor instead.
// 获取 GetTrafficRequest 的描述符
func (*GetTrafficRequest) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{4}
}

// 获取 GetTrafficRequest 中的 User 对象
func (x *GetTrafficRequest) GetUser() *User {
    if x != nil {
        return x.User
    }
    return nil
}

// 重置 GetTrafficResponse 结构体
func (x *GetTrafficResponse) Reset() {
    *x = GetTrafficResponse{}
    if protoimpl.UnsafeEnabled {
        mi := &file_api_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 GetTrafficResponse 结构体转换为字符串
func (x *GetTrafficResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*GetTrafficResponse) ProtoMessage() {}

// 实现 ProtoReflect 接口
func (x *GetTrafficResponse) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[5]
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
// Deprecated: Use GetTrafficResponse.ProtoReflect.Descriptor instead.
// 使用 GetTrafficResponse.ProtoReflect.Descriptor 替代，已废弃
func (*GetTrafficResponse) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{5}
}

// 获取成功标志
func (x *GetTrafficResponse) GetSuccess() bool {
    if x != nil {
        return x.Success
    }
    return false
}

// 获取信息
func (x *GetTrafficResponse) GetInfo() string {
    if x != nil {
        return x.Info
    }
    return ""
}

// 获取总流量
func (x *GetTrafficResponse) GetTrafficTotal() *Traffic {
    if x != nil {
        return x.TrafficTotal
    }
    return nil
}

// 获取当前速度
func (x *GetTrafficResponse) GetSpeedCurrent() *Speed {
    if x != nil {
        return x.SpeedCurrent
    }
    return nil
}

// ListUsersRequest 结构体
type ListUsersRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
}

// 重置 ListUsersRequest 结构体
func (x *ListUsersRequest) Reset() {
    *x = ListUsersRequest{}
    if protoimpl.UnsafeEnabled {
        mi := &file_api_proto_msgTypes[6]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 转换 ListUsersRequest 结构体为字符串
func (x *ListUsersRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*ListUsersRequest) ProtoMessage() {}

// 获取 ProtoReflect 接口
func (x *ListUsersRequest) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[6]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use ListUsersRequest.ProtoReflect.Descriptor instead.
// 使用 ListUsersRequest.ProtoReflect.Descriptor 替代，已废弃
func (*ListUsersRequest) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{6}
}

// ListUsersResponse 结构体
type ListUsersResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Status *UserStatus `protobuf:"bytes,1,opt,name=status,proto3" json:"status,omitempty"`
}
// 重置 ListUsersResponse 对象，将其重置为空对象
func (x *ListUsersResponse) Reset() {
    *x = ListUsersResponse{}
    // 如果启用了不安全模式，则获取消息类型信息并存储消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_api_proto_msgTypes[7]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 ListUsersResponse 对象的字符串表示形式
func (x *ListUsersResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*ListUsersResponse) ProtoMessage() {}

// 返回 ListUsersResponse 对象的反射信息
func (x *ListUsersResponse) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[7]
    // 如果启用了不安全模式并且对象不为空，则获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 ListUsersResponse.ProtoReflect.Descriptor 替代
func (*ListUsersResponse) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{7}
}

// 返回 GetUsersRequest 对象的 UserStatus 字段
func (x *ListUsersResponse) GetStatus() *UserStatus {
    if x != nil {
        return x.Status
    }
    return nil
}

// 定义 GetUsersRequest 结构体
type GetUsersRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    User *User `protobuf:"bytes,1,opt,name=user,proto3" json:"user,omitempty"`
}

// 重置 GetUsersRequest 对象，将其重置为空对象
func (x *GetUsersRequest) Reset() {
    *x = GetUsersRequest{}
    // 如果启用了不安全模式，则获取消息类型信息并存储消息状态
    if protoimpl.UnsafeEnabled {
        mi := &file_api_proto_msgTypes[8]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 GetUsersRequest 对象的字符串表示形式
func (x *GetUsersRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*GetUsersRequest) ProtoMessage() {}

// 返回 GetUsersRequest 对象的反射信息
func (x *GetUsersRequest) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[8]
    // 如果启用了不安全模式并且对象不为空，则获取消息状态并存储消息信息
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}
// Deprecated: Use GetUsersRequest.ProtoReflect.Descriptor instead.
// 返回 GetUsersRequest 的描述符
func (*GetUsersRequest) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{8}
}

// 返回 GetUsersRequest 中的 User 对象
func (x *GetUsersRequest) GetUser() *User {
    if x != nil {
        return x.User
    }
    return nil
}

// GetUsersResponse 结构体定义
type GetUsersResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Success bool        `protobuf:"varint,1,opt,name=success,proto3" json:"success,omitempty"`  // 成功标志
    Info    string      `protobuf:"bytes,2,opt,name=info,proto3" json:"info,omitempty"`  // 信息
    Status  *UserStatus `protobuf:"bytes,3,opt,name=status,proto3" json:"status,omitempty"`  // 用户状态
}

// 重置 GetUsersResponse 结构体
func (x *GetUsersResponse) Reset() {
    *x = GetUsersResponse{}
    if protoimpl.UnsafeEnabled {
        mi := &file_api_proto_msgTypes[9]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 GetUsersResponse 结构体的字符串表示
func (x *GetUsersResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// GetUsersResponse 结构体实现的 ProtoMessage 方法
func (*GetUsersResponse) ProtoMessage() {}

// 返回 GetUsersResponse 结构体的反射信息
func (x *GetUsersResponse) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[9]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use GetUsersResponse.ProtoReflect.Descriptor instead.
// 返回 GetUsersResponse 的描述符
func (*GetUsersResponse) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{9}
}

// 返回 GetUsersResponse 中的 Success 字段
func (x *GetUsersResponse) GetSuccess() bool {
    if x != nil {
        return x.Success
    }
    return false
}

// 返回 GetUsersResponse 中的 Info 字段
func (x *GetUsersResponse) GetInfo() string {
    if x != nil {
        return x.Info
    }
    return ""
}

// 返回 GetUsersResponse 中的 Status 字段
func (x *GetUsersResponse) GetStatus() *UserStatus {
    if x != nil {
        return x.Status
    }
    return nil
}

// SetUsersRequest 结构体定义
type SetUsersRequest struct {
    # 定义状态变量，用于存储消息状态
    state         protoimpl.MessageState
    # 定义大小缓存变量，用于存储消息大小的缓存
    sizeCache     protoimpl.SizeCache
    # 定义未知字段变量，用于存储未知字段
    unknownFields protoimpl.UnknownFields

    # 定义状态变量，用于存储用户状态，是一个指向UserStatus对象的指针
    Status    *UserStatus               `protobuf:"bytes,1,opt,name=status,proto3" json:"status,omitempty"`
    # 定义操作变量，用于存储用户请求的操作类型，是一个SetUsersRequest_Operation枚举类型的变量
    Operation SetUsersRequest_Operation `protobuf:"varint,2,opt,name=operation,proto3,enum=trojan.api.SetUsersRequest_Operation" json:"operation,omitempty"`
}
// 重置 SetUsersRequest 对象
func (x *SetUsersRequest) Reset() {
    // 将 SetUsersRequest 对象重置为空对象
    *x = SetUsersRequest{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_api_proto_msgTypes[10]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SetUsersRequest 对象的字符串表示
func (x *SetUsersRequest) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SetUsersRequest) ProtoMessage() {}

// 返回 SetUsersRequest 对象的反射信息
func (x *SetUsersRequest) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[10]
    // 如果启用了不安全操作并且对象不为空
    if protoimpl.UnsafeEnabled && x != nil {
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 如果消息信息为空
        if ms.LoadMessageInfo() == nil {
            // 存储消息信息
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// 弃用：使用 SetUsersRequest.ProtoReflect.Descriptor 替代
func (*SetUsersRequest) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{10}
}

// 获取 SetUsersRequest 对象的状态
func (x *SetUsersRequest) GetStatus() *UserStatus {
    if x != nil {
        return x.Status
    }
    return nil
}

// 获取 SetUsersRequest 对象的操作
func (x *SetUsersRequest) GetOperation() SetUsersRequest_Operation {
    if x != nil {
        return x.Operation
    }
    return SetUsersRequest_Add
}

// SetUsersResponse 结构体
type SetUsersResponse struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Success bool   `protobuf:"varint,1,opt,name=success,proto3" json:"success,omitempty"`
    Info    string `protobuf:"bytes,2,opt,name=info,proto3" json:"info,omitempty"`
}

// 重置 SetUsersResponse 对象
func (x *SetUsersResponse) Reset() {
    *x = SetUsersResponse{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_api_proto_msgTypes[11]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 返回 SetUsersResponse 对象的字符串表示
func (x *SetUsersResponse) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*SetUsersResponse) ProtoMessage() {}

// 返回 SetUsersResponse 对象的反射信息
func (x *SetUsersResponse) ProtoReflect() protoreflect.Message {
    mi := &file_api_proto_msgTypes[11]
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
// Deprecated: Use SetUsersResponse.ProtoReflect.Descriptor instead.
// 使用 SetUsersResponse.ProtoReflect.Descriptor 替代
func (*SetUsersResponse) Descriptor() ([]byte, []int) {
    return file_api_proto_rawDescGZIP(), []int{11}
}

// 获取 SetUsersResponse 结构体中的 Success 字段值
func (x *SetUsersResponse) GetSuccess() bool {
    if x != nil {
        return x.Success
    }
    return false
}

// 获取 SetUsersResponse 结构体中的 Info 字段值
func (x *SetUsersResponse) GetInfo() string {
    if x != nil {
        return x.Info
    }
    return ""
}

// 定义全局变量 File_api_proto
var File_api_proto protoreflect.FileDescriptor

// 定义全局变量 file_api_proto_rawDesc
var file_api_proto_rawDesc = []byte{
    // 以下为一大段字节流数据，用于描述文件结构
    // ...
}
    # 创建一个十六进制数值列表
    0x61, 0x73, 0x68, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x68, 0x61, 0x73, 0x68, 0x22,
    0x92, 0x02, 0x0a, 0x0a, 0x55, 0x73, 0x65, 0x72, 0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x12, 0x24,
    0x0a, 0x04, 0x75, 0x73, 0x65, 0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x10, 0x2e, 0x74,
    0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04,
    0x75, 0x73, 0x65, 0x72, 0x12, 0x38, 0x0a, 0x0d, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x5f,
    0x74, 0x6f, 0x74, 0x61, 0x6c, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x13, 0x2e, 0x74, 0x72,
    0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63,
    0x52, 0x0c, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x54, 0x6f, 0x74, 0x61, 0x6c, 0x12, 0x36,
    0x0a, 0x0d, 0x73, 0x70, 0x65, 0x65, 0x64, 0x5f, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x18,
    0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x11, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61,
    0x70, 0x69, 0x2e, 0x53, 0x70, 0x65, 0x65, 0x64, 0x52, 0x0c, 0x73, 0x70, 0x65, 0x65, 0x64, 0x43,
    0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x12, 0x32, 0x0a, 0x0b, 0x73, 0x70, 0x65, 0x65, 0x64, 0x5f,
    0x6c, 0x69, 0x6d, 0x69, 0x74, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x11, 0x2e, 0x74, 0x72,
    0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x53, 0x70, 0x65, 0x65, 0x64, 0x52, 0x0a,
    0x73, 0x70, 0x65, 0x65, 0x64, 0x4c, 0x69, 0x6d, 0x69, 0x74, 0x12, 0x1d, 0x0a, 0x0a, 0x69, 0x70,
    0x5f, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28, 0x05, 0x52, 0x09,
    0x69, 0x70, 0x43, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x12, 0x19, 0x0a, 0x08, 0x69, 0x70, 0x5f,
    0x6c, 0x69, 0x6d, 0x69, 0x74, 0x18, 0x06, 0x20, 0x01, 0x28, 0x05, 0x52, 0x07, 0x69, 0x70, 0x4c,
    0x69, 0x6d, 0x69, 0x74, 0x22, 0x39, 0x0a, 0x11, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66,
    0x69, 0x63, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x24, 0x0a, 0x04, 0x75, 0x73, 0x65,
    # 创建一个十六进制数值列表
    0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x10, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e,
    0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65, 0x72, 0x22,
    0xb4, 0x01, 0x0a, 0x12, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x52, 0x65,
    0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x18, 0x0a, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73,
    0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73,
    0x12, 0x12, 0a, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04,
    0x69, 0x6e, 0x66, 0x6f, 0x12, 0x38, 0x0a, 0x0d, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x5f,
    0x74, 0x6f, 0x74, 0x61, 0x6c, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x13, 0x2e, 0x74, 0x72,
    0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63,
    0x52, 0x0c, 0x74, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x54, 0x6f, 0x74, 0x61, 0x6c, 0x12, 0x36,
    0x0a, 0x0d, 0x73, 0x70, 0x65, 0x65, 0x64, 0x5f, 0x63, 0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x18,
    0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x11, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61,
    0x70, 0x69, 0x2e, 0x53, 0x70, 0x65, 0x65, 0x64, 0x52, 0x0c, 0x73, 0x70, 0x65, 0x65, 0x64, 0x43,
    0x75, 0x72, 0x72, 0x65, 0x6e, 0x74, 0x22, 0x12, 0x0a, 0x10, 0x4c, 0x69, 0x73, 0x74, 0x55, 0x73,
    0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x22, 0x43, 0x0a, 0x11, 0x4c, 0x69,
    0x73, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12,
    0x2e, 0x0a, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32,
    0x16, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65,
    0x72, 0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x22,
    0x37, 0x0a, 0x0f, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65,
    # 创建一个十六进制数值列表
    0x73, 0x74, 0x12, 0x24, 0x0a, 0x04, 0x75, 0x73, 0x65, 0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x10, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73,
    0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65, 0x72, 0x22, 0x70, 0x0a, 0x10, 0x47, 0x65, 0x74, 0x55,
    0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x18, 0x0a, 0x07,
    0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x73,
    0x75, 0x63, 0x63, 0x65, 0x73, 0x73, 0x12, 0x12, 0x0a, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x18, 0x02,
    0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x12, 0x2e, 0x0a, 0x06, 0x73, 0x74,
    0x61, 0x74, 0x75, 0x73, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x16, 0x2e, 0x74, 0x72, 0x6f,
    0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x53, 0x74, 0x61, 0x74,
    0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x22, 0xb4, 0x01, 0x0a, 0x0f, 0x53,
    0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x2e,
    0x0a, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x16,
    0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x55, 0x73, 0x65, 0x72,
    0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x12, 0x43,
    0x0a, 0x09, 0x6f, 0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28,
    0x0e, 0x32, 0x25, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x53,
    0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x2e, 0x4f,
    0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x52, 0x09, 0x6f, 0x70, 0x65, 0x72, 0x61, 0x74,
    0x69, 0x6f, 0x6e, 0x22, 0x2c, 0x0a, 0x09, 0x4f, 0x70, 0x65, 0x72, 0x61, 0x74, 0x69, 0x6f, 0x6e,
    0x12, 0x07, 0x0a, 0x03, 0x41, 0x64, 0x64, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x44, 0x65, 0x6c,
    # 创建一个十六进制数值列表
    0x65, 0x74, 0x65, 0x10, 0x01, 0x12, 0x0a, 0x0a, 0x06, 0x4d, 0x6f, 0x64, 0x69, 0x66, 0x79, 0x10,
    0x02, 0x22, 0x40, 0x0a, 0x10, 0x53, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x73,
    0x70, 0x6f, 0x6e, 0x73, 0x65, 0x12, 0x18, 0x0a, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x08, 0x52, 0x07, 0x73, 0x75, 0x63, 0x63, 0x65, 0x73, 0x73, 0x12,
    0x12, 0x0a, 0x04, 0x69, 0x6e, 0x66, 0x6f, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x69,
    0x6e, 0x66, 0x6f, 0x32, 0x64, 0x0a, 0x13, 0x54, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x43, 0x6c, 0x69,
    0x65, 0x6e, 0x74, 0x53, 0x65, 0x72, 0x76, 0x69, 0x63, 0x65, 0x12, 0x4d, 0x0a, 0x0a, 0x47, 0x65,
    0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x12, 0x1d, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61,
    0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63,
    0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x1e, 0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e,
    0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x54, 0x72, 0x61, 0x66, 0x66, 0x69, 0x63, 0x52,
    0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x32, 0xfd, 0x01, 0x0a, 0x13, 0x54, 0x72,
    0x6f, 0x6a, 0x61, 0x6e, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x53, 0x65, 0x72, 0x76, 0x69, 0x63,
    0x65, 0x12, 0x4c, 0x0a, 0x09, 0x4c, 0x69, 0x73, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x12, 0x1c,
    0x2e, 0x74, 0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x4c, 0x69, 0x73, 0x74,
    0x55, 0x73, 0x65, 0x72, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x1d, 0x2e, 0x74,
    0x72, 0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x4c, 0x69, 0x73, 0x74, 0x55, 0x73,
    0x65, 0x72, 0x73, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x22, 0x00, 0x30, 0x01, 0x12,
    0x4b, 0x0a, 0x08, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72, 0x73, 0x12, 0x1b, 0x2e, 0x74, 0x72,
    0x6f, 0x6a, 0x61, 0x6e, 0x2e, 0x61, 0x70, 0x69, 0x2e, 0x47, 0x65, 0x74, 0x55, 0x73, 0x65, 0x72,
    # 这是一系列十六进制数，可能是某种编码或加密的数据
    # 无法确定具体含义和作用
// 定义一个全局变量，用于确保 file_api_proto_rawDescData 只被初始化一次
var (
    file_api_proto_rawDescOnce sync.Once
    file_api_proto_rawDescData = file_api_proto_rawDesc
)

// 定义一个函数，用于将 file_api_proto_rawDescData 压缩为 GZIP 格式的字节数组
func file_api_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保只有一个 goroutine 执行压缩操作
    file_api_proto_rawDescOnce.Do(func() {
        file_api_proto_rawDescData = protoimpl.X.CompressGZIP(file_api_proto_rawDescData)
    })
    return file_api_proto_rawDescData
}

// 定义一些全局变量，包括枚举类型、消息类型和 Go 类型
var file_api_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_api_proto_msgTypes = make([]protoimpl.MessageInfo, 12)
var file_api_proto_goTypes = []interface{}{
    (SetUsersRequest_Operation)(0), // 0: trojan.api.SetUsersRequest.Operation
    (*Traffic)(nil),                // 1: trojan.api.Traffic
    (*Speed)(nil),                  // 2: trojan.api.Speed
    (*User)(nil),                   // 3: trojan.api.User
    (*UserStatus)(nil),             // 4: trojan.api.UserStatus
    (*GetTrafficRequest)(nil),      // 5: trojan.api.GetTrafficRequest
    (*GetTrafficResponse)(nil),     // 6: trojan.api.GetTrafficResponse
    (*ListUsersRequest)(nil),       // 7: trojan.api.ListUsersRequest
    (*ListUsersResponse)(nil),      // 8: trojan.api.ListUsersResponse
    (*GetUsersRequest)(nil),        // 9: trojan.api.GetUsersRequest
    (*GetUsersResponse)(nil),       // 10: trojan.api.GetUsersResponse
    (*SetUsersRequest)(nil),        // 11: trojan.api.SetUsersRequest
    (*SetUsersResponse)(nil),       // 12: trojan.api.SetUsersResponse
}

// 定义一个全局变量，用于存储依赖索引
var file_api_proto_depIdxs = []int32{
    3,  // 0: trojan.api.UserStatus.user:type_name -> trojan.api.User
    1,  // 1: trojan.api.UserStatus.traffic_total:type_name -> trojan.api.Traffic
    2,  // 2: trojan.api.UserStatus.speed_current:type_name -> trojan.api.Speed
    2,  // 3: trojan.api.UserStatus.speed_limit:type_name -> trojan.api.Speed
    3,  // 4: trojan.api.GetTrafficRequest.user:type_name -> trojan.api.User
    1,  // 5: trojan.api.GetTrafficResponse.traffic_total:type_name -> trojan.api.Traffic
    2,  // 6: trojan.api.GetTrafficResponse.speed_current:type_name -> trojan.api.Speed
}
    4,  // 7: trojan.api.ListUsersResponse.status:type_name -> trojan.api.UserStatus
    3,  // 8: trojan.api.GetUsersRequest.user:type_name -> trojan.api.User
    4,  // 9: trojan.api.GetUsersResponse.status:type_name -> trojan.api.UserStatus
    4,  // 10: trojan.api.SetUsersRequest.status:type_name -> trojan.api.UserStatus
    0,  // 11: trojan.api.SetUsersRequest.operation:type_name -> trojan.api.SetUsersRequest.Operation
    5,  // 12: trojan.api.TrojanClientService.GetTraffic:input_type -> trojan.api.GetTrafficRequest
    7,  // 13: trojan.api.TrojanServerService.ListUsers:input_type -> trojan.api.ListUsersRequest
    9,  // 14: trojan.api.TrojanServerService.GetUsers:input_type -> trojan.api.GetUsersRequest
    11, // 15: trojan.api.TrojanServerService.SetUsers:input_type -> trojan.api.SetUsersRequest
    6,  // 16: trojan.api.TrojanClientService.GetTraffic:output_type -> trojan.api.GetTrafficResponse
    8,  // 17: trojan.api.TrojanServerService.ListUsers:output_type -> trojan.api.ListUsersResponse
    10, // 18: trojan.api.TrojanServerService.GetUsers:output_type -> trojan.api.GetUsersResponse
    12, // 19: trojan.api.TrojanServerService.SetUsers:output_type -> trojan.api.SetUsersResponse
    16, // [16:20] is the sub-list for method output_type
    12, // [12:16] is the sub-list for method input_type
    12, // [12:12] is the sub-list for extension type_name
    12, // [12:12] is the sub-list for extension extendee
    0,  // [0:12] is the sub-list for field type_name
func init() { file_api_proto_init() }
func file_api_proto_init() {
    // 检查是否已经初始化了 File_api_proto，如果已经初始化则直接返回
    if File_api_proto != nil {
        return
    }
    // 定义一个空结构体 x
    type x struct{}
    // 使用 protoimpl.TypeBuilder 构建 File_api_proto 对象
    out := protoimpl.TypeBuilder{
        File: protoimpl.DescBuilder{
            // 设置 GoPackagePath 为 x 结构体的包路径
            GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
            // 设置 RawDescriptor 为 file_api_proto_rawDesc
            RawDescriptor: file_api_proto_rawDesc,
            // 设置 NumEnums 为 1
            NumEnums:      1,
            // 设置 NumMessages 为 12
            NumMessages:   12,
            // 设置 NumExtensions 为 0
            NumExtensions: 0,
            // 设置 NumServices 为 2
            NumServices:   2,
        },
        // 设置 GoTypes 为 file_api_proto_goTypes
        GoTypes:           file_api_proto_goTypes,
        // 设置 DependencyIndexes 为 file_api_proto_depIdxs
        DependencyIndexes: file_api_proto_depIdxs,
        // 设置 EnumInfos 为 file_api_proto_enumTypes
        EnumInfos:         file_api_proto_enumTypes,
        // 设置 MessageInfos 为 file_api_proto_msgTypes
        MessageInfos:      file_api_proto_msgTypes,
    }.Build()
    // 将构建好的 File_api_proto 赋值给 File_api_proto
    File_api_proto = out.File
    // 将 file_api_proto_rawDesc 置为 nil
    file_api_proto_rawDesc = nil
    // 将 file_api_proto_goTypes 置为 nil
    file_api_proto_goTypes = nil
    // 将 file_api_proto_depIdxs 置为 nil
    file_api_proto_depIdxs = nil
}
```