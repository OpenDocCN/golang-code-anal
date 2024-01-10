# `grype\internal\format\writer.go`

```
package format

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "fmt" // 导入 fmt 包，用于格式化输入输出
    "io" // 导入 io 包，用于进行 I/O 操作
    "os" // 导入 os 包，提供了对操作系统功能的访问
    "path" // 导入 path 包，用于处理文件路径
    "strings" // 导入 strings 包，用于处理字符串

    "github.com/hashicorp/go-multierror" // 导入第三方包，用于处理多个错误
    "github.com/mitchellh/go-homedir" // 导入第三方包，用于处理用户目录

    "github.com/anchore/grype/grype/presenter/models" // 导入自定义包
    "github.com/anchore/grype/internal/bus" // 导入自定义包
    "github.com/anchore/grype/internal/log" // 导入自定义包
)

// 定义接口 ScanResultWriter
type ScanResultWriter interface {
    Write(result models.PresenterConfig) error
}

// 确保 scanResultMultiWriter 实现了 ScanResultWriter 接口
var _ ScanResultWriter = (*scanResultMultiWriter)(nil)

// 确保 scanResultStreamWriter 实现了 io.Closer 和 ScanResultWriter 接口
var _ interface {
    io.Closer
    ScanResultWriter
} = (*scanResultStreamWriter)(nil)

// MakeScanResultWriter 创建一个 ScanResultWriter 用于输出，如果有错误则返回错误
func MakeScanResultWriter(outputs []string, defaultFile string, cfg PresentationConfig) (ScanResultWriter, error) {
    // 解析输出选项
    outputOptions, err := parseOutputFlags(outputs, defaultFile, cfg)
    if err != nil {
        return nil, err
    }

    // 创建多重写入器
    writer, err := newMultiWriter(outputOptions...)
    if err != nil {
        return nil, err
    }

    return writer, nil
}

// MakeScanResultWriterForFormat 根据给定的格式创建一个 ScanResultWriter，如果有错误则返回错误
func MakeScanResultWriterForFormat(f string, path string, cfg PresentationConfig) (ScanResultWriter, error) {
    // 解析格式
    format := Parse(f)

    if format == UnknownFormat {
        return nil, fmt.Errorf(`unsupported output format "%s", supported formats are: %+v`, f, AvailableFormats)
    }

    // 创建多重写入器
    writer, err := newMultiWriter(newWriterDescription(format, path, cfg))
    if err != nil {
        return nil, err
    }

    return writer, nil
}

// parseOutputFlags 是一个实用工具，用于解析命令行选项字符串，并保留默认格式和文件的现有行为
func parseOutputFlags(outputs []string, defaultFile string, cfg PresentationConfig) (out []scanResultWriterDescription, errs error) {
    // 总是应该有一个选项 -- 通常会得到 "table" 的默认值，但是要确保有值
    if len(outputs) == 0 {
        outputs = append(outputs, TableFormat.String())
    }

    // 遍历输出格式的名称
    for _, name := range outputs {
        // 去除格式名称两侧的空格
        name = strings.TrimSpace(name)

        // 以 "=" 为分隔符，最多分割成两部分，格式名称和文件名
        parts := strings.SplitN(name, "=", 2)

        // 格式名称是第一部分
        name = parts[0]

        // 默认使用 --file 或空字符串，如果未指定文件
        file := defaultFile

        // 如果输出格式名称中指定了文件，使用该文件
        if len(parts) > 1 {
            file = parts[1]
        }

        // 解析格式名称
        format := Parse(name)

        // 如果格式未知，添加错误信息到错误列表，继续下一个输出格式
        if format == UnknownFormat {
            errs = multierror.Append(errs, fmt.Errorf(`unsupported output format "%s", supported formats are: %+v`, name, AvailableFormats))
            continue
        }

        // 添加一个新的输出格式描述到输出列表
        out = append(out, newWriterDescription(format, file, cfg))
    }
    // 返回输出列表和错误列表
    return out, errs
// scanResultWriterDescription 结构体，用于描述 ScanResultWriter 的格式和路径字符串
type scanResultWriterDescription struct {
    Format Format
    Path   string
    Cfg    PresentationConfig
}

// newWriterDescription 函数，用于创建一个新的 scanResultWriterDescription 对象
func newWriterDescription(f Format, p string, cfg PresentationConfig) scanResultWriterDescription {
    // 将路径字符串进行扩展，解析用户主目录和环境变量
    expandedPath, err := homedir.Expand(p)
    if err != nil {
        // 如果扩展路径出错，则记录警告信息，忽略错误，使用原始路径
        log.Warnf("could not expand given writer output path=%q: %w", p, err)
        expandedPath = p
    }
    // 返回一个新的 scanResultWriterDescription 对象
    return scanResultWriterDescription{
        Format: f,
        Path:   expandedPath,
        Cfg:    cfg,
    }
}

// scanResultMultiWriter 结构体，用于保存多个子 ScanResultWriter 对象
type scanResultMultiWriter struct {
    writers []ScanResultWriter
}

// newMultiWriter 函数，用于创建多个报告写入器对象，如果未指定文件，则使用默认的写入器
func newMultiWriter(options ...scanResultWriterDescription) (_ *scanResultMultiWriter, err error) {
    // 如果未提供输出选项，则返回错误
    if len(options) == 0 {
        return nil, fmt.Errorf("no output options provided")
    }
    // 创建一个新的 scanResultMultiWriter 对象
    out := &scanResultMultiWriter{}
    # 遍历选项列表
    for _, option := range options {
        # 根据路径长度进行判断
        switch len(option.Path) {
        # 路径长度为0时
        case 0:
            # 将扫描结果发布者添加到输出写入器列表中
            out.writers = append(out.writers, &scanResultPublisher{
                format: option.Format,
                cfg:    option.Cfg,
            })
        # 默认情况
        default:
            # 创建任何缺失的子目录
            dir := path.Dir(option.Path)
            # 如果目录不为空
            if dir != "" {
                # 获取目录信息
                s, err := os.Stat(dir)
                # 如果出现错误
                if err != nil {
                    # 创建所有缺失的目录
                    err = os.MkdirAll(dir, 0755) // maybe should be os.ModePerm ?
                    # 如果创建目录失败
                    if err != nil {
                        return nil, err
                    }
                # 如果目录存在但不是目录
                } else if !s.IsDir() {
                    return nil, fmt.Errorf("output path does not contain a valid directory: %s", option.Path)
                }
            }
            # 打开或创建报告文件
            fileOut, err := os.OpenFile(option.Path, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0644)
            # 如果出现错误
            if err != nil {
                return nil, fmt.Errorf("unable to create report file: %w", err)
            }
            # 将扫描结果流写入器添加到输出写入器列表中
            out.writers = append(out.writers, &scanResultStreamWriter{
                format: option.Format,
                out:    fileOut,
                cfg:    option.Cfg,
            })
        }
    }

    # 返回输出对象和空错误
    return out, nil
// Write方法将结果写入所有的写入器
func (m *scanResultMultiWriter) Write(s models.PresenterConfig) (errs error) {
    // 遍历所有的写入器
    for _, w := range m.writers {
        // 调用每个写入器的Write方法
        err := w.Write(s)
        // 如果有错误，则将错误追加到errs中
        if err != nil {
            errs = multierror.Append(errs, fmt.Errorf("unable to write result: %w", err))
        }
    }
    return errs
}

// scanResultStreamWriter为给定格式和io.Writer实现了ScanResultWriter，同时提供了一个用于清理的close函数
type scanResultStreamWriter struct {
    format Format
    cfg    PresentationConfig
    out    io.Writer
}

// 将提供的结果写入数据流
func (w *scanResultStreamWriter) Write(s models.PresenterConfig) error {
    // 获取适当格式的Presenter
    pres := GetPresenter(w.format, w.cfg, s)
    // 将结果呈现到输出流
    if err := pres.Present(w.out); err != nil {
        return fmt.Errorf("unable to encode result: %w", err)
    }
    return nil
}

// 关闭任何资源，比如打开的文件
func (w *scanResultStreamWriter) Close() error {
    // 如果out实现了io.Closer接口，则调用其Close方法
    if closer, ok := w.out.(io.Closer); ok {
        return closer.Close()
    }
    return nil
}

// scanResultPublisher实现了将结果发布到事件总线的ScanResultWriter
type scanResultPublisher struct {
    format Format
    cfg    PresentationConfig
}

// 将提供的结果写入数据流
func (w *scanResultPublisher) Write(s models.PresenterConfig) error {
    // 获取适当格式的Presenter
    pres := GetPresenter(w.format, w.cfg, s)
    // 创建一个缓冲区
    buf := &bytes.Buffer{}
    // 将结果呈现到缓冲区
    if err := pres.Present(buf); err != nil {
        return fmt.Errorf("unable to encode result: %w", err)
    }
    // 将缓冲区的内容发布到事件总线
    bus.Report(buf.String())
    return nil
}
```