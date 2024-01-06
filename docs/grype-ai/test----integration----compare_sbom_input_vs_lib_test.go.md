# `grype\test\integration\compare_sbom_input_vs_lib_test.go`

```
package integration
// 导入所需的包

import (
	"fmt"
	"os"
	"testing"

	"github.com/scylladb/go-set/strset"
	"github.com/stretchr/testify/assert"

	"github.com/anchore/grype/grype"
	"github.com/anchore/grype/grype/db"
	"github.com/anchore/grype/internal"
	"github.com/anchore/syft/syft/format/spdxjson"
	"github.com/anchore/syft/syft/format/spdxtagvalue"
	"github.com/anchore/syft/syft/format/syftjson"
	syftPkg "github.com/anchore/syft/syft/pkg"
	"github.com/anchore/syft/syft/sbom"
	"github.com/anchore/syft/syft/source"
)
// 导入所需的包，包括测试框架、断言库、以及其他自定义的包
# 创建一个字符串数组，包含有漏洞的图像名称
var imagesWithVulnerabilities = []string{
	"anchore/test_images:vulnerabilities-alpine",
	"anchore/test_images:gems",
	"anchore/test_images:vulnerabilities-debian",
	"anchore/test_images:vulnerabilities-centos",
	"anchore/test_images:npm",
	"anchore/test_images:java",
	"anchore/test_images:golang-56d52bc",
	"anchore/test_images:arch",
}

# 获取用于更新数据库的 URL 地址
func getListingURL() string {
	# 检查环境变量中是否存在 GRYPE_DB_UPDATE_URL，如果存在则返回其值
	if value, ok := os.LookupEnv("GRYPE_DB_UPDATE_URL"); ok {
		return value
	}
	# 如果环境变量中不存在 GRYPE_DB_UPDATE_URL，则返回内部默认的 DBUpdateURL
	return internal.DBUpdateURL
}

# 检查错误并返回格式化编码器
func must(e sbom.FormatEncoder, err error) sbom.FormatEncoder {
// 如果 err 不为 nil，则触发 panic
if err != nil {
    panic(err)
}
// 返回 e
return e
}

func TestCompareSBOMInputToLibResults(t *testing.T) {
    // 获取一个 grype 数据库
    store, _, closer, err := grype.LoadVulnerabilityDB(db.Config{
        DBRootDir:           "test-fixtures/grype-db",
        ListingURL:          getListingURL(),
        ValidateByHashOnGet: false,
    }, true)
    // 断言 err 为 nil
    assert.NoError(t, err)

    // 如果 closer 不为 nil，则延迟关闭
    if closer != nil {
        defer closer.Close()
    }

    // 创建一个新的字符串集合
    definedPkgTypes := strset.New()
# 遍历 syftPkg.AllPkgs 中的所有元素，并将它们添加到 definedPkgTypes 集合中
for _, p := range syftPkg.AllPkgs {
    definedPkgTypes.Add(string(p))
}
# 从 definedPkgTypes 集合中移除指定的包类型，这些类型不在测试范围内
definedPkgTypes.Remove(
    string(syftPkg.BinaryPkg), 
    string(syftPkg.RustPkg),
    string(syftPkg.KbPkg),
    string(syftPkg.DartPubPkg),
    string(syftPkg.DotnetPkg),
    string(syftPkg.PhpComposerPkg),
    string(syftPkg.ConanPkg),
    string(syftPkg.HexPkg),
    string(syftPkg.PortagePkg),
    string(syftPkg.CocoapodsPkg),
    string(syftPkg.HackagePkg),
    string(syftPkg.NixPkg),
    string(syftPkg.JenkinsPluginPkg), 
    string(syftPkg.LinuxKernelPkg),
    string(syftPkg.LinuxKernelModulePkg),
)
		string(syftPkg.Rpkg), // 将syftPkg.Rpkg转换为字符串
		string(syftPkg.SwiftPkg), // 将syftPkg.SwiftPkg转换为字符串
		string(syftPkg.GithubActionPkg), // 将syftPkg.GithubActionPkg转换为字符串
		string(syftPkg.GithubActionWorkflowPkg), // 将syftPkg.GithubActionWorkflowPkg转换为字符串
	)
	observedPkgTypes := strset.New() // 创建一个新的字符串集合observedPkgTypes
	testCases := []struct { // 创建一个结构体切片testCases
		name   string // 定义结构体字段name为字符串类型
		image  string // 定义结构体字段image为字符串类型
		format sbom.FormatEncoder // 定义结构体字段format为sbom.FormatEncoder类型
	}{
		{
			image:  "anchore/test_images:vulnerabilities-alpine", // 初始化结构体字段image
			format: syftjson.NewFormatEncoder(), // 初始化结构体字段format为syftjson.NewFormatEncoder()的返回值
			name:   "alpine-syft-json", // 初始化结构体字段name
		},

		{
			image:  "anchore/test_images:vulnerabilities-alpine", // 初始化结构体字段image
			format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())), // 初始化结构体字段format为spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())的返回值
```

