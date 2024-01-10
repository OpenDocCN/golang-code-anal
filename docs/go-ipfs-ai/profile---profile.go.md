# `kubo\profile\profile.go`

```
package profile

import (
    "archive/zip"  // 导入压缩包处理的包
    "bytes"  // 导入字节操作的包
    "context"  // 导入上下文包
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "os"  // 导入操作系统功能包
    "runtime"  // 导入运行时包
    "runtime/pprof"  // 导入性能分析包
    "sync"  // 导入同步包
    "time"  // 导入时间包

    "github.com/ipfs/go-log"  // 导入日志包
    version "github.com/ipfs/kubo"  // 导入版本包
)

const (
    CollectorGoroutinesStack = "goroutines-stack"  // 定义常量 CollectorGoroutinesStack
    CollectorGoroutinesPprof = "goroutines-pprof"  // 定义常量 CollectorGoroutinesPprof
    CollectorVersion         = "version"  // 定义常量 CollectorVersion
    CollectorHeap            = "heap"  // 定义常量 CollectorHeap
    CollectorAllocs          = "allocs"  // 定义常量 CollectorAllocs
    CollectorBin             = "bin"  // 定义常量 CollectorBin
    CollectorCPU             = "cpu"  // 定义常量 CollectorCPU
    CollectorMutex           = "mutex"  // 定义常量 CollectorMutex
    CollectorBlock           = "block"  // 定义常量 CollectorBlock
)

var (
    logger = log.Logger("profile")  // 创建日志记录器
    goos   = runtime.GOOS  // 获取操作系统信息
)

type collector struct {
    outputFile   string  // 定义输出文件名
    isExecutable bool  // 定义是否为可执行文件
    collectFunc  func(ctx context.Context, opts Options, writer io.Writer) error  // 定义收集函数
    enabledFunc  func(opts Options) bool  // 定义启用函数
}

func (p *collector) outputFileName() string {
    fName := p.outputFile  // 获取输出文件名
    if p.isExecutable {  // 如果是可执行文件
        if goos == "windows" {  // 如果是 Windows 系统
            fName += ".exe"  // 在文件名后添加 .exe 扩展名
        }
    }
    return fName  // 返回文件名
}

var collectors = map[string]collector{  // 定义收集器映射
    CollectorGoroutinesStack: {  // 收集器名称
        outputFile:  "goroutines.stacks",  // 输出文件名
        collectFunc: goroutineStacksText,  // 收集函数
        enabledFunc: func(opts Options) bool { return true },  // 启用函数
    },
    CollectorGoroutinesPprof: {  // 收集器名称
        outputFile:  "goroutines.pprof",  // 输出文件名
        collectFunc: goroutineStacksProto,  // 收集函数
        enabledFunc: func(opts Options) bool { return true },  // 启用函数
    },
    CollectorVersion: {  // 收集器名称
        outputFile:  "version.json",  // 输出文件名
        collectFunc: versionInfo,  // 收集函数
        enabledFunc: func(opts Options) bool { return true },  // 启用函数
    },
    CollectorHeap: {  // 收集器名称
        outputFile:  "heap.pprof",  // 输出文件名
        collectFunc: heapProfile,  // 收集函数
        enabledFunc: func(opts Options) bool { return true },  // 启用函数
    },
    CollectorAllocs: {  // 收集器名称
        outputFile:  "allocs.pprof",  // 输出文件名
        collectFunc: allocsProfile,  // 收集函数
        enabledFunc: func(opts Options) bool { return true },  // 启用函数
    },
    # CollectorBin: 二进制文件收集器配置
    CollectorBin: {
        # outputFile: 输出文件名为 "ipfs"
        outputFile:   "ipfs",
        # isExecutable: 输出文件是否可执行为 true
        isExecutable: true,
        # collectFunc: 收集函数为 binary
        collectFunc:  binary,
        # enabledFunc: 启用函数为根据选项返回 true
        enabledFunc:  func(opts Options) bool { return true },
    },
    # CollectorCPU: CPU 性能收集器配置
    CollectorCPU: {
        # outputFile: 输出文件名为 "cpu.pprof"
        outputFile:  "cpu.pprof",
        # collectFunc: 收集函数为 profileCPU
        collectFunc: profileCPU,
        # enabledFunc: 启用函数为根据选项返回 true
        enabledFunc: func(opts Options) bool { return opts.ProfileDuration > 0 },
    },
    # CollectorMutex: 互斥锁收集器配置
    CollectorMutex: {
        # outputFile: 输出文件名为 "mutex.pprof"
        outputFile:  "mutex.pprof",
        # collectFunc: 收集函数为 mutexProfile
        collectFunc: mutexProfile,
        # enabledFunc: 启用函数为根据选项返回 true
        enabledFunc: func(opts Options) bool { return opts.ProfileDuration > 0 && opts.MutexProfileFraction > 0 },
    },
    # CollectorBlock: 阻塞事件收集器配置
    CollectorBlock: {
        # outputFile: 输出文件名为 "block.pprof"
        outputFile:  "block.pprof",
        # collectFunc: 收集函数为 blockProfile
        collectFunc: blockProfile,
        # enabledFunc: 启用函数为根据选项返回 true
        enabledFunc: func(opts Options) bool { return opts.ProfileDuration > 0 && opts.BlockProfileRate > 0 },
    },
}


type Options struct {
    Collectors           []string  // 定义选项结构体，包含收集器列表、性能分析持续时间、互斥体性能分析比例、阻塞性能分析速率
    ProfileDuration      time.Duration
    MutexProfileFraction int
    BlockProfileRate     time.Duration
}

func WriteProfiles(ctx context.Context, archive *zip.Writer, opts Options) error {
    p := profiler{  // 创建 profiler 结构体对象
        archive: archive,  // 设置 zip 归档对象
        opts:    opts,  // 设置选项
    }
    return p.runProfile(ctx)  // 运行性能分析
}

// profiler runs the collectors concurrently and writes the results to the zip archive.
type profiler struct {
    archive *zip.Writer  // zip 归档对象
    opts    Options  // 选项
}

func (p *profiler) runProfile(ctx context.Context) error {
    type profileResult struct {  // 定义 profileResult 结构体
        fName string  // 文件名
        buf   *bytes.Buffer  // 字节缓冲区
        err   error  // 错误
    }

    ctx, cancelFn := context.WithCancel(ctx)  // 创建带有取消函数的上下文
    defer cancelFn()  // 延迟调用取消函数

    collectorsToRun := make([]collector, len(p.opts.Collectors))  // 创建要运行的收集器切片
    for i, name := range p.opts.Collectors {  // 遍历收集器列表
        c, ok := collectors[name]  // 获取收集器
        if !ok {
            return fmt.Errorf("unknown collector '%s'", name)  // 返回错误信息
        }
        collectorsToRun[i] = c  // 将收集器添加到要运行的收集器切片中
    }

    results := make(chan profileResult, len(p.opts.Collectors))  // 创建结果通道
    wg := sync.WaitGroup{}  // 创建同步等待组
    for _, c := range collectorsToRun {  // 遍历要运行的收集器切片
        if !c.enabledFunc(p.opts) {  // 如果收集器未启用
            continue  // 继续下一次循环
        }

        fName := c.outputFileName()  // 获取输出文件名

        wg.Add(1)  // 增加等待组计数
        go func(c collector) {  // 启动并发收集器
            defer wg.Done()  // 减少等待组计数
            logger.Infow("collecting profile", "File", fName)  // 记录日志
            defer logger.Infow("profile done", "File", fName)  // 延迟记录日志
            b := bytes.Buffer{}  // 创建字节缓冲区
            err := c.collectFunc(ctx, p.opts, &b)  // 运行收集函数
            if err != nil {  // 如果有错误
                select {  // 选择通道
                case results <- profileResult{err: fmt.Errorf("generating profile data for %q: %w", fName, err)}:  // 发送错误结果到通道
                case <-ctx.Done():  // 如果上下文已完成
                    return  // 返回
                }
            }
            select {  // 选择通道
            case results <- profileResult{buf: &b, fName: fName}:  // 发送结果到通道
            case <-ctx.Done():  // 如果上下文已完成
            }
        }(c)  // 传入收集器
    }
    # 创建一个匿名的 goroutine，等待所有任务完成后关闭 results 通道
    go func() {
        wg.Wait()
        close(results)
    }()
    
    # 遍历 results 通道，处理每个结果
    for res := range results:
        # 如果结果中包含错误，直接返回错误
        if res.err != nil:
            return res.err
        # 创建输出文件，并返回一个写入流
        out, err := p.archive.Create(res.fName)
        # 如果创建输出文件出错，返回错误信息
        if err != nil:
            return fmt.Errorf("creating output file %q: %w", res.fName, err)
        # 将结果数据压缩并写入输出文件
        _, err = io.Copy(out, res.buf)
        # 如果压缩过程出错，返回错误信息
        if err != nil:
            return fmt.Errorf("compressing result %q: %w", res.fName, err)
    
    # 所有结果处理完成，返回 nil 表示没有错误
    return nil
# 输出当前所有 goroutine 的堆栈信息到文本
func goroutineStacksText(ctx context.Context, _ Options, w io.Writer) error {
    return WriteAllGoroutineStacks(w)
}

# 输出当前所有 goroutine 的堆栈信息到 proto 格式
func goroutineStacksProto(ctx context.Context, _ Options, w io.Writer) error {
    return pprof.Lookup("goroutine").WriteTo(w, 0)
}

# 输出当前堆内存分配情况的 profile
func heapProfile(ctx context.Context, _ Options, w io.Writer) error {
    return pprof.Lookup("heap").WriteTo(w, 0)
}

# 输出当前内存分配次数的 profile
func allocsProfile(ctx context.Context, _ Options, w io.Writer) error {
    return pprof.Lookup("allocs").WriteTo(w, 0)
}

# 输出当前程序的版本信息到 JSON 格式
func versionInfo(ctx context.Context, _ Options, w io.Writer) error {
    return json.NewEncoder(w).Encode(version.GetVersionInfo())
}

# 输出当前程序的可执行文件内容
func binary(ctx context.Context, _ Options, w io.Writer) error {
    # 获取当前操作系统的可执行文件路径
    var (
        path string
        err  error
    )
    if goos == "linux" {
        pid := os.Getpid()
        path = fmt.Sprintf("/proc/%d/exe", pid)
    } else {
        path, err = os.Executable()
        if err != nil {
            return fmt.Errorf("finding binary path: %w", err)
        }
    }
    # 打开可执行文件
    fi, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("opening binary %q: %w", path, err)
    }
    # 将可执行文件内容拷贝到输出流
    _, err = io.Copy(w, fi)
    _ = fi.Close()
    if err != nil {
        return fmt.Errorf("copying binary %q: %w", path, err)
    }
    return nil
}

# 输出当前互斥锁的 profile
func mutexProfile(ctx context.Context, opts Options, w io.Writer) error {
    # 设置互斥锁 profile 的采样比例，并在函数结束时恢复原值
    prev := runtime.SetMutexProfileFraction(opts.MutexProfileFraction)
    defer runtime.SetMutexProfileFraction(prev)
    # 等待一段时间或者取消操作，然后输出互斥锁 profile
    err := waitOrCancel(ctx, opts.ProfileDuration)
    if err != nil {
        return err
    }
    return pprof.Lookup("mutex").WriteTo(w, 2)
}

# 输出当前阻塞 profile
func blockProfile(ctx context.Context, opts Options, w io.Writer) error {
    # 设置阻塞 profile 的采样比例，并在函数结束时恢复原值
    runtime.SetBlockProfileRate(int(opts.BlockProfileRate.Nanoseconds()))
    defer runtime.SetBlockProfileRate(0)
    # 等待一段时间或者取消操作，然后输出阻塞 profile
    err := waitOrCancel(ctx, opts.ProfileDuration)
    if err != nil {
        return err
    }
    return pprof.Lookup("block").WriteTo(w, 2)
}
# 根据给定的上下文、选项和写入器开始 CPU 分析，返回可能的错误
func profileCPU(ctx context.Context, opts Options, w io.Writer) error {
    # 开始 CPU 分析，并将结果写入指定的写入器
    err := pprof.StartCPUProfile(w)
    # 如果出现错误，返回错误信息
    if err != nil {
        return err
    }
    # 延迟停止 CPU 分析，确保在函数返回前停止
    defer pprof.StopCPUProfile()
    # 等待指定的时间或者在上下文被取消时返回
    return waitOrCancel(ctx, opts.ProfileDuration)
}

# 等待指定的上下文和持续时间，返回可能的错误
func waitOrCancel(ctx context.Context, d time.Duration) error {
    # 创建一个定时器，设置持续时间
    timer := time.NewTimer(d)
    # 延迟停止定时器，确保在函数返回前停止
    defer timer.Stop()
    # 选择等待定时器到期或者上下文被取消
    select {
    # 定时器到期时返回 nil
    case <-timer.C:
        return nil
    # 上下文被取消时返回上下文的错误信息
    case <-ctx.Done():
        return ctx.Err()
    }
}
```