# `v2ray-core\transport\internet\headers\http\config.pb.go`

```go
// 由 protoc-gen-go 生成的代码。请勿编辑。
// 版本信息:
//     protoc-gen-go v1.25.0
//     protoc        v3.13.0
// 源文件: transport/internet/headers/http/config.proto

package http

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

type Header struct {
    state         protoimpl.MessageState  // 消息状态
    sizeCache     protoimpl.SizeCache  // 大小缓存
    unknownFields protoimpl.UnknownFields  // 未知字段

    // "Accept", "Cookie", 等
    Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`  // 名称
    // 每个条目必须是一个有效的整体。如果存在多个条目，则将选择随机条目。
    Value []string `protobuf:"bytes,2,rep,name=value,proto3" json:"value,omitempty"`  // 值
}

func (x *Header) Reset() {
    *x = Header{}  // 重置 Header 结构体
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[0]  // 获取消息类型
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))  // 获取消息状态
        ms.StoreMessageInfo(mi)  // 存储消息信息
    }
}

func (x *Header) String() string {
    return protoimpl.X.MessageStringOf(x)  // 返回消息的字符串表示形式
}

func (*Header) ProtoMessage() {}  // 实现 ProtoMessage 接口

func (x *Header) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[0]  // 获取消息类型
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
// Deprecated: Use Header.ProtoReflect.Descriptor instead.
// 使用 Header.ProtoReflect.Descriptor 替代此方法
func (*Header) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{0}
}

// 获取 Header 结构体中的 Name 字段值
func (x *Header) GetName() string {
    if x != nil {
        return x.Name
    }
    return ""
}

// 获取 Header 结构体中的 Value 字段值
func (x *Header) GetValue() []string {
    if x != nil {
        return x.Value
    }
    return nil
}

// HTTP version. Default value "1.1".
// HTTP 版本结构体，包含 Value 字段，默认值为 "1.1"
type Version struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value string `protobuf:"bytes,1,opt,name=value,proto3" json:"value,omitempty"`
}

// 重置 Version 结构体
func (x *Version) Reset() {
    *x = Version{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[1]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Version 结构体的字符串表示
func (x *Version) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*Version) ProtoMessage() {}

// 返回 Version 结构体的反射信息
func (x *Version) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[1]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Version.ProtoReflect.Descriptor instead.
// 使用 Version.ProtoReflect.Descriptor 替代此方法
func (*Version) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{1}
}

// 获取 Version 结构体中的 Value 字段值
func (x *Version) GetValue() string {
    if x != nil {
        return x.Value
    }
    return ""
}

// HTTP method. Default value "GET".
// HTTP 方法结构体，包含 Value 字段，默认值为 "GET"
type Method struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Value string `protobuf:"bytes,1,opt,name=value,proto3" json:"value,omitempty"`
}
// 重置 Method 对象为默认值
func (x *Method) Reset() {
    *x = Method{}
    // 如果启用了不安全操作，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[2]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 Method 对象的字符串表示
func (x *Method) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口的方法
func (*Method) ProtoMessage() {}

// 返回 Method 对象的反射信息
func (x *Method) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[2]
    // 如果启用了不安全操作并且 Method 对象不为空，则存储消息类型信息到消息状态中
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Method.ProtoReflect.Descriptor instead.
// 返回 Method 对象的描述符
func (*Method) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{2}
}

// 返回 Method 对象的值
func (x *Method) GetValue() string {
    if x != nil {
        return x.Value
    }
    return ""
}

// RequestConfig 结构体定义
type RequestConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 完整的 HTTP 版本，如 "1.1"
    Version *Version `protobuf:"bytes,1,opt,name=version,proto3" json:"version,omitempty"`
    // GET、POST、CONNECT 等
    Method *Method `protobuf:"bytes,2,opt,name=method,proto3" json:"method,omitempty"`
    // 像 "/login.php" 这样的 URI
    Uri    []string  `protobuf:"bytes,3,rep,name=uri,proto3" json:"uri,omitempty"`
    Header []*Header `protobuf:"bytes,4,rep,name=header,proto3" json:"header,omitempty"`
}

