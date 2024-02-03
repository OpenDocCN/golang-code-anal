# `grype\grype\db\v3\store\store_test.go`

```go
package store

import (
    "encoding/json"  // 导入 JSON 编解码包
    "sort"  // 导入排序包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "github.com/go-test/deep"  // 导入深度比较包
    "github.com/stretchr/testify/assert"  // 导入断言包

    v3 "github.com/anchore/grype/grype/db/v3"  // 导入 v3 包
    "github.com/anchore/grype/grype/db/v3/store/model"  // 导入 model 包
)

func assertIDReader(t *testing.T, reader v3.IDReader, expected v3.ID) {
    t.Helper()  // 标记该函数是测试辅助函数
    if actual, err := reader.GetID(); err != nil {  // 获取 ID 信息
        t.Fatalf("failed to get ID: %+v", err)  // 如果获取失败则输出错误信息
    } else {
        diffs := deep.Equal(&expected, actual)  // 比较预期和实际结果
        if len(diffs) > 0 {  // 如果存在差异
            for _, d := range diffs {  // 遍历差异
                t.Errorf("Diff: %+v", d)  // 输出差异信息
            }
        }
    }
}

func TestStore_GetID_SetID(t *testing.T) {
    dbTempFile := t.TempDir()  // 创建临时目录

    s, err := New(dbTempFile, true)  // 创建新的存储对象
    if err != nil {  // 如果创建失败
        t.Fatalf("could not create store: %+v", err)  // 输出错误信息
    }

    expected := v3.ID{  // 创建预期的 ID 对象
        BuildTimestamp: time.Now().UTC(),  // 设置构建时间戳
        SchemaVersion:  2,  // 设置模式版本
    }

    if err = s.SetID(expected); err != nil {  // 设置 ID
        t.Fatalf("failed to set ID: %+v", err)  // 输出错误信息
    }

    assertIDReader(t, s, expected)  // 断言 ID 读取结果与预期一致

}

func assertVulnerabilityReader(t *testing.T, reader v3.VulnerabilityStoreReader, namespace, name string, expected []v3.Vulnerability) {
    if actual, err := reader.GetVulnerability(namespace, name); err != nil {  // 获取漏洞信息
        t.Fatalf("failed to get Vulnerability: %+v", err)  // 输出错误信息
    } else {
        if len(actual) != len(expected) {  // 如果实际结果数量与预期不一致
            t.Fatalf("unexpected number of vulns: %d", len(actual))  // 输出错误信息
        }
        for idx := range actual {  // 遍历实际结果
            diffs := deep.Equal(expected[idx], actual[idx])  // 比较预期和实际结果
            if len(diffs) > 0 {  // 如果存在差异
                for _, d := range diffs {  // 遍历差异
                    t.Errorf("Diff: %+v", d)  // 输出差异信息
                }
            }
        }
    }
}

func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    dbTempFile := t.TempDir()  // 创建临时目录
    s, err := New(dbTempFile, true)  // 创建新的存储对象
    if err != nil {  // 如果创建失败
        t.Fatalf("could not create store: %+v", err)  // 输出错误信息
    }
    # 创建一个包含漏洞信息的列表
    extra := []v3.Vulnerability{
        # 第一个漏洞信息
        {
            ID:                "my-cve-33333",  # 漏洞ID
            PackageName:       "package-name-2",  # 包名
            Namespace:         "my-namespace",  # 命名空间
            VersionConstraint: "< 1.0",  # 版本约束
            VersionFormat:     "semver",  # 版本格式
            CPEs:              []string{"a-cool-cpe"},  # CPE（通用漏洞披露）条目
            RelatedVulnerabilities: []v3.VulnerabilityReference{  # 相关漏洞引用
                {
                    ID:        "another-cve",  # 相关漏洞ID
                    Namespace: "nvd",  # 相关漏洞命名空间
                },
                {
                    ID:        "an-other-cve",  # 另一个相关漏洞ID
                    Namespace: "nvd",  # 另一个相关漏洞命名空间
                },
            },
            Fix: v3.Fix{  # 漏洞修复信息
                Versions: []string{"2.0.1"},  # 修复版本
                State:    v3.FixedState,  # 修复状态
            },
        },
        # 第二个漏洞信息
        {
            ID:                "my-other-cve-33333",  # 漏洞ID
            PackageName:       "package-name-3",  # 包名
            Namespace:         "my-namespace",  # 命名空间
            VersionConstraint: "< 509.2.2",  # 版本约束
            VersionFormat:     "semver",  # 版本格式
            CPEs:              []string{"a-cool-cpe"},  # CPE（通用漏洞披露）条目
            RelatedVulnerabilities: []v3.VulnerabilityReference{  # 相关漏洞引用
                {
                    ID:        "another-cve",  # 相关漏洞ID
                    Namespace: "nvd",  # 相关漏洞命名空间
                },
                {
                    ID:        "an-other-cve",  # 另一个相关漏洞ID
                    Namespace: "nvd",  # 另一个相关漏洞命名空间
                },
            },
            Fix: v3.Fix{  # 漏洞修复信息
                State: v3.NotFixedState,  # 修复状态
            },
        },
    }
    # 定义一个期望的漏洞列表，包含两个漏洞对象
    expected := []v3.Vulnerability{
        {
            ID:                "my-cve",  # 漏洞ID
            PackageName:       "package-name",  # 包名
            Namespace:         "my-namespace",  # 命名空间
            VersionConstraint: "< 1.0",  # 版本约束
            VersionFormat:     "semver",  # 版本格式
            CPEs:              []string{"a-cool-cpe"},  # 相关CPE
            RelatedVulnerabilities: []v3.VulnerabilityReference{  # 相关漏洞引用列表
                {
                    ID:        "another-cve",  # 相关漏洞ID
                    Namespace: "nvd",  # 相关漏洞命名空间
                },
                {
                    ID:        "an-other-cve",  # 另一个相关漏洞ID
                    Namespace: "nvd",  # 另一个相关漏洞命名空间
                },
            },
            Fix: v3.Fix{  # 漏洞修复信息
                Versions: []string{"1.0.1"},  # 修复版本
                State:    v3.FixedState,  # 修复状态
            },
        },
        {
            ID:                "my-other-cve",  # 另一个漏洞ID
            PackageName:       "package-name",  # 包名
            Namespace:         "my-namespace",  # 命名空间
            VersionConstraint: "< 509.2.2",  # 版本约束
            VersionFormat:     "semver",  # 版本格式
            CPEs:              []string{"a-cool-cpe"},  # 相关CPE
            RelatedVulnerabilities: []v3.VulnerabilityReference{  # 相关漏洞引用列表
                {
                    ID:        "another-cve",  # 相关漏洞ID
                    Namespace: "nvd",  # 相关漏洞命名空间
                },
                {
                    ID:        "an-other-cve",  # 另一个相关漏洞ID
                    Namespace: "nvd",  # 另一个相关漏洞命名空间
                },
            },
            Fix: v3.Fix{  # 漏洞修复信息
                Versions: []string{"4.0.5"},  # 修复版本
                State:    v3.FixedState,  # 修复状态
            },
        },
    }

    # 将期望的漏洞列表和额外的漏洞列表合并成一个总的漏洞列表
    total := append(expected, extra...)

    # 将总的漏洞列表添加到存储中
    if err = s.AddVulnerability(total...); err != nil {
        t.Fatalf("failed to set Vulnerability: %+v", err)  # 如果添加失败则输出错误信息
    }

    # 查询所有的漏洞条目
    var allEntries []model.VulnerabilityModel
    s.(*store).db.Find(&allEntries)
    # 检查查询到的漏洞条目数量是否和总的漏洞数量相等，如果不相等则输出错误信息
    if len(allEntries) != len(total) {
        t.Fatalf("unexpected number of entries: %d", len(allEntries))
    }

    # 断言漏洞读取器的行为
    assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)
// 定义一个函数，用于断言漏洞元数据读取器的行为
func assertVulnerabilityMetadataReader(t *testing.T, reader v3.VulnerabilityMetadataStoreReader, id, namespace string, expected v3.VulnerabilityMetadata) {
    // 如果获取漏洞元数据时出现错误，则输出错误信息
    if actual, err := reader.GetVulnerabilityMetadata(id, namespace); err != nil {
        t.Fatalf("failed to get metadata: %+v", err)
    } else if actual == nil {
        t.Fatalf("no metadata returned for id=%q namespace=%q", id, namespace)
    } else {
        // 对实际和期望的 CVSS 数据进行排序
        sortMetadataCvss(actual.Cvss)
        sortMetadataCvss(expected.Cvss)

        // 确保它们都具有相同数量的 CVSS 条目，以防止后续断言时出现错误
        assert.Len(t, expected.Cvss, len(actual.Cvss))
        for idx, actualCvss := range actual.Cvss {
            // 断言实际的 CVSS 向量、版本和度量与期望的相同
            assert.Equal(t, actualCvss.Vector, expected.Cvss[idx].Vector)
            assert.Equal(t, actualCvss.Version, expected.Cvss[idx].Version)
            assert.Equal(t, actualCvss.Metrics, expected.Cvss[idx].Metrics)

            // 将实际和期望的供应商元数据转换为 JSON 字符串，并进行比较
            actualVendor, err := json.Marshal(actualCvss.VendorMetadata)
            if err != nil {
                t.Errorf("unable to marshal vendor metadata: %q", err)
            }
            expectedVendor, err := json.Marshal(expected.Cvss[idx].VendorMetadata)
            if err != nil {
                t.Errorf("unable to marshal vendor metadata: %q", err)
            }
            assert.Equal(t, string(actualVendor), string(expectedVendor))

        }

        // 将 Cvss 字段设置为 nil，因为它是一个接口 - 在这一点上已经进行了 Cvss 的验证
        expected.Cvss = nil
        actual.Cvss = nil
        // 断言实际的漏洞元数据与期望的相同
        assert.Equal(t, &expected, actual)
    }

}

// 定义一个函数，用于对 CVSS 数据进行排序
func sortMetadataCvss(cvss []v3.Cvss) {
    # 使用sort.Slice函数对cvss切片进行排序，传入比较函数
    sort.Slice(cvss, func(i, j int) bool {
        # 首先按照Vector字段进行排序
        if cvss[i].Vector > cvss[j].Vector {
            return true
        }
        if cvss[i].Vector < cvss[j].Vector {
            return false
        }
        # 如果Vector相同，则尝试按照BaseScore字段进行排序
        return cvss[i].Metrics.BaseScore < cvss[j].Metrics.BaseScore
    })
// CustomMetadata 是一个空操作，其值并不重要，主要用于确保任何类型都可以被存储，然后在这些测试用例中检索以进行断言，其中使用了自定义供应商 CVSS 分数
type CustomMetadata struct {
    SuperScore string
    Vendor     string
}

func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
    // 创建临时目录用于数据库文件
    dbTempFile := t.TempDir()

    // 创建新的存储对象
    s, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 向存储对象添加漏洞元数据
    if err = s.AddVulnerabilityMetadata(total...); err != nil {
        t.Fatalf("failed to set metadata: %+v", err)
    }

    // 查询所有漏洞元数据
    var allEntries []model.VulnerabilityMetadataModel
    s.(*store).db.Find(&allEntries)
    // 检查查询结果是否与期望长度相同
    if len(allEntries) != len(total) {
        t.Fatalf("unexpected number of entries: %d", len(allEntries))
    }
}

func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
    // 测试用例
    tests := []struct {
        name     string
        add      []v3.VulnerabilityMetadata
        expected v3.VulnerabilityMetadata
        err      bool
    }
    // 遍历测试用例
    for _, test := range tests {
        // 使用测试名称创建子测试
        t.Run(test.name, func(t *testing.T) {
            // 创建临时目录用于数据库
            dbTempDir := t.TempDir()
            // 创建新的存储对象
            s, err := New(dbTempDir, true)
            // 如果创建存储对象时出现错误，输出错误信息并终止测试
            if err != nil {
                t.Fatalf("could not create store: %+v", err)
            }

            // 逐个添加元数据
            var theErr error
            for _, metadata := range test.add {
                // 添加漏洞元数据
                err = s.AddVulnerabilityMetadata(metadata)
                // 如果添加时出现错误，记录错误并终止添加
                if err != nil {
                    theErr = err
                    break
                }
            }

            // 检查是否预期出现错误但未出现，如果是则输出错误信息并终止测试
            if test.err && theErr == nil {
                t.Fatalf("expected error but did not get one")
            } else if !test.err && theErr != nil {
                // 检查是否预期不出现错误但出现了，如果是则输出错误信息并终止测试
                t.Fatalf("expected no error but got one: %+v", theErr)
            } else if test.err && theErr != nil {
                // 如果预期出现错误且出现了，测试通过，直接返回
                return
            }

            // 确保只有一个条目
            var allEntries []model.VulnerabilityMetadataModel
            // 从数据库中查找所有条目
            s.(*store).db.Find(&allEntries)
            // 检查条目数量是否为1，如果不是则输出错误信息并终止测试
            if len(allEntries) != 1 {
                t.Fatalf("unexpected number of entries: %d", len(allEntries))
            }

            // 获取预期的元数据对象
            if actual, err := s.GetVulnerabilityMetadata(test.expected.ID, test.expected.Namespace); err != nil {
                // 如果获取元数据时出现错误，输出错误信息并终止测试
                t.Fatalf("failed to get metadata: %+v", err)
            } else {
                // 比较预期和实际的元数据对象，输出差异信息
                diffs := deep.Equal(&test.expected, actual)
                if len(diffs) > 0 {
                    for _, d := range diffs {
                        t.Errorf("Diff: %+v", d)
                    }
                }
            }
        })
    }
// 测试函数，用于测试从元数据中提取 CVSS 分数
func TestCvssScoresInMetadata(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string
        add      []v3.VulnerabilityMetadata
        expected v3.VulnerabilityMetadata
    }
    // 遍历测试用例
    for _, test := range tests {
        // 在临时目录创建数据库
        dbTempDir := t.TempDir()

        // 创建新的存储对象
        s, err := New(dbTempDir, true)
        if err != nil {
            t.Fatalf("could not create s: %+v", err)
        }

        // 逐个添加元数据
        for _, metadata := range test.add {
            err = s.AddVulnerabilityMetadata(metadata)
            if err != nil {
                t.Fatalf("unable to s vulnerability metadata: %+v", err)
            }
        }

        // 确保只有一个条目
        var allEntries []model.VulnerabilityMetadataModel
        s.(*store).db.Find(&allEntries)
        if len(allEntries) != 1 {
            t.Fatalf("unexpected number of entries: %d", len(allEntries))
        }

        // 断言从存储中读取的元数据
        assertVulnerabilityMetadataReader(t, s, test.expected.ID, test.expected.Namespace, test.expected)
    }
}

// 测试函数，用于测试存储差异
func Test_DiffStore(t *testing.T) {
    //GIVEN
    // 在临时目录创建数据库文件
    dbTempFile := t.TempDir()

    // 创建新的存储对象
    s1, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }
    // 重新赋值临时目录
    dbTempFile = t.TempDir()

    // 创建新的存储对象
    s2, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }
}
    # 定义一个包含漏洞信息的列表
    baseVulns := []v3.Vulnerability{
        # 第一个漏洞信息
        {
            Namespace:         "github:python",
            ID:                "CVE-123-4567",
            PackageName:       "pypi:requests",
            VersionConstraint: "< 2.0 >= 1.29",
            CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
        },
        # 第二个漏洞信息
        {
            Namespace:         "github:python",
            ID:                "CVE-123-4567",
            PackageName:       "pypi:requests",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
        },
        # 第三个漏洞信息
        {
            Namespace:         "npm",
            ID:                "CVE-123-7654",
            PackageName:       "npm:axios",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
            # 包含修复状态的信息
            Fix: v3.Fix{
                State: v3.UnknownFixState,
            },
        },
        # 第四个漏洞信息
        {
            Namespace:         "nuget",
            ID:                "GHSA-****-******",
            PackageName:       "nuget:net",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
            # 包含修复状态的信息
            Fix: v3.Fix{
                State: v3.UnknownFixState,
            },
        },
        # 第五个漏洞信息
        {
            Namespace:         "hex",
            ID:                "GHSA-^^^^-^^^^^^",
            PackageName:       "hex:esbuild",
            VersionConstraint: "< 3.0 >= 2.17",
            CPEs:              []string{"cpe:2.3:hex:esbuild:*:*:*:*:*:*"},
        },
    }
    # 定义一个包含漏洞元数据的列表
    baseMetadata := []v3.VulnerabilityMetadata{
        # 漏洞元数据信息
        {
            Namespace:  "nuget",
            ID:         "GHSA-****-******",
            DataSource: "nvd",
        },
    }
    # 定义一个包含漏洞信息的列表
    targetVulns := []v3.Vulnerability{
        # 第一个漏洞信息
        {
            # 漏洞所属命名空间
            Namespace:         "github:python",
            # 漏洞ID
            ID:                "CVE-123-4567",
            # 漏洞所涉及的包名
            PackageName:       "pypi:requests",
            # 版本约束
            VersionConstraint: "< 2.0 >= 1.29",
            # CPE（通用漏洞及漏洞披露）条目
            CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
        },
        # 第二个漏洞信息
        {
            # 漏洞所属命名空间
            Namespace:         "github:go",
            # 漏洞ID
            ID:                "GHSA-....-....",
            # 漏洞所涉及的包名
            PackageName:       "hashicorp:nomad",
            # 版本约束
            VersionConstraint: "< 3.0 >= 2.17",
            # CPE（通用漏洞及漏洞披露）条目
            CPEs:              []string{"cpe:2.3:golang:hashicorp:nomad:*:*:*:*:*"},
        },
        # 第三个漏洞信息
        {
            # 漏洞所属命名空间
            Namespace:         "github:go",
            # 漏洞ID
            ID:                "GHSA-....-....",
            # 漏洞所涉及的包名
            PackageName:       "hashicorp:n",
            # 版本约束
            VersionConstraint: "< 2.0 >= 1.17",
            # CPE（通用漏洞及漏洞披露）条目
            CPEs:              []string{"cpe:2.3:golang:hashicorp:n:*:*:*:*:*"},
        },
        # 第四个漏洞信息
        {
            # 漏洞所属命名空间
            Namespace:         "npm",
            # 漏洞ID
            ID:                "CVE-123-7654",
            # 漏洞所涉及的包名
            PackageName:       "npm:axios",
            # 版本约束
            VersionConstraint: "< 3.0 >= 2.17",
            # CPE（通用漏洞及漏洞披露）条目
            CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
            # 修复状态为不会修复
            Fix: v3.Fix{
                State: v3.WontFixState,
            },
        },
        # 第五个漏洞信息
        {
            # 漏洞所属命名空间
            Namespace:         "nuget",
            # 漏洞ID
            ID:                "GHSA-****-******",
            # 漏洞所涉及的包名
            PackageName:       "nuget:net",
            # 版本约束
            VersionConstraint: "< 3.0 >= 2.17",
            # CPE（通用漏洞及漏洞披露）条目
            CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
            # 修复状态为未知
            Fix: v3.Fix{
                State: v3.UnknownFixState,
            },
        },
    }
    // 定义一个期望的漏洞差异列表，包含了不同的漏洞情况
    expectedDiffs := []v3.Diff{
        {
            Reason:    v3.DiffChanged,  // 漏洞变更的原因
            ID:        "CVE-123-4567",  // 漏洞的ID
            Namespace: "github:python",  // 漏洞所属的命名空间
            Packages:  []string{"pypi:requests"},  // 受影响的软件包列表
        },
        {
            Reason:    v3.DiffChanged,
            ID:        "CVE-123-7654",
            Namespace: "npm",
            Packages:  []string{"npm:axios"},
        },
        {
            Reason:    v3.DiffRemoved,
            ID:        "GHSA-****-******",
            Namespace: "nuget",
            Packages:  []string{"nuget:net"},
        },
        {
            Reason:    v3.DiffAdded,
            ID:        "GHSA-....-....",
            Namespace: "github:go",
            Packages:  []string{"hashicorp:n", "hashicorp:nomad"},
        },
        {
            Reason:    v3.DiffRemoved,
            ID:        "GHSA-^^^^-^^^^^^",
            Namespace: "hex",
            Packages:  []string{"hex:esbuild"},
        },
    }

    // 将基准漏洞列表中的漏洞添加到 s1 中
    for _, vuln := range baseVulns {
        s1.AddVulnerability(vuln)
    }
    // 将目标漏洞列表中的漏洞添加到 s2 中
    for _, vuln := range targetVulns {
        s2.AddVulnerability(vuln)
    }
    // 将基准元数据列表中的元数据添加到 s1 中
    for _, meta := range baseMetadata {
        s1.AddVulnerabilityMetadata(meta)
    }

    // 当执行漏洞差异比较时
    result, err := s1.DiffStore(s2)

    // 然后
    // 对结果列表按照漏洞ID进行稳定排序
    sort.SliceStable(*result, func(i, j int) bool {
        return (*result)[i].ID < (*result)[j].ID
    })
    // 对每个结果中的受影响软件包列表进行排序
    for i := range *result {
        sort.Strings((*result)[i].Packages)
    }

    // 断言：错误应该为空
    assert.NoError(t, err)
    // 断言：结果应该与期望的漏洞差异列表相等
    assert.Equal(t, expectedDiffs, *result)
# 闭合前面的函数定义
```