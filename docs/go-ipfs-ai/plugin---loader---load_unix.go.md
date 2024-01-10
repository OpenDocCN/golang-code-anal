# `kubo\plugin\loader\load_unix.go`

```
// 根据条件编译指令，当满足 cgo 且不满足 noplugin 且满足 (linux || darwin || freebsd) 时，进行编译
// 定义包名为 loader 的包
package loader

// 导入 errors 包，用于处理错误
import (
    "errors"
    "plugin"

    iplugin "github.com/ipfs/kubo/plugin"
)

// 初始化函数，设置 loadPluginFunc 为 unixLoadPlugin 函数
func init() {
    loadPluginFunc = unixLoadPlugin
}

// unixLoadPlugin 函数，根据文件名加载插件
func unixLoadPlugin(fi string) ([]iplugin.Plugin, error) {
    // 打开插件文件
    pl, err := plugin.Open(fi)
    if err != nil {
        return nil, err
    }
    // 查找插件中的 Plugins 符号
    pls, err := pl.Lookup("Plugins")
    if err != nil {
        return nil, err
    }

    // 将查找到的 Plugins 符号转换为 []iplugin.Plugin 类型
    typePls, ok := pls.(*[]iplugin.Plugin)
    if !ok {
        return nil, errors.New("filed 'Plugins' didn't contain correct type")
    }

    // 返回查找到的插件列表
    return *typePls, nil
}
```