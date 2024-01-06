# `grype\grype\db\v5\store\store_test.go`

```
package store

import (
	"encoding/json"  // 导入 JSON 编解码包
	"sort"  // 导入排序包
	"testing"  // 导入测试包
	"time"  // 导入时间包

	"github.com/go-test/deep"  // 导入深度比较包
	"github.com/stretchr/testify/assert"  // 导入断言包

	v5 "github.com/anchore/grype/grype/db/v5"  // 导入 v5 版本的数据库包
	"github.com/anchore/grype/grype/db/v5/store/model"  // 导入数据库模型包
)

func assertIDReader(t *testing.T, reader v5.IDReader, expected v5.ID) {
	t.Helper()  // 标记该函数是测试辅助函数
	if actual, err := reader.GetID(); err != nil {  // 获取 ID 信息
		t.Fatalf("failed to get ID: %+v", err)  // 如果获取失败，输出错误信息
	} else {
		// 使用 deep 包比较 expected 和 actual 两个对象的深层次差异
		diffs := deep.Equal(&expected, actual)
		// 如果存在差异
		if len(diffs) > 0 {
			// 遍历差异列表，输出每个差异
			for _, d := range diffs {
				t.Errorf("Diff: %+v", d)
			}
		}
	}
}

func TestStore_GetID_SetID(t *testing.T) {
	// 在临时目录创建数据库文件
	dbTempFile := t.TempDir()

	// 创建一个新的存储对象
	s, err := New(dbTempFile, true)
	// 如果创建过程中出现错误，输出错误信息
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}

	// 设置预期的 ID 对象
	expected := v5.ID{
		BuildTimestamp: time.Now().UTC(),
		SchemaVersion:  2,
	}

	// 如果设置 ID 失败，输出错误信息
	if err = s.SetID(expected); err != nil {
		t.Fatalf("failed to set ID: %+v", err)
	}

	// 断言 ID 读取器
	assertIDReader(t, s, expected)

}

// 断言漏洞读取器
func assertVulnerabilityReader(t *testing.T, reader v5.VulnerabilityStoreReader, namespace, name string, expected []v5.Vulnerability) {
	// 获取漏洞信息，如果失败输出错误信息
	if actual, err := reader.SearchForVulnerabilities(namespace, name); err != nil {
		t.Fatalf("failed to get Vulnerability: %+v", err)
	} else {
		// 检查实际漏洞数量是否与预期相同
		if len(actual) != len(expected) {
			t.Fatalf("unexpected number of vulns: %d", len(actual))
		}
		// 遍历比较实际漏洞和预期漏洞
		for idx := range actual {
			diffs := deep.Equal(expected[idx], actual[idx])
			// 如果有差异，输出差异信息
			if len(diffs) > 0 {
# 遍历 diffs 切片中的元素，将每个元素输出为错误信息
for _, d := range diffs {
    t.Errorf("Diff: %+v", d)
}
# 结束循环
}

# 结束函数
}

# 定义测试函数 TestStore_GetVulnerability_SetVulnerability
func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    # 创建临时文件夹作为数据库文件路径
    dbTempFile := t.TempDir()
    # 创建一个新的存储对象，并打开数据库文件
    s, err := New(dbTempFile, true)
    # 如果创建存储对象时出现错误，输出错误信息
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    # 创建一个包含 v5.Vulnerability 类型元素的切片
    extra := []v5.Vulnerability{
        {
            ID:                "my-cve-33333",
            PackageName:       "package-name-2",
            Namespace:         "my-namespace",
```

			# 设置版本约束为小于1.0
			VersionConstraint: "< 1.0",
			# 设置版本格式为语义化版本
			VersionFormat:     "semver",
			# 设置CPEs为空字符串数组
			CPEs:              []string{"a-cool-cpe"},
			# 设置相关漏洞引用为包含两个VulnerabilityReference对象的数组
			RelatedVulnerabilities: []v5.VulnerabilityReference{
				{
					# 设置漏洞ID为"another-cve"
					ID:        "another-cve",
					# 设置命名空间为"nvd"
					Namespace: "nvd",
				},
				{
					# 设置漏洞ID为"an-other-cve"
					ID:        "an-other-cve",
					# 设置命名空间为"nvd"
					Namespace: "nvd",
				},
			},
			# 设置修复信息为包含版本和状态的Fix对象
			Fix: v5.Fix{
				# 设置修复版本为"2.0.1"
				Versions: []string{"2.0.1"},
				# 设置状态为已修复状态
				State:    v5.FixedState,
			},
		},
		{
			# 设置漏洞ID为"my-other-cve-33333"
			ID:                "my-other-cve-33333",
# 设置包的名称为 "package-name-3"
PackageName:       "package-name-3",
# 设置命名空间为 "my-namespace"
Namespace:         "my-namespace",
# 设置版本约束为 "< 509.2.2"
VersionConstraint: "< 509.2.2",
# 设置版本格式为 "semver"
VersionFormat:     "semver",
# 设置CPEs为空列表
CPEs:              []string{"a-cool-cpe"},
# 设置相关漏洞引用列表
RelatedVulnerabilities: []v5.VulnerabilityReference{
    # 第一个漏洞引用的ID为 "another-cve"，命名空间为 "nvd"
    {
        ID:        "another-cve",
        Namespace: "nvd",
    },
    # 第二个漏洞引用的ID为 "an-other-cve"，命名空间为 "nvd"
    {
        ID:        "an-other-cve",
        Namespace: "nvd",
    },
},
# 设置修复状态为未修复
Fix: v5.Fix{
    State: v5.NotFixedState,
},
		// 创建一个期望的漏洞切片
		expected := []v5.Vulnerability{
			// 创建一个漏洞对象
			{
				// 设置漏洞ID
				ID:                "my-cve",
				// 设置包名
				PackageName:       "package-name",
				// 设置命名空间
				Namespace:         "my-namespace",
				// 设置版本约束
				VersionConstraint: "< 1.0",
				// 设置版本格式
				VersionFormat:     "semver",
				// 设置CPEs
				CPEs:              []string{"a-cool-cpe"},
				// 设置相关漏洞引用
				RelatedVulnerabilities: []v5.VulnerabilityReference{
					// 创建一个相关漏洞引用对象
					{
						// 设置漏洞ID
						ID:        "another-cve",
						// 设置命名空间
						Namespace: "nvd",
					},
					// 创建另一个相关漏洞引用对象
					{
						// 设置漏洞ID
						ID:        "an-other-cve",
						// 设置命名空间
						Namespace: "nvd",
					},
				},
				// 设置修复信息
				Fix: v5.Fix{
# 定义一个结构体，包含版本、状态等信息
{
    Versions: []string{"1.0.1"},  # 版本号为1.0.1
    State:    v5.FixedState,      # 状态为v5.FixedState
},
# 定义另一个结构体，包含ID、包名、命名空间、版本约束、版本格式、CPEs和相关漏洞引用
{
    ID:                "my-other-cve",  # 漏洞ID为my-other-cve
    PackageName:       "package-name",  # 包名为package-name
    Namespace:         "my-namespace",  # 命名空间为my-namespace
    VersionConstraint: "< 509.2.2",     # 版本约束为小于509.2.2
    VersionFormat:     "semver",        # 版本格式为semver
    CPEs:              nil,             # CPEs为空
    RelatedVulnerabilities: []v5.VulnerabilityReference{  # 相关漏洞引用
        {
            ID:        "another-cve",  # 漏洞ID为another-cve
            Namespace: "nvd",           # 命名空间为nvd
        },
        {
            ID:        "an-other-cve",  # 漏洞ID为an-other-cve
            Namespace: "nvd",           # 命名空间为nvd
        },
    }
}
		},
		// 定义一个 CVE 对象，包含漏洞的相关信息
		Fix: v5.Fix{
			// 指定受影响的版本范围
			Versions: []string{"4.0.5"},
			// 指定漏洞状态为已修复
			State:    v5.FixedState,
		},
	},
	{
		// 定义另一个 CVE 对象，包含漏洞的相关信息
		ID:                     "yet-another-cve",
		PackageName:            "package-name",
		Namespace:              "my-namespace",
		// 指定受影响的版本约束
		VersionConstraint:      "< 1000.0.0",
		// 指定版本格式为语义化版本
		VersionFormat:          "semver",
		// 指定 CPEs 和相关漏洞为空
		CPEs:                   nil,
		RelatedVulnerabilities: nil,
		// 指定修复版本和状态
		Fix: v5.Fix{
			Versions: []string{"1000.0.1"},
			State:    v5.FixedState,
		},
	},
	{
# 设置漏洞信息的数据结构
ID:                     "yet-another-cve-with-advisories",  # 漏洞ID
PackageName:            "package-name",  # 包名
Namespace:              "my-namespace",  # 命名空间
VersionConstraint:      "< 1000.0.0",  # 版本约束
VersionFormat:          "semver",  # 版本格式
CPEs:                   nil,  # CPEs信息
RelatedVulnerabilities: nil,  # 相关漏洞信息
Fix: v5.Fix{  # 漏洞修复信息
    Versions: []string{"1000.0.1"},  # 修复版本
    State:    v5.FixedState,  # 修复状态
},
Advisories: []v5.Advisory{{ID: "ABC-12345", Link: "https://abc.xyz"}},  # 漏洞公告信息
}

total := append(expected, extra...)  # 将期望的漏洞信息和额外的漏洞信息合并为一个总的漏洞信息列表

if err = s.AddVulnerability(total...); err != nil {  # 将总的漏洞信息添加到系统中
    t.Fatalf("failed to set Vulnerability: %+v", err)  # 如果添加失败，则输出错误信息
}
// 声明一个空的VulnerabilityModel切片
var allEntries []model.VulnerabilityModel
// 从数据库中查找所有的VulnerabilityModel并存储到allEntries切片中
s.(*store).db.Find(&allEntries)
// 检查allEntries切片的长度是否与total的长度相等，如果不相等则输出错误信息
if len(allEntries) != len(total) {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}

// 调用assertVulnerabilityReader函数，检查VulnerabilityReader的输出是否符合预期
assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)
}

