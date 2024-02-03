# `grype\grype\pkg\purl_provider.go`

```go
package pkg
// 导入所需的包
import (
    "bufio" // 用于读取输入流
    "fmt" // 用于格式化输出
    "io" // 提供基本的 I/O 接口
    "os" // 提供操作系统功能
    "strings" // 提供字符串操作函数

    "github.com/facebookincubator/nvdtools/wfn" // 导入 Facebook 的 nvdtools/wfn 包
    "github.com/mitchellh/go-homedir" // 导入 mitchellh 的 go-homedir 包

    "github.com/anchore/packageurl-go" // 导入 anchore 的 packageurl-go 包
    "github.com/anchore/syft/syft/cpe" // 导入 anchore 的 syft/cpe 包
    "github.com/anchore/syft/syft/pkg" // 导入 anchore 的 syft/pkg 包
)

const (
    purlInputPrefix  = "purl:" // 定义 purl 输入的前缀
    cpesQualifierKey = "cpes" // 定义 cpes 的限定符键
)

type errEmptyPurlFile struct {
    purlFilepath string // 定义空 purl 文件的错误类型
}

func (e errEmptyPurlFile) Error() string {
    return fmt.Sprintf("purl file is empty: %s", e.purlFilepath) // 返回空 purl 文件的错误信息
}

func purlProvider(userInput string) ([]Package, error) {
    p, err := getPurlPackages(userInput) // 调用 getPurlPackages 函数获取 purl 包列表
    return p, err // 返回 purl 包列表和可能的错误
}

func getPurlPackages(userInput string) ([]Package, error) {
    reader, err := getPurlReader(userInput) // 获取 purl 输入的读取器
    if err != nil {
        return nil, err // 如果出现错误，返回空列表和错误
    }

    return decodePurlFile(reader) // 调用 decodePurlFile 函数解码 purl 文件
}

func decodePurlFile(reader io.Reader) ([]Package, error) {
    scanner := bufio.NewScanner(reader) // 创建一个读取器
    packages := []Package{} // 创建一个空的包列表
    # 遍历扫描器扫描的每一行数据
    for scanner.Scan() {
        # 获取原始行数据
        rawLine := scanner.Text()
        # 将原始行数据解析为 Package URL 对象
        purl, err := packageurl.FromString(rawLine)
        # 如果解析出错，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("unable to decode purl %s: %w", rawLine, err)
        }

        # 初始化 CPEs 数组和 epoch 变量
        cpes := []wfn.Attributes{}
        epoch := "0"
        # 遍历 Package URL 对象的限定符
        for _, qualifier := range purl.Qualifiers {
            # 如果限定符的键为 cpesQualifierKey，则解析 CPEs 数据并添加到数组中
            if qualifier.Key == cpesQualifierKey {
                rawCpes := strings.Split(qualifier.Value, ",")
                for _, rawCpe := range rawCpes {
                    c, err := cpe.New(rawCpe)
                    if err != nil {
                        return nil, fmt.Errorf("unable to decode cpe %s in purl %s: %w", rawCpe, rawLine, err)
                    }
                    cpes = append(cpes, c)
                }
            }
            # 如果限定符的键为 "epoch"，则更新 epoch 变量的值
            if qualifier.Key == "epoch" {
                epoch = qualifier.Value
            }
        }

        # 如果 Package URL 的类型为 TypeRPM，并且版本号不以 epoch 开头，则添加 epoch 前缀
        if purl.Type == packageurl.TypeRPM && !strings.HasPrefix(purl.Version, fmt.Sprintf("%s:", epoch)) {
            purl.Version = fmt.Sprintf("%s:%s", epoch, purl.Version)
        }

        # 将解析后的数据添加到 packages 数组中
        packages = append(packages, Package{
            ID:       ID(purl.String()),
            CPEs:     cpes,
            Name:     purl.Name,
            Version:  purl.Version,
            Type:     pkg.TypeByName(purl.Type),
            Language: pkg.LanguageByName(purl.Type),
            PURL:     purl.String(),
        })
    }

    # 检查扫描器是否出错，如果有错误则返回错误信息
    if err := scanner.Err(); err != nil {
        return nil, err
    }
    # 返回解析后的 packages 数组和 nil 错误信息
    return packages, nil
# 获取 PURL 读取器，接受用户输入并返回读取器和错误信息
func getPurlReader(userInput string) (r io.Reader, err error) {
    # 如果用户输入没有明确指定 PURL，则返回空和错误信息
    if !explicitlySpecifyingPurl(userInput) {
        return nil, errDoesNotProvide
    }

    # 去除用户输入中的 PURL 前缀，得到路径
    path := strings.TrimPrefix(userInput, purlInputPrefix)

    # 打开 PURL 文件并返回文件对象
    return openPurlFile(path)
}

# 打开 PURL 文件，返回文件对象和错误信息
func openPurlFile(path string) (*os.File, error) {
    # 展开路径中的波浪号（~），并处理可能出现的错误
    expandedPath, err := homedir.Expand(path)
    if err != nil {
        return nil, fmt.Errorf("unable to open purls: %w", err)
    }

    # 打开展开后的文件路径，并处理可能出现的错误
    f, err := os.Open(expandedPath)
    if err != nil {
        return nil, fmt.Errorf("unable to open file %s: %w", expandedPath, err)
    }

    # 检查文件是否为空，如果为空则返回错误信息
    if !fileHasContent(f) {
        return nil, errEmptyPurlFile{path}
    }

    # 返回文件对象和空错误信息
    return f, nil
}

# 检查用户输入是否明确指定了 PURL
func explicitlySpecifyingPurl(userInput string) bool {
    return strings.HasPrefix(userInput, purlInputPrefix)
}
```