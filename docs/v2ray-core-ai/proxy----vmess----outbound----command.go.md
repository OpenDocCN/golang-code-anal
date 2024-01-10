# `v2ray-core\proxy\vmess\outbound\command.go`

```
// +build !confonly

package outbound

import (
    "time"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/proxy/vmess"
)

// 处理切换账户命令
func (h *Handler) handleSwitchAccount(cmd *protocol.CommandSwitchAccount) {
    // 创建一个 VMess 账户对象
    rawAccount := &vmess.Account{
        Id:      cmd.ID.String(),
        AlterId: uint32(cmd.AlterIds),
        SecuritySettings: &protocol.SecurityConfig{
            Type: protocol.SecurityType_LEGACY,
        },
    }

    // 将 VMess 账户对象转换为通用账户对象
    account, err := rawAccount.AsAccount()
    common.Must(err)
    // 创建一个内存用户对象
    user := &protocol.MemoryUser{
        Email:   "",
        Level:   cmd.Level,
        Account: account,
    }
    // 创建目标地址对象
    dest := net.TCPDestination(cmd.Host, cmd.Port)
    // 计算账户有效期
    until := time.Now().Add(time.Duration(cmd.ValidMin) * time.Minute)
    // 将服务器信息添加到服务器列表中
    h.serverList.AddServer(protocol.NewServerSpec(dest, protocol.BeforeTime(until), user))
}

// 处理响应命令
func (h *Handler) handleCommand(dest net.Destination, cmd protocol.ResponseCommand) {
    // 根据不同类型的命令进行处理
    switch typedCommand := cmd.(type) {
    case *protocol.CommandSwitchAccount:
        // 如果命令中的主机地址为空，则使用传入的目标地址
        if typedCommand.Host == nil {
            typedCommand.Host = dest.Address
        }
        // 处理切换账户命令
        h.handleSwitchAccount(typedCommand)
    default:
    }
}
```