// 定义一个函数，用于检查VulnerabilityMetadataStoreReader的输出是否符合预期
func assertVulnerabilityMetadataReader(t *testing.T, reader v5.VulnerabilityMetadataStoreReader, id, namespace string, expected v5.VulnerabilityMetadata) {
    // 通过id和namespace从reader中获取VulnerabilityMetadata，并将结果存储到actual中
    if actual, err := reader.GetVulnerabilityMetadata(id, namespace); err != nil {
        // 如果获取失败，则输出错误信息
        t.Fatalf("failed to get metadata: %+v", err)
    } else if actual == nil {
        // 如果获取的结果为空，则输出错误信息
        t.Fatalf("no metadata returned for id=%q namespace=%q", id, namespace)
    } else {
        // 对actual和expected的Cvss字段进行排序
        sortMetadataCvss(actual.Cvss)
        sortMetadataCvss(expected.Cvss)
    }
}
// 确保预期和实际的 CVSS 条目数量相同，以防止后续断言时出现 panic
assert.Len(t, expected.Cvss, len(actual.Cvss))
// 遍历实际的 CVSS 条目，逐一比较向量、版本和度量值
for idx, actualCvss := range actual.Cvss {
    assert.Equal(t, actualCvss.Vector, expected.Cvss[idx].Vector)
    assert.Equal(t, actualCvss.Version, expected.Cvss[idx].Version)
    assert.Equal(t, actualCvss.Metrics, expected.Cvss[idx].Metrics)

    // 将实际和预期的供应商元数据转换为 JSON 字符串并比较
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

// 将 Cvss 字段置空，因为它是一个接口类型，用于验证 Cvss
		// 在这一点上，已经发生了预期的情况
		expected.Cvss = nil
		actual.Cvss = nil
		// 断言预期值和实际值相等
		assert.Equal(t, &expected, actual)
	}

}

