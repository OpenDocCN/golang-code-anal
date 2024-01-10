# `grype\grype\pkg\syft_sbom_provider.go`

```
package pkg

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于进行输入输出操作
    "os" // 导入 os 包，提供操作系统功能
    "strings" // 导入 strings 包，提供字符串操作

    "github.com/gabriel-vasile/mimetype" // 导入第三方库，用于处理 MIME 类型
    "github.com/mitchellh/go-homedir" // 导入第三方库，用于处理用户目录

    "github.com/anchore/grype/internal" // 导入内部包
    "github.com/anchore/grype/internal/log" // 导入内部包，用于日志记录
    "github.com/anchore/syft/syft/format" // 导入格式化包
    "github.com/anchore/syft/syft/sbom" // 导入 SBOM 包
)

type errEmptySBOM struct {
    sbomFilepath string
}

func (e errEmptySBOM) Error() string {
    return fmt.Sprintf("SBOM file is empty: %s", e.sbomFilepath)
}

func syftSBOMProvider(userInput string, config ProviderConfig) ([]Package, Context, *sbom.SBOM, error) {
    s, err := getSBOM(userInput) // 调用 getSBOM 函数获取 SBOM 数据
    if err != nil {
        return nil, Context{}, nil, err
    }

    catalog := removePackagesByOverlap(s.Artifacts.Packages, s.Relationships, s.Artifacts.LinuxDistribution) // 从 SBOM 数据中移除重叠的包

    return FromCollection(catalog, config.SynthesisConfig), Context{
        Source: &s.Source,
        Distro: s.Artifacts.LinuxDistribution,
    }, s, nil
}

func newInputInfo(scheme, contentTye string) *inputInfo {
    return &inputInfo{
        Scheme:      scheme,
        ContentType: contentTye,
    }
}

type inputInfo struct {
    ContentType string
    Scheme      string
}

func getSBOM(userInput string) (*sbom.SBOM, error) {
    reader, err := getSBOMReader(userInput) // 调用 getSBOMReader 函数获取 SBOM 数据的读取器
    if err != nil {
        return nil, err
    }

    s, fmtID, _, err := format.Decode(reader) // 解码 SBOM 数据
    if err != nil {
        return nil, fmt.Errorf("unable to decode sbom: %w", err)
    }

    if fmtID == "" || s == nil {
        return nil, errDoesNotProvide
    }

    return s, nil
}

func getSBOMReader(userInput string) (r io.ReadSeeker, err error) {
    r, _, err = extractReaderAndInfo(userInput) // 调用 extractReaderAndInfo 函数获取 SBOM 数据的读取器和信息
    if err != nil {
        return nil, err
    }

    return r, nil
}

func extractReaderAndInfo(userInput string) (io.ReadSeeker, *inputInfo, error) {
    switch {
    // the order of cases matter
    // 如果用户输入为空字符串，则尝试从标准输入读取数据，否则不假设标准输入中有有效数据
    case userInput == "":
        return decodeStdin(stdinReader())

    // 如果用户显式指定了SBOM（软件构建材料）文件路径，则解析该文件
    case explicitlySpecifyingSBOM(userInput):
        filepath := strings.TrimPrefix(userInput, "sbom:")
        return parseSBOM("sbom", filepath)

    // 如果用户输入可能是SBOM文件，则解析该文件
    case isPossibleSBOM(userInput):
        return parseSBOM("", userInput)

    // 默认情况下，返回空值和错误信息errDoesNotProvide
    default:
        return nil, nil, errDoesNotProvide
    }
// 解析SBOM，返回一个可读可定位的流、输入信息和错误
func parseSBOM(scheme, path string) (io.ReadSeeker, *inputInfo, error) {
    // 打开文件
    r, err := openFile(path)
    if err != nil {
        return nil, nil, err
    }
    // 创建输入信息对象
    info := newInputInfo(scheme, "sbom")
    // 返回文件流、输入信息和空错误
    return r, info, nil
}

// 从标准输入解码，返回一个可读可定位的流、输入信息和错误
func decodeStdin(r io.Reader) (io.ReadSeeker, *inputInfo, error) {
    // 读取标准输入的所有内容
    b, err := io.ReadAll(r)
    if err != nil {
        return nil, nil, fmt.Errorf("failed reading stdin: %w", err)
    }

    // 创建一个字节流读取器
    reader := bytes.NewReader(b)
    // 将读取器的偏移量设置为起始位置
    _, err = reader.Seek(0, io.SeekStart)
    if err != nil {
        return nil, nil, fmt.Errorf("failed to parse stdin: %w", err)
    }

    // 返回读取器、新的输入信息和空错误
    return reader, newInputInfo("", "sbom"), nil
}

// fileHasContent 返回一个布尔值，指示给定文件是否具有可能在下游处理中被利用的数据
func fileHasContent(f *os.File) bool {
    if f == nil {
        return false
    }

    // 获取文件信息
    info, err := f.Stat()
    if err != nil {
        return false
    }

    // 如果文件大小大于0，则返回true
    if size := info.Size(); size > 0 {
        return true
    }

    return false
}

// stdinReader 返回一个标准输入的读取器
func stdinReader() io.Reader {
    // 检查标准输入是否为管道或重定向
    isStdinPipeOrRedirect, err := internal.IsStdinPipeOrRedirect()
    if err != nil {
        log.Warnf("unable to determine if there is piped input: %+v", err)
        return nil
    }

    // 如果不是管道或重定向，则返回nil
    if !isStdinPipeOrRedirect {
        return nil
    }

    return os.Stdin
}

// closeFile 关闭文件
func closeFile(f *os.File) {
    if f == nil {
        return
    }

    // 关闭文件
    err := f.Close()
    if err != nil {
        log.Warnf("failed to close file %s: %v", f.Name(), err)
    }
}

// openFile 打开文件
func openFile(path string) (*os.File, error) {
    // 展开路径
    expandedPath, err := homedir.Expand(path)
    if err != nil {
        return nil, fmt.Errorf("unable to open SBOM: %w", err)
    }

    // 打开文件
    f, err := os.Open(expandedPath)
    if err != nil {
        return nil, fmt.Errorf("unable to open file %s: %w", expandedPath, err)
    }

    // 如果文件没有内容，则返回错误
    if !fileHasContent(f) {
        return nil, errEmptySBOM{path}
    }

    return f, nil
}

// isPossibleSBOM 检查用户输入是否可能是SBOM
func isPossibleSBOM(userInput string) bool {
    // 打开用户输入的文件，并返回文件对象和可能的错误
    f, err := openFile(userInput)
    // 如果有错误发生，则返回 false
    if err != nil {
        return false
    }
    // 延迟执行关闭文件操作，确保文件在函数结束时被关闭
    defer closeFile(f)

    // 检测文件的 MIME 类型，并返回类型和可能的错误
    mType, err := mimetype.DetectReader(f)
    // 如果有错误发生，则返回 false
    if err != nil {
        return false
    }

    // 我们期望输入文档是 application/json、application/xml 或 text/plain 类型的。所有这些类型要么是 text/plain，要么是 text/plain 的子类型。
    // 其他类型的文档不能是 SBOM 文档的输入。
    // 检查文件的 MIME 类型是否是 "text/plain" 或其子类型，如果是则返回 true，否则返回 false
    return isAncestorOfMimetype(mType, "text/plain")
# 检查给定的 MIME 类型是否是期望的类型的祖先
func isAncestorOfMimetype(mType *mimetype.MIME, expected string) bool:
    # 从当前 MIME 类型开始，向上遍历其祖先，直到找到期望的类型，返回 true
    for cur := mType; cur != nil; cur = cur.Parent():
        if cur.Is(expected):
            return true
    # 如果没有找到期望的类型，返回 false
    return false

# 检查用户输入是否以"sbom:"开头，表示明确指定了 SBOM（软件构建材料）类型
func explicitlySpecifyingSBOM(userInput string) bool:
    return strings.HasPrefix(userInput, "sbom:")
```