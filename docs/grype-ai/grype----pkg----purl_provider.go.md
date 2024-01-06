# `grype\grype\pkg\purl_provider.go`

```
// 声明包名为 pkg
package pkg

// 导入所需的包
import (
	"bufio" // 读取输入输出的包
	"fmt" // 格式化输出的包
	"io" // 输入输出的包
	"os" // 操作系统功能的包
	"strings" // 字符串操作的包

	"github.com/facebookincubator/nvdtools/wfn" // 导入 Facebook 开发的 wfn 包
	"github.com/mitchellh/go-homedir" // 导入 mitchellh 开发的 go-homedir 包

	"github.com/anchore/packageurl-go" // 导入 anchore 开发的 packageurl-go 包
	"github.com/anchore/syft/syft/cpe" // 导入 anchore 开发的 cpe 包
	"github.com/anchore/syft/syft/pkg" // 导入 anchore 开发的 pkg 包
)

// 声明常量
const (
	purlInputPrefix  = "purl:" // 定义 purl 输入的前缀
	cpesQualifierKey = "cpes" // 定义 cpes 的限定符键
// 定义一个自定义错误类型，用于表示 purl 文件为空的错误
type errEmptyPurlFile struct {
	purlFilepath string
}

// 实现自定义错误类型的 Error 方法，返回错误信息
func (e errEmptyPurlFile) Error() string {
	return fmt.Sprintf("purl file is empty: %s", e.purlFilepath)
}

// purlProvider 函数，根据用户输入获取 purl 包列表
func purlProvider(userInput string) ([]Package, error) {
	// 调用 getPurlPackages 函数获取 purl 包列表和错误信息
	p, err := getPurlPackages(userInput)
	return p, err
}

// getPurlPackages 函数，根据用户输入获取 purl 包列表
func getPurlPackages(userInput string) ([]Package, error) {
	// 调用 getPurlReader 函数获取 purl 读取器和错误信息
	reader, err := getPurlReader(userInput)
	// 如果出现错误，返回空的包列表和错误信息
	if err != nil {
		return nil, err
	}
# 调用 decodePurlFile 函数并返回结果
return decodePurlFile(reader)
}

# 解码 Purl 文件内容并返回 Package 切片和错误信息
func decodePurlFile(reader io.Reader) ([]Package, error) {
    # 创建一个新的 Scanner 对象来扫描输入流
    scanner := bufio.NewScanner(reader)
    # 创建一个空的 Package 切片
    packages := []Package{}

    # 逐行扫描输入流
    for scanner.Scan() {
        # 获取当前行的原始内容
        rawLine := scanner.Text()
        # 将原始内容解析为 Package URL 对象
        purl, err := packageurl.FromString(rawLine)
        # 如果解析出错，则返回错误信息
        if err != nil {
            return nil, fmt.Errorf("unable to decode purl %s: %w", rawLine, err)
        }

        # 创建一个空的 wfn.Attributes 切片
        cpes := []wfn.Attributes{}
        # 设置 epoch 初始值为 "0"
        epoch := "0"
        # 遍历 Package URL 对象的 qualifiers
        for _, qualifier := range purl.Qualifiers {
            # 如果 qualifier 的 key 为 cpesQualifierKey
            if qualifier.Key == cpesQualifierKey {
                # 将 qualifier 的 value 按逗号分隔并存储到 rawCpes 中
# 遍历 rawCpes 数组中的每个元素
for _, rawCpe := range rawCpes {
    # 使用 rawCpe 创建一个新的 cpe 对象
    c, err := cpe.New(rawCpe)
    # 如果创建过程中出现错误，则返回错误信息
    if err != nil {
        return nil, fmt.Errorf("unable to decode cpe %s in purl %s: %w", rawCpe, rawLine, err)
    }
    # 将新创建的 cpe 对象添加到 cpes 数组中
    cpes = append(cpes, c)
}

# 如果 qualifier 的 Key 为 "epoch"，则将其对应的 Value 赋值给 epoch
if qualifier.Key == "epoch" {
    epoch = qualifier.Value
}

# 如果 purl 的 Type 为 packageurl.TypeRPM，并且其 Version 不以 epoch 开头，则在其 Version 前加上 epoch
if purl.Type == packageurl.TypeRPM && !strings.HasPrefix(purl.Version, fmt.Sprintf("%s:", epoch)) {
    purl.Version = fmt.Sprintf("%s:%s", epoch, purl.Version)
}

# 将当前 purl 对象的信息添加到 packages 数组中
packages = append(packages, Package{
    ID:       ID(purl.String()),
# 将CPEs、Name、Version、Type、Language和PURL添加到packages切片中
packages = append(packages, Package{
    CPEs:     cpes,
    Name:     purl.Name,
    Version:  purl.Version,
    Type:     pkg.TypeByName(purl.Type),
    Language: pkg.LanguageByName(purl.Type),
    PURL:     purl.String(),
})

# 检查是否有错误发生，如果有则返回nil和错误信息
if err := scanner.Err(); err != nil {
    return nil, err
}

# 返回packages切片和nil，表示没有错误发生
return packages, nil
}

# 根据用户输入获取PURL读取器
func getPurlReader(userInput string) (r io.Reader, err error) {
    # 如果用户输入没有明确指定PURL，则返回nil和错误信息
    if !explicitlySpecifyingPurl(userInput) {
        return nil, errDoesNotProvide
    }
// 去除用户输入中的指定前缀
path := strings.TrimPrefix(userInput, purlInputPrefix)

// 调用 openPurlFile 函数打开处理后的路径
return openPurlFile(path)
}

// 打开处理后的路径的文件
func openPurlFile(path string) (*os.File, error) {
	// 将路径中的波浪号（~）扩展为用户主目录
	expandedPath, err := homedir.Expand(path)
	if err != nil {
		return nil, fmt.Errorf("unable to open purls: %w", err)
	}

	// 打开扩展后的路径的文件
	f, err := os.Open(expandedPath)
	if err != nil {
		return nil, fmt.Errorf("unable to open file %s: %w", expandedPath, err)
	}

	// 检查文件是否为空
	if !fileHasContent(f) {
		return nil, errEmptyPurlFile{path}
	}
# 返回 f 和 nil，表示成功执行并返回结果
	return f, nil
# 检查用户输入是否以指定的前缀开头，返回布尔值
func explicitlySpecifyingPurl(userInput string) bool:
	return strings.HasPrefix(userInput, purlInputPrefix)
```