# `kubo\fuse\mount\mount.go`

```
// package mount provides a simple abstraction around a mount point
package mount

import (
    "fmt"
    "io"
    "os/exec"
    "runtime"
    "time"

    logging "github.com/ipfs/go-log"
    goprocess "github.com/jbenet/goprocess"
)

// 创建一个名为log的日志记录器
var log = logging.Logger("mount")

// 设置默认的挂载超时时间为5秒
var MountTimeout = time.Second * 5

// Mount represents a filesystem mount.
type Mount interface {
    // MountPoint is the path at which this mount is mounted
    MountPoint() string

    // Unmounts the mount
    Unmount() error

    // Checks if the mount is still active.
    IsActive() bool

    // Process returns the mount's Process to be able to link it
    // to other processes. Unmount upon closing.
    Process() goprocess.Process
}

// ForceUnmount attempts to forcibly unmount a given mount.
// It does so by calling diskutil or fusermount directly.
func ForceUnmount(m Mount) error {
    // 获取挂载点路径
    point := m.MountPoint()
    // 输出警告信息
    log.Warnf("Force-Unmounting %s...", point)

    // 获取执行强制卸载命令
    cmd, err := UnmountCmd(point)
    if err != nil {
        return err
    }

    // 创建一个用于接收错误信息的通道
    errc := make(chan error, 1)
    go func() {
        defer close(errc)

        // 尝试使用普通的卸载命令
        if err := exec.Command("umount", point).Run(); err == nil {
            return
        }

        // 使用备用命令再次尝试卸载
        errc <- cmd.Run()
    }()

    // 选择先到达的事件，如果超时则返回错误
    select {
    case <-time.After(7 * time.Second):
        return fmt.Errorf("umount timeout")
    case err := <-errc:
        return err
    }
}

// UnmountCmd creates an exec.Cmd that is GOOS-specific
// for unmount a FUSE mount.
func UnmountCmd(point string) (*exec.Cmd, error) {
    switch runtime.GOOS {
    case "darwin":
        return exec.Command("diskutil", "umount", "force", point), nil
    case "linux":
        return exec.Command("fusermount", "-u", point), nil
    default:
        return nil, fmt.Errorf("unmount: unimplemented")
    }
}

// ForceUnmountManyTimes attempts to forcibly unmount a given mount,
// 多次尝试强制卸载挂载点，通过直接调用diskutil或fusermount来实现。
// 尝试指定的次数。
func ForceUnmountManyTimes(m Mount, attempts int) error {
    var err error
    for i := 0; i < attempts; i++ {
        err = ForceUnmount(m)  // 调用ForceUnmount函数尝试卸载挂载点
        if err == nil {
            return err  // 如果成功卸载，则返回nil
        }

        <-time.After(time.Millisecond * 500)  // 等待500毫秒
    }
    return fmt.Errorf("unmount %s failed after 10 seconds of trying", m.MountPoint())  // 返回错误信息
}

// closer结构体，用于实现io.Closer接口
type closer struct {
    M Mount
}

// 实现io.Closer接口的Close方法
func (c *closer) Close() error {
    log.Warn(" (c *closer) Close(),", c.M.MountPoint())  // 记录日志
    return c.M.Unmount()  // 调用Unmount方法卸载挂载点
}

// Closer函数，返回实现io.Closer接口的closer结构体实例
func Closer(m Mount) io.Closer {
    return &closer{m}  // 返回closer结构体实例的指针
}
```