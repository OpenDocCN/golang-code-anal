# `trojan-go\statistic\memory\memory_test.go`

```go
package memory

import (
    "context"  // 导入上下文包
    "runtime"  // 导入运行时包
    "strconv"  // 导入字符串转换包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义包
)

func TestMemoryAuth(t *testing.T) {
    cfg := &Config{  // 创建 Config 结构体指针
        Passwords: nil,  // 初始化密码为 nil
    }
    ctx := config.WithConfig(context.Background(), Name, cfg)  // 使用配置创建上下文
    auth, err := NewAuthenticator(ctx)  // 创建认证器
    common.Must(err)  // 检查错误
    auth.AddUser("user1")  // 添加用户
    valid, user := auth.AuthUser("user1")  // 认证用户
    if !valid {  // 如果认证失败
        t.Fatal("add, auth")  // 报告测试失败
    }
    if user.Hash() != "user1" {  // 如果用户哈希值不等于 "user1"
        t.Fatal("Hash")  // 报告测试失败
    }
    user.AddTraffic(100, 200)  // 添加流量
    sent, recv := user.GetTraffic()  // 获取流量
    if sent != 100 || recv != 200 {  // 如果发送或接收流量不等于预期值
        t.Fatal("traffic")  // 报告测试失败
    }
    sent, recv = user.ResetTraffic()  // 重置流量
    if sent != 100 || recv != 200 {  // 如果发送或接收流量不等于预期值
        t.Fatal("ResetTraffic")  // 报告测试失败
    }
    sent, recv = user.GetTraffic()  // 获取流量
    if sent != 0 || recv != 0 {  // 如果发送或接收流量不等于0
        t.Fatal("ResetTraffic")  // 报告测试失败
    }

    user.AddIP("1234")  // 添加 IP 地址
    user.AddIP("5678")  // 添加 IP 地址
    if user.GetIP() != 0 {  // 如果获取的 IP 地址数量不等于0
        t.Fatal("GetIP")  // 报告测试失败
    }

    user.SetIPLimit(2)  // 设置 IP 地址限制
    user.AddIP("1234")  // 添加 IP 地址
    user.AddIP("5678")  // 添加 IP 地址
    user.DelIP("1234")  // 删除 IP 地址
    if user.GetIP() != 1 {  // 如果获取的 IP 地址数量不等于1
        t.Fatal("DelIP")  // 报告测试失败
    }
    user.DelIP("5678")  // 删除 IP 地址

    user.SetIPLimit(2)  // 设置 IP 地址限制
    if !user.AddIP("1") || !user.AddIP("2") {  // 如果无法添加 IP 地址 "1" 或 "2"
        t.Fatal("AddIP")  // 报告测试失败
    }
    if user.AddIP("3") {  // 如果可以添加 IP 地址 "3"
        t.Fatal("AddIP")  // 报告测试失败
    }
    if !user.AddIP("2") {  // 如果无法添加 IP 地址 "2"
        t.Fatal("AddIP")  // 报告测试失败
    }

    user.SetTraffic(1234, 4321)  // 设置流量
    if a, b := user.GetTraffic(); a != 1234 || b != 4321 {  // 如果获取的流量不等于预期值
        t.Fatal("SetTraffic")  // 报告测试失败
    }

    user.ResetTraffic()  // 重置流量
    go func() {  // 创建匿名 goroutine
        for {
            k := 100  // 定义变量 k
            time.Sleep(time.Second / time.Duration(k))  // 休眠
            user.AddTraffic(2000/k, 1000/k)  // 添加流量
        }
    }()
    time.Sleep(time.Second * 4)  // 休眠
    if sent, recv := user.GetSpeed(); sent > 3000 || sent < 1000 || recv > 1500 || recv < 500 {  // 如果获取的速度不在预期范围内
        t.Error("GetSpeed", sent, recv)  // 报告测试错误
    } else {
        t.Log("GetSpeed", sent, recv)  // 记录测试日志
    }

    user.SetSpeedLimit(30, 20)  // 设置速度限制
}
    # 等待4秒钟
    time.Sleep(time.Second * 4)
    
    # 获取用户的发送和接收速度，并进行判断
    if sent, recv := user.GetSpeed(); sent > 60 || recv > 40:
        # 如果发送速度大于60或接收速度大于40，则记录错误信息
        t.Error("SetSpeedLimit", sent, recv)
    else:
        # 否则记录日志信息
        t.Log("SetSpeedLimit", sent, recv)
    
    # 设置用户的发送和接收速度限制为0
    user.SetSpeedLimit(0, 0)
    # 等待4秒钟
    time.Sleep(time.Second * 4)
    
    # 再次获取用户的发送和接收速度，并进行判断
    if sent, recv := user.GetSpeed(); sent < 30 || recv < 20:
        # 如果发送速度小于30或接收速度小于20，则记录错误信息
        t.Error("SetSpeedLimit", sent, recv)
    else:
        # 否则记录日志信息
        t.Log("SetSpeedLimit", sent, recv)
    
    # 添加用户"user2"到认证系统
    auth.AddUser("user2")
    # 验证用户"user2"是否存在
    valid, _ = auth.AuthUser("user2")
    if !valid:
        # 如果用户"user2"不存在，则终止测试
        t.Fatal()
    
    # 从认证系统中删除用户"user2"
    auth.DelUser("user2")
    # 再次验证用户"user2"是否存在
    valid, _ = auth.AuthUser("user2")
    if valid:
        # 如果用户"user2"存在，则终止测试
        t.Fatal()
    
    # 添加用户"user3"到认证系统
    auth.AddUser("user3")
    # 获取认证系统中的用户列表
    users := auth.ListUsers()
    if len(users) != 2:
        # 如果用户列表长度不等于2，则终止测试
        t.Fatal()
    
    # 关闭用户连接
    user.Close()
    # 关闭认证系统连接
    auth.Close()
func BenchmarkMemoryUsage(b *testing.B) {
    // 创建一个配置对象，其中密码为空
    cfg := &Config{
        Passwords: nil,
    }
    // 使用配置对象创建一个上下文
    ctx := config.WithConfig(context.Background(), Name, cfg)
    // 使用上下文创建一个认证对象
    auth, err := NewAuthenticator(ctx)
    // 检查错误
    common.Must(err)

    // 创建两个内存统计对象
    m1 := runtime.MemStats{}
    m2 := runtime.MemStats{}
    // 读取当前内存统计信息
    runtime.ReadMemStats(&m1)
    // 循环执行 b.N 次，每次向认证对象中添加一个用户
    for i := 0; i < b.N; i++ {
        common.Must(auth.AddUser(common.SHA224String("hash" + strconv.Itoa(i))))
    }
    // 读取执行添加用户操作后的内存统计信息
    runtime.ReadMemStats(&m2)

    // 报告内存使用情况的指标
    b.ReportMetric(float64(m2.Alloc-m1.Alloc)/1024/1024, "MiB(Alloc)")
    b.ReportMetric(float64(m2.TotalAlloc-m1.TotalAlloc)/1024/1024, "MiB(TotalAlloc)")
}
```