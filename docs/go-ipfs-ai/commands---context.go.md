# `kubo\commands\context.go`

```
package commands

import (
    "context"  // 导入 context 包
    "errors"   // 导入 errors 包
    "strings"  // 导入 strings 包
    "time"     // 导入 time 包

    core "github.com/ipfs/kubo/core"  // 导入 core 包，并重命名为 core
    coreapi "github.com/ipfs/kubo/core/coreapi"  // 导入 coreapi 包，并重命名为 coreapi
    loader "github.com/ipfs/kubo/plugin/loader"  // 导入 loader 包，并重命名为 loader

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 cmds 包，并重命名为 cmds
    logging "github.com/ipfs/go-log"    // 导入 logging 包，并重命名为 logging
    config "github.com/ipfs/kubo/config"  // 导入 config 包，并重命名为 config
    coreiface "github.com/ipfs/kubo/core/coreiface"  // 导入 coreiface 包，并重命名为 coreiface
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入 options 包，并重命名为 options
)

var log = logging.Logger("command")  // 定义全局变量 log，使用 logging 包创建 Logger

// Context represents request context.
type Context struct {
    ConfigRoot string  // 请求上下文的配置根目录
    ReqLog     *ReqLog  // 请求日志

    Plugins *loader.PluginLoader  // 插件加载器

    Gateway       bool  // 网关标志
    api           coreiface.CoreAPI  // CoreAPI 实例
    node          *core.IpfsNode  // IpfsNode 实例
    ConstructNode func() (*core.IpfsNode, error)  // 构造 IpfsNode 的函数
}

func (c *Context) GetConfig() (*config.Config, error) {
    node, err := c.GetNode()  // 获取当前命令执行上下文的节点
    if err != nil {
        return nil, err
    }
    return node.Repo.Config()  // 返回节点的配置
}

// GetNode returns the node of the current Command execution
// context. It may construct it with the provided function.
func (c *Context) GetNode() (*core.IpfsNode, error) {
    var err error
    if c.node == nil {  // 如果节点为空
        if c.ConstructNode == nil {  // 如果构造节点的函数为空
            return nil, errors.New("nil ConstructNode function")  // 返回错误
        }
        c.node, err = c.ConstructNode()  // 构造节点
    }
    return c.node, err  // 返回节点和错误
}

// GetAPI returns CoreAPI instance backed by ipfs node.
// It may construct the node with the provided function.
func (c *Context) GetAPI() (coreiface.CoreAPI, error) {
    if c.api == nil {  // 如果 CoreAPI 实例为空
        n, err := c.GetNode()  // 获取当前节点
        if err != nil {
            return nil, err
        }
        fetchBlocks := true  // 设置 fetchBlocks 为 true
        if c.Gateway {  // 如果是网关
            cfg, err := c.GetConfig()  // 获取配置
            if err != nil {
                return nil, err
            }
            fetchBlocks = !cfg.Gateway.NoFetch  // 根据配置设置 fetchBlocks
        }

        c.api, err = coreapi.NewCoreAPI(n, options.Api.FetchBlocks(fetchBlocks))  // 创建 CoreAPI 实例
        if err != nil {
            return nil, err
        }
    }
    # 返回变量 c 的 api 属性和空值
    return c.api, nil
// Context 返回节点的上下文。
func (c *Context) Context() context.Context {
    // 获取节点
    n, err := c.GetNode()
    // 如果出现错误，记录错误信息并返回默认上下文
    if err != nil {
        log.Debug("error getting node: ", err)
        return context.Background()
    }
    // 返回节点的上下文
    return n.Context()
}

// LogRequest 将传入的请求添加到请求日志中，并返回一个在请求生命周期结束时应调用的函数。
func (c *Context) LogRequest(req *cmds.Request) func() {
    // 创建请求日志条目
    rle := &ReqLogEntry{
        StartTime: time.Now(),
        Active:    true,
        Command:   strings.Join(req.Path, "/"),
        Options:   req.Options,
        Args:      req.Arguments,
        log:       c.ReqLog,
    }
    // 将请求日志条目添加到请求日志中
    c.ReqLog.AddEntry(rle)

    // 返回一个函数，用于结束请求日志条目
    return func() {
        c.ReqLog.Finish(rle)
    }
}

// Close 清理应用程序状态。
func (c *Context) Close() {
    // 让我们不要忘记清理。如果节点已初始化，我们必须关闭它。
    // 请注意，这意味着底层的 req.Context().Node 变量被暴露出来。
    // 这很糟糕，当我们提取出执行上下文时应该进行更改。
    if c.node != nil {
        log.Info("Shutting down node...")
        c.node.Close()
    }
}
```