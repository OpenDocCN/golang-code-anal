# `grype\grype\db\v4\store\store_test.go`

```
package store

import (
	"encoding/json"  // 导入 JSON 编码解码包
	"sort"  // 导入排序包
	"testing"  // 导入测试包
	"time"  // 导入时间包

	"github.com/go-test/deep"  // 导入深度比较包
	"github.com/stretchr/testify/assert"  // 导入断言包

	v4 "github.com/anchore/grype/grype/db/v4"  // 导入 v4 版本的数据库包
	"github.com/anchore/grype/grype/db/v4/store/model"  // 导入数据库模型包
)

func assertIDReader(t *testing.T, reader v4.IDReader, expected v4.ID) {
	t.Helper()  // 标记该函数是测试辅助函数
	if actual, err := reader.GetID(); err != nil {  // 获取 ID 信息，如果出错则报错
		t.Fatalf("failed to get ID: %+v", err)  // 输出错误信息
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
	expected := v4.ID{
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
func assertVulnerabilityReader(t *testing.T, reader v4.VulnerabilityStoreReader, namespace, name string, expected []v4.Vulnerability) {
	// 获取漏洞信息，如果失败，输出错误信息
	if actual, err := reader.GetVulnerability(namespace, name); err != nil {
		t.Fatalf("failed to get Vulnerability: %+v", err)
	} else {
		// 比较实际漏洞数量和期望漏洞数量
		if len(actual) != len(expected) {
			t.Fatalf("unexpected number of vulns: %d", len(actual))
		}
		// 遍历比较每个漏洞的差异
		for idx := range actual {
			diffs := deep.Equal(expected[idx], actual[idx])
			if len(diffs) > 0 {
# 遍历 diffs 列表中的元素，将每个元素输出为错误信息
for _, d := range diffs:
    t.Errorf("Diff: %+v", d)

# 结束当前函数
}
}

# 定义测试函数 TestStore_GetVulnerability_SetVulnerability
func TestStore_GetVulnerability_SetVulnerability(t *testing.T):
    # 创建临时文件夹作为数据库文件路径
    dbTempFile := t.TempDir()

    # 创建一个新的存储对象，并检查是否有错误发生
    s, err := New(dbTempFile, true)
    if err != nil:
        t.Fatalf("could not create store: %+v", err)

    # 创建一个包含 v4.Vulnerability 类型元素的列表
    extra := []v4.Vulnerability:
        {
            ID:                "my-cve-33333",
            PackageName:       "package-name-2",
```
（注：由于代码片段不完整，无法对其余部分进行准确注释。）
# 设置命名空间为"my-namespace"
Namespace:         "my-namespace",
# 设置版本约束为小于1.0
VersionConstraint: "< 1.0",
# 设置版本格式为语义化版本
VersionFormat:     "semver",
# 设置CPEs为空列表
CPEs:              []string{"a-cool-cpe"},
# 设置相关漏洞引用列表
RelatedVulnerabilities: []v4.VulnerabilityReference{
    # 第一个漏洞引用
    {
        ID:        "another-cve",
        Namespace: "nvd",
    },
    # 第二个漏洞引用
    {
        ID:        "an-other-cve",
        Namespace: "nvd",
    },
},
# 设置修复信息
Fix: v4.Fix{
    # 设置修复版本为"2.0.1"
    Versions: []string{"2.0.1"},
    # 设置状态为已修复
    State:    v4.FixedState,
},
# 设置漏洞的ID为"my-other-cve-33333"
ID: "my-other-cve-33333",
# 设置漏洞所属的包名为"package-name-3"
PackageName: "package-name-3",
# 设置漏洞所属的命名空间为"my-namespace"
Namespace: "my-namespace",
# 设置版本约束为"< 509.2.2"
VersionConstraint: "< 509.2.2",
# 设置版本格式为"semver"
VersionFormat: "semver",
# 设置CPEs为空列表
CPEs: []string{"a-cool-cpe"},
# 设置相关漏洞引用为包含两个v4.VulnerabilityReference对象的列表
RelatedVulnerabilities: []v4.VulnerabilityReference{
    # 第一个v4.VulnerabilityReference对象的ID为"another-cve"，命名空间为"nvd"
    {
        ID: "another-cve",
        Namespace: "nvd",
    },
    # 第二个v4.VulnerabilityReference对象的ID为"an-other-cve"，命名空间为"nvd"
    {
        ID: "an-other-cve",
        Namespace: "nvd",
    },
},
# 设置修复状态为未修复状态
Fix: v4.Fix{
    State: v4.NotFixedState,
},
	}

	expected := []v4.Vulnerability{  // 创建一个名为expected的v4.Vulnerability类型的切片
		{  // 创建一个v4.Vulnerability类型的结构体
			ID:                "my-cve",  // 设置ID字段的值为"my-cve"
			PackageName:       "package-name",  // 设置PackageName字段的值为"package-name"
			Namespace:         "my-namespace",  // 设置Namespace字段的值为"my-namespace"
			VersionConstraint: "< 1.0",  // 设置VersionConstraint字段的值为"< 1.0"
			VersionFormat:     "semver",  // 设置VersionFormat字段的值为"semver"
			CPEs:              []string{"a-cool-cpe"},  // 设置CPEs字段的值为包含"a-cool-cpe"的字符串切片
			RelatedVulnerabilities: []v4.VulnerabilityReference{  // 创建一个名为RelatedVulnerabilities的v4.VulnerabilityReference类型的切片
				{  // 创建一个v4.VulnerabilityReference类型的结构体
					ID:        "another-cve",  // 设置ID字段的值为"another-cve"
					Namespace: "nvd",  // 设置Namespace字段的值为"nvd"
				},
				{  // 创建一个v4.VulnerabilityReference类型的结构体
					ID:        "an-other-cve",  // 设置ID字段的值为"an-other-cve"
					Namespace: "nvd",  // 设置Namespace字段的值为"nvd"
				},
			},
		// 修复信息，包括修复的版本和状态
		Fix: v4.Fix{
			Versions: []string{"1.0.1"},  // 修复的版本号
			State:    v4.FixedState,      // 修复状态
		},
		// 漏洞的唯一标识符
		ID:                "my-other-cve",
		// 漏洞所属的包名
		PackageName:       "package-name",
		// 漏洞所属的命名空间
		Namespace:         "my-namespace",
		// 版本约束条件
		VersionConstraint: "< 509.2.2",
		// 版本格式
		VersionFormat:     "semver",
		// 与漏洞相关的 CPE（通用漏洞和暴露）条目
		CPEs:              nil,
		// 与漏洞相关的其他漏洞的引用
		RelatedVulnerabilities: []v4.VulnerabilityReference{
			{
				ID:        "another-cve",  // 其他漏洞的唯一标识符
				Namespace: "nvd",           // 其他漏洞所属的命名空间
			},
			{
				ID:        "an-other-cve",  // 另一个漏洞的唯一标识符
				Namespace: "nvd",           // 另一个漏洞所属的命名空间
			},
		},
	},
	Fix: v4.Fix{
		Versions: []string{"4.0.5"},  # 修复的版本号
		State:    v4.FixedState,      # 修复状态
	},
},
{  # 新的漏洞信息
	ID:                     "yet-another-cve",  # 漏洞ID
	PackageName:            "package-name",     # 包名
	Namespace:              "my-namespace",     # 命名空间
	VersionConstraint:      "< 1000.0.0",       # 版本约束
	VersionFormat:          "semver",           # 版本格式
	CPEs:                   nil,                # CPEs
	RelatedVulnerabilities: nil,                # 相关漏洞
	Fix: v4.Fix{                                # 修复信息
		Versions: []string{"1000.0.1"},          # 修复的版本号
		State:    v4.FixedState,                 # 修复状态
	},
},
# 定义一个结构体，包含了漏洞的相关信息，如ID、包名、命名空间、版本约束、版本格式、CPEs、相关漏洞、修复信息和建议
{
    ID:                     "yet-another-cve-with-advisories",  # 漏洞ID
    PackageName:            "package-name",  # 包名
    Namespace:              "my-namespace",  # 命名空间
    VersionConstraint:      "< 1000.0.0",  # 版本约束
    VersionFormat:          "semver",  # 版本格式
    CPEs:                   nil,  # CPEs
    RelatedVulnerabilities: nil,  # 相关漏洞
    Fix: v4.Fix{  # 修复信息
        Versions: []string{"1000.0.1"},  # 修复的版本
        State:    v4.FixedState,  # 修复状态
    },
    Advisories: []v4.Advisory{{ID: "ABC-12345", Link: "https://abc.xyz"}},  # 建议
}

