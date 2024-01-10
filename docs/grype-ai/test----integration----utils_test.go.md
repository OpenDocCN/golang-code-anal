# `grype\test\integration\utils_test.go`

```
package integration

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "errors" // 导入 errors 包，用于处理错误
    "fmt" // 导入 fmt 包，用于格式化输出
    "os" // 导入 os 包，提供对操作系统功能的访问
    "os/exec" // 导入 exec 包，用于执行外部命令
    "path/filepath" // 导入 filepath 包，用于处理文件路径
    "regexp" // 导入 regexp 包，用于正则表达式
    "strings" // 导入 strings 包，用于处理字符串
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/scylladb/go-set/strset" // 导入第三方包
    "github.com/stretchr/testify/require" // 导入第三方包

    "github.com/anchore/grype/grype/match" // 导入自定义包
    "github.com/anchore/syft/syft" // 导入自定义包
    "github.com/anchore/syft/syft/pkg/cataloger" // 导入自定义包
    "github.com/anchore/syft/syft/sbom" // 导入自定义包
    "github.com/anchore/syft/syft/source" // 导入自定义包
)

const cacheDirRelativePath string = "./test-fixtures/cache" // 定义常量 cacheDirRelativePath

func PullThroughImageCache(t testing.TB, imageName string) string {
    cacheDirectory, absErr := filepath.Abs(cacheDirRelativePath) // 获取缓存目录的绝对路径
    if absErr != nil {
        t.Fatalf("could not get absolute path of cache directory %s; %v", cacheDirRelativePath, absErr) // 如果获取失败，输出错误信息
    }

    mkdirError := os.MkdirAll(cacheDirectory, 0755) // 创建缓存目录
    if mkdirError != nil {
        t.Fatalf("could not create cache directory %s; %v", cacheDirRelativePath, absErr) // 如果创建失败，输出错误信息
    }

    re := regexp.MustCompile("[/:]") // 创建正则表达式对象
    archiveFileName := fmt.Sprintf("%s.tar", re.ReplaceAllString(imageName, "-")) // 根据正则表达式替换文件名
    imageArchivePath := filepath.Join(cacheDirectory, archiveFileName) // 拼接文件路径

    if _, err := os.Stat(imageArchivePath); os.IsNotExist(err) { // 判断文件是否存在
        t.Logf("Cache miss for image %s; copying to archive at %s", imageName, imageArchivePath) // 输出缓存未命中信息
        saveImage(t, imageName, imageArchivePath) // 复制镜像到缓存目录
    }

    return imageArchivePath // 返回镜像存档路径
}

func saveImage(t testing.TB, imageName string, destPath string) {
    sourceImage := fmt.Sprintf("docker://docker.io/%s", imageName) // 构建源镜像地址
    destinationString := fmt.Sprintf("docker-archive:%s", destPath) // 构建目标镜像地址
    skopeoPath := filepath.Join(repoRoot(t), ".tmp", "skopeo") // 获取 skopeo 路径
    policyPath := filepath.Join(repoRoot(t), "test", "integration", "test-fixtures", "skopeo-policy.json") // 获取策略文件路径

    skopeoCommand := []string{ // 构建 skopeo 命令
        "--policy", policyPath,
        "copy", "--override-os", "linux", sourceImage, destinationString,
    }

    cmd := exec.Command(skopeoPath, skopeoCommand...) // 创建执行 skopeo 命令的对象

    out, err := cmd.Output() // 执行命令并获取输出
    # 如果错误不为空
    if err != nil:
        # 声明一个指向 exec.ExitError 类型的指针变量 exitError
        var exitError *exec.ExitError
        # 如果错误可以被转换为 exec.ExitError 类型，则执行以下代码
        if errors.As(err, &exitError):
            # 输出 exitError 的标准错误信息
            t.Logf("Stderr: %s", exitError.Stderr)
        # 输出错误信息并终止测试
        t.Fatal(err)

    # 输出标准输出信息
    t.Logf("Stdout: %s\n", out)
// 获取给定镜像的软件构建物件模型（SBOM），并使用指定的编码器进行编码
func getSyftSBOM(t testing.TB, image string, encoder sbom.FormatEncoder) string {
    // 检测给定镜像的软件包信息
    detection, err := source.Detect(image, source.DetectConfig{})
    if err != nil {
        t.Fatalf("could not generate source input for packages command: %+v", err)
    }

    // 根据检测结果创建新的软件包源
    src, err := detection.NewSource(source.DetectionSourceConfig{})
    if err != nil {
        t.Fatalf("can't get the source: %+v", err)
    }
    // 在测试结束时关闭软件包源
    t.Cleanup(func() {
        require.NoError(t, src.Close())
    })

    // 配置默认的目录管理器
    config := cataloger.DefaultConfig()
    config.Search.Scope = source.SquashedScope
    // TODO: relationships are not verified at this time
    // 获取软件包集合、关系和 Linux 发行版信息
    collection, relationships, distro, err := syft.CatalogPackages(src, config)

    // 创建 SBOM 对象
    s := sbom.SBOM{
        Artifacts: sbom.Artifacts{
            Packages:          collection,
            LinuxDistribution: distro,
        },
        Relationships: relationships,
        Source:        src.Describe(),
    }

    var buf bytes.Buffer

    // 使用编码器对 SBOM 对象进行编码
    err = encoder.Encode(&buf, s)
    require.NoError(t, err)

    return buf.String()
}

// 获取匹配集合的字符串集合
func getMatchSet(matches match.Matches) *strset.Set {
    s := strset.New()
    // 遍历匹配集合，将匹配信息格式化为字符串并添加到集合中
    for _, m := range matches.Sorted() {
        s.Add(fmt.Sprintf("%s-%s-%s", m.Vulnerability.ID, m.Package.Name, m.Package.Version))
    }
    return s
}

// 获取仓库根目录
func repoRoot(tb testing.TB) string {
    tb.Helper()
    // 执行命令获取 Git 仓库的根目录
    root, err := exec.Command("git", "rev-parse", "--show-toplevel").Output()
    if err != nil {
        tb.Fatalf("unable to find repo root dir: %+v", err)
    }
    // 获取仓库根目录的绝对路径
    absRepoRoot, err := filepath.Abs(strings.TrimSpace(string(root)))
    if err != nil {
        tb.Fatal("unable to get abs path to repo root:", err)
    }
    return absRepoRoot
}
```