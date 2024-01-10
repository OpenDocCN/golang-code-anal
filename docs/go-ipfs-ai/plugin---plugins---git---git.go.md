# `kubo\plugin\plugins\git\git.go`

```
// 导入必要的包
package git

import (
    "compress/zlib"  // 导入压缩/解压缩包
    "io"  // 导入输入/输出包

    "github.com/ipfs/kubo/plugin"  // 导入插件包

    // 注意，依赖于此包会注册其多编解码器
    git "github.com/ipfs/go-ipld-git"  // 导入 git 包
    "github.com/ipld/go-ipld-prime"  // 导入 ipld-prime 包
    "github.com/ipld/go-ipld-prime/multicodec"  // 导入 multicodec 包
    mc "github.com/multiformats/go-multicodec"  // 导入 multicodec 包
)

// 导出的插件列表，将被加载
var Plugins = []plugin.Plugin{
    &gitPlugin{},  // git 插件
}

type gitPlugin struct{}  // git 插件结构体

var _ plugin.PluginIPLD = (*gitPlugin)(nil)  // 实现 PluginIPLD 接口

func (*gitPlugin) Name() string {
    return "ipld-git"  // 返回插件名称
}

func (*gitPlugin) Version() string {
    return "0.0.1"  // 返回插件版本
}

func (*gitPlugin) Init(_ *plugin.Environment) error {
    return nil  // 初始化插件
}

func (*gitPlugin) Register(reg multicodec.Registry) error {
    // 在保留范围内注册自定义标识符以导入“zlib编码的git对象”
    reg.RegisterDecoder(uint64(mc.ReservedStart+mc.GitRaw), decodeZlibGit)  // 注册解码器
    reg.RegisterEncoder(uint64(mc.GitRaw), git.Encode)  // 注册编码器
    reg.RegisterDecoder(uint64(mc.GitRaw), git.Decode)  // 注册解码器
    return nil  // 返回空
}

func decodeZlibGit(na ipld.NodeAssembler, r io.Reader) error {
    rc, err := zlib.NewReader(r)  // 创建 zlib 读取器
    if err != nil {
        return err  // 如果出错，返回错误
    }

    defer rc.Close()  // 延迟关闭读取器

    return git.Decode(na, rc)  // 解码 git 对象
}
```