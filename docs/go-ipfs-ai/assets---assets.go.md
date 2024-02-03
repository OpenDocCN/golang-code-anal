# `kubo\assets\assets.go`

```go
package assets

import (
    "embed"  // 导入 embed 包，用于嵌入静态资源
    "fmt"  // 导入 fmt 包，用于格式化输出
    gopath "path"  // 导入 path 包，并重命名为 gopath

    "github.com/ipfs/kubo/core"  // 导入 core 包
    "github.com/ipfs/kubo/core/coreapi"  // 导入 coreapi 包

    "github.com/ipfs/boxo/files"  // 导入 files 包
    "github.com/ipfs/boxo/path"  // 导入 path 包
    cid "github.com/ipfs/go-cid"  // 导入 cid 包，并重命名为 cid
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包
)

//go:embed init-doc
var Asset embed.FS  // 声明一个 embed.FS 类型的变量 Asset，用于嵌入静态资源

// initDocPaths lists the paths for the docs we want to seed during --init.
var initDocPaths = []string{  // 声明一个字符串数组，包含要在 --init 期间种子化的文档路径
    gopath.Join("init-doc", "about"),  // 拼接路径
    gopath.Join("init-doc", "readme"),
    gopath.Join("init-doc", "help"),
    gopath.Join("init-doc", "contact"),
    gopath.Join("init-doc", "security-notes"),
    gopath.Join("init-doc", "quick-start"),
    gopath.Join("init-doc", "ping"),
}

// SeedInitDocs adds the list of embedded init documentation to the passed node, pins it and returns the root key.
func SeedInitDocs(nd *core.IpfsNode) (cid.Cid, error) {  // 定义一个函数，将嵌入的初始化文档列表添加到传递的节点中，对其进行固定，并返回根密钥
    return addAssetList(nd, initDocPaths)  // 调用 addAssetList 函数，并返回结果
}

func addAssetList(nd *core.IpfsNode, l []string) (cid.Cid, error) {  // 定义一个函数，接收一个节点和一个字符串数组作为参数
    api, err := coreapi.NewCoreAPI(nd)  // 调用 NewCoreAPI 方法创建一个 CoreAPI 对象
    if err != nil {  // 如果出现错误
        return cid.Cid{}, err  // 返回空的 cid.Cid 对象和错误
    }

    dirb, err := api.Object().New(nd.Context(), options.Object.Type("unixfs-dir"))  // 调用 Object().New 方法创建一个新的 unixfs-dir 类型的对象
    if err != nil {  // 如果出现错误
        return cid.Cid{}, err  // 返回空的 cid.Cid 对象和错误
    }

    basePath := path.FromCid(dirb.Cid())  // 从 Cid 创建一个路径

    for _, p := range l {  // 遍历字符串数组
        d, err := Asset.ReadFile(p)  // 读取嵌入的文件内容
        if err != nil {  // 如果出现错误
            return cid.Cid{}, fmt.Errorf("assets: could load Asset '%s': %s", p, err)  // 返回错误信息
        }

        fp, err := api.Unixfs().Add(nd.Context(), files.NewBytesFile(d))  // 调用 Unixfs().Add 方法将文件添加到 IPFS
        if err != nil {  // 如果出现错误
            return cid.Cid{}, err  // 返回空的 cid.Cid 对象和错误
        }

        fname := gopath.Base(p)  // 获取文件名

        basePath, err = api.Object().AddLink(nd.Context(), basePath, fname, fp)  // 调用 Object().AddLink 方法添加链接
        if err != nil {  // 如果出现错误
            return cid.Cid{}, err  // 返回空的 cid.Cid 对象和错误
        }
    }

    if err := api.Pin().Add(nd.Context(), basePath); err != nil {  // 调用 Pin().Add 方法对路径进行固定
        return cid.Cid{}, err  // 返回空的 cid.Cid 对象和错误
    }

    return basePath.RootCid(), nil  // 返回根密钥和空的错误
}
```