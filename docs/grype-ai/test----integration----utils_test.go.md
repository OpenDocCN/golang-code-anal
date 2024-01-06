# `grype\test\integration\utils_test.go`

```
// 导入 integration 包
package integration

// 导入所需的包
import (
	"bytes"  // 导入 bytes 包，用于操作字节
	"errors"  // 导入 errors 包，用于处理错误
	"fmt"  // 导入 fmt 包，用于格式化输出
	"os"  // 导入 os 包，用于操作系统功能
	"os/exec"  // 导入 exec 包，用于执行外部命令
	"path/filepath"  // 导入 filepath 包，用于处理文件路径
	"regexp"  // 导入 regexp 包，用于正则表达式匹配
	"strings"  // 导入 strings 包，用于字符串操作
	"testing"  // 导入 testing 包，用于编写测试用例

	"github.com/scylladb/go-set/strset"  // 导入第三方包
	"github.com/stretchr/testify/require"  // 导入第三方包

	"github.com/anchore/grype/grype/match"  // 导入 grype 包中的 match 模块
	"github.com/anchore/syft/syft"  // 导入 syft 包中的 syft 模块
	"github.com/anchore/syft/syft/pkg/cataloger"  // 导入 syft 包中的 cataloger 模块
	"github.com/anchore/syft/syft/sbom"  // 导入 syft 包中的 sbom 模块
```
以上是对给定代码的注释。
// 导入所需的包
import (
	"github.com/anchore/syft/syft/source"
)

// 定义缓存目录的相对路径常量
const cacheDirRelativePath string = "./test-fixtures/cache"

// 从镜像缓存中拉取镜像
func PullThroughImageCache(t testing.TB, imageName string) string {
	// 获取缓存目录的绝对路径
	cacheDirectory, absErr := filepath.Abs(cacheDirRelativePath)
	if absErr != nil {
		t.Fatalf("could not get absolute path of cache directory %s; %v", cacheDirRelativePath, absErr)
	}

	// 创建缓存目录
	mkdirError := os.MkdirAll(cacheDirectory, 0755)
	if mkdirError != nil {
		t.Fatalf("could not create cache directory %s; %v", cacheDirRelativePath, absErr)
	}

	// 定义正则表达式，用于替换文件名中的特殊字符
	re := regexp.MustCompile("[/:]")
	archiveFileName := fmt.Sprintf("%s.tar", re.ReplaceAllString(imageName, "-"))
	// 拼接镜像存档文件的路径
	imageArchivePath := filepath.Join(cacheDirectory, archiveFileName)
}
// 检查图片存档路径是否存在，如果不存在则输出日志并将图片复制到存档路径
if _, err := os.Stat(imageArchivePath); os.IsNotExist(err) {
    t.Logf("Cache miss for image %s; copying to archive at %s", imageName, imageArchivePath)
    saveImage(t, imageName, imageArchivePath)
}

// 返回图片存档路径
return imageArchivePath
}

// 保存图片到指定路径
func saveImage(t testing.TB, imageName string, destPath string) {
    // 构建源图片和目标路径字符串
    sourceImage := fmt.Sprintf("docker://docker.io/%s", imageName)
    destinationString := fmt.Sprintf("docker-archive:%s", destPath)
    // 构建 skopeo 工具路径和策略文件路径
    skopeoPath := filepath.Join(repoRoot(t), ".tmp", "skopeo")
    policyPath := filepath.Join(repoRoot(t), "test", "integration", "test-fixtures", "skopeo-policy.json")

    // 构建 skopeo 命令
    skopeoCommand := []string{
        "--policy", policyPath,
        "copy", "--override-os", "linux", sourceImage, destinationString,
    }

    // 执行 skopeo 命令
    cmd := exec.Command(skopeoPath, skopeoCommand...)
}
// 执行命令并获取输出
out, err := cmd.Output()
// 如果执行命令出现错误
if err != nil {
	// 如果错误是 ExitError 类型
	var exitError *exec.ExitError
	if errors.As(err, &exitError) {
		// 输出标准错误信息
		t.Logf("Stderr: %s", exitError.Stderr)
	}
	// 输出错误信息并终止测试
	t.Fatal(err)
}

// 输出标准输出信息
t.Logf("Stdout: %s\n", out)
}

// 获取 Syft SBOM
func getSyftSBOM(t testing.TB, image string, encoder sbom.FormatEncoder) string {
	// 检测镜像并生成源输入
	detection, err := source.Detect(image, source.DetectConfig{})
	if err != nil {
		// 如果无法生成源输入，输出错误信息并终止测试
		t.Fatalf("could not generate source input for packages command: %+v", err)
	}

	// 创建新的源
	src, err := detection.NewSource(source.DetectionSourceConfig{})
// 如果发生错误，输出错误信息并终止测试
if err != nil {
    t.Fatalf("can't get the source: %+v", err)
}
// 在测试结束时，确保关闭源
t.Cleanup(func() {
    require.NoError(t, src.Close())
})

// 创建默认配置
config := cataloger.DefaultConfig()
// 设置搜索范围为压缩范围
config.Search.Scope = source.SquashedScope
// TODO: 目前不验证关系

// 从源中目录软件包，并返回软件包集合、关系、Linux 发行版和可能的错误
collection, relationships, distro, err := syft.CatalogPackages(src, config)

// 创建 SBOM 结构
s := sbom.SBOM{
    Artifacts: sbom.Artifacts{
        Packages:          collection,
        LinuxDistribution: distro,
    },
    Relationships: relationships,
    Source:        src.Describe(),
}
// 创建一个字节缓冲区
var buf bytes.Buffer

// 使用编码器将结构体编码到字节缓冲区中
err = encoder.Encode(&buf, s)
require.NoError(t, err)

// 将字节缓冲区转换为字符串并返回
return buf.String()
}

// 获取匹配集合的字符串集合
func getMatchSet(matches match.Matches) *strset.Set {
    s := strset.New()
    for _, m := range matches.Sorted() {
        // 将匹配对象的信息格式化为字符串并添加到字符串集合中
        s.Add(fmt.Sprintf("%s-%s-%s", m.Vulnerability.ID, m.Package.Name, m.Package.Version))
    }
    return s
}

// 获取仓库根目录
func repoRoot(tb testing.TB) string {
    tb.Helper()
    // 执行命令获取git仓库的根目录
    root, err := exec.Command("git", "rev-parse", "--show-toplevel").Output()
# 如果发生错误，则输出错误信息并终止测试
if err != nil:
    tb.Fatalf("unable to find repo root dir: %+v", err)
# 获取去除空白字符后的根目录路径，并转换为绝对路径
absRepoRoot, err := filepath.Abs(strings.TrimSpace(string(root)))
# 如果发生错误，则输出错误信息并终止测试
if err != nil:
    tb.Fatal("unable to get abs path to repo root:", err)
# 返回绝对路径的根目录
return absRepoRoot
```