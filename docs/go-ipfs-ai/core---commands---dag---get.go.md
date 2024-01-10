# `kubo\core\commands\dag\get.go`

```
package dagcmd

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于输入输出操作

    "github.com/ipfs/boxo/path"  // 导入路径相关的包
    ipldlegacy "github.com/ipfs/go-ipld-legacy"  // 导入 IPLD 相关的包
    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入命令环境相关的包
    "github.com/ipfs/kubo/core/commands/cmdutils"  // 导入命令工具相关的包

    "github.com/ipld/go-ipld-prime"  // 导入 IPLD Prime 相关的包
    "github.com/ipld/go-ipld-prime/multicodec"  // 导入多编解码相关的包
    "github.com/ipld/go-ipld-prime/traversal"  // 导入遍历相关的包
    mc "github.com/multiformats/go-multicodec"  // 导入多编解码相关的包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 IPFS 命令相关的包
)

func dagGet(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    api, err := cmdenv.GetApi(env, req)  // 获取 API 对象
    if err != nil {
        return err
    }

    codecStr, _ := req.Options["output-codec"].(string)  // 获取输出编解码格式
    var codec mc.Code
    if err := codec.Set(codecStr); err != nil {  // 设置编解码格式
        return err
    }

    p, err := cmdutils.PathOrCidPath(req.Arguments[0])  // 获取路径或 CID 路径
    if err != nil {
        return err
    }

    rp, remainder, err := api.ResolvePath(req.Context, p)  // 解析路径
    if err != nil {
        return err
    }

    obj, err := api.Dag().Get(req.Context, rp.RootCid())  // 获取 DAG 对象
    if err != nil {
        return err
    }

    universal, ok := obj.(ipldlegacy.UniversalNode)  // 转换为 UniversalNode 对象
    if !ok {
        return fmt.Errorf("%T is not a valid IPLD node", obj)  // 返回错误信息
    }

    finalNode := universal.(ipld.Node)  // 转换为 IPLD Node 对象

    if len(remainder) > 0 {  // 如果有剩余路径
        remainderPath := ipld.ParsePath(path.SegmentsToString(remainder...))  // 解析剩余路径
        finalNode, err = traversal.Get(finalNode, remainderPath)  // 获取最终节点
        if err != nil {
            return err
        }
    }

    encoder, err := multicodec.LookupEncoder(uint64(codec))  // 查找编码器
    if err != nil {
        return fmt.Errorf("invalid encoding: %s - %s", codec, err)  // 返回错误信息
    }

    r, w := io.Pipe()  // 创建管道
    go func() {
        defer w.Close()  // 延迟关闭管道
        if err := encoder(finalNode, w); err != nil {  // 使用编码器将节点编码写入管道
            _ = w.CloseWithError(err)  // 关闭管道并返回错误
        }
    }()

    return res.Emit(r)  // 发送编码后的节点
}
```