# `kubo\plugin\loader\load_noplugin.go`

```go
// 标记当前代码块为适用于 noplugin 构建标记的代码
// 构建标记为 noplugin
package loader

import (
    "errors"

    iplugin "github.com/ipfs/kubo/plugin"
)

// 初始化函数，设置 loadPluginFunc 为 nopluginLoadPlugin 函数
func init() {
    loadPluginFunc = nopluginLoadPlugin
}

// nopluginLoadPlugin 函数，接受一个字符串参数，返回一个插件数组和一个错误
func nopluginLoadPlugin(string) ([]iplugin.Plugin, error) {
    // 返回空的插件数组和一个包含错误信息的错误对象
    return nil, errors.New("not built with plugin support")
}
```