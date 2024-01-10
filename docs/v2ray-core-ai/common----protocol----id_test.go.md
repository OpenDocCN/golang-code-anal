# `v2ray-core\common\protocol\id_test.go`

```
package protocol_test

import (
    "testing"  // 导入测试包
    . "v2ray.com/core/common/protocol"  // 导入 v2ray 协议包
    "v2ray.com/core/common/uuid"  // 导入 v2ray UUID 包
)

func TestIdEquals(t *testing.T) {
    id1 := NewID(uuid.New())  // 创建一个新的 ID 对象，使用新生成的 UUID
    id2 := NewID(id1.UUID())  // 创建一个新的 ID 对象，使用 id1 的 UUID

    if !id1.Equals(id2) {  // 如果 id1 不等于 id2
        t.Error("expected id1 to equal id2, but actually not")  // 输出错误信息
    }

    if id1.String() != id2.String() {  // 如果 id1 的字符串表示不等于 id2 的字符串表示
        t.Error(id1.String(), " != ", id2.String())  // 输出错误信息，显示两个 ID 的字符串表示
    }
}
```