# 定义一个包含镜像信息的列表，每个元素是一个字典
# 第一个元素
{
    # 镜像名称
    image:  "anchore/test_images:alpine",
    # 使用默认配置创建 SPDX JSON 格式的编码器
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    # 名称
    name:   "alpine-spdx-json",
},

# 第二个元素
{
    # 镜像名称
    image:  "anchore/test_images:vulnerabilities-alpine",
    # 使用默认配置创建 SPDX Tag-Value 格式的编码器
    format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),
    # 名称
    name:   "alpine-spdx-tag-value",
},

# 第三个元素
{
    # 镜像名称
    image:  "anchore/test_images:gems",
    # 使用 Syft JSON 格式的编码器
    format: syftjson.NewFormatEncoder(),
    # 名称
    name:   "gems-syft-json",
},

# 第四个元素
{
    # 镜像名称
    image:  "anchore/test_images:gems",
    # 使用默认配置创建 SPDX JSON 格式的编码器
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    # 名称
    name:   "gems-spdx-json",
},
# 创建一个包含镜像信息的对象，包括镜像名称和格式
{
    # 镜像名称
    image:  "anchore/test_images:gems",
    # 使用默认配置创建 SPDX 格式编码器
    format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),
    # 镜像名称
    name:   "gems-spdx-tag-value",
},

{
    # 镜像名称
    image:  "anchore/test_images:vulnerabilities-debian",
    # 使用 Syft JSON 格式编码器
    format: syftjson.NewFormatEncoder(),
    # 镜像名称
    name:   "debian-syft-json",
},

{
    # 镜像名称
    image:  "anchore/test_images:vulnerabilities-debian",
    # 使用默认配置创建 SPDX JSON 格式编码器
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    # 镜像名称
    name:   "debian-spdx-json",
},
# 定义一个镜像名称为 "anchore/test_images:vulnerabilities-debian" 的镜像信息
# 使用 SPDXTAGVALUE 格式编码器，并使用默认配置
# 定义名称为 "debian-spdx-tag-value" 的镜像格式
{
    image:  "anchore/test_images:vulnerabilities-debian",
    format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),
    name:   "debian-spdx-tag-value",
},

# 定义一个镜像名称为 "anchore/test_images:vulnerabilities-centos" 的镜像信息
# 使用 SYFTJSON 格式编码器
# 定义名称为 "centos-syft-json" 的镜像格式
{
    image:  "anchore/test_images:vulnerabilities-centos",
    format: syftjson.NewFormatEncoder(),
    name:   "centos-syft-json",
},

# 定义一个镜像名称为 "anchore/test_images:vulnerabilities-centos" 的镜像信息
# 使用 SPDJSON 格式编码器，并使用默认配置
# 定义名称为 "centos-spdx-json" 的镜像格式
{
    image:  "anchore/test_images:vulnerabilities-centos",
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    name:   "centos-spdx-json",
},

# 定义一个镜像名称为 "anchore/test_images:vulnerabilities-centos" 的镜像信息
# 使用 SPDXTAGVALUE 格式编码器，并使用默认配置
# 创建一个包含镜像信息的结构体，包括镜像名称和格式
{
    image:  "anchore/test_images:centos",
    format: syftjson.NewFormatEncoder(),
    name:   "centos-syft-json",
},

# 创建一个包含镜像信息的结构体，包括镜像名称和格式
{
    image:  "anchore/test_images:centos",
    format: syftjson.NewFormatEncoder(),
    name:   "centos-syft-json",
},

# 创建一个包含镜像信息的结构体，包括镜像名称和格式
{
    image:  "anchore/test_images:npm",
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    name:   "npm-spdx-json",
},

# 创建一个包含镜像信息的结构体，包括镜像名称和格式
{
    image:  "anchore/test_images:npm",
    format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),
    name:   "npm-spdx-tag-value",
},
# 创建一个包含镜像信息的对象，镜像名称为 "anchore/test_images:java"，格式为 syftjson
{
    image:  "anchore/test_images:java",
    format: syftjson.NewFormatEncoder(),
    name:   "java-syft-json",
},

# 创建一个包含镜像信息的对象，镜像名称为 "anchore/test_images:java"，格式为 spdxjson
{
    image:  "anchore/test_images:java",
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    name:   "java-spdx-json",
},

