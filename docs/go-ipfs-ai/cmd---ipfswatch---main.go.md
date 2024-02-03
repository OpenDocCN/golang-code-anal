# `kubo\cmd\ipfswatch\main.go`

```go
//go:build !plan9
// +build !plan9

package main

import (
    "context" // 导入上下文包，用于控制程序的执行流程
    "flag" // 导入 flag 包，用于解析命令行参数
    "log" // 导入 log 包，用于打印日志信息
    "os" // 导入 os 包，提供对操作系统功能的访问
    "os/signal" // 导入 os/signal 包，用于处理信号
    "path/filepath" // 导入 path/filepath 包，用于处理文件路径
    "syscall" // 导入 syscall 包，提供对底层操作系统的访问

    commands "github.com/ipfs/kubo/commands" // 导入自定义的命令包
    core "github.com/ipfs/kubo/core" // 导入自定义的核心包
    coreapi "github.com/ipfs/kubo/core/coreapi" // 导入自定义的核心 API 包
    corehttp "github.com/ipfs/kubo/core/corehttp" // 导入自定义的核心 HTTP 包
    fsrepo "github.com/ipfs/kubo/repo/fsrepo" // 导入自定义的文件系统仓库包

    fsnotify "github.com/fsnotify/fsnotify" // 导入文件系统监视包
    "github.com/ipfs/boxo/files" // 导入文件操作包
    process "github.com/jbenet/goprocess" // 导入进程管理包
    homedir "github.com/mitchellh/go-homedir" // 导入用户主目录包
)

var (
    http      = flag.Bool("http", false, "expose IPFS HTTP API") // 定义一个布尔类型的命令行参数，用于指定是否暴露 IPFS HTTP API
    repoPath  = flag.String("repo", os.Getenv("IPFS_PATH"), "IPFS_PATH to use") // 定义一个字符串类型的命令行参数，用于指定 IPFS_PATH
    watchPath = flag.String("path", ".", "the path to watch") // 定义一个字符串类型的命令行参数，用于指定要监视的路径
)

func main() {
    flag.Parse() // 解析命令行参数

    // precedence
    // 1. --repo flag
    // 2. IPFS_PATH environment variable
    // 3. default repo path
    var ipfsPath string // 定义一个字符串变量，用于存储 IPFS 路径
    if *repoPath != "" { // 如果命令行参数中指定了 --repo
        ipfsPath = *repoPath // 使用命令行参数中指定的 IPFS 路径
    } else {
        var err error
        ipfsPath, err = fsrepo.BestKnownPath() // 获取最佳已知路径
        if err != nil {
            log.Fatal(err) // 如果出错，打印错误信息并退出程序
        }
    }

    if err := run(ipfsPath, *watchPath); err != nil { // 调用 run 函数，并传入 IPFS 路径和监视路径
        log.Fatal(err) // 如果出错，打印错误信息并退出程序
    }
}

func run(ipfsPath, watchPath string) error {
    proc := process.WithParent(process.Background()) // 创建一个带有父进程的进程
    log.Printf("running IPFSWatch on '%s' using repo at '%s'...", watchPath, ipfsPath) // 打印日志信息，指示正在运行 IPFSWatch，并指定监视路径和 IPFS 路径

    ipfsPath, err := homedir.Expand(ipfsPath) // 将 IPFS 路径扩展为绝对路径
    if err != nil {
        return err // 如果出错，返回错误信息
    }
    watcher, err := fsnotify.NewWatcher() // 创建一个文件系统监视器
    if err != nil {
        return err // 如果出错，返回错误信息
    }
    defer watcher.Close() // 在函数返回前关闭监视器

    if err := addTree(watcher, watchPath); err != nil { // 调用 addTree 函数，将监视路径添加到监视器中
        return err // 如果出错，返回错误信息
    }

    r, err := fsrepo.Open(ipfsPath) // 打开 IPFS 仓库
    if err != nil {
        // TODO handle case: daemon running
        // TODO handle case: repo doesn't exist or isn't initialized
        return err // 如果出错，返回错误信息
    }

    node, err := core.NewNode(context.Background(), &core.BuildCfg{ // 创建一个新的 IPFS 节点
        Online: true, // 在线模式
        Repo:   r, // 使用指定的仓库
    }) 
    // 如果发生错误，返回错误信息
    if err != nil {
        return err
    }
    // 延迟关闭节点
    defer node.Close()

    // 创建新的核心 API
    api, err := coreapi.NewCoreAPI(node)
    // 如果发生错误，返回错误信息
    if err != nil {
        return err
    }

    // 如果启用 HTTP
    if *http {
        // 设置地址
        addr := "/ip4/127.0.0.1/tcp/5001"
        // 设置选项
        opts := []corehttp.ServeOption{
            corehttp.GatewayOption("/ipfs", "/ipns"),
            corehttp.WebUIOption,
            corehttp.CommandsOption(cmdCtx(node, ipfsPath)),
        }
        // 启动 HTTP 服务
        proc.Go(func(p process.Process) {
            if err := corehttp.ListenAndServe(node, addr, opts...); err != nil {
                return
            }
        })
    }

    // 创建中断信号通道
    interrupts := make(chan os.Signal, 1)
    // 监听中断信号
    signal.Notify(interrupts, os.Interrupt, syscall.SIGTERM)
    // 无限循环，监听多个 channel 的事件
    for {
        // 选择监听的事件
        select {
        // 如果接收到中断信号，则返回空值
        case <-interrupts:
            return nil
        // 如果接收到文件系统事件
        case e := <-watcher.Events:
            // 打印接收到的事件
            log.Printf("received event: %s", e)
            // 判断是否为目录
            isDir, err := IsDirectory(e.Name)
            // 如果出现错误，则继续下一个事件
            if err != nil {
                continue
            }
            // 根据事件类型进行处理
            switch e.Op {
            // 如果是文件删除事件
            case fsnotify.Remove:
                // 如果是目录，则移除监听
                if isDir {
                    if err := watcher.Remove(e.Name); err != nil {
                        return err
                    }
                }
            // 其他事件
            default:
                // 除了删除事件外，其他事件都会触发 IPFS.Add 操作，但只有目录创建会触发新的监听
                switch e.Op {
                // 如果是文件创建事件
                case fsnotify.Create:
                    // 如果是目录，则添加监听
                    if isDir {
                        if err := addTree(watcher, e.Name); err != nil {
                            return err
                        }
                    }
                }
                // 异步处理事件
                proc.Go(func(p process.Process) {
                    // 打开文件
                    file, err := os.Open(e.Name)
                    // 如果出现错误，则打印错误信息并返回
                    if err != nil {
                        log.Println(err)
                        return
                    }
                    // 延迟关闭文件
                    defer file.Close()

                    // 获取文件信息
                    st, err := file.Stat()
                    // 如果出现错误，则打印错误信息并返回
                    if err != nil {
                        log.Println(err)
                        return
                    }

                    // 创建文件读取器
                    f, err := files.NewReaderPathFile(e.Name, file, st)
                    // 如果出现错误，则打印错误信息并返回
                    if err != nil {
                        log.Println(err)
                        return
                    }

                    // 将文件添加到 IPFS
                    k, err := api.Unixfs().Add(node.Context(), f)
                    // 如果出现错误，则打印错误信息
                    if err != nil {
                        log.Println(err)
                    }
                    // 打印添加成功的信息
                    log.Printf("added %s... key: %s", e.Name, k)
                })
            }
        // 如果接收到监听错误
        case err := <-watcher.Errors:
            // 打印错误信息
            log.Println(err)
        }
    }
# 添加监视器以监视指定根目录下的文件变化
func addTree(w *fsnotify.Watcher, root string) error {
    # 遍历指定根目录下的所有文件和子目录
    err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        # 如果出现错误，记录错误信息并返回
        if err != nil {
            log.Println(err)
            return nil
        }
        # 检查当前路径是否为目录
        isDir, err := IsDirectory(path)
        if err != nil {
            log.Println(err)
            return nil
        }
        # 根据不同情况进行处理
        switch {
        # 如果是隐藏目录，则记录并跳过
        case isDir && IsHidden(path):
            log.Println(path)
            return filepath.SkipDir
        # 如果是普通目录，则记录并添加到监视器中
        case isDir:
            log.Println(path)
            if err := w.Add(path); err != nil {
                return err
            }
        # 如果是文件，则不做处理
        default:
            return nil
        }
        return nil
    })
    return err
}

# 判断指定路径是否为目录
func IsDirectory(path string) (bool, error) {
    # 获取路径的文件信息
    fileInfo, err := os.Stat(path)
    # 返回是否为目录和可能出现的错误
    return fileInfo.IsDir(), err
}

# 判断指定路径是否为隐藏文件或目录
func IsHidden(path string) bool {
    # 获取路径的基本名称
    path = filepath.Base(path)
    # 如果路径为当前目录或空字符串，则不是隐藏文件
    if path == "." || path == "" {
        return false
    }
    # 如果路径以"."开头，则是隐藏文件
    if rune(path[0]) == rune('.') {
        return true
    }
    return false
}

# 创建命令上下文
func cmdCtx(node *core.IpfsNode, repoPath string) commands.Context {
    # 返回命令上下文对象
    return commands.Context{
        ConfigRoot: repoPath,
        ConstructNode: func() (*core.IpfsNode, error) {
            return node, nil
        },
    }
}
```