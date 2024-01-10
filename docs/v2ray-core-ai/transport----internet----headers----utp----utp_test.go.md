# `v2ray-core\transport\internet\headers\utp\utp_test.go`

```
package utp_test

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/buf"  // 导入缓冲包
    . "v2ray.com/core/transport/internet/headers/utp"  // 导入 UTP 头部包
)

func TestUTPWrite(t *testing.T) {
    content := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g'}  // 定义测试内容
    utpRaw, err := New(context.Background(), &Config{})  // 创建 UTP 对象
    common.Must(err)  // 检查错误

    utp := utpRaw.(*UTP)  // 将原始 UTP 对象转换为 UTP 类型

    payload := buf.New()  // 创建缓冲区
    utp.Serialize(payload.Extend(utp.Size()))  // 序列化 UTP 头部并写入缓冲区
    payload.Write(content)  // 将测试内容写入缓冲区

    if payload.Len() != int32(len(content))+utp.Size() {  // 检查缓冲区长度是否符合预期
        t.Error("unexpected payload length: ", payload.Len())  // 输出错误信息
    }
}
```