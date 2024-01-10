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
    syftPkg "github.com/anchore/syft/syft/pkg"  // 导入 syftPkg 包
    syftSource "github.com/anchore/syft/syft/source"  // 导入 syftSource 包
)

func TestPackagesAreSorted(t *testing.T) {
    // 定义 pkg1 变量
    var pkg1 = pkg.Package{
        ID:      "package-1-id",
        Name:    "package-1",
        Version: "1.1.1",
        Type:    syftPkg.DebPkg,
    }

    // 定义 pkg2 变量
    var pkg2 = pkg.Package{
        ID:      "package-2-id",
        Name:    "package-2",
        Version: "2.2.2",
        Type:    syftPkg.DebPkg,
    }

    // 定义 match1 变量
    var match1 = match.Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-1999-0003",
        },
        Package: pkg1,
        Details: match.Details{
            {
                Type: match.ExactDirectMatch,
            },
        },
    }

    // 定义 match2 变量
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

    // 定义 match3 变量
    var match3 = match.Match{
        Vulnerability: vulnerability.Vulnerability{
            ID: "CVE-1999-0001",
        },
        Package: pkg1,
        Details: match.Details{
            {
                Type: match.ExactIndirectMatch,
            },
        },
    }

    // 创建新的 matches 对象
    matches := match.NewMatches()
    // 向 matches 对象中添加 match1, match2, match3
    matches.Add(match1, match2, match3)

    // 定义 packages 切片
    packages := []pkg.Package{pkg1, pkg2}

    // 定义 ctx 变量
    ctx := pkg.Context{
        Source: &syftSource.Description{
            Metadata: syftSource.DirectorySourceMetadata{},
        },
        Distro: &linux.Release{
            ID:      "centos",
            IDLike:  []string{"rhel"},
            Version: "8.0",
        },
    }
}
    // 使用给定的参数创建一个新的文档对象
    doc, err := NewDocument(clio.Identification{}, packages, ctx, matches, nil, NewMetadataMock(), nil, nil)
    // 如果创建文档对象时发生错误，输出错误信息并终止测试
    if err != nil {
        t.Fatalf("unable to get document: %+v", err)
    }

    // 创建一个空的实际漏洞列表
    var actualVulnerabilities []string
    // 遍历文档对象的匹配列表，将每个匹配的漏洞 ID 添加到实际漏洞列表中
    for _, m := range doc.Matches {
        actualVulnerabilities = append(actualVulnerabilities, m.Vulnerability.ID)
    }

    // 断言实际漏洞列表与预期的漏洞列表相等
    assert.Equal(t, []string{"CVE-1999-0003", "CVE-1999-0002", "CVE-1999-0001"}, actualVulnerabilities)
func TestTimestampValidFormat(t *testing.T) {
    // 创建一个新的匹配对象
    matches := match.NewMatches()
    
    // 创建一个上下文对象，包含源和发行版信息
    ctx := pkg.Context{
        Source: nil,
        Distro: nil,
    }
    
    // 创建一个新的文档对象，使用空的身份验证、上下文、匹配对象和其他参数
    doc, err := NewDocument(clio.Identification{}, nil, ctx, matches, nil, nil, nil, nil)
    // 如果创建文档对象时发生错误，输出错误信息
    if err != nil {
        t.Fatalf("unable to get document: %+v", err)
    }
    
    // 断言文档描述符的时间戳不为空
    assert.NotEmpty(t, doc.Descriptor.Timestamp)
    // 检查时间戳的格式是否符合 RFC3339 标准，例如 2023-04-21T00:22:06.491137+01:00
    _, timeErr := time.Parse(time.RFC3339, doc.Descriptor.Timestamp)
    // 如果解析时间戳时发生错误，输出错误信息
    if timeErr != nil {
        t.Fatalf("unable to parse time: %+v", timeErr)
    }
}
```