# `kubo\cmd\ipfs\kubo\daemon_linux.go`

```
// 如果是在 Linux 系统下编译该包
// 构建标签为 linux
package kubo

// 导入 coreos/go-systemd/v22/daemon 包
import (
    daemon "github.com/coreos/go-systemd/v22/daemon"
)

// 通知 systemd 服务已准备就绪
func notifyReady() {
    // 使用 systemd 的 SdNotify 函数通知服务已准备就绪
    _, _ = daemon.SdNotify(false, daemon.SdNotifyReady)
}

// 通知 systemd 服务正在停止
func notifyStopping() {
    // 使用 systemd 的 SdNotify 函数通知服务正在停止
    _, _ = daemon.SdNotify(false, daemon.SdNotifyStopping)
}
```