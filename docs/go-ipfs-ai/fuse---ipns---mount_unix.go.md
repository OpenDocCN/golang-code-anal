# `kubo\fuse\ipns\mount_unix.go`

```go
// 根据条件编译指令，当操作系统为 linux、darwin、freebsd、netbsd、openbsd 且不是 nofuse 时进行编译
// 导入所需的包
package ipns

import (
    core "github.com/ipfs/kubo/core"  // 导入 core 包并重命名为 core
    coreapi "github.com/ipfs/kubo/core/coreapi"  // 导入 coreapi 包并重命名为 coreapi
    mount "github.com/ipfs/kubo/fuse/mount"  // 导入 mount 包并重命名为 mount
)

// Mount 函数挂载给定位置的 ipns，并返回一个 mount.Mount 实例
func Mount(ipfs *core.IpfsNode, ipnsmp, ipfsmp string) (mount.Mount, error) {
    // 创建 coreAPI 实例
    coreAPI, err := coreapi.NewCoreAPI(ipfs)
    if err != nil {
        return nil, err
    }

    // 获取 ipfs 节点的配置信息
    cfg, err := ipfs.Repo.Config()
    if err != nil {
        return nil, err
    }

    // 获取是否允许其他用户访问的配置信息
    allowOther := cfg.Mounts.FuseAllowOther

    // 创建文件系统实例
    fsys, err := NewFileSystem(ipfs.Context(), coreAPI, ipfsmp, ipnsmp)
    if err != nil {
        return nil, err
    }

    // 返回挂载实例
    return mount.NewMount(ipfs.Process, fsys, ipnsmp, allowOther)
}
```