# 创建一个包含镜像信息的对象，镜像名称为 "anchore/test_images:java"，格式为 spdxtagvalue
{
    image:  "anchore/test_images:java",
    format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),
    name:   "java-spdx-tag-value",
},
# 定义一个包含镜像信息的数据结构，包括镜像名称和格式编码器
{
    # 镜像名称
    image:  "anchore/test_images:golang-56d52bc",
    # 使用 syftjson 格式的编码器
    format: syftjson.NewFormatEncoder(),
    # 格式名称
    name:   "go-syft-json",
},

{
    # 镜像名称
    image:  "anchore/test_images:golang-56d52bc",
    # 使用 spdxjson 格式的编码器
    format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),
    # 格式名称
    name:   "go-spdx-json",
},

{
    # 镜像名称
    image:  "anchore/test_images:golang-56d52bc",
    # 使用 spdxtagvalue 格式的编码器
    format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),
    # 格式名称
    name:   "go-spdx-tag-value",
},

{
    # 镜像名称
    image:  "anchore/test_images:arch",
    # 使用 syftjson 格式的编码器
    format: syftjson.NewFormatEncoder(),
    # 格式名称
    name:   "arch-syft-json",
},
# 定义测试用例，每个测试用例包含镜像名称、格式编码器和名称
testCases := []struct {
		image  string  // 镜像名称
		format FormatEncoder  // 格式编码器
		name   string  // 名称
	}{
		{
			image:  "anchore/test_images:arch",
			format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),  // 使用默认配置创建 JSON 格式编码器
			name:   "arch-syft-json",  // 设置名称
		},

		{
			image:  "anchore/test_images:arch",
			format: must(spdxjson.NewFormatEncoderWithConfig(spdxjson.DefaultEncoderConfig())),  // 使用默认配置创建 JSON 格式编码器
			name:   "arch-spdx-json",  // 设置名称
		},

		{
			image:  "anchore/test_images:arch",
			format: must(spdxtagvalue.NewFormatEncoderWithConfig(spdxtagvalue.DefaultEncoderConfig())),  // 使用默认配置创建 SPDX 标签值格式编码器
			name:   "arch-spdx-tag-value",  // 设置名称
		},
	}
	# 遍历测试用例
	for _, tc := range testCases {
		# 从镜像缓存中拉取镜像
		imageArchive := PullThroughImageCache(t, tc.image)
		# 格式化镜像来源
		imageSource := fmt.Sprintf("docker-archive:%s", imageArchive)

		# 运行测试用例
		t.Run(tc.name, func(t *testing.T) {
// 从syft获取SBOM，写入临时文件
sbomBytes := getSyftSBOM(t, imageSource, tc.format)
sbomFile, err := os.CreateTemp("", "")
assert.NoError(t, err)
// 在测试结束时删除临时文件
t.Cleanup(func() {
    assert.NoError(t, os.Remove(sbomFile.Name()))
})
// 将SBOM数据写入临时文件
_, err = sbomFile.WriteString(sbomBytes)
assert.NoError(t, err)
assert.NoError(t, sbomFile.Close())

// 获取漏洞信息（从SBOM）
matchesFromSbom, _, pkgsFromSbom, err := grype.FindVulnerabilities(*store, fmt.Sprintf("sbom:%s", sbomFile.Name()), source.SquashedScope, nil)
assert.NoError(t, err)

// 获取漏洞信息（从镜像）
matchesFromImage, _, _, err := grype.FindVulnerabilities(*store, imageSource, source.SquashedScope, nil)
assert.NoError(t, err)

// 比较软件包（浅层比较）
// 从软件构建清单（sbom）中获取匹配集合
matchSetFromSbom := getMatchSet(matchesFromSbom)
// 从镜像中获取匹配集合
matchSetFromImage := getMatchSet(matchesFromImage)

// 断言：确保使用软件构建清单作为输入时，结果中仅包含的漏洞为空
assert.Empty(t, strset.Difference(matchSetFromSbom, matchSetFromImage).List(), "vulnerabilities present only in results when using sbom as input")
// 断言：确保使用镜像作为输入时，结果中仅包含的漏洞为空
assert.Empty(t, strset.Difference(matchSetFromImage, matchSetFromSbom).List(), "vulnerabilities present only in results when using image as input")

// 跟踪所有已覆盖的软件包类型（用于测试后使用）
for _, p := range pkgsFromSbom {
    observedPkgTypes.Add(string(p.Type))
}

// 结束测试
})

// 确保我们已经覆盖了所有的软件包类型（-rust，-kb）
unobservedPackageTypes := strset.Difference(definedPkgTypes, observedPkgTypes)
// 断言：确保在测试中覆盖了所有的软件包类型
assert.Empty(t, unobservedPackageTypes.List(), "not all package type were covered in testing")
```