func sortMetadataCvss(cvss []v5.Cvss) {
	// 对 Cvss 数组进行排序
	sort.Slice(cvss, func(i, j int) bool {
		// 首先，按照 Vector 进行排序
		if cvss[i].Vector > cvss[j].Vector {
			return true
		}
		if cvss[i].Vector < cvss[j].Vector {
			return false
		}
		// 如果 Vector 相同，则尝试按照 BaseScore 进行排序
		return cvss[i].Metrics.BaseScore < cvss[j].Metrics.BaseScore
	})
```
// CustomMetadata 是一个空操作，它的值并不具有实际意义，主要用于确保任何类型都可以被存储然后在这些测试用例中进行断言，其中使用自定义供应商 CVSS 分数
type CustomMetadata struct {
    SuperScore string
    Vendor     string
}

func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
    // 创建临时文件夹用于数据库
    dbTempFile := t.TempDir()

    // 创建一个新的存储对象
    s, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 定义一个包含 v5.VulnerabilityMetadata 结构的切片
    total := []v5.VulnerabilityMetadata{
        {
# 设置 CVE 的 ID
ID:           "my-cve",
# 设置记录来源
RecordSource: "record-source",
# 设置命名空间
Namespace:    "namespace",
# 设置严重程度
Severity:     "pretty bad",
# 设置相关 URL
URLs:         []string{"https://ancho.re"},
# 设置描述
Description:  "best description ever",
# 设置 CVSS 评分
Cvss: []v5.Cvss{
    # 设置厂商元数据
    {
        VendorMetadata: CustomMetadata{
            Vendor:     "redhat",
            SuperScore: "1000",
        },
        # 设置版本
        Version: "2.0",
        # 设置 CVSS 指标
        Metrics: v5.NewCvssMetrics(
            1.1,
            2.2,
            3.3,
        ),
        # 设置向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--NOT",
    },
# 创建一个包含漏洞信息的数据结构
{
    # 漏洞版本号
    Version: "3.0",
    # 使用 v5 包中的 NewCvssMetrics 方法创建漏洞的 CVSS 指标
    Metrics: v5.NewCvssMetrics(
        1.3,
        2.1,
        3.2,
    ),
    # 漏洞向量
    Vector:         "AV:N/AC:L/Au:N/C:P/I:P/A:P--NICE",
    # 供应商元数据
    VendorMetadata: nil,
},
# 创建另一个漏洞信息的数据结构
{
    # 漏洞 ID
    ID:           "my-other-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace:    "namespace",
    # 漏洞严重程度
    Severity:     "pretty bad",
    # 相关 URL
    URLs:         []string{"https://ancho.re"},
    # 漏洞描述
    Description:  "worst description ever",
    # 漏洞的 CVSS 指标
    Cvss: []v5.Cvss{
# 创建一个包含两个元素的列表，每个元素都是一个字典
{
    # 第一个元素的字典
    Version: "2.0",
    # 使用 v5 包中的 NewCvssMetrics 函数创建一个包含 CVSS 指标的对象
    Metrics: v5.NewCvssMetrics(
        4.1,
        5.2,
        6.3,
    ),
    # 设置向量字符串
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
},
{
    # 第二个元素的字典
    Version: "3.0",
    # 使用 v5 包中的 NewCvssMetrics 函数创建一个包含 CVSS 指标的对象
    Metrics: v5.NewCvssMetrics(
        1.4,
        2.5,
        3.6,
    ),
    # 设置向量字符串
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
// 如果添加漏洞元数据失败，则输出错误信息
if err = s.AddVulnerabilityMetadata(total...); err != nil {
    t.Fatalf("failed to set metadata: %+v", err)
}

// 查找所有漏洞元数据并存储在allEntries中
var allEntries []model.VulnerabilityMetadataModel
s.(*store).db.Find(&allEntries)

// 检查存储的漏洞元数据数量是否与预期数量相同，如果不同则输出错误信息
if len(allEntries) != len(total) {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}
```


```
}

// 测试存储合并漏洞元数据的功能
func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string
        add      []v5.VulnerabilityMetadata
        expected v5.VulnerabilityMetadata
        err      bool
# 创建一个包含漏洞元数据的结构体切片
add: []v5.VulnerabilityMetadata{
    # 添加一个漏洞元数据对象
    {
        # 漏洞ID
        ID: "my-cve",
        # 记录来源
        RecordSource: "record-source",
        # 命名空间
        Namespace: "namespace",
        # 严重程度
        Severity: "pretty bad",
        # 相关URL
        URLs: []string{"https://ancho.re"},
        # 描述
        Description: "worst description ever",
        # Cvss评分
        Cvss: []v5.Cvss{
            {
                # Cvss版本
                Version: "2.0",
                # Cvss指标
                Metrics: v5.NewCvssMetrics(
                    4.1,
                    5.2,
                    6.3,
                ),
                # Cvss向量
                Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
						},
						{
							// 版本号为3.0
							Version: "3.0",
							// 使用v5.NewCvssMetrics创建CVSS指标
							Metrics: v5.NewCvssMetrics(
								1.4, // 基础分数
								2.5, // 攻击向量
								3.6, // 攻击复杂性
							),
							// 漏洞向量
							Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
						},
					},
				},
			},
			// 期望的漏洞元数据
			expected: v5.VulnerabilityMetadata{
				ID:           "my-cve", // 漏洞ID
				RecordSource: "record-source", // 记录来源
				Namespace:    "namespace", // 命名空间
				Severity:     "pretty bad", // 严重程度
				URLs:         []string{"https://ancho.re"}, // 相关URL
				Description:  "worst description ever", // 描述
# 创建一个包含多个 CVSS 对象的列表
Cvss: []v5.Cvss{
    # 创建第一个 CVSS 对象
    {
        # 设置 CVSS 版本号
        Version: "2.0",
        # 设置 CVSS 指标
        Metrics: v5.NewCvssMetrics(
            4.1,  # 设置基本指标
            5.2,  # 设置基本指标
            6.3,  # 设置基本指标
        ),
        # 设置 CVSS 向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 创建第二个 CVSS 对象
    {
        # 设置 CVSS 版本号
        Version: "3.0",
        # 设置 CVSS 指标
        Metrics: v5.NewCvssMetrics(
            1.4,  # 设置基本指标
            2.5,  # 设置基本指标
            3.6,  # 设置基本指标
        ),
        # 设置 CVSS 向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
# 创建一个名为 "merge-links" 的对象，包含一个名为 "add" 的属性，属性值为一个包含多个漏洞元数据的数组
{
    name: "merge-links",
    add: []v5.VulnerabilityMetadata{
        # 第一个漏洞元数据
        {
            ID:           "my-cve",
            RecordSource: "record-source",
            Namespace:    "namespace",
            Severity:     "pretty bad",
            URLs:         []string{"https://ancho.re"},
        },
        # 第二个漏洞元数据
        {
            ID:           "my-cve",
            RecordSource: "record-source",
            Namespace:    "namespace",
            Severity:     "pretty bad",
            URLs:         []string{"https://google.com"},
        },
        {
# 设置漏洞的ID
ID:           "my-cve",
# 设置记录来源
RecordSource: "record-source",
# 设置命名空间
Namespace:    "namespace",
# 设置漏洞严重程度
Severity:     "pretty bad",
# 设置漏洞相关URL
URLs:         []string{"https://yahoo.com"},
# 设置预期的漏洞元数据
expected: v5.VulnerabilityMetadata{
    ID:           "my-cve",
    RecordSource: "record-source",
    Namespace:    "namespace",
    Severity:     "pretty bad",
    URLs:         []string{"https://ancho.re", "https://google.com", "https://yahoo.com"},
    Cvss:         []v5.Cvss{},
},
# 设置漏洞名称为"bad-severity"
name: "bad-severity",
# 添加漏洞元数据
add: []v5.VulnerabilityMetadata{
    {
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 漏洞的记录来源
    Namespace:    "namespace",  # 漏洞的命名空间
    Severity:     "pretty bad",  # 漏洞的严重程度
    URLs:         []string{"https://ancho.re"},  # 漏洞相关的 URL 列表
},
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 漏洞的记录来源
    Namespace:    "namespace",  # 漏洞的命名空间
    Severity:     "meh, push that for next tuesday...",  # 漏洞的严重程度
    URLs:         []string{"https://redhat.com"},  # 漏洞相关的 URL 列表
},
# 设置错误标志为 true
err: true,
},
{
    name: "mismatch-description",  # 漏洞描述不匹配的情况
    err:  true,  # 设置错误标志为 true
    add: []v5.VulnerabilityMetadata{  # 添加漏洞元数据
# 创建一个包含 CVE 信息的结构体
{
    # CVE ID
    ID:           "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace:    "namespace",
    # 严重程度
    Severity:     "pretty bad",
    # 相关 URL
    URLs:         []string{"https://ancho.re"},
    # 描述
    Description:  "best description ever",
    # CVE 的 CVSS 评分
    Cvss: []v5.Cvss{
        # 第一个 CVE 的 CVSS 评分
        {
            # 版本
            Version: "2.0",
            # 评分指标
            Metrics: v5.NewCvssMetrics(
                4.1,
                5.2,
                6.3,
            ),
            # 向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
        # 第二个 CVE 的 CVSS 评分
        {
            # 版本
            Version: "3.0",
# 创建一个新的 CVE 记录
{
    # CVE 记录的 ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 严重程度
    Severity: "pretty bad",
    # 相关 URL
    URLs: []string{"https://ancho.re"},
    # 描述
    Description: "worst description ever",
    # CVE 记录的 CVSS 评分
    Cvss: []v5.Cvss{
        {
            # CVSS 版本
            Version: "2.0",
            # CVSS 评分指标
            Metrics: v5.NewCvssMetrics(
                # 基础评分
                1.4,
                # 攻击向量复杂性
                2.5,
                # 攻击向量影响
                3.6,
            ),
            # CVSS 向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
        },
    },
},
# 创建一个包含多个漏洞信息的数据结构
{
    # 漏洞的名称
    Name: "CVE-2021-12345",
    # 漏洞的描述
    Description: "This is a vulnerability description.",
    # 漏洞的解决方案
    Solution: "Apply the latest security patch.",
    # 漏洞的影响
    Impact: "This vulnerability may lead to unauthorized access.",
    # 漏洞的相关链接
    References: ["https://example.com/cve-2021-12345"],
    # 漏洞的CVSS评分
    Cvss: {
        # CVSS的版本
        Version: "3.1",
        # CVSS的指标
        Metrics: v3.NewCvssMetrics(
            # 基本指标：攻击向量、攻击复杂度、身份认证、机密性、完整性、可用性
            4.1,
            5.2,
            6.3,
        ),
        # CVSS的向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 另一个漏洞的CVSS评分
    Cvss: {
        # CVSS的版本
        Version: "3.0",
        # CVSS的指标
        Metrics: v5.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
        # CVSS的向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
			# 定义名称为 "mismatch-cvss2" 的漏洞
			name: "mismatch-cvss2",
			# 错误标志位为 false
			err:  false,
			# 添加一个 VulnerabilityMetadata 对象到 add 列表中
			add: []v5.VulnerabilityMetadata{
				{
					# 漏洞的唯一标识符为 "my-cve"
					ID:           "my-cve",
					# 记录来源为 "record-source"
					RecordSource: "record-source",
					# 命名空间为 "namespace"
					Namespace:    "namespace",
					# 严重程度为 "pretty bad"
					Severity:     "pretty bad",
					# URL 列表包含 "https://ancho.re"
					URLs:         []string{"https://ancho.re"},
					# 描述为 "best description ever"
					Description:  "best description ever",
					# Cvss 列表包含一个 Cvss 对象
					Cvss: []v5.Cvss{
						{
							# Cvss 版本为 "2.0"
							Version: "2.0",
							# Cvss 指标为 (4.1, 5.2, 6.3)
							Metrics: v5.NewCvssMetrics(
								4.1,
								5.2,
								6.3,
							),
							# Cvss 向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
							Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
						},
# 创建一个包含漏洞信息的数据结构
{
    # 漏洞版本号
    Version: "3.0",
    # 使用 v5.NewCvssMetrics 创建漏洞的 CVSS 指标
    Metrics: v5.NewCvssMetrics(
        1.4,
        2.5,
        3.6,
    ),
    # 漏洞向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 创建另一个漏洞信息的数据结构
{
    # 漏洞 ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 漏洞严重程度
    Severity: "pretty bad",
    # 相关 URL
    URLs: []string{"https://ancho.re"},
    # 漏洞描述
    Description: "best description ever",
    # 漏洞的 CVSS 指标
    Cvss: []v5.Cvss{
        {
# 创建一个包含两个元素的列表，每个元素都是一个字典
[
    {
        # 第一个元素的字典包含 Version、Metrics 和 Vector 三个键值对
        Version: "2.0",
        Metrics: v5.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        Vector: "AV:P--VERY",
    },
    {
        # 第二个元素的字典包含 Version、Metrics 和 Vector 三个键值对
        Version: "3.0",
        Metrics: v5.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
],
# 创建一个 v5.VulnerabilityMetadata 结构体实例，包含漏洞的元数据信息
expected: v5.VulnerabilityMetadata{
    # 设置漏洞的 ID
    ID:           "my-cve",
    # 设置记录来源
    RecordSource: "record-source",
    # 设置命名空间
    Namespace:    "namespace",
    # 设置漏洞的严重程度
    Severity:     "pretty bad",
    # 设置漏洞相关的 URL
    URLs:         []string{"https://ancho.re"},
    # 设置漏洞的描述
    Description:  "best description ever",
    # 设置漏洞的 CVSS 评分
    Cvss: []v5.Cvss{
        # 创建一个 v5.Cvss 结构体实例，包含 CVSS 2.0 版本的评分信息
        {
            Version: "2.0",
            # 设置 CVSS 2.0 版本的评分指标
            Metrics: v5.NewCvssMetrics(
                4.1,
                5.2,
                6.3,
            ),
            # 设置 CVSS 2.0 版本的向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
        # 创建一个 v5.Cvss 结构体实例，包含 CVSS 3.0 版本的评分信息
        {
            Version: "3.0",
            # 设置 CVSS 3.0 版本的评分指标
            Metrics: v5.NewCvssMetrics(
# 创建一个包含漏洞信息的列表
[
    {
        # 漏洞名称
        name: "mismatch-cvss2",
        # 漏洞的CVSS版本
        cvssVersion: "2.0",
        # 漏洞的CVSS指标
        cvssMetrics: {
            # 创建一个CVSS指标对象
            Metrics: v2.NewCvssMetrics(
                # 设置CVSS指标的值
                1.4,
                2.5,
                3.6,
            ),
            # 设置CVSS向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
        },
    },
    {
        # 漏洞名称
        name: "mismatch-cvss3",
// 初始化错误标志为false
err: false,
// 初始化漏洞元数据数组
add: []v5.VulnerabilityMetadata{
    // 添加漏洞元数据对象
    {
        // 设置漏洞ID
        ID: "my-cve",
        // 设置记录来源
        RecordSource: "record-source",
        // 设置命名空间
        Namespace: "namespace",
        // 设置严重程度
        Severity: "pretty bad",
        // 设置URL数组
        URLs: []string{"https://ancho.re"},
        // 设置描述
        Description: "best description ever",
        // 设置CVSS评分数组
        Cvss: []v5.Cvss{
            {
                // 设置CVSS版本
                Version: "2.0",
                // 设置CVSS指标
                Metrics: v5.NewCvssMetrics(
                    4.1,
                    5.2,
                    6.3,
                ),
                // 设置CVSS向量
                Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
            },
            {
                // ...
# 设置版本号为3.0
Version: "3.0",
# 创建一个新的CVSS指标对象，设置相应的指标值
Metrics: v5.NewCvssMetrics(
    1.4,
    2.5,
    3.6,
),
# 设置向量为"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
# 设置ID为"my-cve"
ID: "my-cve",
# 设置记录来源为"record-source"
RecordSource: "record-source",
# 设置命名空间为"namespace"
Namespace: "namespace",
# 设置严重程度为"pretty bad"
Severity: "pretty bad",
# 设置URLs为包含"https://ancho.re"的字符串数组
URLs: []string{"https://ancho.re"},
# 设置描述为"best description ever"
Description: "best description ever",
# 设置CVSS为包含一个CVSS对象的数组
Cvss: []v5.Cvss{
    {
        # 设置版本号为2.0
        Version: "2.0",
# 创建一个新的 CVSS 指标对象，包括基础分数、基础向量
Metrics: v5.NewCvssMetrics(
    4.1,    # 基础分数
    5.2,    # 基础分数
    6.3,    # 基础分数
),
# 设置基础向量
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",

# 创建另一个新的 CVSS 指标对象，包括基础分数、基础向量
{
    Version: "3.0",    # 版本号
    Metrics: v5.NewCvssMetrics(
        1.4,    # 基础分数
        0,      # 基础分数
        3.6,    # 基础分数
    ),
    # 设置基础向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 设置 CVE 的 ID
ID:           "my-cve",
# 设置记录来源
RecordSource: "record-source",
# 设置命名空间
Namespace:    "namespace",
# 设置严重程度
Severity:     "pretty bad",
# 设置相关 URL
URLs:         []string{"https://ancho.re"},
# 设置描述
Description:  "best description ever",
# 设置 CVE 的 CVSS 评分
Cvss: []v5.Cvss{
    # 设置 CVSS 版本为 2.0 的评分
    {
        Version: "2.0",
        # 设置 CVSS 2.0 的指标
        Metrics: v5.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        # 设置 CVSS 2.0 的向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 设置 CVSS 版本为 3.0 的评分
    {
        Version: "3.0",
        # 设置 CVSS 3.0 的指标
        Metrics: v5.NewCvssMetrics(
            1.4,
# 定义一个包含多个测试用例的测试集合
tests := []struct {
	// 定义每个测试用例的结构体
	Version string // 版本号
	Metrics v5.CvssMetrics // CVSS指标
	Vector string // 向量
}{
	{
		// 第一个测试用例
		Version: "2.0", // 版本号为2.0
		Metrics: v5.NewCvssMetrics( // 创建一个新的CVSS指标对象
			2.5, // 设置基本指标
			3.6,
			0,
		),
		Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD", // 设置向量
	},
	{
		// 第二个测试用例
		Version: "3.0", // 版本号为3.0
		Metrics: v5.NewCvssMetrics( // 创建一个新的CVSS指标对象
			1.4, // 设置基本指标
			0,
			3.6,
		),
		Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD", // 设置向量
	},
}

for _, test := range tests {
	// 遍历测试集合中的每个测试用例
		// 使用测试名称运行测试函数
		t.Run(test.name, func(t *testing.T) {
			// 创建临时目录用于数据库
			dbTempDir := t.TempDir()

			// 创建新的存储对象
			s, err := New(dbTempDir, true)
			if err != nil {
				t.Fatalf("could not create store: %+v", err)
			}

			// 按顺序添加每个元数据
			var theErr error
			for _, metadata := range test.add {
				err = s.AddVulnerabilityMetadata(metadata)
				if err != nil {
					theErr = err
					break
				}
			}

			// 如果期望出现错误但未出现，则报错
			if test.err && theErr == nil {
				t.Fatalf("expected error but did not get one")
			} else if !test.err && theErr != nil {
				// 如果预期没有错误但实际有错误，则测试失败
				t.Fatalf("expected no error but got one: %+v", theErr)
			} else if test.err && theErr != nil {
				// 如果预期有错误且实际也有错误，则测试通过，直接返回
				return
			}

			// 确保只有一个条目
			var allEntries []model.VulnerabilityMetadataModel
			s.(*store).db.Find(&allEntries)
			if len(allEntries) != 1 {
				// 如果条目数量不是1，则测试失败
				t.Fatalf("unexpected number of entries: %d", len(allEntries))
			}

			// 获取结果的元数据对象
			if actual, err := s.GetVulnerabilityMetadata(test.expected.ID, test.expected.Namespace); err != nil {
				// 如果获取元数据对象时出错，则测试失败
				t.Fatalf("failed to get metadata: %+v", err)
			} else {
				// 比较预期和实际的元数据对象
				diffs := deep.Equal(&test.expected, actual)
				if len(diffs) > 0 {
					// 如果存在差异，则测试失败
# 遍历 diffs 列表中的元素，将每个元素输出为错误信息
for _, d := range diffs {
    t.Errorf("Diff: %+v", d)
}
# 结束当前测试用例
}
}
})
}
}

# 定义测试函数 TestCvssScoresInMetadata
func TestCvssScoresInMetadata(t *testing.T) {
    # 定义测试用例列表
    tests := []struct {
        name     string
        add      []v5.VulnerabilityMetadata
        expected v5.VulnerabilityMetadata
    }{
        {
            name: "append-cvss",
            add: []v5.VulnerabilityMetadata{
                {
                    ID:           "my-cve",
```

					RecordSource: "record-source",  // 设置记录来源为 "record-source"
					Namespace:    "namespace",     // 设置命名空间为 "namespace"
					Severity:     "pretty bad",     // 设置严重程度为 "pretty bad"
					URLs:         []string{"https://ancho.re"},  // 设置URL数组为 ["https://ancho.re"]
					Description:  "worst description ever",  // 设置描述为 "worst description ever"
					Cvss: []v5.Cvss{  // 设置Cvss数组
						{
							Version: "2.0",  // 设置版本为 "2.0"
							Metrics: v5.NewCvssMetrics(  // 设置Cvss指标
								4.1,  // 设置基本评分为 4.1
								5.2,  // 设置严重程度为 5.2
								6.3,  // 设置可利用性为 6.3
							),
							Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  // 设置向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
						},
					},
				},
				{
					ID:           "my-cve",  // 设置ID为 "my-cve"
					RecordSource: "record-source",  // 设置记录来源为 "record-source"
# 定义命名空间为 "namespace"
Namespace:    "namespace",
# 定义严重程度为 "pretty bad"
Severity:     "pretty bad",
# 定义 URL 列表为空
URLs:         []string{"https://ancho.re"},
# 定义描述为 "worst description ever"
Description:  "worst description ever",
# 定义 CVSS 列表，包含一个 v5.Cvss 对象
Cvss: []v5.Cvss{
    {
        # 定义 CVSS 版本为 "3.0"
        Version: "3.0",
        # 定义 CVSS 指标
        Metrics: v5.NewCvssMetrics(
            1.4,    # 基础分数
            2.5,    # 攻击向量
            3.6,    # 攻击复杂度
        ),
        # 定义 CVSS 向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
# 定义预期的漏洞元数据
expected: v5.VulnerabilityMetadata{
    # 定义漏洞 ID 为 "my-cve"
    ID:           "my-cve",
    # 定义记录来源为 "record-source"
    RecordSource: "record-source",
# 设置命名空间为 "namespace"
Namespace:    "namespace",
# 设置严重程度为 "pretty bad"
Severity:     "pretty bad",
# 设置 URL 列表为空
URLs:         []string{"https://ancho.re"},
# 设置描述为 "worst description ever"
Description:  "worst description ever",
# 设置 CVSS 列表
Cvss: []v5.Cvss{
    # 设置第一个 CVSS 版本为 "2.0"，指标为 (4.1, 5.2, 6.3)，向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
    {
        Version: "2.0",
        Metrics: v5.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 设置第二个 CVSS 版本为 "3.0"，指标为 (1.4, 2.5, 3.6)
    {
        Version: "3.0",
        Metrics: v5.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
# 定义一个名为 "append-vendor-cvss" 的元数据记录，包含漏洞的相关信息
{
    name: "append-vendor-cvss",
    add: []v5.VulnerabilityMetadata{
        # 定义一个名为 "my-cve" 的漏洞 ID
        {
            ID: "my-cve",
            # 漏洞记录来源
            RecordSource: "record-source",
            # 命名空间
            Namespace: "namespace",
            # 漏洞严重程度
            Severity: "pretty bad",
            # 相关 URL
            URLs: []string{"https://ancho.re"},
            # 漏洞描述
            Description: "worst description ever",
            # 定义 CVSS 版本 2.0 的指标
            Cvss: []v5.Cvss{
                {
                    Version: "2.0",
                    # 创建 CVSS 指标
                    Metrics: v5.NewCvssMetrics(
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 漏洞严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 漏洞描述
    Cvss: []v5.Cvss{  # CVSS评分
        {
            Version: "2.0",  # CVSS版本
            Metrics: v5.NewCvssMetrics(  # CVSS指标
                4.1,  # 指标数值
                5.2,
                6.3,
            ),
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # CVSS向量
        },
    },
},
# 定义一个包含多个键值对的数据结构，包括 ID、RecordSource、Namespace、Severity、URLs、Description 和 Cvss
expected: v5.VulnerabilityMetadata{
    # 漏洞的唯一标识符
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 严重程度
    Severity: "pretty bad",
    # 相关链接
    URLs: []string{"https://ancho.re"},
    # 描述信息
    Description: "worst description ever",
    # 包含 CVSS 评分的数据结构
    Cvss: []v5.Cvss{
# 创建一个包含漏洞信息的对象
{
    # 指定漏洞信息的版本号
    Version: "2.0",
    # 创建一个包含 CVSS 指标的对象
    Metrics: v5.NewCvssMetrics(
        4.1,    # 基础指标
        5.2,    # 基础指标
        6.3,    # 基础指标
    ),
    # 指定漏洞的向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    # 添加厂商自定义的元数据
    VendorMetadata: CustomMetadata{
        SuperScore: "100",  # 自定义的漏洞评分
        Vendor: "debian",    # 自定义的厂商信息
    }
}
# 定义一个名为 "avoids-duplicate-cvss" 的测试用例
{
    name: "avoids-duplicate-cvss",
    # 添加一个漏洞元数据对象到测试用例中
    add: []v5.VulnerabilityMetadata{
        {
            ID:           "my-cve",
            RecordSource: "record-source",
            Namespace:    "namespace",
            Severity:     "pretty bad",
            URLs:         []string{"https://ancho.re"},
            Description:  "worst description ever",
            # 添加一个 CVSS 指标对象到漏洞元数据中
            Cvss: []v5.Cvss{
                {
                    Version: "3.0",
                    # 创建一个新的 CVSS 指标对象
                    Metrics: v5.NewCvssMetrics(
                        # 设置 CVSS 指标的基础分数
                        1.4,
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 漏洞严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 漏洞描述
    Cvss: []v5.Cvss{  # 使用CVSS标准的漏洞评分
        {
            Version: "3.0",  # CVSS版本
            Metrics: v5.NewCvssMetrics(  # 创建CVSS指标
                1.4,  # 基础指标
                2.5,  # 基础指标
                3.6,  # 基础指标
            ),
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # CVSS向量
        },
    },
},
# 定义一个预期的漏洞元数据对象
expected: v5.VulnerabilityMetadata{
    # 漏洞ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 严重程度
    Severity: "pretty bad",
    # 相关URL
    URLs: []string{"https://ancho.re"},
    # 描述
    Description: "worst description ever",
    # CVSS评分
    Cvss: []v5.Cvss{
        {
            # CVSS版本
            Version: "3.0",
            # CVSS指标
            Metrics: v5.NewCvssMetrics(
                1.4, # 基础分数
                2.5, # 基础向量
                3.6, # 基础向量
                Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD", # 向量
            ),
        },
    },
},
# 定义一个包含测试用例的切片
tests := []struct {
	// 定义测试用例的名称
	name string
	// 定义测试用例的输入数据
	add []VulnerabilityMetadata
	// 定义测试用例的期望输出结果
	expected map[string]map[string]map[string]map[string]float64
}{
	// 定义测试用例1
	{
		name: "Test1",
		add: []VulnerabilityMetadata{
			// 定义测试用例1的输入数据
			{
				// 定义测试用例1的输入数据中的子字段
				CVSS: map[string]map[string]map[string]float64{
					"3.0": {
						"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD": {
							// 定义测试用例1的输入数据中的子字段
							BaseScore: 3.6,
						},
					},
				},
			},
		},
		// 定义测试用例1的期望输出结果
		expected: map[string]map[string]map[string]map[string]float64{
			"3.0": {
				"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD": {
					// 定义测试用例1的期望输出结果中的子字段
					BaseScore: 3.6,
				},
			},
		},
	},
}

# 遍历测试用例切片，对每个测试用例执行测试
for _, test := range tests {
	# 使用测试框架的子测试功能，对每个测试用例创建一个子测试
	t.Run(test.name, func(t *testing.T) {
		# 在临时目录创建一个数据库
		dbTempDir := t.TempDir()

		# 创建一个新的对象s，并检查是否有错误发生
		s, err := New(dbTempDir, true)
		if err != nil {
			t.Fatalf("could not create s: %+v", err)
		}

		# 依次添加每个元数据
		for _, metadata := range test.add {
			err = s.AddVulnerabilityMetadata(metadata)
// 如果发生错误，输出错误信息并终止测试
if err != nil {
    t.Fatalf("unable to s vulnerability metadata: %+v", err)
}

// 确保只有一个条目
var allEntries []model.VulnerabilityMetadataModel
s.(*store).db.Find(&allEntries)
if len(allEntries) != 1 {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}

// 调用 assertVulnerabilityMetadataReader 函数，验证读取的漏洞元数据
assertVulnerabilityMetadataReader(t, s, test.expected.ID, test.expected.Namespace, test.expected)
```

这段代码主要是对漏洞元数据进行读取和验证的过程，包括处理错误、确保只有一个条目以及验证读取的漏洞元数据。
	} else {
		// 如果条件不满足，记录 actual 的值
		t.Logf("%+v", actual)
		// 检查 actual 和 expected 的长度是否相等，如果不相等则输出错误信息
		if len(actual) != len(expected) {
			t.Fatalf("unexpected number of vulnerability match exclusions: expected=%d, actual=%d", len(expected), len(actual))
		}
		// 遍历 actual 数组
		for idx := range actual {
			// 比较 expected 和 actual 的值，记录差异
			diffs := deep.Equal(expected[idx], actual[idx])
			// 如果有差异，输出差异信息
			if len(diffs) > 0 {
				for _, d := range diffs {
					t.Errorf("Diff: %+v", d)
				}
			}
		}
	}
}

func TestStore_GetVulnerabilityMatchExclusion_SetVulnerabilityMatchExclusion(t *testing.T) {
	// 创建临时数据库文件
	dbTempFile := t.TempDir()
	// 创建新的数据库实例
	s, err := New(dbTempFile, true)
// 如果发生错误，输出错误信息并终止测试
if err != nil {
    t.Fatalf("could not create store: %+v", err)
}

// 创建一个包含漏洞匹配排除的额外信息的数组
extra := []v5.VulnerabilityMatchExclusion{
    // 创建一个漏洞匹配排除对象，包括ID和约束条件
    {
        ID: "CVE-1234-14567",
        Constraints: []v5.VulnerabilityMatchExclusionConstraint{
            // 创建一个漏洞匹配排除约束条件，包括漏洞和包的信息
            {
                Vulnerability: v5.VulnerabilityExclusionConstraint{
                    Namespace: "extra-namespace:cpe",
                },
                Package: v5.PackageExclusionConstraint{
                    Name:     "abc",
                    Language: "ruby",
                    Version:  "1.2.3",
                },
            },
            {
                // 创建另一个漏洞匹配排除约束条件
                Vulnerability: v5.VulnerabilityExclusionConstraint{
# 创建一个包含漏洞和包排除约束的列表
[
    # 第一个排除约束
    {
        # 漏洞排除约束的命名空间
        Vulnerability: v5.VulnerabilityExclusionConstraint{
            Namespace: "extra-namespace:cpe",
        },
        # 包排除约束的详细信息
        Package: v5.PackageExclusionConstraint{
            Name:     "abc",
            Language: "ruby",
            Version:  "4.5.6",
        },
    },
    # 第二个排除约束
    {
        # 漏洞排除约束的命名空间
        Vulnerability: v5.VulnerabilityExclusionConstraint{
            Namespace: "extra-namespace:cpe",
        },
        # 包排除约束的详细信息
        Package: v5.PackageExclusionConstraint{
            Name:     "time-1",
            Language: "ruby",
        },
    },
    # 第三个排除约束
    {
        # 漏洞排除约束的命名空间
        Vulnerability: v5.VulnerabilityExclusionConstraint{
            Namespace: "extra-namespace:cpe",
        },
        # 包排除约束的详细信息
        # （缺少版本信息）
    },
]
# 创建一个包含漏洞匹配排除规则的切片
expected := []v5.VulnerabilityMatchExclusion{
    # 创建一个漏洞匹配排除规则对象
    {
        # 设置漏洞ID
        ID: "CVE-1234-9999999",
        # 设置漏洞匹配排除规则约束
        Constraints: []v5.VulnerabilityMatchExclusionConstraint{
            # 创建一个漏洞匹配排除规则约束对象
# 创建一个漏洞排除约束对象，指定漏洞的命名空间为"old-namespace:cpe"
# 指定要排除的包的语言为python，名称为abc，版本为1.2.3
{
    Vulnerability: v5.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v5.PackageExclusionConstraint{
        Language: "python",
        Name: "abc",
        Version: "1.2.3",
    },
},

# 创建另一个漏洞排除约束对象，指定漏洞的命名空间为"old-namespace:cpe"
# 指定要排除的包的语言为python，名称为abc，版本为4.5.6
{
    Vulnerability: v5.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v5.PackageExclusionConstraint{
        Language: "python",
        Name: "abc",
        Version: "4.5.6",
    },
},
# 创建一个漏洞排除约束对象，指定漏洞的命名空间为"old-namespace:cpe"
# 并且指定要排除的包的语言为python，包名为"time-245"
{
    Vulnerability: v5.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v5.PackageExclusionConstraint{
        Language: "python",
        Name: "time-245",
    },
},
# 创建另一个漏洞排除约束对象，指定漏洞的命名空间为"old-namespace:cpe"
# 并且指定要排除的包的类型为npm，包名为"everything"
{
    Vulnerability: v5.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v5.PackageExclusionConstraint{
        Type: "npm",
        Name: "everything",
    },
},
# 给出排除这些漏洞的理由
Justification: "This is a false positive",
		},
		{
			# 定义漏洞ID
			ID: "CVE-1234-9999999",
			# 定义漏洞匹配排除约束条件列表
			Constraints: []v5.VulnerabilityMatchExclusionConstraint{
				# 定义漏洞匹配排除约束条件
				{
					# 定义漏洞排除约束条件的命名空间
					Vulnerability: v5.VulnerabilityExclusionConstraint{
						Namespace: "old-namespace:cpe",
					},
					# 定义包排除约束条件
					Package: v5.PackageExclusionConstraint{
						# 定义包的语言
						Language: "go",
						# 定义包的类型
						Type:     "go-module",
						# 定义包的名称
						Name:     "abc",
					},
				},
				# 定义另一个漏洞匹配排除约束条件
				{
					# 定义漏洞排除约束条件的命名空间
					Vulnerability: v5.VulnerabilityExclusionConstraint{
						Namespace: "some-other-namespace:cpe",
					},
					# 定义包排除约束条件
					Package: v5.PackageExclusionConstraint{
						# 定义包的语言
						Language: "go",
# 定义一个类型为 "go-module" 的结构体，并设置 Name 为 "abc"
{
    Type:     "go-module",
    Name:     "abc",
},

# 定义一个漏洞排除约束的结构体，并设置 FixState 为 "wont-fix"
{
    Vulnerability: v5.VulnerabilityExclusionConstraint{
        FixState: "wont-fix",
    },
},

# 定义一个漏洞排除约束的结构体，并设置 Justification 为 "This is also a false positive"
{
    Justification: "This is also a false positive",
},

# 定义一个 ID 为 "CVE-1234-9999999" 的结构体，并设置 Justification 为 "global exclude"
{
    ID:            "CVE-1234-9999999",
    Justification: "global exclude",
},

# 将 extra 切片追加到 expected 切片中，并将结果赋给 total
total := append(expected, extra...)
// 如果出现错误，将漏洞匹配排除添加到总数中
if err = s.AddVulnerabilityMatchExclusion(total...); err != nil {
    t.Fatalf("failed to set Vulnerability Match Exclusion: %+v", err)
}

// 查找所有的漏洞匹配排除模型
var allEntries []model.VulnerabilityMatchExclusionModel
s.(*store).db.Find(&allEntries)
// 检查是否返回的条目数量与总数相等
if len(allEntries) != len(total) {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}
// 断言漏洞匹配排除读取器
assertVulnerabilityMatchExclusionReader(t, s, expected[0].ID, expected)
}

func Test_DiffStore(t *testing.T) {
    //GIVEN
    // 创建临时文件夹作为数据库文件路径
    dbTempFile := t.TempDir()
    // 创建新的存储对象
    s1, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }
    // 重新分配临时文件夹作为数据库文件路径
    dbTempFile = t.TempDir()
```

	// 使用 New 函数创建一个新的存储对象，传入 dbTempFile 和 true 作为参数
	s2, err := New(dbTempFile, true)
	// 如果创建存储对象时发生错误，输出错误信息并终止测试
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}

	// 创建一个包含两个漏洞信息的数组
	baseVulns := []v5.Vulnerability{
		{
			Namespace:         "github:language:python",
			ID:                "CVE-123-4567",
			PackageName:       "pypi:requests",
			VersionConstraint: "< 2.0 >= 1.29",
			CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
		},
		{
			Namespace:         "github:language:python",
			ID:                "CVE-123-4567",
			PackageName:       "pypi:requests",
			VersionConstraint: "< 3.0 >= 2.17",
			CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
		},
# 创建一个对象，表示命名空间为 "npm" 的漏洞信息
{
    Namespace:         "npm",  # 命名空间为 "npm"
    ID:                "CVE-123-7654",  # 漏洞ID
    PackageName:       "npm:axios",  # 包名
    VersionConstraint: "< 3.0 >= 2.17",  # 版本约束
    CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},  # CPEs
    Fix: v5.Fix{  # 修复信息
        State: v5.UnknownFixState,  # 修复状态
    },
},
# 创建一个对象，表示命名空间为 "nuget" 的漏洞信息
{
    Namespace:         "nuget",  # 命名空间为 "nuget"
    ID:                "GHSA-****-******",  # 漏洞ID
    PackageName:       "nuget:net",  # 包名
    VersionConstraint: "< 3.0 >= 2.17",  # 版本约束
    CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},  # CPEs
    Fix: v5.Fix{  # 修复信息
        State: v5.UnknownFixState,  # 修复状态
    },
},
# 创建一个包含漏洞信息的结构体，包括命名空间、ID、包名、版本约束和CPEs
{
    Namespace:         "hex",
    ID:                "GHSA-^^^^-^^^^^^",
    PackageName:       "hex:esbuild",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:hex:esbuild:*:*:*:*:*:*"},
}

# 创建一个包含基本漏洞元数据的结构体，包括命名空间、ID和数据源
baseMetadata := []v5.VulnerabilityMetadata{
    {
        Namespace:  "nuget",
        ID:         "GHSA-****-******",
        DataSource: "nvd",
    },
}

# 创建一个包含目标漏洞信息的结构体，包括命名空间、ID、包名
targetVulns := []v5.Vulnerability{
    {
        Namespace:         "github:language:python",
        ID:                "CVE-123-4567",
        PackageName:       "pypi:requests",
    }
}
# 定义版本约束为小于2.0且大于等于1.29
VersionConstraint: "< 2.0 >= 1.29",
# 定义CPEs为空字符串数组
CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
# 定义命名空间为github:language:go
Namespace:         "github:language:go",
# 定义ID为GHSA-....-....
ID:                "GHSA-....-....",
# 定义包名为hashicorp:nomad
PackageName:       "hashicorp:nomad",
# 定义版本约束为小于3.0且大于等于2.17
VersionConstraint: "< 3.0 >= 2.17",
# 定义CPEs为字符串数组
CPEs:              []string{"cpe:2.3:golang:hashicorp:nomad:*:*:*:*:*"},
# 定义命名空间为github:language:go
Namespace:         "github:language:go",
# 定义ID为GHSA-....-....
ID:                "GHSA-....-....",
# 定义包名为hashicorp:n
PackageName:       "hashicorp:n",
# 定义版本约束为小于2.0且大于等于1.17
VersionConstraint: "< 2.0 >= 1.17",
# 定义CPEs为字符串数组
CPEs:              []string{"cpe:2.3:golang:hashicorp:n:*:*:*:*:*"},
# 定义命名空间为npm
Namespace:         "npm",
# 定义ID为CVE-123-7654
ID:                "CVE-123-7654",
# 定义一个包的信息，包名为 "npm:axios"，版本约束为 "< 3.0 >= 2.17"，CPEs 为空数组，修复状态为 WontFixState
{
    Namespace:         "npm",
    PackageName:       "npm:axios",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
    Fix: v5.Fix{
        State: v5.WontFixState,
    },
},
# 定义另一个包的信息，命名空间为 "nuget"，ID 为 "GHSA-****-******"，包名为 "nuget:net"，版本约束为 "< 3.0 >= 2.17"，CPEs 为空数组，修复状态为 UnknownFixState
{
    Namespace:         "nuget",
    ID:                "GHSA-****-******",
    PackageName:       "nuget:net",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
    Fix: v5.Fix{
        State: v5.UnknownFixState,
    },
},
# 定义一个期望的差异数组
expectedDiffs := []v5.Diff{
    {
		{
			# 原因为版本差异变化
			Reason:    v5.DiffChanged,
			# 漏洞ID为CVE-123-4567
			ID:        "CVE-123-4567",
			# 命名空间为Python语言的GitHub
			Namespace: "github:language:python",
			# 受影响的包为pypi:requests
			Packages:  []string{"pypi:requests"},
		},
		{
			# 原因为版本差异变化
			Reason:    v5.DiffChanged,
			# 漏洞ID为CVE-123-7654
			ID:        "CVE-123-7654",
			# 命名空间为npm
			Namespace: "npm",
			# 受影响的包为npm:axios
			Packages:  []string{"npm:axios"},
		},
		{
			# 原因为版本差异移除
			Reason:    v5.DiffRemoved,
			# 漏洞ID为GHSA-****-******
			ID:        "GHSA-****-******",
			# 命名空间为nuget
			Namespace: "nuget",
			# 受影响的包为nuget:net
			Packages:  []string{"nuget:net"},
		},
		{
			# 原因为版本差异添加
			Reason:    v5.DiffAdded,
			# 漏洞ID为GHSA-....-....
# 定义一个结构体切片，包含了语言为Go的命名空间和包列表
baseVulns := []struct {
    Reason    string
    ID        string
    Namespace string
    Packages  []string
}{
    {
        Reason:    v5.DiffAdded,
        ID:        "GHSA-^^^^-^^^^^^",
        Namespace: "github:language:go",
        Packages:  []string{"hashicorp:n", "hashicorp:nomad"},
    },
    {
        Reason:    v5.DiffRemoved,
        ID:        "GHSA-^^^^-^^^^^^",
        Namespace: "hex",
        Packages:  []string{"hex:esbuild"},
    },
}

# 遍历baseVulns切片，将每个漏洞添加到s1中
for _, vuln := range baseVulns {
    s1.AddVulnerability(vuln)
}

# 遍历targetVulns切片，将每个漏洞添加到s2中
for _, vuln := range targetVulns {
    s2.AddVulnerability(vuln)
}

# 遍历baseMetadata切片，将每个元数据添加到s1中
for _, meta := range baseMetadata {
    s1.AddVulnerabilityMetadata(meta)
}
	// 调用DiffStore方法，将s2与s1进行比较，返回比较结果和可能的错误
	result, err := s1.DiffStore(s2)

	// 对比较结果按照ID进行稳定排序
	sort.SliceStable(*result, func(i, j int) bool {
		return (*result)[i].ID < (*result)[j].ID
	})
	// 对比较结果中的每个元素的Packages字段进行排序
	for i := range *result {
		sort.Strings((*result)[i].Packages)
	}

	// 断言，验证错误是否为空
	assert.NoError(t, err)
	// 断言，验证比较结果是否与期望的结果相等
	assert.Equal(t, expectedDiffs, *result)
}
```