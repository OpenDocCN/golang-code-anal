# `kubo\core\coreiface\unixfs.go`

```go
package iface

import (
    "context"  // 导入上下文包

    "github.com/ipfs/boxo/files"  // 导入文件包
    "github.com/ipfs/boxo/path"  // 导入路径包
    "github.com/ipfs/go-cid"  // 导入CID包
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包
)

type AddEvent struct {
    Name  string  // 事件名称
    Path  path.ImmutablePath `json:",omitempty"`  // 不可变路径
    Bytes int64  `json:",omitempty"`  // 字节数
    Size  string  `json:",omitempty"`  // 大小
}

// FileType is an enum of possible UnixFS file types.
type FileType int32  // UnixFS文件类型的枚举

const (
    // TUnknown means the file type isn't known (e.g., it hasn't been
    // resolved).
    TUnknown FileType = iota  // 未知文件类型
    // TFile is a regular file.
    TFile  // 普通文件
    // TDirectory is a directory.
    TDirectory  // 目录
    // TSymlink is a symlink.
    TSymlink  // 符号链接
)

func (t FileType) String() string {
    switch t {
    case TUnknown:
        return "unknown"  // 未知
    case TFile:
        return "file"  // 文件
    case TDirectory:
        return "directory"  // 目录
    case TSymlink:
        return "symlink"  // 符号链接
    default:
        return "<unknown file type>"  // 未知文件类型
    }
}

// DirEntry is a directory entry returned by `Ls`.
type DirEntry struct {
    Name string  // 名称
    Cid  cid.Cid  // CID

    // Only filled when asked to resolve the directory entry.
    Size   uint64  // 文件的大小（或符号链接的大小）
    Type   FileType  // 文件类型
    Target string  // 符号链接的目标

    Err error  // 错误
}

// UnixfsAPI is the basic interface to immutable files in IPFS
// NOTE: This API is heavily WIP, things are guaranteed to break frequently
type UnixfsAPI interface {
    // Add imports the data from the reader into merkledag file
    //
    // TODO: a long useful comment on how to use this for many different scenarios
    Add(context.Context, files.Node, ...options.UnixfsAddOption) (path.ImmutablePath, error)  // 将读取器中的数据导入merkledag文件

    // Get returns a read-only handle to a file tree referenced by a path
    //
    // Note that some implementations of this API may apply the specified context
    // to operations performed on the returned file
}
    // Get函数接受一个上下文对象和一个路径对象作为参数，返回一个文件节点和一个错误对象
    Get(context.Context, path.Path) (files.Node, error)

    // Ls函数返回一个目录中链接的列表。不能保证链接按顺序返回
    Ls(context.Context, path.Path, ...options.UnixfsLsOption) (<-chan DirEntry, error)
# 闭合前面的函数定义
```