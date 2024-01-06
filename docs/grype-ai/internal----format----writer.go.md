# `grype\internal\format\writer.go`

```
# 导入所需的包
package format

# 导入所需的包
import (
	"bytes"  # 导入 bytes 包，用于操作字节流
	"fmt"  # 导入 fmt 包，用于格式化输出
	"io"  # 导入 io 包，用于实现 I/O 操作
	"os"  # 导入 os 包，用于操作系统功能
	"path"  # 导入 path 包，用于处理文件路径
	"strings"  # 导入 strings 包，用于处理字符串

	"github.com/hashicorp/go-multierror"  # 导入第三方包，用于处理多个错误
	"github.com/mitchellh/go-homedir"  # 导入第三方包，用于处理用户主目录路径

	"github.com/anchore/grype/grype/presenter/models"  # 导入自定义包
	"github.com/anchore/grype/internal/bus"  # 导入自定义包
	"github.com/anchore/grype/internal/log"  # 导入自定义包
)

# 定义接口类型 ScanResultWriter
type ScanResultWriter interface {
	# 定义接口方法 Write，用于写入扫描结果
	Write(result models.PresenterConfig) error
// 定义一个接口类型为 ScanResultWriter 的变量，值为 nil
var _ ScanResultWriter = (*scanResultMultiWriter)(nil)

// 定义一个接口类型为 io.Closer 和 ScanResultWriter 的变量，值为 nil
var _ interface {
	io.Closer
	ScanResultWriter
} = (*scanResultStreamWriter)(nil)

// MakeScanResultWriter 函数用于创建一个 ScanResultWriter 用于输出，或者返回一个错误。如果没有错误，则应调用 ScanResultWriter.Close()
func MakeScanResultWriter(outputs []string, defaultFile string, cfg PresentationConfig) (ScanResultWriter, error) {
	// 解析输出选项
	outputOptions, err := parseOutputFlags(outputs, defaultFile, cfg)
	if err != nil {
		return nil, err
	}

	// 创建一个多重写入器
	writer, err := newMultiWriter(outputOptions...)
	if err != nil {
		return nil, err
	}
	}

	return writer, nil
}

// MakeScanResultWriterForFormat creates a ScanResultWriter for the given format or returns an error.
// 为给定的格式创建一个ScanResultWriter，或者返回一个错误。
func MakeScanResultWriterForFormat(f string, path string, cfg PresentationConfig) (ScanResultWriter, error) {
	// 解析输出格式
	format := Parse(f)

	// 如果格式为UnknownFormat，则返回错误
	if format == UnknownFormat {
		return nil, fmt.Errorf(`unsupported output format "%s", supported formats are: %+v`, f, AvailableFormats)
	}

	// 创建一个新的ScanResultWriter
	writer, err := newMultiWriter(newWriterDescription(format, path, cfg))
	if err != nil {
		return nil, err
	}

	// 返回创建的ScanResultWriter或者错误
	return writer, nil
}
// parseOutputFlags 是一个解析命令行选项字符串的实用工具，用于保留默认格式和文件的现有行为
func parseOutputFlags(outputs []string, defaultFile string, cfg PresentationConfig) (out []scanResultWriterDescription, errs error) {
	// 总是应该有一个选项 -- 通常我们得到默认的 "table"，但要确保一下
	if len(outputs) == 0 {
		outputs = append(outputs, TableFormat.String())
	}

	for _, name := range outputs {
		name = strings.TrimSpace(name)

		// 将 <format>=<file> 拆分为最多两部分
		parts := strings.SplitN(name, "=", 2)

		// 格式名称是第一部分
		name = parts[0]

		// 默认为 --file 或如果未指定则为空字符串
		file := defaultFile
// 如果输出格式中指定了文件名，使用该文件名
if len(parts) > 1 {
    file = parts[1]
}

// 解析输出格式名称
format := Parse(name)

// 如果解析出的格式为未知格式，将错误信息添加到errs中，并继续循环
if format == UnknownFormat {
    errs = multierror.Append(errs, fmt.Errorf(`unsupported output format "%s", supported formats are: %+v`, name, AvailableFormats))
    continue
}

// 将新的写入器描述添加到输出列表中
out = append(out, newWriterDescription(format, file, cfg))
}
return out, errs
}

// scanResultWriterDescription 用于创建ScanResultWriter的格式和路径字符串
type scanResultWriterDescription struct {
    Format Format
```
// 定义一个结构体成员变量，表示文件路径
Path   string
// 定义一个结构体成员变量，表示演示配置
Cfg    PresentationConfig
}

// 创建一个新的扫描结果写入描述
func newWriterDescription(f Format, p string, cfg PresentationConfig) scanResultWriterDescription {
	// 将路径进行扩展，如果出现错误则记录警告并忽略错误
	expandedPath, err := homedir.Expand(p)
	if err != nil {
		log.Warnf("could not expand given writer output path=%q: %w", p, err)
		// 忽略错误，使用原始路径
		expandedPath = p
	}
	// 返回扫描结果写入描述
	return scanResultWriterDescription{
		Format: f,
		Path:   expandedPath,
		Cfg:    cfg,
	}
}

