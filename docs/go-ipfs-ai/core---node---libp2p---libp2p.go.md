# `kubo\core\node\libp2p\libp2p.go`

```go
package libp2p

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "sort"  // 导入 sort 包，用于对数据进行排序
    "time"  // 导入 time 包，用于处理时间

    version "github.com/ipfs/kubo"  // 导入版本信息
    config "github.com/ipfs/kubo/config"  // 导入配置信息

    logging "github.com/ipfs/go-log"  // 导入日志记录包
    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/core/crypto"  // 导入 libp2p 加密包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 对等节点包
    "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 libp2p 对等节点存储包
    "github.com/libp2p/go-libp2p/p2p/net/connmgr"  // 导入 libp2p 网络连接管理包
    "go.uber.org/fx"  // 导入 fx 包
)

var log = logging.Logger("p2pnode")  // 定义日志记录器

type Libp2pOpts struct {
    fx.Out  // 使用 fx.Out 标记为输出参数

    Opts []libp2p.Option `group:"libp2p"`  // 定义 libp2p 选项切片
}

func ConnectionManager(low, high int, grace time.Duration) func() (opts Libp2pOpts, err error) {
    return func() (opts Libp2pOpts, err error) {
        cm, err := connmgr.NewConnManager(low, high, connmgr.WithGracePeriod(grace))  // 创建连接管理器
        if err != nil {
            return opts, err
        }
        opts.Opts = append(opts.Opts, libp2p.ConnectionManager(cm))  // 将连接管理器选项添加到 opts 中
        return
    }
}

func PstoreAddSelfKeys(id peer.ID, sk crypto.PrivKey, ps peerstore.Peerstore) error {
    if err := ps.AddPubKey(id, sk.GetPublic()); err != nil {  // 如果添加公钥出错
        return err
    }

    return ps.AddPrivKey(id, sk)  // 添加私钥
}

func UserAgent() func() (opts Libp2pOpts, err error) {
    return simpleOpt(libp2p.UserAgent(version.GetUserAgentVersion()))  // 返回用户代理选项
}

func simpleOpt(opt libp2p.Option) func() (opts Libp2pOpts, err error) {
    return func() (opts Libp2pOpts, err error) {
        opts.Opts = append(opts.Opts, opt)  // 将选项添加到 opts 中
        return
    }
}

type priorityOption struct {
    priority, defaultPriority config.Priority  // 定义优先级选项
    opt                       libp2p.Option  // 定义 libp2p 选项
}

func prioritizeOptions(opts []priorityOption) libp2p.Option {
    type popt struct {
        priority int64 // lower priority values mean higher priority  // 优先级值越低，优先级越高
        opt      libp2p.Option  // 定义 libp2p 选项
    }
    enabledOptions := make([]popt, 0, len(opts))  // 创建一个空的优先级选项切片
    # 遍历选项列表
    for _, o := range opts:
        # 获取优先级，如果不存在则使用默认优先级
        if prio, ok := o.priority.WithDefault(o.defaultPriority); ok {
            # 将具有优先级的选项添加到已启用选项列表中
            enabledOptions = append(enabledOptions, popt{
                priority: prio,
                opt:      o.opt,
            })
        }
    # 根据优先级对已启用选项列表进行排序
    sort.Slice(enabledOptions, func(i, j int) bool {
        return enabledOptions[i].priority < enabledOptions[j].priority
    })
    # 创建 libp2p 选项列表
    p2pOpts := make([]libp2p.Option, len(enabledOptions))
    # 将已启用选项列表中的选项添加到 libp2p 选项列表中
    for i, opt := range enabledOptions {
        p2pOpts[i] = opt.opt
    }
    # 返回 libp2p 选项链
    return libp2p.ChainOptions(p2pOpts...)
# 定义一个函数，接受一个指向可选字符串的指针，并返回一个函数，该函数返回Libp2pOpts和错误
func ForceReachability(val *config.OptionalString) func() (opts Libp2pOpts, err error) {
    # 返回一个函数，该函数返回Libp2pOpts和错误
    return func() (opts Libp2pOpts, err error) {
        # 如果val的值是默认值，则返回空
        if val.IsDefault() {
            return
        }
        # 使用默认值创建一个新的可选字符串
        v := val.WithDefault("unrecognized")
        # 根据新的可选字符串的值进行判断
        switch v {
        # 如果值为"public"，则将ForceReachabilityPublic()选项添加到opts中
        case "public":
            opts.Opts = append(opts.Opts, libp2p.ForceReachabilityPublic())
        # 如果值为"private"，则将ForceReachabilityPrivate()选项添加到opts中
        case "private":
            opts.Opts = append(opts.Opts, libp2p.ForceReachabilityPrivate())
        # 如果值为其他值，则返回错误信息
        default:
            return opts, fmt.Errorf("unrecognized reachability option: %s", v)
        }
        # 返回opts和空错误
        return
    }
}
```