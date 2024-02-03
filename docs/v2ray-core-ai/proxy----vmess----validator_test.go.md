# `v2ray-core\proxy\vmess\validator_test.go`

```go
package vmess_test

import (
    "testing"
    "time"

    "v2ray.com/core/common"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/common/uuid"
    . "v2ray.com/core/proxy/vmess"
)

func toAccount(a *Account) protocol.Account {
    // 将 Account 转换为 protocol.Account
    account, err := a.AsAccount()
    // 如果有错误，立即终止程序
    common.Must(err)
    return account
}

func TestUserValidator(t *testing.T) {
    // 使用默认的 IDHash 创建一个新的 TimedUserValidator
    hasher := protocol.DefaultIDHash
    v := NewTimedUserValidator(hasher)
    // 在函数返回时关闭 v
    defer common.Close(v)

    // 生成一个新的 UUID
    id := uuid.New()
    // 创建一个 MemoryUser 对象
    user := &protocol.MemoryUser{
        Email: "test",
        Account: toAccount(&Account{
            Id:      id.String(),
            AlterId: 8,
        }),
    }
    // 如果有错误，立即终止程序
    common.Must(v.Add(user))

    {
        // 定义一个内部函数 testSmallLag，用于测试时间延迟
        testSmallLag := func(lag time.Duration) {
            // 生成一个时间戳
            ts := protocol.Timestamp(time.Now().Add(time.Second * lag).Unix())
            // 使用 IDHash 对 UUID 进行哈希
            idHash := hasher(id.Bytes())
            // 将时间戳写入 idHash
            common.Must2(serial.WriteUint64(idHash, uint64(ts)))
            // 对 idHash 进行哈希，得到用户哈希
            userHash := idHash.Sum(nil)

            // 从 v 中获取用户信息和时间戳
            euser, ets, found, _ := v.Get(userHash)
            // 如果用户信息未找到，终止程序
            if !found {
                t.Fatal("user not found")
            }
            // 如果用户信息的邮箱不符合预期，输出错误信息
            if euser.Email != user.Email {
                t.Error("unexpected user email: ", euser.Email, " want ", user.Email)
            }
            // 如果时间戳不符合预期，输出错误信息
            if ets != ts {
                t.Error("unexpected timestamp: ", ets, " want ", ts)
            }
        }

        // 分别测试不同的时间延迟
        testSmallLag(0)
        testSmallLag(40)
        testSmallLag(-40)
        testSmallLag(80)
        testSmallLag(-80)
        testSmallLag(120)
        testSmallLag(-120)
    }
}
    {
        // 定义测试函数，测试时间偏移量对应的操作
        testBigLag := func(lag time.Duration) {
            // 根据时间偏移量创建时间戳
            ts := protocol.Timestamp(time.Now().Add(time.Second * lag).Unix())
            // 对用户ID进行哈希处理
            idHash := hasher(id.Bytes())
            // 将时间戳转换为 uint64 类型并写入哈希值
            common.Must2(serial.WriteUint64(idHash, uint64(ts)))
            // 对用户ID哈希值进行处理
            userHash := idHash.Sum(nil)
    
            // 从存储中获取用户信息
            euser, _, found, _ := v.Get(userHash)
            // 如果找到用户信息或者 euser 不为空，则报错
            if found || euser != nil {
                t.Error("unexpected user")
            }
        }
    
        // 分别测试不同时间偏移量对应的操作
        testBigLag(121)
        testBigLag(-121)
        testBigLag(310)
        testBigLag(-310)
        testBigLag(500)
        testBigLag(-500)
    }
    
    // 从存储中移除用户信息，并检查是否成功
    if v := v.Remove(user.Email); !v {
        t.Error("unable to remove user")
    }
    // 再次尝试从存储中移除用户信息，并检查是否成功
    if v := v.Remove(user.Email); v {
        t.Error("remove user twice")
    }
# 基准测试用户验证器函数
func BenchmarkUserValidator(b *testing.B) {
    # 循环执行基准测试
    for i := 0; i < b.N; i++ {
        # 使用默认的 ID 哈希算法创建哈希器
        hasher := protocol.DefaultIDHash
        # 创建一个新的定时用户验证器实例
        v := NewTimedUserValidator(hasher)

        # 循环添加1500个内存用户到验证器中
        for j := 0; j < 1500; j++ {
            # 生成一个新的 UUID 作为用户 ID
            id := uuid.New()
            # 将内存用户添加到验证器中
            v.Add(&protocol.MemoryUser{
                Email: "test",
                Account: toAccount(&Account{
                    Id:      id.String(),
                    AlterId: 16,
                }),
            })
        }

        # 关闭验证器
        common.Close(v)
    }
}
```