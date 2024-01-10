# `kubo\fuse\node\mount_test.go`

```
// 根据构建标记来确定是否在特定操作系统上构建代码
// 如果不是在 openbsd、nofuse、netbsd、plan9 上构建，则执行下面的代码
// +build !openbsd,!nofuse,!netbsd,!plan9

package node

import (
    "context"
    "os"
    "strings"
    "testing"
    "time"

    "bazil.org/fuse"

    core "github.com/ipfs/kubo/core"
    ipns "github.com/ipfs/kubo/fuse/ipns"
    mount "github.com/ipfs/kubo/fuse/mount"

    ci "github.com/libp2p/go-libp2p-testing/ci"
)

// 在运行测试时，如果不支持 FUSE，则跳过测试
func maybeSkipFuseTests(t *testing.T) {
    if ci.NoFuse() {
        t.Skip("Skipping FUSE tests")
    }
}

// 创建目录
func mkdir(t *testing.T, path string) {
    err := os.Mkdir(path, os.ModeDir|os.ModePerm)
    if err != nil {
        t.Fatal(err)
    }
}

// 测试外部卸载，然后尝试在代码中卸载
func TestExternalUnmount(t *testing.T) {
    if testing.Short() {
        t.SkipNow()
    }

    // TODO: needed?
    maybeSkipFuseTests(t)

    // 创建一个新的节点
    node, err := core.NewNode(context.Background(), &core.BuildCfg{})
    if err != nil {
        t.Fatal(err)
    }

    // 初始化 IPNS 的密钥空间
    err = ipns.InitializeKeyspace(node, node.PrivateKey)
    if err != nil {
        t.Fatal(err)
    }

    // 获取测试目录路径 (/tmp/TestExternalUnmount)
    dir := t.TempDir()

    ipfsDir := dir + "/ipfs"
    ipnsDir := dir + "/ipns"
    // 创建 IPFS 和 IPNS 目录
    mkdir(t, ipfsDir)
    mkdir(t, ipnsDir)

    // 挂载 IPFS 和 IPNS
    err = Mount(node, ipfsDir, ipnsDir)
    if err != nil {
        // 如果出现特定错误或者找不到 OSXFUSE，则跳过
        if strings.Contains(err.Error(), "unable to check fuse version") || err == fuse.ErrOSXFUSENotFound {
            t.Skip(err)
        }
    }

    if err != nil {
        t.Fatalf("error mounting: %v", err)
    }

    // 运行外部命令来卸载目录
    cmd, err := mount.UnmountCmd(ipfsDir)
    if err != nil {
        t.Fatal(err)
    }

    if err := cmd.Run(); err != nil {
        t.Fatal(err)
    }

    // TODO(noffle): it takes a moment for the goroutine that's running fs.Serve to be notified and do its cleanup.
    // 等待一段时间，以便运行 fs.Serve 的 goroutine 被通知并进行清理
    time.Sleep(time.Millisecond * 100)

    // 尝试卸载 IPFS；应该成功卸载
    err = node.Mounts.Ipfs.Unmount()
}
    # 如果错误不是未挂载错误，则执行以下代码
    if err != mount.ErrNotMounted:
        # 输出致命错误信息
        t.Fatal("Unmount should have failed")
    # 如果错误是未挂载错误，则不执行以上代码
# 闭合前面的函数定义
```