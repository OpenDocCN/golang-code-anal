# `grype\grype\presenter\models\document_test.go`

```
package models

import (
	"testing"  // 导入测试包
	"time"  // 导入时间包

	"github.com/stretchr/testify/assert"  // 导入断言包

	"github.com/anchore/clio"  // 导入 clio 包
	"github.com/anchore/grype/grype/match"  // 导入 match 包
	"github.com/anchore/grype/grype/pkg"  // 导入 pkg 包
	"github.com/anchore/grype/grype/vulnerability"  // 导入 vulnerability 包
	"github.com/anchore/syft/syft/linux"  // 导入 linux 包
	syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包并重命名为 syftPkg
	syftSource "github.com/anchore/syft/syft/source"  // 导入 source 包并重命名为 syftSource
)

func TestPackagesAreSorted(t *testing.T) {
    // 定义测试函数 TestPackagesAreSorted
	var pkg1 = pkg.Package{  // 定义变量 pkg1 为 pkg.Package 结构体
	// 定义一个名为 pkg1 的包对象，包含 ID、名称、版本和类型信息
	var pkg1 = pkg.Package{
		ID:      "package-1-id",
		Name:    "package-1",
		Version: "1.1.1",
		Type:    syftPkg.DebPkg,
	}

	// 定义一个名为 pkg2 的包对象，包含 ID、名称、版本和类型信息
	var pkg2 = pkg.Package{
		ID:      "package-2-id",
		Name:    "package-2",
		Version: "2.2.2",
		Type:    syftPkg.DebPkg,
	}

	// 定义一个名为 match1 的匹配对象，包含漏洞、包和详细信息
	var match1 = match.Match{
		Vulnerability: vulnerability.Vulnerability{
			ID: "CVE-1999-0003",
		},
		Package: pkg1, // 将 pkg1 包对象作为匹配对象的包属性
		Details: match.Details{
			// 匹配对象的详细信息
			{
				// 详细信息的内容
			},
		},
	}
# 创建一个名为match1的匹配对象，使用ExactDirectMatch类型进行匹配
var match1 = match.Match{
    Vulnerability: vulnerability.Vulnerability{
        ID: "CVE-1999-0001",
    },
    Package: pkg1,
    Details: match.Details{
        {
            Type: match.ExactDirectMatch,
        },
    },
}

# 创建一个名为match2的匹配对象，使用ExactIndirectMatch类型进行匹配
var match2 = match.Match{
    Vulnerability: vulnerability.Vulnerability{
        ID: "CVE-1999-0002",
    },
    Package: pkg1,
    Details: match.Details{
        {
            Type: match.ExactIndirectMatch,
        },
    },
}

# 创建一个名为match3的匹配对象，使用ExactDirectMatch类型进行匹配
var match3 = match.Match{
    Vulnerability: vulnerability.Vulnerability{
        ID: "CVE-1999-0001",
		},
		Package: pkg1,  // 设置 Package 字段为 pkg1
		Details: match.Details{  // 设置 Details 字段为 match.Details 结构体
			{
				Type: match.ExactIndirectMatch,  // 设置 Type 字段为 match.ExactIndirectMatch
			},
		},
	}

	matches := match.NewMatches()  // 创建一个新的匹配对象

	// 向 matches 对象中添加 match1, match2, match3 三个匹配对象
	matches.Add(match1, match2, match3)

	// 创建一个包含 pkg1, pkg2 两个包的数组
	packages := []pkg.Package{pkg1, pkg2}

	// 创建一个包含 Source 和 Distro 字段的 pkg.Context 对象
	ctx := pkg.Context{
		Source: &syftSource.Description{  // 设置 Source 字段为 syftSource.Description 结构体
			Metadata: syftSource.DirectorySourceMetadata{},  // 设置 Metadata 字段为 syftSource.DirectorySourceMetadata 结构体
		},
		Distro: &linux.Release{  // 设置 Distro 字段为 linux.Release 结构体
			ID:      "centos",  // 设置 ID 字段为 "centos"
// 创建一个新的 Identification 结构体，其中 IDLike 字段为 ["rhel"]，Version 字段为 "8.0"
IDLike:  []string{"rhel"},
Version: "8.0",
},

// 使用 NewDocument 函数创建一个新的文档对象，传入空的 Identification 结构体、packages、ctx、matches、nil、NewMetadataMock()、nil、nil
doc, err := NewDocument(clio.Identification{}, packages, ctx, matches, nil, NewMetadataMock(), nil, nil)
if err != nil {
    // 如果出现错误，打印错误信息
    t.Fatalf("unable to get document: %+v", err)
}

// 创建一个空的字符串切片 actualVulnerabilities
var actualVulnerabilities []string
// 遍历 doc.Matches 中的每个元素，将其 Vulnerability.ID 添加到 actualVulnerabilities 中
for _, m := range doc.Matches {
    actualVulnerabilities = append(actualVulnerabilities, m.Vulnerability.ID)
}

// 使用 assert.Equal 函数断言 actualVulnerabilities 是否等于 ["CVE-1999-0003", "CVE-1999-0002", "CVE-1999-0001"]
assert.Equal(t, []string{"CVE-1999-0003", "CVE-1999-0002", "CVE-1999-0001"}, actualVulnerabilities)
}

// 创建一个名为 TestTimestampValidFormat 的测试函数
func TestTimestampValidFormat(t *testing.T) {

// 创建一个新的 matches 对象
matches := match.NewMatches()
// 创建一个上下文对象，包含源和发行版信息
ctx := pkg.Context{
    Source: nil,
    Distro: nil,
}

// 使用给定的参数创建一个新的文档对象
doc, err := NewDocument(clio.Identification{}, nil, ctx, matches, nil, nil, nil, nil)
if err != nil {
    t.Fatalf("unable to get document: %+v", err)
}

// 断言文档描述符的时间戳不为空
assert.NotEmpty(t, doc.Descriptor.Timestamp)

// 检查时间戳的格式是否符合 RFC3339 标准，例如 2023-04-21T00:22:06.491137+01:00
_, timeErr := time.Parse(time.RFC3339, doc.Descriptor.Timestamp)
if timeErr != nil {
    t.Fatalf("unable to parse time: %+v", timeErr)
}
```