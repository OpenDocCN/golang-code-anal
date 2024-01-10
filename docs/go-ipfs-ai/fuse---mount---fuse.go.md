# `kubo\fuse\mount\fuse.go`

```
// 根据构建标签，判断是否满足条件，如果不满足则不包含在构建中
// +build !nofuse,!windows,!openbsd,!netbsd,!plan9

package mount

import (
    "errors"
    "fmt"
    "sync"
    "time"

    "bazil.org/fuse"
    "bazil.org/fuse/fs"
    "github.com/jbenet/goprocess"
)

// 定义错误变量，表示未挂载
var ErrNotMounted = errors.New("not mounted")

// mount 结构体实现了 go-ipfs/fuse/mount 接口
type mount struct {
    mpoint     string
    filesys    fs.FS
    fuseConn   *fuse.Conn
    active     bool
    activeLock *sync.RWMutex
    proc       goprocess.Process
}

// NewMount 函数用于挂载 fuse fs.FS 到指定位置，并返回 Mount 实例
// parent 是一个 ContextGroup，用于绑定挂载的 ContextGroup
func NewMount(p goprocess.Process, fsys fs.FS, mountpoint string, allowOther bool) (Mount, error) {
    var conn *fuse.Conn
    var err error

    // 设置挂载选项
    mountOpts := []fuse.MountOption{
        fuse.MaxReadahead(64 * 1024 * 1024),
        fuse.AsyncRead(),
    }

    // 如果允许其他用户访问，则添加 AllowOther 选项
    if allowOther {
        mountOpts = append(mountOpts, fuse.AllowOther())
    }
    // 挂载到指定位置
    conn, err = fuse.Mount(mountpoint, mountOpts...)

    if err != nil {
        return nil, err
    }

    // 创建 mount 实例
    m := &mount{
        mpoint:     mountpoint,
        fuseConn:   conn,
        filesys:    fsys,
        active:     false,
        activeLock: &sync.RWMutex{},
        proc:       goprocess.WithParent(p), // 将其链接到父进程
    }
    m.proc.SetTeardown(m.unmount)

    // 启动挂载过程
    if err := m.mount(); err != nil {
        _ = m.Unmount() // 以防万一
        return nil, err
    }

    return m, nil
}

// mount 方法用于执行挂载操作
func (m *mount) mount() error {
    log.Infof("Mounting %s", m.MountPoint())

    errs := make(chan error, 1)
    go func() {
        // fs.Serve 会阻塞直到文件系统被卸载
        err := fs.Serve(m.fuseConn, m.filesys)
        log.Debugf("%s is unmounted", m.MountPoint())
        if err != nil {
            log.Debugf("fs.Serve returned (%s)", err)
            errs <- err
        }
        m.setActive(false)
    }()
}
    // 等待挂载过程完成，或者超时
    select {
    case <-time.After(MountTimeout):
        return fmt.Errorf("mounting %s timed out", m.MountPoint())
    case err := <-errs:
        return err
    case <-m.fuseConn.Ready:
    }

    // 检查挂载过程是否有错误报告
    if err := m.fuseConn.MountError; err != nil {
        return err
    }

    // 设置挂载状态为活跃
    m.setActive(true)

    // 记录挂载点信息
    log.Infof("Mounted %s", m.MountPoint())
    // 返回空值
    return nil
// umount is called exactly once to unmount this service.
// note that closing the connection will not always unmount
// properly. If that happens, we bring out the big guns
// (mount.ForceUnmountManyTimes, exec unmount).
func (m *mount) unmount() error {
    log.Infof("Unmounting %s", m.MountPoint())

    // try unmounting with fuse lib
    err := fuse.Unmount(m.MountPoint())
    if err == nil {
        m.setActive(false)
        return nil
    }
    log.Warnf("fuse unmount err: %s", err)

    // try closing the fuseConn
    err = m.fuseConn.Close()
    if err == nil {
        m.setActive(false)
        return nil
    }
    log.Warnf("fuse conn error: %s", err)

    // try mount.ForceUnmountManyTimes
    if err := ForceUnmountManyTimes(m, 10); err != nil {
        return err
    }

    log.Infof("Seemingly unmounted %s", m.MountPoint())
    m.setActive(false)
    return nil
}

// 返回挂载点所在的进程
func (m *mount) Process() goprocess.Process {
    return m.proc
}

// 返回挂载点的路径
func (m *mount) MountPoint() string {
    return m.mpoint
}

// 尝试卸载挂载点，如果未挂载则返回 ErrNotMounted
func (m *mount) Unmount() error {
    if !m.IsActive() {
        return ErrNotMounted
    }

    // 调用 Process Close() 方法，确保调用 unmount() 仅一次
    return m.proc.Close()
}

// 返回挂载点是否处于活动状态
func (m *mount) IsActive() bool {
    m.activeLock.RLock()
    defer m.activeLock.RUnlock()

    return m.active
}

// 设置挂载点的活动状态
func (m *mount) setActive(a bool) {
    m.activeLock.Lock()
    m.active = a
    m.activeLock.Unlock()
}
```