// scanResultMultiWriter 持有一个子扫描结果写入器的列表，用于应用所有写入和关闭操作
type scanResultMultiWriter struct {
// 定义一个结构体，包含一个写入器的切片
type scanResultMultiWriter struct {
	writers []ScanResultWriter
}

// newMultiWriter 从输入选项中创建所有报告写入器；如果未指定文件，则使用给定的默认写入器
func newMultiWriter(options ...scanResultWriterDescription) (_ *scanResultMultiWriter, err error) {
	// 如果未提供输出选项，则返回错误
	if len(options) == 0 {
		return nil, fmt.Errorf("no output options provided")
	}

	// 创建一个scanResultMultiWriter对象
	out := &scanResultMultiWriter{}

	// 遍历所有选项
	for _, option := range options {
		// 根据路径长度进行不同的处理
		switch len(option.Path) {
		case 0:
			// 如果路径长度为0，则将格式和配置信息添加到writers切片中
			out.writers = append(out.writers, &scanResultPublisher{
				format: option.Format,
				cfg:    option.Cfg,
			})
		default:
			// 创建任何缺失的子目录
// 获取文件路径的目录部分
dir := path.Dir(option.Path)
// 如果目录部分不为空
if dir != "" {
    // 获取目录的状态信息
    s, err := os.Stat(dir)
    // 如果获取状态信息出错
    if err != nil {
        // 创建目录及其所有父目录，权限为 0755
        err = os.MkdirAll(dir, 0755) // 可能应该使用 os.ModePerm ?
        // 如果创建目录出错
        if err != nil {
            return nil, err
        }
    } else if !s.IsDir() {
        // 如果目录不是一个有效的目录
        return nil, fmt.Errorf("output path does not contain a valid directory: %s", option.Path)
    }
}
// 打开文件，如果不存在则创建，以读写模式打开，权限为 0644
fileOut, err := os.OpenFile(option.Path, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0644)
// 如果打开文件出错
if err != nil {
    return nil, fmt.Errorf("unable to create report file: %w", err)
}
// 将文件写入输出流列表
out.writers = append(out.writers, &scanResultStreamWriter{
    format: option.Format,
    out:    fileOut,
    cfg:    option.Cfg,
})
// 结构体方法，将结果写入多个写入器
func (m *scanResultMultiWriter) Write(s models.PresenterConfig) (errs error) {
    // 遍历所有写入器，将结果写入
    for _, w := range m.writers {
        // 调用写入器的Write方法，将结果写入
        err := w.Write(s)
        // 如果写入出错，将错误信息添加到errs中
        if err != nil {
            errs = multierror.Append(errs, fmt.Errorf("unable to write result: %w", err))
        }
    }
    // 返回所有写入器的错误信息
    return errs
}

// 结构体类型，实现了给定格式和io.Writer的ScanResultWriter接口，并提供了一个关闭函数进行清理
type scanResultStreamWriter struct {
    // 这里可以添加该结构体的属性说明
}
// 定义了一个结构体，包含了格式、展示配置和输出流
format Format
cfg    PresentationConfig
out    io.Writer
}

// 将提供的结果写入数据流
func (w *scanResultStreamWriter) Write(s models.PresenterConfig) error {
	// 根据格式、展示配置和结果获取展示器
	pres := GetPresenter(w.format, w.cfg, s)
	// 调用展示器的Present方法将结果写入输出流
	if err := pres.Present(w.out); err != nil {
		return fmt.Errorf("unable to encode result: %w", err)
	}
	return nil
}

// 关闭任何资源，比如打开的文件
func (w *scanResultStreamWriter) Close() error {
	// 如果输出流实现了io.Closer接口，则调用其Close方法关闭资源
	if closer, ok := w.out.(io.Closer); ok {
		return closer.Close()
	}
	return nil
// scanResultPublisher 实现了将结果发布到事件总线的 ScanResultWriter 接口
type scanResultPublisher struct {
    format Format  // 存储结果格式
    cfg    PresentationConfig  // 存储展示配置
}

// Write 方法将提供的结果写入数据流
func (w *scanResultPublisher) Write(s models.PresenterConfig) error {
    // 根据结果格式、展示配置和结果数据创建展示对象
    pres := GetPresenter(w.format, w.cfg, s)
    // 创建一个字节缓冲区
    buf := &bytes.Buffer{}
    // 将展示对象的展示结果写入缓冲区
    if err := pres.Present(buf); err != nil {
        return fmt.Errorf("unable to encode result: %w", err)  // 如果写入失败，返回错误信息
    }

    // 将展示结果发布到事件总线
    bus.Report(buf.String())
    return nil  // 返回空错误，表示写入成功
}
```