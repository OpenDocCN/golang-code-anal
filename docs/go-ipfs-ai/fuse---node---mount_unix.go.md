# `kubo\fuse\node\mount_unix.go`

```go
//go:build !windows && !openbsd && !netbsd && !plan9 && !nofuse
// +build !windows,!openbsd,!netbsd,!plan9,!nofuse

package node

import (
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输出
    "strings"  // 导入 strings 包，用于处理字符串
    "sync"  // 导入 sync 包，用于实现同步功能

    core "github.com/ipfs/kubo/core"  // 导入 core 包，用于处理 IPFS 核心功能
    ipns "github.com/ipfs/kubo/fuse/ipns"  // 导入 ipns 包，用于处理 IPNS 功能
    mount "github.com/ipfs/kubo/fuse/mount"  // 导入 mount 包，用于挂载文件系统
    rofs "github.com/ipfs/kubo/fuse/readonly"  // 导入 rofs 包，用于只读文件系统

    logging "github.com/ipfs/go-log"  // 导入 logging 包，用于日志记录
)

var log = logging.Logger("node")  // 创建名为 log 的日志记录器

// fuseNoDirectory used to check the returning fuse error.
const fuseNoDirectory = "fusermount: failed to access mountpoint"  // 定义常量 fuseNoDirectory，用于检查返回的 fuse 错误

// fuseExitStatus1 used to check the returning fuse error.
const fuseExitStatus1 = "fusermount: exit status 1"  // 定义常量 fuseExitStatus1，用于检查返回的 fuse 错误

// platformFuseChecks can get overridden by arch-specific files
// to run fuse checks (like checking the OSXFUSE version).
var platformFuseChecks = func(*core.IpfsNode) error {
    return nil  // 定义变量 platformFuseChecks，用于运行 fuse 检查
}

func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
    // check if we already have live mounts.
    // if the user said "Mount", then there must be something wrong.
    // so, close them and try again.
    if node.Mounts.Ipfs != nil && node.Mounts.Ipfs.IsActive() {
        // best effort
        _ = node.Mounts.Ipfs.Unmount()  // 如果已经存在活动的挂载，则尝试卸载
    }
    if node.Mounts.Ipns != nil && node.Mounts.Ipns.IsActive() {
        // best effort
        _ = node.Mounts.Ipns.Unmount()  // 如果已经存在活动的挂载，则尝试卸载
    }

    if err := platformFuseChecks(node); err != nil {
        return err  // 运行平台特定的 fuse 检查，如果出错则返回错误
    }

    return doMount(node, fsdir, nsdir)  // 执行挂载操作
}

func doMount(node *core.IpfsNode, fsdir, nsdir string) error {
    // 定义一个匿名函数，用于格式化并处理挂载错误
    fmtFuseErr := func(err error, mountpoint string) error {
        s := err.Error()
        // 如果错误信息包含指定字符串，进行替换处理
        if strings.Contains(s, fuseNoDirectory) {
            s = strings.Replace(s, `fusermount: "fusermount:`, "", -1)
            s = strings.Replace(s, `\n", exit status 1`, "", -1)
            return errors.New(s)
        }
        // 如果错误信息等于指定字符串，进行格式化处理
        if s == fuseExitStatus1 {
            s = fmt.Sprintf("fuse failed to access mountpoint %s", mountpoint)
            return errors.New(s)
        }
        // 返回原始错误信息
        return err
    }

    // 创建两个 Mount 对象和两个错误对象
    var fsmount, nsmount mount.Mount
    var err1, err2 error

    // 创建一个同步等待组
    var wg sync.WaitGroup

    // 增加等待组计数
    wg.Add(1)
    // 启动一个 goroutine，用于挂载只读文件系统
    go func() {
        defer wg.Done()
        fsmount, err1 = rofs.Mount(node, fsdir)
    }()

    // 如果节点在线，增加等待组计数
    if node.IsOnline {
        wg.Add(1)
        // 启动一个 goroutine，用于挂载 IPNS 文件系统
        go func() {
            defer wg.Done()
            nsmount, err2 = ipns.Mount(node, nsdir, fsdir)
        }()
    }

    // 等待所有 goroutine 完成
    wg.Wait()

    // 如果挂载只读文件系统出错，记录错误日志
    if err1 != nil {
        log.Errorf("error mounting: %s", err1)
    }

    // 如果挂载 IPNS 文件系统出错，记录错误日志
    if err2 != nil {
        log.Errorf("error mounting: %s", err2)
    }

    // 如果有任何一个挂载出错
    if err1 != nil || err2 != nil {
        // 如果只读文件系统已经挂载，执行卸载操作
        if fsmount != nil {
            _ = fsmount.Unmount()
        }
        // 如果 IPNS 文件系统已经挂载，执行卸载操作
        if nsmount != nil {
            _ = nsmount.Unmount()
        }

        // 如果只读文件系统出错，返回格式化处理后的错误信息
        if err1 != nil {
            return fmtFuseErr(err1, fsdir)
        }
        // 如果 IPNS 文件系统出错，返回格式化处理后的错误信息
        return fmtFuseErr(err2, nsdir)
    }

    // 设置节点状态的挂载信息，并返回空值
    node.Mounts.Ipfs = fsmount
    node.Mounts.Ipns = nsmount
    return nil
# 闭合前面的函数定义
```