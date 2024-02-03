# `v2ray-core\app\policy\manager_test.go`

```go
package policy_test

import (
    "context"
    "testing"
    "time"

    . "v2ray.com/core/app/policy"  // 导入 v2ray.com/core/app/policy 包，使用 . 表示在后续代码中可以省略包名
    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/features/policy"  // 导入 v2ray.com/core/features/policy 包
)

func TestPolicy(t *testing.T) {
    manager, err := New(context.Background(), &Config{  // 创建一个 PolicyManager 对象，并传入 context 和 Config 对象的指针
        Level: map[uint32]*Policy{  // 创建一个 uint32 到 *Policy 的映射
            0: {  // 当 Level 为 0 时
                Timeout: &Policy_Timeout{  // 设置超时策略
                    Handshake: &Second{  // 设置握手超时时间
                        Value: 2,  // 设置超时时间为 2 秒
                    },
                },
            },
        },
    })
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    pDefault := policy.SessionDefault()  // 获取默认的会话策略

    {
        p := manager.ForLevel(0)  // 获取 Level 为 0 的策略
        if p.Timeouts.Handshake != 2*time.Second {  // 检查握手超时时间是否为 2 秒
            t.Error("expect 2 sec timeout, but got ", p.Timeouts.Handshake)  // 输出错误信息
        }
        if p.Timeouts.ConnectionIdle != pDefault.Timeouts.ConnectionIdle {  // 检查连接空闲超时时间是否与默认值相同
            t.Error("expect ", pDefault.Timeouts.ConnectionIdle, " sec timeout, but got ", p.Timeouts.ConnectionIdle)  // 输出错误信息
        }
    }

    {
        p := manager.ForLevel(1)  // 获取 Level 为 1 的策略
        if p.Timeouts.Handshake != pDefault.Timeouts.Handshake {  // 检查握手超时时间是否与默认值相同
            t.Error("expect ", pDefault.Timeouts.Handshake, " sec timeout, but got ", p.Timeouts.Handshake)  // 输出错误信息
        }
    }
}
```