# `kubo\client\rpc\apifile.go`

```
package rpc

import (
    "context"
    "encoding/json"
    "fmt"
    "io"

    "github.com/ipfs/boxo/files"
    unixfs "github.com/ipfs/boxo/ipld/unixfs"
    "github.com/ipfs/boxo/path"
    "github.com/ipfs/go-cid"
)

const forwardSeekLimit = 1 << 14 // 16k

// Get retrieves the node at the given path
func (api *UnixfsAPI) Get(ctx context.Context, p path.Path) (files.Node, error) {
    // Check if the path is mutable and resolve it if necessary
    if p.Mutable() {
        var err error
        p, _, err = api.core().ResolvePath(ctx, p)
        if err != nil {
            return nil, err
        }
    }

    // Retrieve the hash, type, and size of the node at the given path
    var stat struct {
        Hash string
        Type string
        Size int64 // unixfs size
    }
    err := api.core().Request("files/stat", p.String()).Exec(ctx, &stat)
    if err != nil {
        return nil, err
    }

    // Based on the type of the node, call the appropriate method to retrieve the node
    switch stat.Type {
    case "file":
        return api.getFile(ctx, p, stat.Size)
    case "directory":
        return api.getDir(ctx, p, stat.Size)
    default:
        return nil, fmt.Errorf("unsupported file type '%s'", stat.Type)
    }
}

// apiFile represents a file in the API
type apiFile struct {
    ctx  context.Context
    core *HttpApi
    size int64
    path path.Path

    r  *Response
    at int64
}

// reset resets the file to its initial state
func (f *apiFile) reset() error {
    // If there is an ongoing request, cancel it
    if f.r != nil {
        _ = f.r.Cancel()
        f.r = nil
    }
    // Create a new request to read the file at the specified path and offset
    req := f.core.Request("cat", f.path.String())
    if f.at != 0 {
        req.Option("offset", f.at)
    }
    resp, err := req.Send(f.ctx)
    if err != nil {
        return err
    }
    if resp.Error != nil {
        return resp.Error
    }
    f.r = resp
    return nil
}

// Read reads data from the file
func (f *apiFile) Read(p []byte) (int, error) {
    n, err := f.r.Output.Read(p)
    if n > 0 {
        f.at += int64(n)
    }
    return n, err
}

// ReadAt reads data from the file at the specified offset
func (f *apiFile) ReadAt(p []byte, off int64) (int, error) {
    // Always make a new request. This method should be parallel-safe.
    resp, err := f.core.Request("cat", f.path.String()).
        Option("offset", off).Option("length", len(p)).Send(f.ctx)
    if err != nil {
        return 0, err
    }
    // ...
}
    }
    # 如果响应中存在错误，则返回 0 和错误信息
    if resp.Error != nil:
        return 0, resp.Error
    # 延迟关闭响应输出流
    defer resp.Output.Close()

    # 从响应输出流中读取数据到 p 中，返回读取的字节数和可能的错误
    n, err := io.ReadFull(resp.Output, p)
    # 如果出现意外的文件结尾错误，则将其转换为文件结尾错误
    if err == io.ErrUnexpectedEOF:
        err = io.EOF
    # 返回读取的字节数和可能的错误
    return n, err
}

// Seek 方法用于移动文件指针到指定位置
func (f *apiFile) Seek(offset int64, whence int) (int64, error) {
    // 根据 whence 参数确定移动方式
    switch whence {
    case io.SeekEnd:
        offset = f.size + offset
    case io.SeekCurrent:
        offset = f.at + offset
    }
    // 如果当前位置等于目标位置，则不进行移动
    if f.at == offset { // noop
        return offset, nil
    }

    // 如果目标位置在当前位置之后，并且距离不超过 forwardSeekLimit，则进行快速跳转
    if f.at < offset && offset-f.at < forwardSeekLimit { // forward skip
        r, err := io.CopyN(io.Discard, f.r.Output, offset-f.at)

        f.at += r
        return f.at, err
    }
    // 更新当前位置为目标位置
    f.at = offset
    return f.at, f.reset()
}

// Close 方法用于关闭文件
func (f *apiFile) Close() error {
    // 如果文件读取器不为空，则取消读取操作
    if f.r != nil {
        return f.r.Cancel()
    }
    return nil
}

// Size 方法用于获取文件大小
func (f *apiFile) Size() (int64, error) {
    return f.size, nil
}

// getFile 方法用于获取指定路径的文件
func (api *UnixfsAPI) getFile(ctx context.Context, p path.Path, size int64) (files.Node, error) {
    // 创建 apiFile 对象
    f := &apiFile{
        ctx:  ctx,
        core: api.core(),
        size: size,
        path: p,
    }

    return f, f.reset()
}

// apiIter 结构体用于迭代文件
type apiIter struct {
    ctx  context.Context
    core *UnixfsAPI

    err error

    dec     *json.Decoder
    curFile files.Node
    cur     lsLink
}

// Err 方法用于获取迭代器的错误信息
func (it *apiIter) Err() error {
    return it.err
}

// Name 方法用于获取当前迭代的文件名
func (it *apiIter) Name() string {
    return it.cur.Name
}

// Next 方法用于移动到下一个文件
func (it *apiIter) Next() bool {
    // 如果上下文出现错误，则返回错误信息
    if it.ctx.Err() != nil {
        it.err = it.ctx.Err()
        return false
    }

    // 解析 JSON 数据并进行迭代
    var out lsOutput
    if err := it.dec.Decode(&out); err != nil {
        if err != io.EOF {
            it.err = err
        }
        return false
    }

    // 检查返回的对象和链接数量是否符合预期
    if len(out.Objects) != 1 {
        it.err = fmt.Errorf("ls returned more objects than expected (%d)", len(out.Objects))
        return false
    }

    if len(out.Objects[0].Links) != 1 {
        it.err = fmt.Errorf("ls returned more links than expected (%d)", len(out.Objects[0].Links))
        return false
    }

    // 获取当前链接的哈希值并解析为 CID
    it.cur = out.Objects[0].Links[0]
    c, err := cid.Parse(it.cur.Hash)
    if err != nil {
        it.err = err
        return false
    }

    // 根据链接类型进行处理
    switch it.cur.Type {
    # 根据不同的文件类型进行不同的处理
    case unixfs.THAMTShard, unixfs.TMetadata, unixfs.TDirectory:
        # 如果是目录类型，则调用getDir方法获取目录信息
        it.curFile, err = it.core.getDir(it.ctx, path.FromCid(c), int64(it.cur.Size))
        # 如果出现错误，则设置错误信息并返回false
        if err != nil:
            it.err = err
            return false
        # 如果没有错误，则返回true
    case unixfs.TFile:
        # 如果是文件类型，则调用getFile方法获取文件信息
        it.curFile, err = it.core.getFile(it.ctx, path.FromCid(c), int64(it.cur.Size))
        # 如果出现错误，则设置错误信息并返回false
        if err != nil:
            it.err = err
            return false
    default:
        # 如果是其他类型的文件，则设置错误信息并返回false
        it.err = fmt.Errorf("file type %d not supported", it.cur.Type)
        return false
    }
    # 返回true
    return true
# 返回当前迭代器指向的文件节点
func (it *apiIter) Node() files.Node {
    return it.curFile
}

# 定义 apiDir 结构体
type apiDir struct {
    ctx  context.Context
    core *UnixfsAPI
    size int64
    path path.Path

    dec *json.Decoder
}

# 关闭 apiDir 对象，返回错误
func (d *apiDir) Close() error {
    return nil
}

# 返回 apiDir 对象的大小和错误
func (d *apiDir) Size() (int64, error) {
    return d.size, nil
}

# 返回 apiDir 对象的文件夹迭代器
func (d *apiDir) Entries() files.DirIterator {
    return &apiIter{
        ctx:  d.ctx,
        core: d.core,
        dec:  d.dec,
    }
}

# 获取指定路径的文件夹节点
func (api *UnixfsAPI) getDir(ctx context.Context, p path.Path, size int64) (files.Node, error) {
    # 发送 ls 命令请求，获取指定路径的文件夹信息
    resp, err := api.core().Request("ls", p.String()).
        Option("resolve-size", true).
        Option("stream", true).Send(ctx)
    if err != nil {
        return nil, err
    }
    if resp.Error != nil {
        return nil, resp.Error
    }

    # 创建 apiDir 对象，用于表示指定路径的文件夹
    d := &apiDir{
        ctx:  ctx,
        core: api,
        size: size,
        path: p,

        dec: json.NewDecoder(resp.Output),
    }

    return d, nil
}

# 空白标识符，用于断言接口实现
var (
    _ files.File      = &apiFile{}
    _ files.Directory = &apiDir{}
)
```