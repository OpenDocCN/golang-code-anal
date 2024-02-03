# `v2ray-core\proxy\vmess\encoding\commands_test.go`

```go
package encoding_test

import (
    "testing"

    "github.com/google/go-cmp/cmp"  // 导入用于比较的包

    "v2ray.com/core/common"  // 导入通用功能包
    "v2ray.com/core/common/buf"  // 导入缓冲区包
    "v2ray.com/core/common/protocol"  // 导入协议包
    "v2ray.com/core/common/uuid"  // 导入 UUID 包
    . "v2ray.com/core/proxy/vmess/encoding"  // 导入 VMess 编码包
)

func TestSwitchAccount(t *testing.T) {
    sa := &protocol.CommandSwitchAccount{  // 创建 CommandSwitchAccount 结构体实例
        Port:     1234,  // 设置端口号
        ID:       uuid.New(),  // 生成新的 UUID
        AlterIds: 1024,  // 设置 AlterIds
        Level:    128,  // 设置 Level
        ValidMin: 16,  // 设置 ValidMin
    }

    buffer := buf.New()  // 创建新的缓冲区
    common.Must(MarshalCommand(sa, buffer))  // 使用 MarshalCommand 将 sa 序列化到缓冲区中

    cmd, err := UnmarshalCommand(1, buffer.BytesFrom(2))  // 使用 UnmarshalCommand 从缓冲区中解析命令
    common.Must(err)  // 检查错误

    sa2, ok := cmd.(*protocol.CommandSwitchAccount)  // 将解析后的命令转换为 CommandSwitchAccount 类型
    if !ok {
        t.Fatal("failed to convert command to CommandSwitchAccount")  // 如果转换失败，则输出错误信息
    }
    if r := cmp.Diff(sa2, sa); r != "" {  // 比较两个结构体实例的差异
        t.Error(r)  // 输出差异信息
    }
}
```