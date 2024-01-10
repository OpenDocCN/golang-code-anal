# `kubo\fuse\readonly\mount_unix.go`

```
// 根据条件构建标签，指定在 linux、darwin、freebsd 平台上构建，并且不包括 nofuse
// +build linux darwin freebsd
// +build !nofuse

package readonly

import (
    core "github.com/ipfs/kubo/core"  // 导入 core 包，并重命名为 core
    mount "github.com/ipfs/kubo/fuse/mount"  // 导入 mount 包，并重命名为 mount
)

// 在给定位置挂载 IPFS，并返回一个 mount.Mount 实例
func Mount(ipfs *core.IpfsNode, mountpoint string) (mount.Mount, error) {
    // 获取 IPFS 节点的配置信息
    cfg, err := ipfs.Repo.Config()
    if err != nil {
        return nil, err
    }
    // 获取是否允许其他用户挂载的配置信息
    allowOther := cfg.Mounts.FuseAllowOther
    // 创建一个新的文件系统实例
    fsys := NewFileSystem(ipfs)
    // 返回一个新的挂载实例
    return mount.NewMount(ipfs.Process, fsys, mountpoint, allowOther)
}
```