# `grype\grype\db\v2\store\store_test.go`

```
package store

import (
    "testing"
    "time"

    "github.com/go-test/deep"

    v2 "github.com/anchore/grype/grype/db/v2"
    "github.com/anchore/grype/grype/db/v2/store/model"
)

func assertIDReader(t *testing.T, reader v2.IDReader, expected v2.ID) {
    t.Helper()
    if actual, err := reader.GetID(); err != nil {
        t.Fatalf("failed to get ID: %+v", err)
    } else {
        diffs := deep.Equal(&expected, actual)
        if len(diffs) > 0 {
            for _, d := range diffs {
                t.Errorf("Diff: %+v", d)
            }
        }
    }
}

func TestStore_GetID_SetID(t *testing.T) {
    dbTempFile := t.TempDir()  // 创建临时目录用于存储数据库文件

    s, err := New(dbTempFile, true)  // 创建新的数据库存储实例
    if err != nil {
        t.Fatalf("could not create store: %+v", err)  // 如果创建失败，输出错误信息
    }

    expected := v2.ID{  // 创建预期的 ID 对象
        BuildTimestamp: time.Now().UTC(),  // 设置构建时间戳为当前时间的 UTC 时间
        SchemaVersion:  2,  // 设置模式版本为 2
    }

    if err = s.SetID(expected); err != nil {  // 将预期的 ID 对象设置到数据库存储实例中
        t.Fatalf("failed to set ID: %+v", err)  // 如果设置失败，输出错误信息
    }

    assertIDReader(t, s, expected)  // 调用断言函数，验证读取的 ID 是否与预期相符

}

func assertVulnerabilityReader(t *testing.T, reader v2.VulnerabilityStoreReader, namespace, name string, expected []v2.Vulnerability) {
    if actual, err := reader.GetVulnerability(namespace, name); err != nil {
        t.Fatalf("failed to get Vulnerability: %+v", err)  // 如果获取漏洞信息失败，输出错误信息
    } else {
        if len(actual) != len(expected) {
            t.Fatalf("unexpected number of vulns: %d", len(actual))  // 如果获取的漏洞数量与预期不符，输出错误信息
        }

        for idx := range actual {
            diffs := deep.Equal(expected[idx], actual[idx])  // 比较预期的漏洞信息和实际获取的漏洞信息
            if len(diffs) > 0 {
                for _, d := range diffs {
                    t.Errorf("Diff: %+v", d)  // 如果存在差异，输出差异信息
                }
            }
        }
    }
}

func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    dbTempFile := t.TempDir()  // 创建临时目录用于存储数据库文件
    s, err := New(dbTempFile, true)  // 创建新的数据库存储实例
    if err != nil {
        t.Fatalf("could not create store: %+v", err)  // 如果创建失败，输出错误信息
    }
    # 创建一个包含漏洞信息的列表
    extra := []v2.Vulnerability{
        # 创建一个漏洞对象，包含漏洞的各种信息
        {
            ID:                   "my-cve-33333",
            RecordSource:         "record-source",
            PackageName:          "package-name-2",
            Namespace:            "my-namespace",
            VersionConstraint:    "< 1.0",
            VersionFormat:        "semver",
            CPEs:                 []string{"a-cool-cpe"},
            ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
            FixedInVersion:       "2.0.1",
        },
        # 创建另一个漏洞对象，包含漏洞的各种信息
        {
            ID:                   "my-other-cve-33333",
            RecordSource:         "record-source",
            PackageName:          "package-name-3",
            Namespace:            "my-namespace",
            VersionConstraint:    "< 509.2.2",
            VersionFormat:        "semver",
            CPEs:                 []string{"a-cool-cpe"},
            ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
        },
    }

    # 创建一个期望的漏洞信息列表
    expected := []v2.Vulnerability{
        # 创建一个期望的漏洞对象，包含漏洞的各种信息
        {
            ID:                   "my-cve",
            RecordSource:         "record-source",
            PackageName:          "package-name",
            Namespace:            "my-namespace",
            VersionConstraint:    "< 1.0",
            VersionFormat:        "semver",
            CPEs:                 []string{"a-cool-cpe"},
            ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
            FixedInVersion:       "1.0.1",
        },
        # 创建另一个期望的漏洞对象，包含漏洞的各种信息
        {
            ID:                   "my-other-cve",
            RecordSource:         "record-source",
            PackageName:          "package-name",
            Namespace:            "my-namespace",
            VersionConstraint:    "< 509.2.2",
            VersionFormat:        "semver",
            CPEs:                 []string{"a-cool-cpe"},
            ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
            FixedInVersion:       "4.0.5",
        },
    }
    # 将 expected 切片和 extra 切片合并成一个新的切片 total
    total := append(expected, extra...)

    # 调用 AddVulnerability 方法，将 total 切片中的元素作为参数传入，如果出现错误则输出错误信息
    if err = s.AddVulnerability(total...); err != nil {
        t.Fatalf("failed to set Vulnerability: %+v", err)
    }

    # 声明一个空的 model.VulnerabilityModel 切片 allEntries
    var allEntries []model.VulnerabilityModel
    # 从数据库中查询所有的 VulnerabilityModel 记录，并将结果存入 allEntries 切片
    s.(*store).db.Find(&allEntries)
    # 检查 allEntries 切片的长度是否等于 total 切片的长度，如果不相等则输出错误信息
    if len(allEntries) != len(total) {
        t.Fatalf("unexpected number of entries: %d", len(allEntries))
    }

    # 调用 assertVulnerabilityReader 方法，检查 s 对象中的数据是否符合预期
    assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)
# 定义一个函数，用于断言漏洞元数据读取器的行为是否符合预期
func assertVulnerabilityMetadataReader(t *testing.T, reader v2.VulnerabilityMetadataStoreReader, id, recordSource string, expected v2.VulnerabilityMetadata) {
    # 如果通过读取器获取漏洞元数据时发生错误，则输出错误信息
    if actual, err := reader.GetVulnerabilityMetadata(id, recordSource); err != nil {
        t.Fatalf("failed to get metadata: %+v", err)
    } else {
        # 比较预期的漏洞元数据和实际获取的漏洞元数据，输出差异信息
        diffs := deep.Equal(&expected, actual)
        if len(diffs) > 0 {
            for _, d := range diffs {
                t.Errorf("Diff: %+v", d)
            }
        }
    }
}

# 定义一个测试函数，用于测试存储器的获取和设置漏洞元数据的功能
func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
    # 在临时目录中创建一个数据库临时文件
    dbTempFile := t.TempDir()
    
    # 创建一个新的存储器实例，并检查是否出现错误
    s, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }
}
    # 创建一个包含漏洞元数据的列表
    total := []v2.VulnerabilityMetadata{
        # 第一个漏洞元数据
        {
            ID:           "my-cve",  # 漏洞ID
            RecordSource: "record-source",  # 记录来源
            Severity:     "pretty bad",  # 严重程度
            Links:        []string{"https://ancho.re"},  # 相关链接
            Description:  "best description ever",  # 描述
            CvssV2: &v2.Cvss{  # CVSS V2 评分
                BaseScore:           1.1,  # 基础分数
                ExploitabilityScore: 2.2,  # 可利用性分数
                ImpactScore:         3.3,  # 影响分数
                Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--NOT",  # 向量
            },
            CvssV3: &v2.Cvss{  # CVSS V3 评分
                BaseScore:           1.3,  # 基础分数
                ExploitabilityScore: 2.1,  # 可利用性分数
                ImpactScore:         3.2,  # 影响分数
                Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--NICE",  # 向量
            },
        },
        # 第二个漏洞元数据
        {
            ID:           "my-other-cve",  # 漏洞ID
            RecordSource: "record-source",  # 记录来源
            Severity:     "pretty bad",  # 严重程度
            Links:        []string{"https://ancho.re"},  # 相关链接
            Description:  "worst description ever",  # 描述
            CvssV2: &v2.Cvss{  # CVSS V2 评分
                BaseScore:           4.1,  # 基础分数
                ExploitabilityScore: 5.2,  # 可利用性分数
                ImpactScore:         6.3,  # 影响分数
                Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量
            },
            CvssV3: &v2.Cvss{  # CVSS V3 评分
                BaseScore:           1.4,  # 基础分数
                ExploitabilityScore: 2.5,  # 可利用性分数
                ImpactScore:         3.6,  # 影响分数
                Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 向量
            },
        },
    }
    
    # 将漏洞元数据添加到存储中
    if err = s.AddVulnerabilityMetadata(total...); err != nil {
        t.Fatalf("failed to set metadata: %+v", err)  # 如果添加失败，输出错误信息
    }
    
    # 查询所有漏洞元数据
    var allEntries []model.VulnerabilityMetadataModel
    s.(*store).db.Find(&allEntries)
    # 检查查询结果是否和添加的漏洞元数据数量一致
    if len(allEntries) != len(total) {
        t.Fatalf("unexpected number of entries: %d", len(allEntries))  # 如果数量不一致，输出错误信息
    }
func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string
        add      []v2.VulnerabilityMetadata
        expected v2.VulnerabilityMetadata
        err      bool
    }

    // 遍历测试用例
    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            // 创建临时目录
            dbTempDir := t.TempDir()

            // 创建新的存储对象
            s, err := New(dbTempDir, true)
            if err != nil {
                t.Fatalf("could not create store: %+v", err)
            }

            // 逐个添加元数据
            var theErr error
            for _, metadata := range test.add {
                err = s.AddVulnerabilityMetadata(metadata)
                if err != nil {
                    theErr = err
                    break
                }
            }

            // 检查是否预期出现错误
            if test.err && theErr == nil {
                t.Fatalf("expected error but did not get one")
            } else if !test.err && theErr != nil {
                t.Fatalf("expected no error but got one: %+v", theErr)
            } else if test.err && theErr != nil {
                // 测试通过...
                return
            }

            // 确保只有一个条目
            var allEntries []model.VulnerabilityMetadataModel
            s.(*store).db.Find(&allEntries)
            if len(allEntries) != 1 {
                t.Fatalf("unexpected number of entries: %d", len(allEntries))
            }

            // 获取结果的元数据对象
            if actual, err := s.GetVulnerabilityMetadata(test.expected.ID, test.expected.RecordSource); err != nil {
                t.Fatalf("failed to get metadata: %+v", err)
            } else {
                // 比较预期结果和实际结果
                diffs := deep.Equal(&test.expected, actual)
                if len(diffs) > 0 {
                    for _, d := range diffs {
                        t.Errorf("Diff: %+v", d)
                    }
                }
            }
        })
    }
}
```