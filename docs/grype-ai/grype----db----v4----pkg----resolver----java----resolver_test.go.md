# `grype\grype\db\v4\pkg\resolver\java\resolver_test.go`

```
package java
// 导入所需的包

import (
	"testing"
	// 导入测试包

	"github.com/google/uuid"
	// 导入 uuid 包
	"github.com/stretchr/testify/assert"
	// 导入 testify 包

	grypePkg "github.com/anchore/grype/grype/pkg"
	// 导入 grypePkg 包
)

func TestResolver_Normalize(t *testing.T) {
	// 定义测试函数 TestResolver_Normalize
	tests := []struct {
		packageName string
		normalized  string
	}{
		// 定义测试用例
		{
			packageName: "PyYAML",
			normalized:  "pyyaml",
		},
		// 测试用例1
# 创建一个包含不同包名和其规范化形式的对象数组
{
    packageName: "oslo.concurrency",  # 包名为"oslo.concurrency"
    normalized:  "oslo.concurrency",  # 规范化形式也为"oslo.concurrency"
},
{
    packageName: "",  # 空包名
    normalized:  "",  # 空的规范化形式
},
{
    packageName: "test---1",  # 包名为"test---1"
    normalized:  "test---1",  # 规范化形式也为"test---1"
},
{
    packageName: "AbCd.-__.--.-___.__.--1234____----....XyZZZ",  # 包名为"AbCd.-__.--.-___.__.--1234____----....XyZZZ"
    normalized:  "abcd.-__.--.-___.__.--1234____----....xyzzz",  # 规范化形式为"abcd.-__.--.-___.__.--1234____----....xyzzz"
}

# 创建一个名为resolver的Resolver对象
resolver := Resolver{}
# 遍历测试用例列表
for _, test := range tests:
    # 对包名进行规范化处理
    resolvedNames := resolver.Normalize(test.packageName)
    # 断言规范化后的包名与预期结果相等
    assert.Equal(t, resolvedNames, test.normalized)
}

# 测试解析器的解析功能
func TestResolver_Resolve(t *testing.T):
    # 测试用例列表
    tests := []struct {
        name     string
        pkg      grypePkg.Package
        resolved []string
    }{
        # 测试用例1
        {
            name: "both artifact and manifest 1",
            pkg: grypePkg.Package{
                Name:     "ABCD",
                Version:  "1.2.3.4",
                Language: "java",
                Metadata: grypePkg.JavaMetadata{
                    VirtualPath:   "virtual-path-info",
# 设置PomArtifactID、PomGroupID和ManifestName字段的值为对应的信息
PomArtifactID: "pom-ARTIFACT-ID-info",
PomGroupID:    "pom-group-ID-info",
ManifestName:  "main-section-name-info",
# 设置resolved字段的值为包含两个字符串的列表
resolved: []string{"pom-group-id-info:pom-artifact-id-info", "pom-group-id-info:main-section-name-info"},
# 设置name字段的值为"both artifact and manifest 2"
name: "both artifact and manifest 2",
# 设置pkg字段的值为一个包含ID、Name和Metadata字段的Package对象
pkg: grypePkg.Package{
    ID:   grypePkg.ID(uuid.NewString()),
    Name: "a-name",
    # 设置Metadata字段的值为一个包含VirtualPath、PomArtifactID、PomGroupID和ManifestName字段的JavaMetadata对象
    Metadata: grypePkg.JavaMetadata{
        VirtualPath:   "v-path",
        PomArtifactID: "art-id",
        PomGroupID:    "g-id",
        ManifestName:  "man-name",
    },
},
# 设置resolved字段的值为一个包含字符串的列表
resolved: []string{
# 定义一个测试用例，包含不同情况下的包信息
{
    name: "with group id",
    pkg: grypePkg.Package{
        ID:   grypePkg.ID(uuid.NewString()),  # 生成一个新的唯一 ID
        Name: "a-name",  # 包的名称
        Metadata: grypePkg.JavaMetadata{  # Java 包的元数据
            VirtualPath:   "v-path",  # 虚拟路径
            PomArtifactID: "art-id",  # POM 文件的 Artifact ID
            ManifestName:  "man-name",  # Manifest 文件的名称
        },
        Resolved: []string{  # 已解析的依赖包列表
            "g-id:art-id",  # 依赖包的组 ID 和 Artifact ID
            "g-id:man-name",  # 依赖包的组 ID 和 Manifest 文件的名称
        },
    },
},
{
    name: "no group id",
    pkg: grypePkg.Package{
        ID:   grypePkg.ID(uuid.NewString()),  # 生成一个新的唯一 ID
        Name: "a-name",  # 包的名称
        Metadata: grypePkg.JavaMetadata{  # Java 包的元数据
            VirtualPath:   "v-path",  # 虚拟路径
            PomArtifactID: "art-id",  # POM 文件的 Artifact ID
            ManifestName:  "man-name",  # Manifest 文件的名称
        },
        Resolved: []string{},  # 没有已解析的依赖包
    },
},
{
    name: "only manifest",
    pkg: grypePkg.Package{  # 定义一个只包含 Manifest 信息的包
# 创建一个新的唯一标识符作为包的ID
ID:   grypePkg.ID(uuid.NewString()),
# 设置包的名称为"a-name"
Name: "a-name",
# 设置包的元数据为JavaMetadata类型，包括虚拟路径、POM组ID和清单名称
Metadata: grypePkg.JavaMetadata{
    VirtualPath:  "v-path",
    PomGroupID:   "g-id",
    ManifestName: "man-name",
},
# 设置已解析的依赖列表，包括组ID和清单名称的组合
resolved: []string{
    "g-id:man-name",
},
# 创建另一个包对象，设置名称为"only artifact"
{
    name: "only artifact",
    # 设置包的ID
    pkg: grypePkg.Package{
        ID:   grypePkg.ID(uuid.NewString()),
        # 设置包的名称为"a-name"
        Name: "a-name",
        # 设置包的元数据为JavaMetadata类型，包括虚拟路径和POM的artifactID
        Metadata: grypePkg.JavaMetadata{
            VirtualPath:   "v-path",
            PomArtifactID: "art-id",
# 定义一个包含PomGroupID的PomGroupID字段的结构体
PomGroupID:    "g-id",
# 定义一个包含resolved字段的结构体
			},
			# 定义一个包含resolved字段的结构体
			resolved: []string{
				"g-id:art-id",
			},
		},
		# 定义一个包含name、pkg和resolved字段的结构体
		{
			# 定义一个包含name、pkg和resolved字段的结构体
			name: "no artifact or manifest",
			# 定义一个包含ID、Name和Metadata字段的结构体
			pkg: grypePkg.Package{
				ID:   grypePkg.ID(uuid.NewString()),
				Name: "a-name",
				# 定义一个包含VirtualPath和PomGroupID字段的JavaMetadata结构体
				Metadata: grypePkg.JavaMetadata{
					VirtualPath: "v-path",
					PomGroupID:  "g-id",
				},
			},
			# 定义一个空的字符串数组
			resolved: []string{},
		},
		{
# 创建一个名为 "with valid purl" 的测试用例
{
    # 设置包的属性
    name: "with valid purl",
    pkg: grypePkg.Package{
        # 设置包的唯一标识符
        ID:   grypePkg.ID(uuid.NewString()),
        # 设置包的名称
        Name: "a-name",
        # 设置包的 PURL
        PURL: "pkg:maven/org.anchore/b-name@0.2",
    },
    # 设置已解析的字符串列表
    resolved: ["org.anchore:b-name"],
},
# 创建一个名为 "ignore invalid pURLs" 的测试用例
{
    # 设置包的属性
    name: "ignore invalid pURLs",
    pkg: grypePkg.Package{
        # 设置包的唯一标识符
        ID:   grypePkg.ID(uuid.NewString()),
        # 设置包的名称
        Name: "a-name",
        # 设置包的 PURL
        PURL: "pkg:BAD/",
        # 设置包的元数据
        Metadata: grypePkg.JavaMetadata{
            # 设置虚拟路径
            VirtualPath:   "v-path",
            # 设置 POM 项目的 artifact ID
            PomArtifactID: "art-id",
            # 设置 POM 项目的 group ID
            PomGroupID:    "g-id",
        },
    },
}
# 创建一个包含字符串的切片，用于存储解析后的结果
resolved: []string{
    "g-id:art-id",
},

# 创建一个测试用例的结构体切片，包含包名和期望的解析结果
tests := []struct {
    name     string
    pkg      string
    resolved []string
}{
    {
        name: "test case name",
        pkg:  "package name",
        resolved: []string{
            "g-id:art-id",
        },
    },
}

# 创建一个解析器对象
resolver := Resolver{}

# 遍历测试用例切片，对每个测试用例进行解析并进行断言
for _, test := range tests {
    t.Run(test.name, func(t *testing.T) {
        resolvedNames := resolver.Resolve(test.pkg)
        assert.ElementsMatch(t, resolvedNames, test.resolved)
    })
}
```