# `kubo\core\commands\get.go`

```go
package commands

import (
    "bufio"  // 导入 bufio 包，提供了缓冲读写功能
    "compress/gzip"  // 导入 gzip 压缩包
    "errors"  // 导入 errors 包，用于处理错误
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，提供了基本的 I/O 接口
    "os"  // 导入 os 包，提供了操作系统函数
    gopath "path"  // 导入 path 包，用于处理文件路径
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "strings"  // 导入 strings 包，提供了操作字符串的函数

    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包
    "github.com/ipfs/kubo/core/commands/cmdutils"  // 导入 cmdutils 包
    "github.com/ipfs/kubo/core/commands/e"  // 导入 e 包

    "github.com/cheggaaa/pb"  // 导入 pb 包，用于显示进度条
    "github.com/ipfs/boxo/files"  // 导入 files 包
    "github.com/ipfs/boxo/tar"  // 导入 tar 包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 go-ipfs-cmds 包
)

var ErrInvalidCompressionLevel = errors.New("compression level must be between 1 and 9")  // 定义错误变量

const (
    outputOptionName           = "output"  // 定义输出选项名称
    archiveOptionName          = "archive"  // 定义归档选项名称
    compressOptionName         = "compress"  // 定义压缩选项名称
    compressionLevelOptionName = "compression-level"  // 定义压缩级别选项名称
)

var GetCmd = &cmds.Command{  // 定义 GetCmd 变量为命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Download IPFS objects.",  // 标语
        ShortDescription: `  // 简短描述
Stores to disk the data contained an IPFS or IPNS object(s) at the given path.

By default, the output will be stored at './<ipfs-path>', but an alternate
path can be specified with '--output=<path>' or '-o=<path>'.

To output a TAR archive instead of unpacked files, use '--archive' or '-a'.

