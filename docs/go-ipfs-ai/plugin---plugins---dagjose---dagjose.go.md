# `kubo\plugin\plugins\dagjose\dagjose.go`

```
package dagjose

import (
    "github.com/ipfs/kubo/plugin"  // 导入 kubo 插件包

    "github.com/ceramicnetwork/go-dag-jose/dagjose"  // 导入 go-dag-jose/dagjose 包
    "github.com/ipld/go-ipld-prime/multicodec"  // 导入 go-ipld-prime/multicodec 包
    mc "github.com/multiformats/go-multicodec"  // 导入 multiformats/go-multicodec 包，并重命名为 mc
)

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{  // 定义插件列表
    &dagjosePlugin{},  // 使用 dagjosePlugin 结构体作为插件
}

type dagjosePlugin struct{}  // 定义 dagjosePlugin 结构体

var _ plugin.PluginIPLD = (*dagjosePlugin)(nil)  // 实现 PluginIPLD 接口

func (*dagjosePlugin) Name() string {  // 实现 Name 方法
    return "ipld-codec-dagjose"  // 返回插件名称
}

func (*dagjosePlugin) Version() string {  // 实现 Version 方法
    return "0.0.1"  // 返回插件版本
}

func (*dagjosePlugin) Init(_ *plugin.Environment) error {  // 实现 Init 方法
    return nil  // 初始化方法，返回空错误
}

func (*dagjosePlugin) Register(reg multicodec.Registry) error {  // 实现 Register 方法
    reg.RegisterEncoder(uint64(mc.DagJose), dagjose.Encode)  // 注册 DagJose 编码器
    reg.RegisterDecoder(uint64(mc.DagJose), dagjose.Decode)  // 注册 DagJose 解码器
    return nil  // 返回空错误
}
```