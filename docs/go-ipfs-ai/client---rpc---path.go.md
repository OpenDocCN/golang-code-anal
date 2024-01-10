# `kubo\client\rpc\path.go`

```
package rpc

import (
    "context"  // 导入上下文包

    "github.com/ipfs/boxo/path"  // 导入路径包
    cid "github.com/ipfs/go-cid"  // 导入CID包
    ipld "github.com/ipfs/go-ipld-format"  // 导入IPLD包
)

func (api *HttpApi) ResolvePath(ctx context.Context, p path.Path) (path.ImmutablePath, []string, error) {
    var out struct {  // 定义结构体变量out
        Cid     cid.Cid  // CID类型的Cid字段
        RemPath string  // 字符串类型的RemPath字段
    }

    var err error  // 定义错误变量err
    if p.Namespace() == path.IPNSNamespace {  // 如果路径的命名空间是IPNS
        if p, err = api.Name().Resolve(ctx, p.String()); err != nil {  // 解析路径
            return path.ImmutablePath{}, nil, err  // 返回空的不可变路径、空数组和错误
        }
    }

    if err := api.Request("dag/resolve", p.String()).Exec(ctx, &out); err != nil {  // 执行请求
        return path.ImmutablePath{}, nil, err  // 返回空的不可变路径、空数组和错误
    }

    p, err = path.NewPathFromSegments(p.Namespace(), out.Cid.String(), out.RemPath)  // 从段创建新路径
    if err != nil {
        return path.ImmutablePath{}, nil, err  // 返回空的不可变路径、空数组和错误
    }

    imPath, err := path.NewImmutablePath(p)  // 创建不可变路径
    if err != nil {
        return path.ImmutablePath{}, nil, err  // 返回空的不可变路径、空数组和错误
    }

    return imPath, path.StringToSegments(out.RemPath), nil  // 返回不可变路径、RemPath的段数组和空错误
}

func (api *HttpApi) ResolveNode(ctx context.Context, p path.Path) (ipld.Node, error) {
    rp, _, err := api.ResolvePath(ctx, p)  // 解析路径
    if err != nil {
        return nil, err  // 返回空和错误
    }

    return api.Dag().Get(ctx, rp.RootCid())  // 返回DAG中根CID对应的节点
}
```