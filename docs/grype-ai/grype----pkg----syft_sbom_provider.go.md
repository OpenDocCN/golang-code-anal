# `grype\grype\pkg\syft_sbom_provider.go`

```
// 声明一个名为 pkg 的包
package pkg

// 导入所需的包
import (
	"bytes" // 用于操作字节流
	"fmt" // 用于格式化输出
	"io" // 用于输入输出操作
	"os" // 提供对操作系统功能的访问
	"strings" // 提供对字符串操作的函数

	"github.com/gabriel-vasile/mimetype" // 用于处理 MIME 类型
	"github.com/mitchellh/go-homedir" // 用于处理用户目录

	"github.com/anchore/grype/internal" // 导入内部包
	"github.com/anchore/grype/internal/log" // 导入内部日志包
	"github.com/anchore/syft/syft/format" // 导入格式化包
	"github.com/anchore/syft/syft/sbom" // 导入 sbom 包
)

// 定义一个自定义错误类型 errEmptySBOM，包含 sbomFilepath 字段
type errEmptySBOM struct {
	sbomFilepath string
// errEmptySBOM 是一个自定义的错误类型，用于表示 SBOM 文件为空的错误
func (e errEmptySBOM) Error() string {
	// 返回错误信息，包含空的 SBOM 文件路径
	return fmt.Sprintf("SBOM file is empty: %s", e.sbomFilepath)
}

// syftSBOMProvider 是一个函数，用于提供 SBOM 数据
func syftSBOMProvider(userInput string, config ProviderConfig) ([]Package, Context, *sbom.SBOM, error) {
	// 获取 SBOM 数据
	s, err := getSBOM(userInput)
	// 如果获取失败，返回错误
	if err != nil {
		return nil, Context{}, nil, err
	}

	// 从 SBOM 数据中移除重叠的软件包
	catalog := removePackagesByOverlap(s.Artifacts.Packages, s.Relationships, s.Artifacts.LinuxDistribution)

	// 将移除重叠后的软件包转换为合成配置，并返回上下文信息和 SBOM 数据
	return FromCollection(catalog, config.SynthesisConfig), Context{
		Source: &s.Source,
		Distro: s.Artifacts.LinuxDistribution,
	}, s, nil
}
# 创建一个新的输入信息对象，包含指定的方案和内容类型
func newInputInfo(scheme, contentTye string) *inputInfo {
	# 返回一个指向新创建的输入信息对象的指针
	return &inputInfo{
		Scheme:      scheme,
		ContentType: contentTye,
	}
}

# 定义输入信息结构体
type inputInfo struct {
	ContentType string
	Scheme      string
}

# 获取软件构建材料对象
func getSBOM(userInput string) (*sbom.SBOM, error) {
	# 获取用户输入的软件构建材料读取器
	reader, err := getSBOMReader(userInput)
	# 如果出现错误，返回空指针和错误信息
	if err != nil {
		return nil, err
	}

	# 解析软件构建材料数据
	s, fmtID, _, err := format.Decode(reader)
	# 如果出现错误，返回空指针和错误信息
	if err != nil {
		// 如果解码 sbom 失败，则返回 nil 和带有错误信息的错误对象
		return nil, fmt.Errorf("unable to decode sbom: %w", err)
	}

	// 如果 fmtID 为空或者 s 为空，则返回 nil 和错误对象 errDoesNotProvide
	if fmtID == "" || s == nil {
		return nil, errDoesNotProvide
	}

	// 返回 s 和 nil，表示成功获取 sbom 读取器
	return s, nil
}

// 获取 sbom 读取器
func getSBOMReader(userInput string) (r io.ReadSeeker, err error) {
	// 调用 extractReaderAndInfo 函数获取读取器和信息
	r, _, err = extractReaderAndInfo(userInput)
	// 如果出现错误，则返回 nil 和错误对象
	if err != nil {
		return nil, err
	}

	// 返回读取器和 nil，表示成功获取 sbom 读取器
	return r, nil
}

// 从用户输入中提取读取器和信息
func extractReaderAndInfo(userInput string) (io.ReadSeeker, *inputInfo, error) {
// switch 语句用于根据不同的条件执行不同的代码块
// 注意：case 的顺序很重要，因为匹配是按顺序进行的
switch {
    // 如果用户输入为空字符串，则尝试从标准输入读取数据
    // 注意：只有在用户没有通过命令行指定其他选项时，我们才会尝试从标准输入读取数据，否则我们不应该假设标准输入中有有效输入
    case userInput == "":
        return decodeStdin(stdinReader())

    // 如果用户明确指定了 SBOM（软件构建材料）文件路径，则解析该文件
    // 注意：使用 strings.TrimPrefix 函数去除输入字符串中的 "sbom:" 前缀
    case explicitlySpecifyingSBOM(userInput):
        filepath := strings.TrimPrefix(userInput, "sbom:")
        return parseSBOM("sbom", filepath)

    // 如果用户输入可能是 SBOM，则解析该输入
    case isPossibleSBOM(userInput):
        return parseSBOM("", userInput)

    // 如果以上条件都不满足，则返回 nil, nil, errDoesNotProvide
    default:
        return nil, nil, errDoesNotProvide
}

// 解析 SBOM 文件的函数
func parseSBOM(scheme, path string) (io.ReadSeeker, *inputInfo, error) {
// 打开文件并返回读取器，如果出现错误则返回 nil 和错误信息
r, err := openFile(path)
if err != nil {
    return nil, nil, err
}
// 创建一个新的输入信息对象
info := newInputInfo(scheme, "sbom")
// 返回读取器、输入信息对象和 nil 错误
return r, info, nil
}

// 从输入流中解码数据并返回读取器、输入信息对象和错误信息
func decodeStdin(r io.Reader) (io.ReadSeeker, *inputInfo, error) {
    // 读取整个输入流的数据
    b, err := io.ReadAll(r)
    if err != nil {
        return nil, nil, fmt.Errorf("failed reading stdin: %w", err)
    }

    // 创建一个新的字节流读取器
    reader := bytes.NewReader(b)
    // 将读取器的位置设置为起始位置
    _, err = reader.Seek(0, io.SeekStart)
    if err != nil {
        return nil, nil, fmt.Errorf("failed to parse stdin: %w", err)
    }
// 返回reader、新的输入信息和nil
return reader, newInputInfo("", "sbom"), nil
}

// fileHasContent函数返回一个布尔值，指示给定文件是否具有可能在下游处理中被利用的数据。
func fileHasContent(f *os.File) bool {
	// 如果文件为空，则返回false
	if f == nil {
		return false
	}

	// 获取文件信息
	info, err := f.Stat()
	// 如果获取文件信息出错，则返回false
	if err != nil {
		return false
	}

	// 如果文件大小大于0，则返回true
	if size := info.Size(); size > 0 {
		return true
	}

	// 否则返回false
	return false
// 定义一个函数，用于读取标准输入流
func stdinReader() io.Reader {
    // 检查标准输入是否为管道或重定向
    isStdinPipeOrRedirect, err := internal.IsStdinPipeOrRedirect()
    if err != nil {
        // 如果无法确定是否有管道输入，则记录警告并返回空
        log.Warnf("unable to determine if there is piped input: %+v", err)
        return nil
    }

    // 如果标准输入是管道或重定向，则返回标准输入流
    if !isStdinPipeOrRedirect {
        return nil
    }
    return os.Stdin
}

// 定义一个函数，用于关闭文件
func closeFile(f *os.File) {
    // 如果文件为空，则直接返回
    if f == nil {
        return
    }
}
// 关闭文件并检查是否有错误发生，如果有则记录日志
err := f.Close()
if err != nil {
    log.Warnf("failed to close file %s: %v", f.Name(), err)
}

// 打开指定路径的文件，并返回文件对象和可能发生的错误
func openFile(path string) (*os.File, error) {
    // 将路径进行扩展，解析可能包含的波浪号（~）等特殊字符
    expandedPath, err := homedir.Expand(path)
    if err != nil {
        return nil, fmt.Errorf("unable to open SBOM: %w", err)
    }

    // 打开扩展后的路径对应的文件
    f, err := os.Open(expandedPath)
    if err != nil {
        return nil, fmt.Errorf("unable to open file %s: %w", expandedPath, err)
    }

    // 检查文件是否为空，如果是则返回错误
    if !fileHasContent(f) {
        return nil, errEmptySBOM{path}
    }
	}

	// 返回文件对象和空错误
	return f, nil
}

// 判断用户输入是否可能是 SBOM（软件构建材料）文档
func isPossibleSBOM(userInput string) bool {
	// 打开用户输入的文件
	f, err := openFile(userInput)
	// 如果出现错误，返回 false
	if err != nil {
		return false
	}
	// 延迟关闭文件
	defer closeFile(f)

	// 检测文件的 MIME 类型
	mType, err := mimetype.DetectReader(f)
	// 如果出现错误，返回 false
	if err != nil {
		return false
	}

	// 我们期望应用程序/json、应用程序/xml和文本/纯文本输入文档。所有这些要么是文本/纯文本，要么是文本/纯文本的后代。其他任何类型都不能是输入的 SBOM 文档。
	// 返回 MIME 类型是否是 text/plain 或其后代
	return isAncestorOfMimetype(mType, "text/plain")
}

func isAncestorOfMimetype(mType *mimetype.MIME, expected string) bool {
	// 检查给定的 MIME 类型是否是期望类型的祖先
	for cur := mType; cur != nil; cur = cur.Parent() {
		// 如果当前 MIME 类型是期望类型，则返回 true
		if cur.Is(expected) {
			return true
		}
	}
	// 如果没有找到期望类型的祖先，则返回 false
	return false
}

func explicitlySpecifyingSBOM(userInput string) bool {
	// 检查用户输入是否以"sbom:"开头
	return strings.HasPrefix(userInput, "sbom:")
}
```