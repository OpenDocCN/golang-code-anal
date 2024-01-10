# `kubo\client\rpc\pin.go`

```
package rpc

import (
    "context"  // 导入上下文包，用于处理请求的上下文
    "encoding/json"  // 导入 JSON 包，用于处理 JSON 数据
    "io"  // 导入输入输出包，用于处理输入输出流
    "strings"  // 导入字符串包，用于处理字符串操作

    "github.com/ipfs/boxo/path"  // 导入路径包，用于处理路径相关操作
    "github.com/ipfs/go-cid"  // 导入 CID 包，用于处理 CID 相关操作
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入接口包，用于定义接口
    caopts "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包，用于处理选项
    "github.com/pkg/errors"  // 导入错误包，用于处理错误
)

type PinAPI HttpApi  // 定义 PinAPI 结构体，继承自 HttpApi

type pinRefKeyObject struct {
    Type string  // 定义 pinRefKeyObject 结构体，包含 Type 字段
}

type pinRefKeyList struct {
    Keys map[string]pinRefKeyObject  // 定义 pinRefKeyList 结构体，包含 Keys 字段，类型为 map，键为字符串，值为 pinRefKeyObject 结构体
}

type pin struct {
    path path.ImmutablePath  // 定义 pin 结构体，包含 path 字段，类型为不可变路径
    typ  string  // 包含 typ 字段，类型为字符串
    name string  // 包含 name 字段，类型为字符串
    err  error  // 包含 err 字段，类型为错误
}

func (p pin) Err() error {  // 定义 pin 结构体的 Err 方法，返回错误
    return p.err
}

func (p pin) Path() path.ImmutablePath {  // 定义 pin 结构体的 Path 方法，返回不可变路径
    return p.path
}

func (p pin) Name() string {  // 定义 pin 结构体的 Name 方法，返回字符串
    return p.name
}

func (p pin) Type() string {  // 定义 pin 结构体的 Type 方法，返回字符串
    return p.typ
}

func (api *PinAPI) Add(ctx context.Context, p path.Path, opts ...caopts.PinAddOption) error {  // 定义 PinAPI 结构体的 Add 方法，接收上下文、路径和选项，返回错误
    options, err := caopts.PinAddOptions(opts...)  // 获取 PinAddOptions 选项
    if err != nil {  // 如果出现错误
        return err  // 返回错误
    }

    return api.core().Request("pin/add", p.String()).  // 发送 pin/add 请求，传入路径字符串
        Option("recursive", options.Recursive).Exec(ctx, nil)  // 设置递归选项并执行请求
}

type pinLsObject struct {
    Cid  string  // 定义 pinLsObject 结构体，包含 Cid 字段，类型为字符串
    Name string  // 包含 Name 字段，类型为字符串
    Type string  // 包含 Type 字段，类型为字符串
}

func (api *PinAPI) Ls(ctx context.Context, opts ...caopts.PinLsOption) (<-chan iface.Pin, error) {  // 定义 PinAPI 结构体的 Ls 方法，接收上下文和选项，返回通道和错误
    options, err := caopts.PinLsOptions(opts...)  // 获取 PinLsOptions 选项
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空值和错误
    }

    res, err := api.core().Request("pin/ls").  // 发送 pin/ls 请求
        Option("type", options.Type).  // 设置类型选项
        Option("stream", true).  // 设置流选项
        Send(ctx)  // 发送请求
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空值和错误
    }

    pins := make(chan iface.Pin)  // 创建一个通道用于存储 Pin
    # 使用匿名函数处理通道和输出
    go func(ch chan<- iface.Pin) {
        # 延迟关闭输出通道
        defer res.Output.Close()
        # 延迟关闭通道
        defer close(ch)

        # 创建 JSON 解码器
        dec := json.NewDecoder(res.Output)
        # 定义用于存储 pinLsObject 结构的变量
        var out pinLsObject
        # 循环解析 JSON 数据
        for {
            # 解析 JSON 数据并检查错误
            switch err := dec.Decode(&out); err {
            case nil:
                # 无错误，继续循环
            case io.EOF:
                # 文件结束，返回
                return
            default:
                # 其他错误情况
                select {
                case ch <- pin{err: err}:
                    return
                case <-ctx.Done():
                    return
                }
            }

            # 解析 CID 字符串
            c, err := cid.Parse(out.Cid)
            if err != nil {
                # CID 解析错误
                select {
                case ch <- pin{err: err}:
                    return
                case <-ctx.Done():
                    return
                }
            }

            # 发送 pin 结构到通道
            select {
            case ch <- pin{typ: out.Type, name: out.Name, path: path.FromCid(c)}:
            case <-ctx.Done():
                return
            }
        }
    }(pins)
    # 返回 pins 和 nil
    return pins, nil
// IsPinned 返回给定 CID 是否被固定，并解释为什么它被固定。
func (api *PinAPI) IsPinned(ctx context.Context, p path.Path, opts ...caopts.PinIsPinnedOption) (string, bool, error) {
    // 解析选项
    options, err := caopts.PinIsPinnedOptions(opts...)
    if err != nil {
        return "", false, err
    }
    var out pinRefKeyList
    // 发送 pin/ls 请求，获取固定信息
    err = api.core().Request("pin/ls").
        Option("type", options.WithType).
        Option("arg", p.String()).
        Exec(ctx, &out)
    if err != nil {
        // 如果出现错误，检查错误类型
        // TODO: 基于子字符串匹配的错误类型判断是脆弱的，这个问题在这个开放问题中得到了解决：https://github.com/ipfs/go-ipfs/issues/7563
        if strings.Contains(err.Error(), "is not pinned") {
            return "", false, nil
        }
        return "", false, err
    }

    for _, obj := range out.Keys {
        return obj.Type, true, nil
    }
    return "", false, errors.New("http api returned no error and no results")
}

// 从固定中删除给定路径
func (api *PinAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.PinRmOption) error {
    // 解析选项
    options, err := caopts.PinRmOptions(opts...)
    if err != nil {
        return err
    }
    // 发送 pin/rm 请求，删除给定路径的固定
    return api.core().Request("pin/rm", p.String()).
        Option("recursive", options.Recursive).
        Exec(ctx, nil)
}

// 更新给定路径的固定
func (api *PinAPI) Update(ctx context.Context, from path.Path, to path.Path, opts ...caopts.PinUpdateOption) error {
    // 解析选项
    options, err := caopts.PinUpdateOptions(opts...)
    if err != nil {
        return err
    }
    // 发送 pin/update 请求，更新给定路径的固定
    return api.core().Request("pin/update", from.String(), to.String()).
        Option("unpin", options.Unpin).Exec(ctx, nil)
}

// pinVerifyRes 是固定验证的结果
type pinVerifyRes struct {
    ok       bool
    badNodes []iface.BadPinNode
    err      error
}

// Ok 返回验证结果是否成功
func (r pinVerifyRes) Ok() bool {
    return r.ok
}

// BadNodes 返回错误的固定节点列表
func (r pinVerifyRes) BadNodes() []iface.BadPinNode {
    return r.badNodes
}

// Err 返回验证过程中的错误
func (r pinVerifyRes) Err() error {
    return r.err
}

// badNode 是一个错误的固定节点
type badNode struct {
    err error
    cid cid.Cid
}
# 返回节点的路径
func (n badNode) Path() path.ImmutablePath {
    return path.FromCid(n.cid)
}

# 返回节点的错误信息
func (n badNode) Err() error {
    return n.err
}

# 验证固定节点的状态
func (api *PinAPI) Verify(ctx context.Context) (<-chan iface.PinStatus, error) {
    # 发送验证固定节点的请求，并获取响应
    resp, err := api.core().Request("pin/verify").Option("verbose", true).Send(ctx)
    # 如果请求出错，返回错误信息
    if err != nil {
        return nil, err
    }
    # 如果响应中包含错误信息，返回错误信息
    if resp.Error != nil {
        return nil, resp.Error
    }
    # 创建一个通道用于存储固定节点的状态
    res := make(chan iface.PinStatus)
    # 创建一个匿名的 goroutine 函数
    go func() {
        # 延迟关闭 resp
        defer resp.Close()
        # 延迟关闭 res
        defer close(res)
        # 创建一个 JSON 解码器，使用 resp.Output 作为输入
        dec := json.NewDecoder(resp.Output)
        # 无限循环，解析 JSON 数据
        for {
            # 定义一个结构体 out，用于存储解析后的数据
            var out struct {
                Cid string
                Err string
                Ok  bool
                BadNodes []struct {
                    Cid string
                    Err string
                }
            }
            # 解码 JSON 数据到 out 结构体
            if err := dec.Decode(&out); err != nil {
                # 如果解码出错
                if err == io.EOF {
                    # 如果是文件结束错误，退出循环
                    return
                }
                # 如果不是文件结束错误，根据情况发送错误信息到 res 或者退出循环
                select {
                case res <- pinVerifyRes{err: err}:
                    return
                case <-ctx.Done():
                    return
                }
            }

            # 如果 out 中包含错误信息，根据情况发送错误信息到 res 或者退出循环
            if out.Err != "" {
                select {
                case res <- pinVerifyRes{err: errors.New(out.Err)}:
                    return
                case <-ctx.Done():
                    return
                }
            }

            # 创建一个长度为 out.BadNodes 长度的 iface.BadPinNode 切片
            badNodes := make([]iface.BadPinNode, len(out.BadNodes))
            # 遍历 out.BadNodes，解析其中的数据并存储到 badNodes 中
            for i, n := range out.BadNodes {
                c, err := cid.Decode(n.Cid)
                if err != nil {
                    badNodes[i] = badNode{cid: c, err: err}
                    continue
                }

                if n.Err != "" {
                    err = errors.New(n.Err)
                }
                badNodes[i] = badNode{cid: c, err: err}
            }

            # 根据情况发送验证结果到 res 或者退出循环
            select {
            case res <- pinVerifyRes{ok: out.Ok, badNodes: badNodes}:
            case <-ctx.Done():
                return
            }
        }
    }()

    # 返回结果 res 和空错误
    return res, nil
# 定义一个方法 core()，该方法属于 PinAPI 结构体
func (api *PinAPI) core() *HttpApi:
    # 返回一个指向 HttpApi 类型的指针，该指针指向 api 对象
    return (*HttpApi)(api)
```