# 将extra追加到expected中
total := append(expected, extra...)

# 将total作为参数传递给AddVulnerability方法，如果出现错误则打印错误信息
if err = s.AddVulnerability(total...); err != nil {
    t.Fatalf("failed to set Vulnerability: %+v", err)
}
	}

	// 声明一个空的VulnerabilityModel切片
	var allEntries []model.VulnerabilityModel
	// 从数据库中查找所有的VulnerabilityModel，并存储到allEntries切片中
	s.(*store).db.Find(&allEntries)
	// 检查从数据库中获取的VulnerabilityModel数量是否与期望的数量相同，如果不同则输出错误信息
	if len(allEntries) != len(total) {
		t.Fatalf("unexpected number of entries: %d", len(allEntries))
	}

	// 调用assertVulnerabilityReader函数，对VulnerabilityModel进行断言
	assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)

}

// 定义一个函数，用于断言VulnerabilityMetadataStoreReader接口的行为
func assertVulnerabilityMetadataReader(t *testing.T, reader v4.VulnerabilityMetadataStoreReader, id, namespace string, expected v4.VulnerabilityMetadata) {
	// 通过id和namespace从reader中获取VulnerabilityMetadata，并存储到actual中
	if actual, err := reader.GetVulnerabilityMetadata(id, namespace); err != nil {
		// 如果获取失败，则输出错误信息
		t.Fatalf("failed to get metadata: %+v", err)
	} else if actual == nil {
		// 如果获取的VulnerabilityMetadata为空，则输出错误信息
		t.Fatalf("no metadata returned for id=%q namespace=%q", id, namespace)
	} else {
		// 对获取的VulnerabilityMetadata和期望的VulnerabilityMetadata进行排序
		sortMetadataCvss(actual.Cvss)
		sortMetadataCvss(expected.Cvss)
// 确保预期和实际的 CVSS 条目数量相同，以防止后续断言时出现 panic
assert.Len(t, expected.Cvss, len(actual.Cvss))
// 遍历实际的 CVSS 条目，比较向量、版本和度量值是否与预期相同
for idx, actualCvss := range actual.Cvss {
    assert.Equal(t, actualCvss.Vector, expected.Cvss[idx].Vector)
    assert.Equal(t, actualCvss.Version, expected.Cvss[idx].Version)
    assert.Equal(t, actualCvss.Metrics, expected.Cvss[idx].Metrics)

    // 将实际和预期的供应商元数据转换为 JSON 字符串，并比较它们是否相同
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
		// 将Cvss字段置为nil，因为它是一个接口 - 在这一点上已经进行了Cvss的验证
		expected.Cvss = nil
		actual.Cvss = nil
	// 使用assert.Equal函数比较expected和actual，如果不相等则会引发测试失败
		assert.Equal(t, &expected, actual)
	}

}

// 根据Cvss字段对metadata进行排序
func sortMetadataCvss(cvss []v4.Cvss) {
	// 使用sort.Slice函数对cvss进行排序
	sort.Slice(cvss, func(i, j int) bool {
		// 首先按照Vector字段进行排序
		if cvss[i].Vector > cvss[j].Vector {
			return true
		}
		if cvss[i].Vector < cvss[j].Vector {
			return false
		}
		// 如果Vector相同，则尝试按照BaseScore进行排序
		return cvss[i].Metrics.BaseScore < cvss[j].Metrics.BaseScore
	})
}

// CustomMetadata是一个空操作，它的值并不重要，主要用于确保任何类型都可以被存储，然后在这些测试用例中进行断言，其中使用自定义供应商CVSS分数
type CustomMetadata struct {
	SuperScore string
	Vendor     string
}

func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
	// 创建临时文件夹作为数据库文件
	dbTempFile := t.TempDir()

	// 创建一个新的存储对象
	s, err := New(dbTempFile, true)
	if err != nil {
		// 如果创建失败，输出错误信息
		t.Fatalf("could not create store: %+v", err)
	}

	// 定义一个v4.VulnerabilityMetadata类型的切片
	total := []v4.VulnerabilityMetadata{
# 定义一个包含 CVE 信息的结构体
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
    Cvss: []v4.Cvss{
        {
            # 供应商元数据
            VendorMetadata: CustomMetadata{
                # 供应商
                Vendor:     "redhat",
                # 超级分数
                SuperScore: "1000",
            },
            # 版本
            Version: "2.0",
            # CVSS 指标
            Metrics: v4.NewCvssMetrics(
                1.1,
                2.2,
                3.3,
            ),
            # 向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--NOT",
				},
				{
					// 设置版本号为 "3.0"
					Version: "3.0",
					// 使用 v4.NewCvssMetrics 创建新的 CVSS 指标
					Metrics: v4.NewCvssMetrics(
						1.3, // 设置基本指标
						2.1, // 设置基本指标
						3.2, // 设置基本指标
					),
					// 设置向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--NICE"
					Vector:         "AV:N/AC:L/Au:N/C:P/I:P/A:P--NICE",
					// 设置供应商元数据为 nil
					VendorMetadata: nil,
				},
			},
		},
		{
			// 设置 ID 为 "my-other-cve"
			ID:           "my-other-cve",
			// 设置记录来源为 "record-source"
			RecordSource: "record-source",
			// 设置命名空间为 "namespace"
			Namespace:    "namespace",
			// 设置严重程度为 "pretty bad"
			Severity:     "pretty bad",
			// 设置 URL 列表为 ["https://ancho.re"]
			URLs:         []string{"https://ancho.re"},
			// 设置描述为 "worst description ever"
			Description:  "worst description ever",
# 创建一个包含多个Cvss对象的列表
Cvss: []v4.Cvss{
    # 创建第一个Cvss对象
    {
        # 设置CVSS版本号
        Version: "2.0",
        # 设置CVSS指标
        Metrics: v4.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        # 设置CVSS向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 创建第二个Cvss对象
    {
        # 设置CVSS版本号
        Version: "3.0",
        # 设置CVSS指标
        Metrics: v4.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
        # 设置CVSS向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
		},
	}

	// 将漏洞元数据添加到存储中
	if err = s.AddVulnerabilityMetadata(total...); err != nil {
		t.Fatalf("failed to set metadata: %+v", err)
	}

	// 从数据库中获取所有漏洞元数据
	var allEntries []model.VulnerabilityMetadataModel
	s.(*store).db.Find(&allEntries)
	// 检查获取的漏洞元数据数量是否与预期一致
	if len(allEntries) != len(total) {
		t.Fatalf("unexpected number of entries: %d", len(allEntries))
	}

}

// 测试存储中的漏洞元数据合并功能
func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
	tests := []struct {
		name     string
		add      []v4.VulnerabilityMetadata
		expected v4.VulnerabilityMetadata
```

在上面的代码中，注释解释了每个语句的作用，包括将漏洞元数据添加到存储中，从数据库中获取所有漏洞元数据以及测试存储中的漏洞元数据合并功能。
		err      bool  // 定义一个名为err的布尔变量
	}{  // 匿名结构体开始
		{  // 结构体字段开始
			name: "go-case",  // 设置name字段的值为"go-case"
			add: []v4.VulnerabilityMetadata{  // 设置add字段的值为v4.VulnerabilityMetadata类型的切片
				{  // 结构体字段开始
					ID:           "my-cve",  // 设置ID字段的值为"my-cve"
					RecordSource: "record-source",  // 设置RecordSource字段的值为"record-source"
					Namespace:    "namespace",  // 设置Namespace字段的值为"namespace"
					Severity:     "pretty bad",  // 设置Severity字段的值为"pretty bad"
					URLs:         []string{"https://ancho.re"},  // 设置URLs字段的值为包含一个字符串元素"https://ancho.re"的字符串切片
					Description:  "worst description ever",  // 设置Description字段的值为"worst description ever"
					Cvss: []v4.Cvss{  // 设置Cvss字段的值为v4.Cvss类型的切片
						{  // 结构体字段开始
							Version: "2.0",  // 设置Version字段的值为"2.0"
							Metrics: v4.NewCvssMetrics(  // 设置Metrics字段的值为v4.NewCvssMetrics函数的返回值
								4.1,  // 设置v4.NewCvssMetrics函数的第一个参数为4.1
								5.2,  // 设置v4.NewCvssMetrics函数的第二个参数为5.2
								6.3,  // 设置v4.NewCvssMetrics函数的第三个参数为6.3
							),  // 结构体字段结束
# 创建一个包含漏洞元数据的结构体
expected: v4.VulnerabilityMetadata{
    # 漏洞的唯一标识符
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 漏洞的严重程度
    Severity: "pretty bad",
    # 漏洞相关的 URL 列表
    URLs: []string{"https://ancho.re"},
    # 漏洞的 CVSS 评分列表
    Cvss: []v4.Cvss{
        {
            # CVSS 版本号
            Version: "3.0",
            # CVSS 评分指标
            Metrics: v4.NewCvssMetrics(
                # 基础评分
                1.4,
                # 攻击向量
                2.5,
                # 攻击复杂度
                3.6,
            ),
            # 攻击向量字符串
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
        },
        {
            # CVSS 版本号
            Version: "2.0",
            # CVSS 评分指标
            Metrics: v4.NewCvssMetrics(
                # 基础评分
                3.5,
                # 攻击向量
                4.2,
                # 攻击复杂度
                5.1,
            ),
            # 攻击向量字符串
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
    },
},
# 描述漏洞的描述信息
Description:  "worst description ever",
# 漏洞的 CVSS 评分
Cvss: []v4.Cvss{
    # 第一个 CVSS 评分
    {
        # CVSS 版本
        Version: "2.0",
        # CVSS 评分指标
        Metrics: v4.NewCvssMetrics(
            # 基础评分
            4.1,
            # 攻击向量复杂度
            5.2,
            # 攻击向量影响
            6.3,
        ),
        # 攻击向量字符串表示
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 第二个 CVSS 评分
    {
        # CVSS 版本
        Version: "3.0",
        # CVSS 评分指标
        Metrics: v4.NewCvssMetrics(
            # 基础评分
            1.4,
            # 攻击向量复杂度
            2.5,
            # 攻击向量影响
            3.6,
        ),
        # 攻击向量字符串表示
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
# 创建一个名为 "merge-links" 的对象
{
    name: "merge-links",
    # 添加一个名为 "my-cve" 的漏洞元数据到对象中
    add: []v4.VulnerabilityMetadata{
        {
            ID:           "my-cve",
            RecordSource: "record-source",
            Namespace:    "namespace",
            Severity:     "pretty bad",
            URLs:         []string{"https://ancho.re"},
        },
        # 添加另一个名为 "my-cve" 的漏洞元数据到对象中
        {
            ID:           "my-cve",
            RecordSource: "record-source",
            Namespace:    "namespace",
            Severity:     "pretty bad",
            URLs:         []string{"https://google.com"},
        },
# 创建一个包含漏洞元数据的切片
{
    # 设置漏洞ID
    ID:           "my-cve",
    # 设置记录来源
    RecordSource: "record-source",
    # 设置命名空间
    Namespace:    "namespace",
    # 设置严重程度
    Severity:     "pretty bad",
    # 设置URL列表
    URLs:         []string{"https://yahoo.com"},
},
# 期望的漏洞元数据
expected: v4.VulnerabilityMetadata{
    # 设置漏洞ID
    ID:           "my-cve",
    # 设置记录来源
    RecordSource: "record-source",
    # 设置命名空间
    Namespace:    "namespace",
    # 设置严重程度
    Severity:     "pretty bad",
    # 设置URL列表
    URLs:         []string{"https://ancho.re", "https://google.com", "https://yahoo.com"},
    # 设置CVSS列表
    Cvss:         []v4.Cvss{},
},
# 测试用例：严重程度为bad-severity
{
    name: "bad-severity",
    # 添加漏洞元数据到切片
    add: []v4.VulnerabilityMetadata{
# 创建一个包含 CVE（通用漏洞与漏洞）信息的列表
{
    # CVE ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 严重程度
    Severity: "pretty bad",
    # 相关 URL 列表
    URLs: []string{"https://ancho.re"},
},
{
    # CVE ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 严重程度
    Severity: "meh, push that for next tuesday...",
    # 相关 URL 列表
    URLs: []string{"https://redhat.com"},
},
# 错误标志
err: true,
},
{
    # 测试用例名称
    name: "mismatch-description",
    # 错误标志
    err: true,
```
# 添加漏洞元数据到列表中
add: []v4.VulnerabilityMetadata{
    # 漏洞的唯一标识符
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 漏洞严重程度
    Severity: "pretty bad",
    # 相关链接
    URLs: []string{"https://ancho.re"},
    # 漏洞描述
    Description: "best description ever",
    # CVSS 评分
    Cvss: []v4.Cvss{
        {
            # CVSS 版本
            Version: "2.0",
            # CVSS 指标
            Metrics: v4.NewCvssMetrics(
                4.1,  # 基础评分
                5.2,  # 环境评分
                6.3,  # 基础向量
            ),
            # CVSS 向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
        {
            # 其他 CVSS 数据
# 设置版本号为3.0
Version: "3.0",
# 创建一个新的CVSS指标对象，设置相应的基本指标值
Metrics: v4.NewCvssMetrics(
    1.4,
    2.5,
    3.6,
),
# 设置CVSS向量
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
# 设置CVE记录的其他属性
ID: "my-cve",
RecordSource: "record-source",
Namespace: "namespace",
Severity: "pretty bad",
URLs: []string{"https://ancho.re"},
Description: "worst description ever",
# 设置CVE记录的CVSS指标
Cvss: []v4.Cvss{
    {
        # 设置版本号为2.0
        Version: "2.0",
# 创建一个新的 CVSS 指标对象，指定相应的分数
Metrics: v4.NewCvssMetrics(
    4.1,  # 基础分数
    5.2,  # 基础分数
    6.3,  # 基础分数
),
# 指定向量字符串
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
# 创建另一个新的 CVSS 指标对象，指定相应的分数
Metrics: v4.NewCvssMetrics(
    1.4,  # 基础分数
    2.5,  # 基础分数
    3.6,  # 基础分数
),
# 指定向量字符串
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
# 创建一个包含漏洞元数据的对象
{
    # 漏洞名称
    name: "mismatch-cvss2",
    # 错误标志
    err:  false,
    # 添加漏洞元数据
    add: []v4.VulnerabilityMetadata{
        {
            # 漏洞ID
            ID:           "my-cve",
            # 记录来源
            RecordSource: "record-source",
            # 命名空间
            Namespace:    "namespace",
            # 严重程度
            Severity:     "pretty bad",
            # URL列表
            URLs:         []string{"https://ancho.re"},
            # 描述
            Description:  "best description ever",
            # CVSS评分
            Cvss: []v4.Cvss{
                {
                    # CVSS版本
                    Version: "2.0",
                    # CVSS指标
                    Metrics: v4.NewCvssMetrics(
                        4.1,
                        5.2,
                        6.3,
                    ),
                    # CVSS向量
                    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
						},
						{
							// 版本号为3.0
							Version: "3.0",
							// 使用v4.NewCvssMetrics创建CVSS指标
							Metrics: v4.NewCvssMetrics(
								1.4, // 基础指标
								2.5, // 基础指标
								3.6, // 基础指标
							),
							// 安全向量
							Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
						},
					},
				},
				{
					// CVE ID
					ID:           "my-cve",
					// 记录来源
					RecordSource: "record-source",
					// 命名空间
					Namespace:    "namespace",
					// 严重程度
					Severity:     "pretty bad",
					// 相关链接
					URLs:         []string{"https://ancho.re"},
					// 描述
					Description:  "best description ever",
					// CVE的CVSS指标
					Cvss: []v4.Cvss{
# 创建一个包含不同版本、指标和向量的 CVSS 数据列表
{
    # 版本号为 "2.0"
    Version: "2.0",
    # 创建一个包含指标的 v4 版本的 CVSS 数据
    Metrics: v4.NewCvssMetrics(
        # 基础指标为 4.1
        4.1,
        # 攻击向量指标为 5.2
        5.2,
        # 攻击复杂度指标为 6.3
        6.3,
    ),
    # 攻击向量为 "AV:P--VERY"
    Vector: "AV:P--VERY",
},
{
    # 版本号为 "3.0"
    Version: "3.0",
    # 创建一个包含指标的 v4 版本的 CVSS 数据
    Metrics: v4.NewCvssMetrics(
        # 基础指标为 1.4
        1.4,
        # 攻击向量指标为 2.5
        2.5,
        # 攻击复杂度指标为 3.6
        3.6,
    ),
    # 攻击向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
			},
			expected: v4.VulnerabilityMetadata{
				ID:           "my-cve",  # 漏洞的唯一标识符
				RecordSource: "record-source",  # 记录来源
				Namespace:    "namespace",  # 命名空间
				Severity:     "pretty bad",  # 漏洞严重程度
				URLs:         []string{"https://ancho.re"},  # 相关链接
				Description:  "best description ever",  # 漏洞描述
				Cvss: []v4.Cvss{  # CVSS评分
					{
						Version: "2.0",  # CVSS版本
						Metrics: v4.NewCvssMetrics(  # CVSS指标
							4.1,  # 基本评分
							5.2,  # 环境评分
							6.3,  # 基本评分
						),
						Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量描述
					},
					{
						Version: "3.0",  # CVSS版本
# 创建一个新的 CVSS 指标对象，指定相应的分数
Metrics: v4.NewCvssMetrics(
    1.4,    # 基础分数
    2.5,    # 基础向量
    3.6,    # 基础影响
),
# 指定向量字符串
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",

# 创建另一个新的 CVSS 指标对象，指定相应的分数
Metrics: v4.NewCvssMetrics(
    4.1,    # 基础分数
    5.2,    # 基础向量
    6.3,    # 基础影响
),
# 指定向量字符串
Vector: "AV:P--VERY",
# 定义一个名为 "mismatch-cvss3" 的变量，用于存储漏洞信息
name: "mismatch-cvss3",
# 定义一个名为 "err" 的变量，表示是否有错误
err:  false,
# 定义一个名为 "add" 的变量，用于存储漏洞元数据
add: []v4.VulnerabilityMetadata{
    # 定义一个名为 "ID" 的变量，表示漏洞的唯一标识
    ID:           "my-cve",
    # 定义一个名为 "RecordSource" 的变量，表示记录来源
    RecordSource: "record-source",
    # 定义一个名为 "Namespace" 的变量，表示命名空间
    Namespace:    "namespace",
    # 定义一个名为 "Severity" 的变量，表示漏洞严重程度
    Severity:     "pretty bad",
    # 定义一个名为 "URLs" 的变量，表示漏洞相关的网址
    URLs:         []string{"https://ancho.re"},
    # 定义一个名为 "Description" 的变量，表示漏洞描述
    Description:  "best description ever",
    # 定义一个名为 "Cvss" 的变量，表示漏洞的 CVSS 信息
    Cvss: []v4.Cvss{
        {
            # 定义一个名为 "Version" 的变量，表示 CVSS 版本
            Version: "2.0",
            # 定义一个名为 "Metrics" 的变量，表示 CVSS 的度量
            Metrics: v4.NewCvssMetrics(
                4.1,    # 基本度量
                5.2,    # 基本度量
                6.3,    # 基本度量
            ),
            # 定义一个名为 "Vector" 的变量，表示 CVSS 向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
# 创建一个包含漏洞信息的数据结构
{
    # 漏洞版本号
    Version: "3.0",
    # 使用 v4.NewCvssMetrics 创建漏洞的 CVSS 指标
    Metrics: v4.NewCvssMetrics(
        # 基础指标
        1.4,
        # 攻击向量指标
        2.5,
        # 攻击复杂度指标
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
    Cvss: []v4.Cvss{
        {
# 设置版本号为 "2.0"
Version: "2.0",
# 创建一个新的 CVSS 指标对象，设置基本指标的值
Metrics: v4.NewCvssMetrics(
    4.1,  # 设置基本指标的值：攻击向量
    5.2,  # 设置基本指标的值：攻击复杂度
    6.3,  # 设置基本指标的值：认证要求
),
# 设置向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
# 创建另一个 CVSS 指标对象
{
    Version: "3.0",  # 设置版本号为 "3.0"
    Metrics: v4.NewCvssMetrics(
        1.4,  # 设置基本指标的值：攻击向量
        0,    # 设置基本指标的值：攻击复杂度
        3.6,  # 设置基本指标的值：认证要求
    ),
    # 设置向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 创建一个 v4.VulnerabilityMetadata 结构体实例，包含漏洞的元数据信息
expected: v4.VulnerabilityMetadata{
    # 设置漏洞ID
    ID:           "my-cve",
    # 设置记录来源
    RecordSource: "record-source",
    # 设置命名空间
    Namespace:    "namespace",
    # 设置漏洞严重程度
    Severity:     "pretty bad",
    # 设置漏洞相关URL
    URLs:         []string{"https://ancho.re"},
    # 设置漏洞描述
    Description:  "best description ever",
    # 设置漏洞的CVSS评分信息
    Cvss: []v4.Cvss{
        # 创建一个 v4.Cvss 结构体实例，包含CVSS 2.0版本的评分信息
        {
            # 设置CVSS版本
            Version: "2.0",
            # 设置CVSS评分指标
            Metrics: v4.NewCvssMetrics(
                4.1,
                5.2,
                6.3,
            ),
            # 设置CVSS向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
        # 创建一个 v4.Cvss 结构体实例，包含CVSS 3.0版本的评分信息
        {
            # 设置CVSS版本
            Version: "3.0",
            # 设置CVSS评分指标
            Metrics: v4.NewCvssMetrics(
# 创建一个包含漏洞信息的数据结构
{
    # 漏洞名称
    Name: "CVE-2021-1234",
    # 漏洞描述
    Description: "This is a sample vulnerability",
    # 漏洞的影响版本
    Versions: {
        # 版本号
        "1.0": {
            # 使用 v4 包中的 NewCvssMetrics 方法创建漏洞的 CVSS 指标
            Metrics: v4.NewCvssMetrics(
                # 基础指标
                1.4,
                # 基础指标
                2.5,
                # 基础指标
                3.6,
            ),
            # 漏洞的向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
        },
        # 版本号
        "3.0": {
            # 使用 v4 包中的 NewCvssMetrics 方法创建漏洞的 CVSS 指标
            Metrics: v4.NewCvssMetrics(
                # 基础指标
                1.4,
                # 基础指标
                0,
                # 基础指标
                3.6,
            ),
            # 漏洞的向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
        },
    },
}
# 遍历测试用例列表
for _, test := range tests {
    # 使用测试名称创建子测试
    t.Run(test.name, func(t *testing.T) {
        # 在临时目录创建数据库临时目录
        dbTempDir := t.TempDir()
        # 创建新的存储对象
        s, err := New(dbTempDir, true)
        # 如果创建存储对象出错，输出错误信息
        if err != nil {
            t.Fatalf("could not create store: %+v", err)
        }

        # 依次添加每个元数据
        var theErr error
        for _, metadata := range test.add {
            err = s.AddVulnerabilityMetadata(metadata)
            # 如果添加元数据出错，记录错误并跳出循环
            if err != nil {
                theErr = err
                break
            }
        }

        # 如果测试用例期望出错但未出错，输出错误信息
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
    # 定义测试用例
    tests := []struct {
        name     string
        add      []v4.VulnerabilityMetadata
        expected v4.VulnerabilityMetadata
    }{
        {
            name: "append-cvss",
            add: []v4.VulnerabilityMetadata{
                {
                    ID:           "my-cve",
```

					RecordSource: "record-source",  // 设置记录来源为"record-source"
					Namespace:    "namespace",     // 设置命名空间为"namespace"
					Severity:     "pretty bad",    // 设置严重程度为"pretty bad"
					URLs:         []string{"https://ancho.re"},  // 设置URL数组为["https://ancho.re"]
					Description:  "worst description ever",  // 设置描述为"worst description ever"
					Cvss: []v4.Cvss{  // 设置Cvss数组
						{
							Version: "2.0",  // 设置版本为"2.0"
							Metrics: v4.NewCvssMetrics(  // 设置Cvss指标
								4.1,  // 设置指标值
								5.2,  // 设置指标值
								6.3,  // 设置指标值
							),
							Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  // 设置向量为"AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
						},
					},
				},
				{
					ID:           "my-cve",  // 设置ID为"my-cve"
					RecordSource: "record-source",  // 设置记录来源为"record-source"
# 设置命名空间为 "namespace"
Namespace:    "namespace",
# 设置严重程度为 "pretty bad"
Severity:     "pretty bad",
# 设置 URL 列表为空
URLs:         []string{"https://ancho.re"},
# 设置描述为 "worst description ever"
Description:  "worst description ever",
# 设置 CVSS 列表
Cvss: []v4.Cvss{
    # 设置 CVSS 版本为 "3.0"
    {
        Version: "3.0",
        # 设置 CVSS 指标
        Metrics: v4.NewCvssMetrics(
            # 设置基础分数为 1.4
            1.4,
            # 设置攻击向量为 2.5
            2.5,
            # 设置攻击复杂度为 3.6
            3.6,
        ),
        # 设置 CVSS 向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
# 设置预期的漏洞元数据
expected: v4.VulnerabilityMetadata{
    # 设置漏洞 ID 为 "my-cve"
    ID:           "my-cve",
    # 设置记录来源为 "record-source"
    RecordSource: "record-source",
# 设置命名空间为"namespace"
Namespace:    "namespace",
# 设置严重程度为"pretty bad"
Severity:     "pretty bad",
# 设置URL列表为空
URLs:         []string{"https://ancho.re"},
# 设置描述为"worst description ever"
Description:  "worst description ever",
# 设置CVSS评分，包括版本、指标和向量
Cvss: []v4.Cvss{
    {
        Version: "2.0",
        Metrics: v4.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    {
        Version: "3.0",
        Metrics: v4.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
# ...
# 定义一个名为 "append-vendor-cvss" 的漏洞元数据
{
    name: "append-vendor-cvss",
    # 添加一个名为 "my-cve" 的漏洞元数据
    add: []v4.VulnerabilityMetadata{
        {
            ID: "my-cve",
            RecordSource: "record-source",
            Namespace: "namespace",
            Severity: "pretty bad",
            URLs: []string{"https://ancho.re"},
            Description: "worst description ever",
            # 添加一个名为 "2.0" 的 CVSS 指标
            Cvss: []v4.Cvss{
                {
                    Version: "2.0",
                    # 创建一个新的 CVSS 指标
                    Metrics: v4.NewCvssMetrics(
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 漏洞严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 漏洞描述
    Cvss: []v4.Cvss{  # CVSS评分
        {
            Version: "2.0",  # CVSS版本
            Metrics: v4.NewCvssMetrics(  # CVSS指标
                4.1,  # 基本指标
                5.2,  # 基本指标
                6.3,  # 基本指标
            ),
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # CVSS向量
        },
    },
},
# 定义一个包含多个键值对的结构体
# 包含多个字段，如ID、RecordSource、Namespace、Severity、URLs、Description、Cvss等
# 其中Cvss字段是一个包含多个Cvss对象的数组
# 每个Cvss对象包含多个字段，如Score、Vector、VendorMetadata等
# VendorMetadata字段是一个自定义的结构体，包含SuperScore和Vendor字段
# 最后定义了一个expected变量，包含了一个v4.VulnerabilityMetadata对象的实例化
# 创建一个包含漏洞信息的对象
{
    # 指定漏洞信息的版本号
    Version: "2.0",
    # 创建一个包含 CVSS 指标的对象
    Metrics: v4.NewCvssMetrics(
        4.1,    # 基础指标
        5.2,    # 基础指标
        6.3,    # 基础指标
    ),
    # 指定漏洞的向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    # 添加厂商自定义的元数据
    VendorMetadata: CustomMetadata{
        SuperScore: "100",  # 自定义的超级分数
        Vendor: "debian",   # 厂商名称
# 定义一个名为 "avoids-duplicate-cvss" 的测试用例
{
    name: "avoids-duplicate-cvss",
    # 添加一个漏洞元数据对象到测试用例中
    add: []v4.VulnerabilityMetadata{
        {
            ID: "my-cve",
            RecordSource: "record-source",
            Namespace: "namespace",
            Severity: "pretty bad",
            URLs: []string{"https://ancho.re"},
            Description: "worst description ever",
            # 添加一个 CVSS 对象到漏洞元数据中
            Cvss: []v4.Cvss{
                {
                    Version: "3.0",
                    # 创建一个新的 CVSS 指标对象
                    Metrics: v4.NewCvssMetrics(
                        1.4,
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 漏洞严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 漏洞描述
    Cvss: []v4.Cvss{  # CVSS评分
        {
            Version: "3.0",  # CVSS版本
            Metrics: v4.NewCvssMetrics(  # CVSS指标
                1.4,  # 基础分数
                2.5,  # 攻击向量
                3.6,  # 攻击复杂性
                # 其他CVSS指标
            ),
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # CVSS向量
        },
    },
},
# 创建一个预期的漏洞元数据对象
expected: v4.VulnerabilityMetadata{
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
    # 漏洞描述
    Description: "worst description ever",
    # CVSS评分
    Cvss: []v4.Cvss{
        {
            # CVSS版本
            Version: "3.0",
            # CVSS指标
            Metrics: v4.NewCvssMetrics(
                # 基础分数
                1.4,
                # 攻击向量
                2.5,
                # ...
# 定义一个包含测试用例的切片
tests := []struct {
	// 定义测试用例的名称
	name string
	// 定义测试用例的输入数据
	add []VulnerabilityMetadata
	// 定义测试用例的期望输出
	expected map[string]map[string]map[string]map[string]float64
}{
	// 定义测试用例1
	{
		name: "Test1",
		add: []VulnerabilityMetadata{
			// 定义测试用例1的输入数据
			{
				// 定义测试用例1的输入数据中的字段
				CVSS: map[string]map[string]map[string]float64{
					"3.0": {
						"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD": {
							// 定义测试用例1的输入数据中的字段
							BaseScore: 3.6,
						},
					},
				},
			},
		},
		// 定义测试用例1的期望输出
		expected: map[string]map[string]map[string]map[string]float64{
			"3.0": {
				"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD": {
					// 定义测试用例1的期望输出中的字段
					BaseScore: 3.6,
				},
			},
		},
	},
}

# 遍历测试用例切片，对每个测试用例执行测试
for _, test := range tests {
	# 使用测试框架创建一个临时目录
	dbTempDir := t.TempDir()

	# 创建一个新的对象s，并检查是否有错误发生
	s, err := New(dbTempDir, true)
	if err != nil {
		t.Fatalf("could not create s: %+v", err)
	}

	# 对每个测试用例中的metadata执行添加操作
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

```
// 断言漏洞匹配排除读取器的功能
func assertVulnerabilityMatchExclusionReader(t *testing.T, reader v4.VulnerabilityMatchExclusionStoreReader, id string, expected []v4.VulnerabilityMatchExclusion) {
    // 如果获取漏洞匹配排除失败，输出错误信息并终止测试
    if actual, err := reader.GetVulnerabilityMatchExclusion(id); err != nil {
        t.Fatalf("failed to get Vulnerability Match Exclusion: %+v", err)
    }
}
	} else {
		// 如果条件不成立，记录 actual 的值
		t.Logf("%+v", actual)
		// 如果 actual 的长度不等于 expected 的长度，输出错误信息
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
	// 创建临时文件夹
	dbTempFile := t.TempDir()
	// 创建一个新的数据库实例
	s, err := New(dbTempFile, true)
	// 如果发生错误，输出错误信息并终止测试
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}

	// 创建一个包含漏洞匹配排除项的数组
	extra := []v4.VulnerabilityMatchExclusion{
		// 创建一个漏洞匹配排除项对象
		{
			// 设置漏洞ID
			ID: "CVE-1234-14567",
			// 设置漏洞匹配排除项的约束条件数组
			Constraints: []v4.VulnerabilityMatchExclusionConstraint{
				// 创建一个漏洞匹配排除项约束条件对象
				{
					// 设置漏洞匹配排除项的漏洞约束条件
					Vulnerability: v4.VulnerabilityExclusionConstraint{
						// 设置漏洞的命名空间
						Namespace: "extra-namespace:cpe",
					},
					// 设置漏洞匹配排除项的包约束条件
					Package: v4.PackageExclusionConstraint{
						// 设置包的名称
						Name:     "abc",
						// 设置包的语言
						Language: "ruby",
						// 设置包的版本
						Version:  "1.2.3",
					},
				},
				{
					// 创建另一个漏洞匹配排除项约束条件对象
					Vulnerability: v4.VulnerabilityExclusionConstraint{
# 创建一个包含漏洞和包排除约束的列表
{
    # 漏洞排除约束的命名空间
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "extra-namespace:cpe",
    },
    # 包排除约束的具体信息
    Package: v4.PackageExclusionConstraint{
        # 包的名称
        Name:     "abc",
        # 包的语言
        Language: "ruby",
        # 包的版本
        Version:  "4.5.6",
    },
},
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "extra-namespace:cpe",
    },
    Package: v4.PackageExclusionConstraint{
        Name:     "time-1",
        Language: "ruby",
    },
},
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "extra-namespace:cpe",
    },
}
		},
		Package: v4.PackageExclusionConstraint{
			Name: "abc.xyz:nothing-of-interest",
			Type: "java-archive",
		},
	},
},
Justification: "Because I said so.",
},
{
ID:            "CVE-1234-10",
Constraints:   nil,
Justification: "Because I said so.",
},
}

expected := []v4.VulnerabilityMatchExclusion{
{
ID: "CVE-1234-9999999",
Constraints: []v4.VulnerabilityMatchExclusionConstraint{
```

注释：
- 定义了一个名为expected的变量，类型为v4.VulnerabilityMatchExclusion的切片
- 切片中包含了一个v4.VulnerabilityMatchExclusion类型的结构体，其中包含了ID和Constraints字段
- ID字段为"CVE-1234-9999999"
- Constraints字段为一个v4.VulnerabilityMatchExclusionConstraint类型的切片，其中包含了Package字段和Type字段
- Package字段为v4.PackageExclusionConstraint类型的结构体，包含了Name和Type字段
- Name字段为"abc.xyz:nothing-of-interest"
- Type字段为"java-archive"
- Justification字段为"Because I said so."，表示排除此漏洞的理由
- 接下来还有一个ID为"CVE-1234-10"的漏洞匹配排除，Constraints为nil，Justification为"Because I said so."
# 创建一个漏洞排除约束对象，指定漏洞所属的命名空间为"old-namespace:cpe"
# 指定要排除的包的语言为Python，名称为abc，版本为1.2.3
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v4.PackageExclusionConstraint{
        Language: "python",
        Name: "abc",
        Version: "1.2.3",
    },
},
# 创建另一个漏洞排除约束对象，指定漏洞所属的命名空间为"old-namespace:cpe"
# 指定要排除的包的语言为Python，名称为abc，版本为4.5.6
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v4.PackageExclusionConstraint{
        Language: "python",
        Name: "abc",
        Version: "4.5.6",
    },
},
# 创建一个漏洞排除约束对象，指定漏洞的命名空间为"old-namespace:cpe"
# 创建一个包排除约束对象，指定语言为Python，包名为"time-245"
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v4.PackageExclusionConstraint{
        Language: "python",
        Name: "time-245",
    },
},
# 创建另一个漏洞排除约束对象，指定漏洞的命名空间为"old-namespace:cpe"
# 创建另一个包排除约束对象，指定类型为npm，包名为"everything"
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        Namespace: "old-namespace:cpe",
    },
    Package: v4.PackageExclusionConstraint{
        Type: "npm",
        Name: "everything",
    },
},
# 设置排除约束的理由为"This is a false positive"
Justification: "This is a false positive",
		},
		{
			# 定义漏洞ID
			ID: "CVE-1234-9999999",
			# 定义漏洞匹配排除约束条件列表
			Constraints: []v4.VulnerabilityMatchExclusionConstraint{
				# 定义漏洞匹配排除约束条件
				{
					# 定义漏洞排除约束条件的命名空间
					Vulnerability: v4.VulnerabilityExclusionConstraint{
						Namespace: "old-namespace:cpe",
					},
					# 定义包排除约束条件
					Package: v4.PackageExclusionConstraint{
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
					Vulnerability: v4.VulnerabilityExclusionConstraint{
						Namespace: "some-other-namespace:cpe",
					},
					# 定义包排除约束条件
					Package: v4.PackageExclusionConstraint{
						# 定义包的语言
						Language: "go",
# 定义一个类型为 "go-module" 的结构体，并设置 Name 字段为 "abc"
{
    Type:     "go-module",
    Name:     "abc",
},

# 定义一个漏洞排除约束的结构体，并设置 FixState 字段为 "wont-fix"
{
    Vulnerability: v4.VulnerabilityExclusionConstraint{
        FixState: "wont-fix",
    },
},

# 定义一个漏洞排除约束的结构体，并设置 Justification 字段为 "This is also a false positive"
{
    Justification: "This is also a false positive",
},

# 定义一个漏洞排除约束的结构体，并设置 ID 字段为 "CVE-1234-9999999"，并设置 Justification 字段为 "global exclude"
{
    ID:            "CVE-1234-9999999",
    Justification: "global exclude",
},

# 将 extra 切片追加到 expected 切片中，并将结果赋值给 total
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
    // 创建临时数据库文件
    dbTempFile := t.TempDir()

    // 创建新的存储对象
    s1, err := New(dbTempFile, true)
    // 如果出现错误，报告无法创建存储对象
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }
	// 创建临时目录用于存储数据库文件
	dbTempFile := t.TempDir()

	// 创建一个新的存储实例，并指定为临时数据库文件
	s2, err := New(dbTempFile, true)
	if err != nil {
		// 如果创建存储实例出错，则输出错误信息并终止测试
		t.Fatalf("could not create store: %+v", err)
	}

	// 创建基础漏洞数据
	baseVulns := []v4.Vulnerability{
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
```
```go
			// 其他漏洞数据...
		},
		// 其他漏洞数据...
	}
```
```go
	// 其他操作...
```
```go
```

注释：
- 创建临时目录用于存储数据库文件
- 创建一个新的存储实例，并指定为临时数据库文件
- 如果创建存储实例出错，则输出错误信息并终止测试
- 创建基础漏洞数据
- 其他漏洞数据...
- 其他漏洞数据...
- 其他操作...
# 定义一个包含漏洞信息的结构体，包括漏洞的命名空间、ID、包名称、版本约束、CPEs和修复状态
{
    Namespace:         "pypi",  # 漏洞的命名空间为pypi
    ID:                "CVE-2021-3456",  # 漏洞的ID
    PackageName:       "pypi:requests",  # 漏洞所在的包名称
    VersionConstraint: "*",  # 版本约束
    CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},  # 漏洞相关的CPEs
},
{
    Namespace:         "npm",  # 漏洞的命名空间为npm
    ID:                "CVE-123-7654",  # 漏洞的ID
    PackageName:       "npm:axios",  # 漏洞所在的包名称
    VersionConstraint: "< 3.0 >= 2.17",  # 版本约束
    CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},  # 漏洞相关的CPEs
    Fix: v4.Fix{  # 漏洞的修复状态
        State: v4.UnknownFixState,  # 修复状态为未知
    },
},
{
    Namespace:         "nuget",  # 漏洞的命名空间为nuget
    ID:                "GHSA-****-******",  # 漏洞的ID
    PackageName:       "nuget:net",  # 漏洞所在的包名称
    VersionConstraint: "< 3.0 >= 2.17",  # 版本约束
    CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},  # 漏洞相关的CPEs
    Fix: v4.Fix{  # 漏洞的修复状态
        State: v4.UnknownFixState,  # 修复状态为未知
    },
}
		},
		},
		{
			Namespace:         "hex",  # 漏洞所属的命名空间
			ID:                "GHSA-^^^^-^^^^^^",  # 漏洞的唯一标识符
			PackageName:       "hex:esbuild",  # 漏洞所属的包名
			VersionConstraint: "< 3.0 >= 2.17",  # 受影响的版本范围
			CPEs:              []string{"cpe:2.3:hex:esbuild:*:*:*:*:*:*"},  # 受影响的 CPE（通用漏洞披露标识符）
		},
	}
	baseMetadata := []v4.VulnerabilityMetadata{  # 基础元数据列表
		{
			Namespace:  "nuget",  # 漏洞所属的命名空间
			ID:         "GHSA-****-******",  # 漏洞的唯一标识符
			DataSource: "nvd",  # 数据源
		},
	}
	targetVulns := []v4.Vulnerability{  # 目标漏洞列表
		{
			Namespace:         "github:language:python",  # 漏洞所属的命名空间
```

# 定义漏洞ID为 "CVE-123-4567" 的漏洞信息
{
    # 漏洞所属包的名称
    PackageName: "pypi:requests",
    # 版本约束，表示版本小于 2.0 且大于等于 1.29
    VersionConstraint: "< 2.0 >= 1.29",
    # 漏洞相关的 CPE（通用漏洞和暴露）条目列表
    CPEs: []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
},
# 定义漏洞ID为 "GHSA-....-...." 的漏洞信息
{
    # 漏洞所属包的语言和名称
    Namespace: "github:language:go",
    # 漏洞ID
    ID: "GHSA-....-....",
    # 漏洞所属包的名称
    PackageName: "hashicorp:nomad",
    # 版本约束，表示版本小于 3.0 且大于等于 2.17
    VersionConstraint: "< 3.0 >= 2.17",
    # 漏洞相关的 CPE（通用漏洞和暴露）条目列表
    CPEs: []string{"cpe:2.3:golang:hashicorp:nomad:*:*:*:*:*"},
},
# 定义漏洞ID为 "GHSA-....-...." 的漏洞信息
{
    # 漏洞所属包的语言和名称
    Namespace: "github:language:go",
    # 漏洞ID
    ID: "GHSA-....-....",
    # 漏洞所属包的名称
    PackageName: "hashicorp:n",
    # 版本约束，表示版本小于 2.0 且大于等于 1.17
    VersionConstraint: "< 2.0 >= 1.17",
    # 漏洞相关的 CPE（通用漏洞和暴露）条目列表
    CPEs: []string{"cpe:2.3:golang:hashicorp:n:*:*:*:*:*"},
},
# 设置命名空间为 "npm"，漏洞ID为 "CVE-123-7654"，包名为 "npm:axios"，版本约束为 "< 3.0 >= 2.17"，CPEs为空数组，修复状态为不修复
{
    Namespace:         "npm",
    ID:                "CVE-123-7654",
    PackageName:       "npm:axios",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
    Fix: v4.Fix{
        State: v4.WontFixState,
    },
},
# 设置命名空间为 "nuget"，漏洞ID为 "GHSA-****-******"，包名为 "nuget:net"，版本约束为 "< 3.0 >= 2.17"，CPEs为 "cpe:2.3:nuget:net:*:*:*:*:*:*"，修复状态为未知
{
    Namespace:         "nuget",
    ID:                "GHSA-****-******",
    PackageName:       "nuget:net",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
    Fix: v4.Fix{
        State: v4.UnknownFixState,
    },
},
# 定义一个包含v4.Diff类型的切片，用于存储预期的差异
expectedDiffs := []v4.Diff{
    # 定义第一个差异，表示发生了变化的漏洞
    {
        Reason:    v4.DiffChanged,  # 差异的原因为发生了变化
        ID:        "CVE-123-4567",  # 漏洞的ID
        Namespace: "github:language:python",  # 漏洞所属的命名空间
        Packages:  []string{"pypi:requests"},  # 受影响的包
    },
    # 定义第二个差异，表示发生了变化的漏洞
    {
        Reason:    v4.DiffChanged,  # 差异的原因为发生了变化
        ID:        "CVE-123-7654",  # 漏洞的ID
        Namespace: "npm",  # 漏洞所属的命名空间
        Packages:  []string{"npm:axios"},  # 受影响的包
    },
    # 定义第三个差异，表示移除的漏洞
    {
        Reason:    v4.DiffRemoved,  # 差异的原因为移除
        ID:        "GHSA-****-******",  # 漏洞的ID
        Namespace: "nuget",  # 漏洞所属的命名空间
        Packages:  []string{"nuget:net"},  # 受影响的包
    },
    # ...
# 添加一个新的漏洞信息到 baseVulns 数组中
{
    Reason:    v4.DiffAdded,
    ID:        "GHSA-....-....",
    Namespace: "github:language:go",
    Packages:  []string{"hashicorp:n", "hashicorp:nomad"},
},
# 添加一个新的漏洞信息到 baseVulns 数组中
{
    Reason:    v4.DiffRemoved,
    ID:        "GHSA-^^^^-^^^^^^",
    Namespace: "hex",
    Packages:  []string{"hex:esbuild"},
},

# 遍历 baseVulns 数组，将每个漏洞信息添加到 s1 中
for _, vuln := range baseVulns {
    s1.AddVulnerability(vuln)
}

# 遍历 targetVulns 数组，将每个漏洞信息添加到 s2 中
for _, vuln := range targetVulns {
    s2.AddVulnerability(vuln)
}

# 遍历 baseMetadata 数组
		// 向 s1 中添加漏洞元数据
		s1.AddVulnerabilityMetadata(meta)
	}

	// 当
	// 调用 s1 的 DiffStore 方法，将 s2 与 s1 进行比较
	result, err := s1.DiffStore(s2)

	// 然后
	// 对比结果按照 ID 进行排序
	sort.SliceStable(*result, func(i, j int) bool {
		return (*result)[i].ID < (*result)[j].ID
	})
	// 对比结果中的包名列表按照字母顺序进行排序
	for i := range *result {
		sort.Strings((*result)[i].Packages)
	}

	// 断言
	// 断言错误对象为空
	assert.NoError(t, err)
	// 断言预期的差异与实际的差异相等
	assert.Equal(t, expectedDiffs, *result)
}
```