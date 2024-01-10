# `kubo\plugin\ipld.go`

```
// 导入 plugin 包
package plugin

// 导入 multicodec 包，并重命名为 multicodec
import (
    multicodec "github.com/ipld/go-ipld-prime/multicodec"
)

// PluginIPLD 是一个接口，可以实现不同的 IPLD 编解码器的处理程序
type PluginIPLD interface {
    // 继承 Plugin 接口
    Plugin

    // 注册 IPLD 编解码器到 multicodec 注册表中，并返回错误信息
    Register(multicodec.Registry) error
}
```