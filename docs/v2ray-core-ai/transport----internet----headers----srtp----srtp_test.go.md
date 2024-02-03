# `v2ray-core\transport\internet\headers\srtp\srtp_test.go`

```go
// 定义一个名为 srtp_test 的包
package srtp_test

// 导入所需的包
import (
    "context"
    "testing"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    . "v2ray.com/core/transport/internet/headers/srtp"
)

// 定义名为 TestSRTPWrite 的测试函数
func TestSRTPWrite(t *testing.T) {
    // 定义一个包含字节 'a', 'b', 'c', 'd', 'e', 'f', 'g' 的切片
    content := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g'}
    // 使用 New 函数创建一个 SRTP 对象
    srtpRaw, err := New(context.Background(), &Config{})
    // 检查错误并处理
    common.Must(err)

    // 将 srtpRaw 转换为 SRTP 类型
    srtp := srtpRaw.(*SRTP)

    // 创建一个新的缓冲区
    payload := buf.New()
    // 将 SRTP 对象序列化后写入缓冲区
    srtp.Serialize(payload.Extend(srtp.Size()))
    // 将 content 写入缓冲区
    payload.Write(content)

    // 计算预期的长度
    expectedLen := int32(len(content)) + srtp.Size()
    // 检查实际长度是否与预期长度相等
    if payload.Len() != expectedLen {
        // 输出错误信息
        t.Error("expected ", expectedLen, " of bytes, but got ", payload.Len())
    }
}
```