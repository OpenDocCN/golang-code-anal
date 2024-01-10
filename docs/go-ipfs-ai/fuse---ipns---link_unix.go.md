# `kubo\fuse\ipns\link_unix.go`

```
//go:build !nofuse && !openbsd && !netbsd && !plan9
// +build !nofuse,!openbsd,!netbsd,!plan9
// 如果编译标记不包括 nofuse、openbsd、netbsd 和 plan9，则编译此文件

package ipns
// 定义包名为 ipns

import (
    "context"
    "os"
    // 导入所需的包
    "bazil.org/fuse"
    "bazil.org/fuse/fs"
)

type Link struct {
    Target string
}
// 定义 Link 结构体，包含 Target 字段

func (l *Link) Attr(ctx context.Context, a *fuse.Attr) error {
    log.Debug("Link attr.")
    // 记录日志，表示获取链接属性
    a.Mode = os.ModeSymlink | 0o555
    // 设置属性为符号链接并且可执行
    return nil
}

func (l *Link) Readlink(ctx context.Context, req *fuse.ReadlinkRequest) (string, error) {
    log.Debugf("ReadLink: %s", l.Target)
    // 记录日志，表示读取链接目标
    return l.Target, nil
    // 返回链接目标和无错误
}

var _ fs.NodeReadlinker = (*Link)(nil)
// 确保 Link 类型实现了 fs.NodeReadlinker 接口
```