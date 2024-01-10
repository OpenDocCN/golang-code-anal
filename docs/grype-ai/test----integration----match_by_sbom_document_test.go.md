# `grype\test\integration\match_by_sbom_document_test.go`

```
package integration

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp" // 导入 go-cmp 包，用于比较数据结构
    "github.com/google/go-cmp/cmp/cmpopts" // 导入 go-cmp 包的 cmpopts 模块，用于定制比较选项
    "github.com/scylladb/go-set/strset" // 导入 strset 模块，用于操作字符串集合
    "github.com/stretchr/testify/assert" // 导入 testify 包的 assert 模块，用于编写断言
    "github.com/stretchr/testify/require" // 导入 testify 包的 require 模块，用于编写测试所需的条件

    "github.com/anchore/grype/grype" // 导入 grype 包，用于漏洞扫描
    "github.com/anchore/grype/grype/db" // 导入 grype 包的 db 模块，用于数据库操作
    "github.com/anchore/grype/grype/match" // 导入 grype 包的 match 模块，用于匹配漏洞
    "github.com/anchore/grype/grype/pkg" // 导入 grype 包的 pkg 模块，用于处理软件包
    "github.com/anchore/grype/grype/store" // 导入 grype 包的 store 模块，用于存储数据
    "github.com/anchore/syft/syft/source" // 导入 syft 包的 source 模块，用于处理源数据
)

func TestMatchBySBOMDocument(t *testing.T) {
    tests := []struct {
        name            string // 测试用例名称
        fixture         string // 测试用例的数据文件
        expectedIDs     []string // 期望的漏洞 ID 列表
        expectedDetails []match.Detail // 期望的漏洞详情列表
    }
    # 遍历测试用例列表
    for _, test := range tests {
        # 使用测试名称创建子测试
        t.Run(test.name, func(t *testing.T) {
            # 创建模拟的数据库存储
            mkStr := newMockDbStore()
            # 创建漏洞提供程序
            vp, err := db.NewVulnerabilityProvider(mkStr)
            require.NoError(t, err)
            # 创建漏洞元数据提供程序
            mp := db.NewVulnerabilityMetadataProvider(mkStr)
            # 创建匹配排除提供程序
            ep := db.NewMatchExclusionProvider(mkStr)
            # 创建存储对象
            str := store.Store{
                Provider:          vp,
                MetadataProvider:  mp,
                ExclusionProvider: ep,
            }
            # 查找漏洞
            matches, _, _, err := grype.FindVulnerabilities(str, fmt.Sprintf("sbom:%s", test.fixture), source.SquashedScope, nil)
            assert.NoError(t, err)
            # 创建空的匹配详情列表
            details := make([]match.Detail, 0)
            # 创建漏洞 ID 集合
            ids := strset.New()
            # 遍历匹配列表
            for _, m := range matches.Sorted() {
                # 将匹配的详情追加到详情列表
                details = append(details, m.Details...)
                # 将漏洞 ID 添加到集合中
                ids.Add(m.Vulnerability.ID)
            }

            # 断言匹配详情列表的长度与预期长度相等
            require.Len(t, details, len(test.expectedDetails))

            # 定义比较选项
            cmpOpts := []cmp.Option{
                cmpopts.IgnoreFields(pkg.Package{}, "Locations"),
            }

            # 遍历预期匹配详情列表
            for i := range test.expectedDetails {
                # 比较预期匹配详情和实际匹配详情
                if d := cmp.Diff(test.expectedDetails[i], details[i], cmpOpts...); d != "" {
                    t.Errorf("unexpected match details (-want +got):\n%s", d)
                }
            }

            # 断言预期漏洞 ID 集合与实际漏洞 ID 集合相等
            assert.ElementsMatch(t, test.expectedIDs, ids.List())
        })
    }
# 闭合前面的函数定义
```