// 重置 RequestConfig 对象为默认值
func (x *RequestConfig) Reset() {
    *x = RequestConfig{}
    // 如果启用了不安全操作，则将消息类型信息存储到消息状态中
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[3]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 返回 RequestConfig 对象的字符串表示
func (x *RequestConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}
// 定义 RequestConfig 结构体的 ProtoMessage 方法
func (*RequestConfig) ProtoMessage() {}

// 定义 RequestConfig 结构体的 ProtoReflect 方法
func (x *RequestConfig) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[3]
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

// Deprecated: Use RequestConfig.ProtoReflect.Descriptor instead.
// 定义 RequestConfig 结构体的 Descriptor 方法
func (*RequestConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{3}
}

// 获取 RequestConfig 结构体的 Version 字段
func (x *RequestConfig) GetVersion() *Version {
    if x != nil {
        return x.Version
    }
    return nil
}

// 获取 RequestConfig 结构体的 Method 字段
func (x *RequestConfig) GetMethod() *Method {
    if x != nil {
        return x.Method
    }
    return nil
}

// 获取 RequestConfig 结构体的 Uri 字段
func (x *RequestConfig) GetUri() []string {
    if x != nil {
        return x.Uri
    }
    return nil
}

// 获取 RequestConfig 结构体的 Header 字段
func (x *RequestConfig) GetHeader() []*Header {
    if x != nil {
        return x.Header
    }
    return nil
}

// 定义 Status 结构体
type Status struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // Status code. Default "200".
    Code string `protobuf:"bytes,1,opt,name=code,proto3" json:"code,omitempty"`
    // Statue reason. Default "OK".
    Reason string `protobuf:"bytes,2,opt,name=reason,proto3" json:"reason,omitempty"`
}

// 重置 Status 结构体
func (x *Status) Reset() {
    *x = Status{}
    // 如果启用了不安全操作
    if protoimpl.UnsafeEnabled {
        // 获取消息类型信息
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[4]
        // 获取消息状态
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        // 存储消息信息
        ms.StoreMessageInfo(mi)
    }
}

// 获取 Status 结构体的字符串表示
func (x *Status) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 定义 Status 结构体的 ProtoMessage 方法
func (*Status) ProtoMessage() {}

// 定义 Status 结构体的 ProtoReflect 方法
func (x *Status) ProtoReflect() protoreflect.Message {
    // 获取消息类型信息
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[4]
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
// Deprecated: Use Status.ProtoReflect.Descriptor instead.
// 使用 Status.ProtoReflect.Descriptor 替代
func (*Status) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{4}
}

// 获取状态码
func (x *Status) GetCode() string {
    if x != nil {
        return x.Code
    }
    return ""
}

// 获取状态原因
func (x *Status) GetReason() string {
    if x != nil {
        return x.Reason
    }
    return ""
}

// 响应配置结构体
type ResponseConfig struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Version *Version  `protobuf:"bytes,1,opt,name=version,proto3" json:"version,omitempty"`
    Status  *Status   `protobuf:"bytes,2,opt,name=status,proto3" json:"status,omitempty"`
    Header  []*Header `protobuf:"bytes,3,rep,name=header,proto3" json:"header,omitempty"`
}

