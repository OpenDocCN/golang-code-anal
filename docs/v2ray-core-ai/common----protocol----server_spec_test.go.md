# `v2ray-core\common\protocol\server_spec_test.go`

```
package protocol_test

import (
    "strings" // 导入 strings 包
    "testing" // 导入 testing 包
    "time" // 导入 time 包

    "v2ray.com/core/common" // 导入 common 包
    "v2ray.com/core/common/net" // 导入 net 包
    . "v2ray.com/core/common/protocol" // 导入 protocol 包，并将其所有公开的成员导入当前命名空间
    "v2ray.com/core/common/uuid" // 导入 uuid 包
    "v2ray.com/core/proxy/vmess" // 导入 vmess 包
)

func TestAlwaysValidStrategy(t *testing.T) {
    strategy := AlwaysValid() // 创建 AlwaysValid 策略对象
    if !strategy.IsValid() { // 如果策略无效
        t.Error("strategy not valid") // 输出错误信息
    }
    strategy.Invalidate() // 使策略失效
    if !strategy.IsValid() { // 如果策略无效
        t.Error("strategy not valid") // 输出错误信息
    }
}

func TestTimeoutValidStrategy(t *testing.T) {
    strategy := BeforeTime(time.Now().Add(2 * time.Second)) // 创建在指定时间之前有效的策略对象
    if !strategy.IsValid() { // 如果策略无效
        t.Error("strategy not valid") // 输出错误信息
    }
    time.Sleep(3 * time.Second) // 休眠3秒
    if strategy.IsValid() { // 如果策略有效
        t.Error("strategy is valid") // 输出错误信息
    }

    strategy = BeforeTime(time.Now().Add(2 * time.Second)) // 创建在指定时间之前有效的策略对象
    strategy.Invalidate() // 使策略失效
    if strategy.IsValid() { // 如果策略有效
        t.Error("strategy is valid") // 输出错误信息
    }
}

func TestUserInServerSpec(t *testing.T) {
    uuid1 := uuid.New() // 生成新的 UUID
    uuid2 := uuid.New() // 生成新的 UUID

    toAccount := func(a *vmess.Account) Account { // 定义将 vmess.Account 转换为 Account 的函数
        account, err := a.AsAccount() // 调用 AsAccount 方法
        common.Must(err) // 检查错误
        return account // 返回 Account 对象
    }

    spec := NewServerSpec(net.Destination{}, AlwaysValid(), &MemoryUser{ // 创建新的服务器规范对象
        Email:   "test1@v2ray.com", // 设置邮箱
        Account: toAccount(&vmess.Account{Id: uuid1.String()}), // 设置账户
    })
    if spec.HasUser(&MemoryUser{ // 如果规范中有指定的用户
        Email:   "test1@v2ray.com", // 设置邮箱
        Account: toAccount(&vmess.Account{Id: uuid2.String()}), // 设置账户
    }) {
        t.Error("has user: ", uuid2) // 输出错误信息
    }

    spec.AddUser(&MemoryUser{Email: "test2@v2ray.com"}) // 向规范中添加用户
    if !spec.HasUser(&MemoryUser{ // 如果规范中没有指定的用户
        Email:   "test1@v2ray.com", // 设置邮箱
        Account: toAccount(&vmess.Account{Id: uuid1.String()}), // 设置账户
    }) {
        t.Error("not having user: ", uuid1) // 输出错误信息
    }
}

func TestPickUser(t *testing.T) {
    spec := NewServerSpec(net.Destination{}, AlwaysValid(), &MemoryUser{Email: "test1@v2ray.com"}, &MemoryUser{Email: "test2@v2ray.com"}, &MemoryUser{Email: "test3@v2ray.com"}) // 创建新的服务器规范对象
}
    # 从 spec 对象中选择用户
    user := spec.PickUser()
    # 如果用户的邮箱不是以 "@v2ray.com" 结尾
    if !strings.HasSuffix(user.Email, "@v2ray.com") {
        # 输出错误信息，包括用户的邮箱
        t.Error("user: ", user.Email)
    }
# 闭合前面的函数定义
```