To compress the output with GZIP compression, use '--compress' or '-C'. You
may also specify the level of compression by specifying '-l=<1-9>'.
`,
    },

    Arguments: []cmds.Argument{  // 参数
        cmds.StringArg("ipfs-path", true, false, "The path to the IPFS object(s) to be outputted.").EnableStdin(),  // 字符串参数
    },
    Options: []cmds.Option{  // 选项
        cmds.StringOption(outputOptionName, "o", "The path where the output should be stored."),  // 字符串选项
        cmds.BoolOption(archiveOptionName, "a", "Output a TAR archive."),  // 布尔选项
        cmds.BoolOption(compressOptionName, "C", "Compress the output with GZIP compression."),  // 布尔选项
        cmds.IntOption(compressionLevelOptionName, "l", "The level of compression (1-9)."),  // 整数选项
        cmds.BoolOption(progressOptionName, "p", "Stream progress data.").WithDefault(true),  // 布尔选项
    },
    # 在运行之前执行的函数，用于获取压缩选项并返回错误
    PreRun: func(req *cmds.Request, env cmds.Environment) error {
        _, err := getCompressOptions(req)
        return err
    },
    # 实际运行的函数，处理请求并返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取请求的上下文
        ctx := req.Context
        # 获取压缩选项
        cmplvl, err := getCompressOptions(req)
        if err != nil {
            return err
        }
        # 从环境中获取 API
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        # 获取文件路径或 CID 路径
        p, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }
        # 从 API 中获取文件
        file, err := api.Unixfs().Get(ctx, p)
        if err != nil {
            return err
        }
        # 获取文件大小
        size, err := file.Size()
        if err != nil {
            return err
        }
        # 设置响应的长度
        res.SetLength(uint64(size))
        # 获取是否为归档选项
        archive, _ := req.Options[archiveOptionName].(bool)
        # 获取文件的读取器
        reader, err := fileArchive(file, p.String(), archive, cmplvl)
        if err != nil {
            return err
        }
        # 异步关闭读取器
        go func() {
            # 无法在响应写入器中延迟关闭
            # 因为命令框架会在上下文结束时不调用响应
            <-ctx.Done()
            reader.Close()
        }()
        # 发送读取器的内容作为响应
        return res.Emit(reader)
    },
    # 定义 PostRunMap 结构体的 cmds.CLI 方法，处理命令行接口的响应
    PostRun: cmds.PostRunMap{
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 获取响应对象的请求信息
            req := res.Request()

            # 从响应对象中获取下一个值和可能的错误
            v, err := res.Next()
            if err != nil {
                return err
            }

            # 将获取的值转换为 io.Reader 接口类型
            outReader, ok := v.(io.Reader)
            if !ok {
                return e.New(e.TypeErr(outReader, v))
            }

            # 获取输出路径
            outPath := getOutPath(req)

            # 获取压缩选项
            cmplvl, err := getCompressOptions(req)
            if err != nil {
                return err
            }

            # 获取归档和进度选项
            archive, _ := req.Options[archiveOptionName].(bool)
            progress, _ := req.Options[progressOptionName].(bool)

            # 创建 getWriter 结构体
            gw := getWriter{
                Out:         os.Stdout,
                Err:         os.Stderr,
                Archive:     archive,
                Compression: cmplvl,
                Size:        int64(res.Length()),
                Progress:    progress,
            }

            # 调用 getWriter 结构体的 Write 方法
            return gw.Write(outReader, outPath)
        },
    },
}
// 定义一个结构体 clearlineReader，包含一个 io.Reader 和一个 io.Writer
type clearlineReader struct {
    io.Reader
    out io.Writer
}

// 重写 clearlineReader 的 Read 方法
func (r *clearlineReader) Read(p []byte) (n int, err error) {
    // 调用原始 Reader 的 Read 方法
    n, err = r.Reader.Read(p)
    // 如果读取到文件末尾
    if err == io.EOF {
        // 回调函数
        fmt.Fprintf(r.out, "\033[2K\r") // 清除进度条行
    }
    return
}

// 为 Reader 创建一个进度条
func progressBarForReader(out io.Writer, r io.Reader, l int64) (*pb.ProgressBar, io.Reader) {
    // 创建进度条
    bar := makeProgressBar(out, l)
    // 创建进度条的代理 Reader
    barR := bar.NewProxyReader(r)
    return bar, &clearlineReader{barR, out}
}

// 创建进度条
func makeProgressBar(out io.Writer, l int64) *pb.ProgressBar {
    // 设置进度条的总长度
    bar := pb.New64(l).SetUnits(pb.U_BYTES)
    bar.Output = out

    // 进度条库没有提供获取输出宽度的方法，所以使用回调函数来测量输出，然后将其删除
    bar.Callback = func(line string) {
        terminalWidth := len(line)
        bar.Callback = nil
        log.Infof("terminal width: %v\n", terminalWidth)
    }
    return bar
}

// 获取输出路径
func getOutPath(req *cmds.Request) string {
    outPath, _ := req.Options[outputOptionName].(string)
    if outPath == "" {
        trimmed := strings.TrimRight(req.Arguments[0], "/")
        _, outPath = filepath.Split(trimmed)
        outPath = filepath.Clean(outPath)
    }
    return outPath
}

// 定义一个结构体 getWriter，包含一个输出到用户的 io.Writer 和一个进度条输出的 io.Writer
type getWriter struct {
    Out io.Writer // 输出给用户
    Err io.Writer // 进度条输出

    Archive     bool
    Compression int
    Size        int64
    Progress    bool
}

// 实现 getWriter 的 Write 方法
func (gw *getWriter) Write(r io.Reader, fpath string) error {
    if gw.Archive || gw.Compression != gzip.NoCompression {
        return gw.writeArchive(r, fpath)
    }
    return gw.writeExtracted(r, fpath)
}

// 写入压缩文件
func (gw *getWriter) writeArchive(r io.Reader, fpath string) error {
    // 如果是 tar 文件，调整文件名
    if gw.Archive {
        if !strings.HasSuffix(fpath, ".tar") && !strings.HasSuffix(fpath, ".tar.gz") {
            fpath += ".tar"
        }
    // 如果文件压缩方式不是无压缩，则检查文件名是否以".gz"结尾，如果不是则添加".gz"后缀
    if gw.Compression != gzip.NoCompression {
        if !strings.HasSuffix(fpath, ".gz") {
            fpath += ".gz"
        }
    }

    // 创建文件
    file, err := os.Create(fpath)
    if err != nil {
        return err
    }
    defer file.Close()

    // 在控制台输出保存归档文件的路径
    fmt.Fprintf(gw.Out, "Saving archive to %s\n", fpath)

    // 如果启用了进度条显示
    if gw.Progress {
        // 创建进度条并将读取器传入，获取进度条和读取器
        var bar *pb.ProgressBar
        bar, r = progressBarForReader(gw.Err, r, gw.Size)
        // 启动进度条
        bar.Start()
        // 在函数返回时关闭进度条
        defer bar.Finish()
    }

    // 将读取器的内容拷贝到文件中
    _, err = io.Copy(file, r)
    return err
// writeExtracted 将从输入流中读取的数据写入到指定路径的文件中
func (gw *getWriter) writeExtracted(r io.Reader, fpath string) error {
    // 在输出流中打印保存文件的路径
    fmt.Fprintf(gw.Out, "Saving file(s) to %s\n", fpath)
    // 定义进度回调函数
    var progressCb func(int64) int64
    // 如果需要显示进度条
    if gw.Progress {
        // 创建进度条并开始显示
        bar := makeProgressBar(gw.Err, gw.Size)
        bar.Start()
        // 在函数返回时结束进度条显示，并设置进度为总大小
        defer bar.Finish()
        defer bar.Set64(gw.Size)
        // 设置进度回调函数
        progressCb = bar.Add64
    }

    // 创建一个 tar.Extractor 对象，用于解压文件
    extractor := &tar.Extractor{Path: fpath, Progress: progressCb}
    // 调用 Extract 方法进行解压，并返回可能的错误
    return extractor.Extract(r)
}

// getCompressOptions 从命令请求中获取压缩选项
func getCompressOptions(req *cmds.Request) (int, error) {
    // 从请求中获取是否需要压缩的选项
    cmprs, _ := req.Options[compressOptionName].(bool)
    // 从请求中获取压缩级别选项
    cmplvl, cmplvlFound := req.Options[compressionLevelOptionName].(int)
    // 根据不同情况返回不同的压缩选项和可能的错误
    switch {
    case !cmprs:
        return gzip.NoCompression, nil
    case cmprs && !cmplvlFound:
        return gzip.DefaultCompression, nil
    case cmprs && (cmplvl < 1 || cmplvl > 9):
        return gzip.NoCompression, ErrInvalidCompressionLevel
    }
    return cmplvl, nil
}

// identityWriteCloser 是一个实现了 io.WriteCloser 接口的结构体
type identityWriteCloser struct {
    w io.Writer
}

// Write 实现了 io.WriteCloser 接口的 Write 方法
func (i *identityWriteCloser) Write(p []byte) (int, error) {
    return i.w.Write(p)
}

// Close 实现了 io.WriteCloser 接口的 Close 方法
func (i *identityWriteCloser) Close() error {
    return nil
}

// fileArchive 对文件进行归档操作
func fileArchive(f files.Node, name string, archive bool, compression int) (io.ReadCloser, error) {
    // 清理文件名
    cleaned := gopath.Clean(name)
    // 获取文件名
    _, filename := gopath.Split(cleaned)

    // 创建一个管道，用于连接写入和读取
    piper, pipew := io.Pipe()
    // 检查错误并关闭管道
    checkErrAndClosePipe := func(err error) bool {
        if err != nil {
            _ = pipew.CloseWithError(err)
            return true
        }
        return false
    }

    // 使用缓冲写入器并指定默认缓冲大小
    bufw := bufio.NewWriterSize(pipew, DefaultBufSize)

    // 根据压缩选项创建可能的 gzip.Writer 对象
    maybeGzw, err := newMaybeGzWriter(bufw, compression)
    # 检查错误并关闭管道，如果有错误则返回nil和错误
    if checkErrAndClosePipe(err) {
        return nil, err
    }

    # 定义关闭Gzip Writer和管道的函数
    closeGzwAndPipe := func() {
        # 如果可能的Gzip Writer关闭时出现错误，则检查错误并关闭管道
        if err := maybeGzw.Close(); checkErrAndClosePipe(err) {
            return
        }
        # 如果缓冲区写入出现错误，则检查错误并关闭管道
        if err := bufw.Flush(); checkErrAndClosePipe(err) {
            return
        }
        # 关闭管道，一切似乎正常
        pipew.Close() // everything seems to be ok.
    }

    # 如果不是归档并且压缩方式不是gzip.NoCompression
    if !archive && compression != gzip.NoCompression {
        # 当节点是文件时的情况
        r := files.ToFile(f)
        # 如果r为空，则返回错误信息
        if r == nil {
            return nil, errors.New("file is not regular")
        }

        # 启动一个goroutine，将文件内容拷贝到Gzip Writer中
        go func() {
            if _, err := io.Copy(maybeGzw, r); checkErrAndClosePipe(err) {
                return
            }
            # 关闭Gzip Writer和管道，一切似乎正常
            closeGzwAndPipe() // everything seems to be ok
        }()
    } else {
        # 归档的情况，以及未归档且未压缩的情况，其中tar仍然作为传输格式使用

        # 构造tar writer
        w, err := files.NewTarWriter(maybeGzw)
        # 如果出现错误，则检查错误并关闭管道
        if checkErrAndClosePipe(err) {
            return nil, err
        }

        # 启动一个goroutine，递归写入所有节点
        go func() {
            if err := w.WriteFile(f, filename); checkErrAndClosePipe(err) {
                return
            }
            w.Close()         // 关闭tar writer
            closeGzwAndPipe() // 一切似乎正常
        }()
    }

    # 返回管道和nil
    return piper, nil
# 定义一个新的函数，用于创建可能是 Gzip 压缩的写入器
func newMaybeGzWriter(w io.Writer, compression int) (io.WriteCloser, error) {
    # 如果压缩级别不是 NoCompression，则创建指定压缩级别的 Gzip 写入器
    if compression != gzip.NoCompression {
        return gzip.NewWriterLevel(w, compression)
    }
    # 如果压缩级别是 NoCompression，则创建一个 identityWriteCloser 结构体的指针和一个空的错误
    return &identityWriteCloser{w}, nil
}
```