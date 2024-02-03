# `kubo\core\commands\sysdiag.go`

```go
package commands

import (
    "os"  // 导入操作系统相关的包
    "path"  // 导入路径相关的包
    "runtime"  // 导入运行时相关的包

    version "github.com/ipfs/kubo"  // 导入版本相关的包
    "github.com/ipfs/kubo/core"  // 导入核心功能相关的包
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入命令环境相关的包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令相关的包
    manet "github.com/multiformats/go-multiaddr/net"  // 导入网络地址相关的包
    sysi "github.com/whyrusleeping/go-sysinfo"  // 导入系统信息相关的包
)

var sysDiagCmd = &cmds.Command{  // 定义系统诊断命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Print system diagnostic information.",  // 简短描述
        ShortDescription: `  // 简要描述
Prints out information about your computer to aid in easier debugging.
`,
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  // 运行函数
        nd, err := cmdenv.GetNode(env)  // 获取节点信息
        if err != nil {
            return err
        }

        info, err := getInfo(nd)  // 获取信息
        if err != nil {
            return err
        }
        return cmds.EmitOnce(res, info)  // 发送信息
    },
}

func getInfo(nd *core.IpfsNode) (map[string]interface{}, error) {  // 获取信息函数
    info := make(map[string]interface{})  // 创建信息映射
    err := runtimeInfo(info)  // 运行时信息
    if err != nil {
        return nil, err
    }

    err = envVarInfo(info)  // 环境变量信息
    if err != nil {
        return nil, err
    }

    err = diskSpaceInfo(info)  // 磁盘空间信息
    if err != nil {
        return nil, err
    }

    err = memInfo(info)  // 内存信息
    if err != nil {
        return nil, err
    }

    err = netInfo(nd.IsOnline, info)  // 网络信息
    if err != nil {
        return nil, err
    }

    info["ipfs_version"] = version.CurrentVersionNumber  // IPFS版本号
    info["ipfs_commit"] = version.CurrentCommit  // IPFS提交信息
    return info, nil
}

func runtimeInfo(out map[string]interface{}) error {  // 运行时信息函数
    rt := make(map[string]interface{})  // 创建运行时信息映射
    rt["os"] = runtime.GOOS  // 操作系统
    rt["arch"] = runtime.GOARCH  // 架构
    rt["compiler"] = runtime.Compiler  // 编译器
    rt["version"] = runtime.Version()  // 版本
    rt["numcpu"] = runtime.NumCPU()  // CPU数量
    rt["gomaxprocs"] = runtime.GOMAXPROCS(0)  // 最大处理器数量
    rt["numgoroutines"] = runtime.NumGoroutine()  // Goroutine数量

    out["runtime"] = rt  // 运行时信息存入输出映射
    return nil
}

func envVarInfo(out map[string]interface{}) error {  // 环境变量信息函数
    # 创建一个空的字符串到接口类型的映射
    ev := make(map[string]interface{})
    # 将环境变量GOPATH的值存储到映射ev中的键"GOPATH"中
    ev["GOPATH"] = os.Getenv("GOPATH")
    # 将环境变量IPFS_PATH的值存储到映射ev中的键"IPFS_PATH"中
    ev["IPFS_PATH"] = os.Getenv("IPFS_PATH")
    
    # 将映射ev存储到输出映射out中的键"environment"中
    out["environment"] = ev
    # 返回空值
    return nil
// 获取 IPFS 存储路径，如果环境变量中没有设置，则使用默认路径
func ipfsPath() string {
    p := os.Getenv("IPFS_PATH")
    if p == "" {
        p = path.Join(os.Getenv("HOME"), ".ipfs")
    }
    return p
}

// 获取磁盘空间信息并存入指定的 map 中
func diskSpaceInfo(out map[string]interface{}) error {
    di := make(map[string]interface{})
    // 获取指定路径的磁盘使用情况
    dinfo, err := sysi.DiskUsage(ipfsPath())
    if err != nil {
        return err
    }

    di["fstype"] = dinfo.FsType
    di["total_space"] = dinfo.Total
    di["free_space"] = dinfo.Free

    out["diskinfo"] = di
    return nil
}

// 获取内存信息并存入指定的 map 中
func memInfo(out map[string]interface{}) error {
    m := make(map[string]interface{})

    // 获取内存使用情况
    meminf, err := sysi.MemoryInfo()
    if err != nil {
        return err
    }

    m["swap"] = meminf.Swap
    m["virt"] = meminf.Used
    out["memory"] = m
    return nil
}

// 获取网络信息并存入指定的 map 中
func netInfo(online bool, out map[string]interface{}) error {
    n := make(map[string]interface{})
    // 获取网络接口的多地址
    addrs, err := manet.InterfaceMultiaddrs()
    if err != nil {
        return err
    }

    straddrs := make([]string, len(addrs))
    for i, a := range addrs {
        straddrs[i] = a.String()
    }

    n["interface_addresses"] = straddrs
    n["online"] = online
    out["net"] = n
    return nil
}
```