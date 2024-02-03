# `kubo\plugin\loader\load_nocgo.go`

```go
//go:build !cgo && !noplugin && (linux || darwin || freebsd)
// +build !cgo
// +build !noplugin
// +build linux darwin freebsd

// 定义了构建条件，要求不使用 cgo，并且不是 noplugin，并且是在 linux、darwin 或 freebsd 系统上构建

package loader

import (
    "errors"

    iplugin "github.com/ipfs/kubo/plugin"
)

// 初始化函数，在包加载时被调用
func init() {
    // 设置加载插件的函数为 nocgoLoadPlugin
    loadPluginFunc = nocgoLoadPlugin
}

// 不使用 cgo 加载插件的函数
func nocgoLoadPlugin(fi string) ([]iplugin.Plugin, error) {
    // 返回空的插件列表和一个错误，说明不支持 cgo
    return nil, errors.New("not built with cgo support")
}
```