// 重置响应配置结构体
func (x *ResponseConfig) Reset() {
    *x = ResponseConfig{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[5]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 转换响应配置结构体为字符串
func (x *ResponseConfig) String() string {
    return protoimpl.X.MessageStringOf(x)
}

func (*ResponseConfig) ProtoMessage() {}

// 获取响应配置结构体的反射信息
func (x *ResponseConfig) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[5]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use ResponseConfig.ProtoReflect.Descriptor instead.
// 使用 ResponseConfig.ProtoReflect.Descriptor 替代
func (*ResponseConfig) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{5}
}

// 获取响应配置结体的版本信息
func (x *ResponseConfig) GetVersion() *Version {
    if x != nil {
        return x.Version
    }
    return nil
}

// 获取响应配置结体的状态信息
func (x *ResponseConfig) GetStatus() *Status {
    # 如果 x 不为空
    if x != nil:
        # 返回 x 的状态
        return x.Status
    # 否则返回空
    return nil
// 获取响应配置的头部信息
func (x *ResponseConfig) GetHeader() []*Header {
    // 如果 x 不为空，则返回其头部信息
    if x != nil {
        return x.Header
    }
    // 如果 x 为空，则返回空值
    return nil
}

// Config 结构体定义
type Config struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    // 用于认证请求的设置。如果未设置，客户端将不发送认证头部，服务器端将绕过认证。
    Request *RequestConfig `protobuf:"bytes,1,opt,name=request,proto3" json:"request,omitempty"`
    // 用于认证响应的设置。如果未设置，客户端将绕过认证，服务器端将不发送认证头部。
    Response *ResponseConfig `protobuf:"bytes,2,opt,name=response,proto3" json:"response,omitempty"`
}

// 重置 Config 结构体
func (x *Config) Reset() {
    *x = Config{}
    if protoimpl.UnsafeEnabled {
        mi := &file_transport_internet_headers_http_config_proto_msgTypes[6]
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        ms.StoreMessageInfo(mi)
    }
}

// 将 Config 结构体转换为字符串
func (x *Config) String() string {
    return protoimpl.X.MessageStringOf(x)
}

// 实现 ProtoMessage 接口
func (*Config) ProtoMessage() {}

// 获取 Config 结构体的反射信息
func (x *Config) ProtoReflect() protoreflect.Message {
    mi := &file_transport_internet_headers_http_config_proto_msgTypes[6]
    if protoimpl.UnsafeEnabled && x != nil {
        ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
        if ms.LoadMessageInfo() == nil {
            ms.StoreMessageInfo(mi)
        }
        return ms
    }
    return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
// 获取 Config 结构体的描述符
func (*Config) Descriptor() ([]byte, []int) {
    return file_transport_internet_headers_http_config_proto_rawDescGZIP(), []int{6}
}

// 获取请求配置
func (x *Config) GetRequest() *RequestConfig {
    // 如果 x 不为空，则返回其请求配置
    if x != nil {
        return x.Request
    }
    // 如果 x 为空，则返回空值
    return nil
}

// 获取响应配置
func (x *Config) GetResponse() *ResponseConfig {
    // 如果 x 不为空，则返回其响应配置
    if x != nil {
        return x.Response
    }
    // 如果 x 为空，则返回空值
    return nil
}
// 声明一个名为 File_transport_internet_headers_http_config_proto 的变量，类型为 protoreflect.FileDescriptor
var File_transport_internet_headers_http_config_proto protoreflect.FileDescriptor

// 声明一个名为 file_transport_internet_headers_http_config_proto_rawDesc 的变量，类型为 []byte
var file_transport_internet_headers_http_config_proto_rawDesc = []byte{
    // 以下为字节流内容，用于描述文件结构和数据
    // ...
}
    # 创建一个十六进制数值列表，可能是某种配置或参数的表示方式
    0x56, 0x65, 0x72, 0x73, 0x69, 0x6f, 0x6e, 0x52, 0x07, 0x76, 0x65, 0x72, 0x73, 0x69, 0x6f, 0x6e,
    0x12, 0x4a, 0x0a, 0x06, 0x6d, 0x65, 0x74, 0x68, 0x6f, 0x64, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x32, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72,
    0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
    0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x2e, 0x4d, 0x65,
    0x74, 0x68, 0x6f, 0x64, 0x52, 0x06, 0x6d, 0x65, 0x74, 0x68, 0x6f, 0x64,
    # 可能是某种配置或参数的表示方式
    0x12, 0x10, 0x0a, 0x03, 0x75, 0x72, 0x69, 0x18, 0x03, 0x20, 0x03, 0x28, 0x09, 0x52, 0x03, 0x75, 0x72, 0x69,
    0x12, 0x4a, 0x0a, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x32,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65,
    0x72, 0x73, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x2e, 0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x52, 0x06, 0x68, 0x65,
    0x61, 0x64, 0x65, 0x72,
    # 可能是某种配置或参数的表示方式
    0x22, 0x34, 0x0a, 0x06, 0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x12, 0x12, 0x0a, 0x04, 0x63, 0x6f, 0x64, 0x65,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x63, 0x6f, 0x64, 0x65, 0x12, 0x16, 0x0a, 0x06, 0x72, 0x65,
    0x61, 0x73, 0x6f, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x06, 0x72, 0x65, 0x61, 0x73, 0x6f, 0x6e,
    # 可能是某种配置或参数的表示方式
    0x22, 0xf7, 0x01, 0x0a, 0x0e, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x43, 0x6f, 0x6e, 0x66, 0x69,
    0x67, 0x12, 0x4d, 0x0a, 0x07, 0x76, 0x65, 0x72, 0x73, 0x69, 0x6f, 0x6e, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b,
    0x32, 0x33, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e,
    0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61,
    0x64, 0x65, 0x72, 0x73, 0x2e, 0x68, 0x74,
    # 创建一个十六进制数值列表，表示一个特定的数据结构
    0x70, 0x2e, 0x56, 0x65, 0x72, 0x73, 0x69, 0x6f, 0x6e, 0x52, 0x07, 0x76, 0x65, 0x72, 0x73, 0x69,
    0x6f, 0x6e, 0x12, 0x4a, 0x0a, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x18, 0x02, 0x20, 0x01,
    0x28, 0x0b, 0x32, 0x32, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
    0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e,
    0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x2e,
    0x53, 0x74, 0x61, 0x74, 0x75, 0x73, 0x52, 0x06, 0x73, 0x74, 0x61, 0x74, 0x75, 0x73, 0x12, 0x4a,
    0x0a, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x32,
    0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e,
    0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68,
    0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x2e, 0x48, 0x65, 0x61, 0x64,
    0x65, 0x72, 0x52, 0x06, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x22, 0xb5, 0x01, 0x0a, 0x06, 0x43,
    0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x53, 0x0a, 0x07, 0x72, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74,
    0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x39, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e,
    0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x68,
    0x74, 0x74, 0x70, 0x2e, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69,
    0x67, 0x52, 0x07, 0x72, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x56, 0x0a, 0x08, 0x72, 0x65,
    0x73, 0x70, 0x6f, 0x6e, 0x73, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x3a, 0x2e, 0x76,
    0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70,
    0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61,
    # 以十六进制表示的字节码序列
    0x64, 0x65, 0x72, 0x73, 0x2e, 0x68, 0x74, 0x74, 0x70, 0x2e, 0x52, 0x65, 0x73, 0x70, 0x6f, 0x6e,
    0x73, 0x65, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x08, 0x72, 0x65, 0x73, 0x70, 0x6f, 0x6e,
    0x73, 0x65, 0x42, 0x8f, 0x01, 0x0a, 0x2e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
    0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e,
    0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73,
    0x2e, 0x68, 0x74, 0x74, 0x70, 0x50, 0x01, 0x5a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
    0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72,
    0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65,
    0x72, 0x73, 0x2f, 0x68, 0x74, 0x74, 0x70, 0xaa, 0x02, 0x2a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e,
    0x43, 0x6f, 0x72, 0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49,
    0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x48, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e,
    0x48, 0x74, 0x74, 0x70, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
// 定义一个全局变量，用于确保只执行一次初始化操作
var (
    file_transport_internet_headers_http_config_proto_rawDescOnce sync.Once
    file_transport_internet_headers_http_config_proto_rawDescData = file_transport_internet_headers_http_config_proto_rawDesc
)

// 定义一个函数，用于对原始描述数据进行 GZIP 压缩
func file_transport_internet_headers_http_config_proto_rawDescGZIP() []byte {
    // 使用 sync.Once 确保只执行一次 GZIP 压缩操作
    file_transport_internet_headers_http_config_proto_rawDescOnce.Do(func() {
        file_transport_internet_headers_http_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_headers_http_config_proto_rawDescData)
    })
    // 返回 GZIP 压缩后的原始描述数据
    return file_transport_internet_headers_http_config_proto_rawDescData
}

// 定义消息类型的切片
var file_transport_internet_headers_http_config_proto_msgTypes = make([]protoimpl.MessageInfo, 7)
// 定义 Go 类型的切片
var file_transport_internet_headers_http_config_proto_goTypes = []interface{}{
    (*Header)(nil),         // 0: v2ray.core.transport.internet.headers.http.Header
    (*Version)(nil),        // 1: v2ray.core.transport.internet.headers.http.Version
    (*Method)(nil),         // 2: v2ray.core.transport.internet.headers.http.Method
    (*RequestConfig)(nil),  // 3: v2ray.core.transport.internet.headers.http.RequestConfig
    (*Status)(nil),         // 4: v2ray.core.transport.internet.headers.http.Status
    (*ResponseConfig)(nil), // 5: v2ray.core.transport.internet.headers.http.ResponseConfig
    (*Config)(nil),         // 6: v2ray.core.transport.internet.headers.http.Config
}
// 定义依赖索引的切片
var file_transport_internet_headers_http_config_proto_depIdxs = []int32{
    1, // 0: v2ray.core.transport.internet.headers.http.RequestConfig.version:type_name -> v2ray.core.transport.internet.headers.http.Version
    2, // 1: v2ray.core.transport.internet.headers.http.RequestConfig.method:type_name -> v2ray.core.transport.internet.headers.http.Method
    0, // 2: v2ray.core.transport.internet.headers.http.RequestConfig.header:type_name -> v2ray.core.transport.internet.headers.http.Header
    1, // 3: v2ray.core.transport.internet.headers.http.ResponseConfig.version:type_name -> v2ray.core.transport.internet.headers.http.Version
    // 第3个元素表示 v2ray.core.transport.internet.headers.http.ResponseConfig.version 的类型为 v2ray.core.transport.internet.headers.http.Version
    4, // 4: v2ray.core.transport.internet.headers.http.ResponseConfig.status:type_name -> v2ray.core.transport.internet.headers.http.Status
    // 第4个元素表示 v2ray.core.transport.internet.headers.http.ResponseConfig.status 的类型为 v2ray.core.transport.internet.headers.http.Status
    0, // 5: v2ray.core.transport.internet.headers.http.ResponseConfig.header:type_name -> v2ray.core.transport.internet.headers.http.Header
    // 第5个元素表示 v2ray.core.transport.internet.headers.http.ResponseConfig.header 的类型为 v2ray.core.transport.internet.headers.http.Header
    3, // 6: v2ray.core.transport.internet.headers.http.Config.request:type_name -> v2ray.core.transport.internet.headers.http.RequestConfig
    // 第6个元素表示 v2ray.core.transport.internet.headers.http.Config.request 的类型为 v2ray.core.transport.internet.headers.http.RequestConfig
    5, // 7: v2ray.core.transport.internet.headers.http.Config.response:type_name -> v2ray.core.transport.internet.headers.http.ResponseConfig
    // 第7个元素表示 v2ray.core.transport.internet.headers.http.Config.response 的类型为 v2ray.core.transport.internet.headers.http.ResponseConfig
    8, // [8:8] is the sub-list for method output_type
    // 第8个元素表示方法的输出类型
    8, // [8:8] is the sub-list for method input_type
    // 第8个元素表示方法的输入类型
    8, // [8:8] is the sub-list for extension type_name
    // 第8个元素表示扩展的类型名
    8, // [8:8] is the sub-list for extension extendee
    // 第8个元素表示扩展的扩展对象
    0, // [0:8] is the sub-list for field type_name
    // 第0个元素表示字段的类型名
func init() { file_transport_internet_headers_http_config_proto_init() }
func file_transport_internet_headers_http_config_proto_init() {
    // 如果文件已经被初始化，则直接返回
    if File_transport_internet_headers_http_config_proto != nil {
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
            RawDescriptor: file_transport_internet_headers_http_config_proto_rawDesc,
            // 设置枚举数量为 0
            NumEnums:      0,
            // 设置消息数量为 7
            NumMessages:   7,
            // 设置扩展数量为 0
            NumExtensions: 0,
            // 设置服务数量为 0
            NumServices:   0,
        },
        // 设置 Go 类型
        GoTypes:           file_transport_internet_headers_http_config_proto_goTypes,
        // 设置依赖索引
        DependencyIndexes: file_transport_internet_headers_http_config_proto_depIdxs,
        // 设置消息信息
        MessageInfos:      file_transport_internet_headers_http_config_proto_msgTypes,
    }.Build()
    // 将构建的类型赋值给 File_transport_internet_headers_http_config_proto
    File_transport_internet_headers_http_config_proto = out.File
    // 清空原始描述
    file_transport_internet_headers_http_config_proto_rawDesc = nil
    // 清空 Go 类型
    file_transport_internet_headers_http_config_proto_goTypes = nil
    // 清空依赖索引
    file_transport_internet_headers_http_config_proto_depIdxs = nil
}
```