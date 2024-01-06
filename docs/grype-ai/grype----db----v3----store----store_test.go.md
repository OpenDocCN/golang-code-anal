# `grype\grype\db\v3\store\store_test.go`

```
package store

import (
	"encoding/json"  // 导入 JSON 编解码包
	"sort"  // 导入排序包
	"testing"  // 导入测试包
	"time"  // 导入时间包

	"github.com/go-test/deep"  // 导入深度比较包
	"github.com/stretchr/testify/assert"  // 导入断言包

	v3 "github.com/anchore/grype/grype/db/v3"  // 导入 v3 版本的数据库包
	"github.com/anchore/grype/grype/db/v3/store/model"  // 导入数据库模型包
)

func assertIDReader(t *testing.T, reader v3.IDReader, expected v3.ID) {
	t.Helper()  // 标记该函数是测试辅助函数
	if actual, err := reader.GetID(); err != nil {  // 获取 ID 信息
		t.Fatalf("failed to get ID: %+v", err)  // 如果获取失败则输出错误信息
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
	expected := v3.ID{
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
func assertVulnerabilityReader(t *testing.T, reader v3.VulnerabilityStoreReader, namespace, name string, expected []v3.Vulnerability) {
	// 获取漏洞信息
	if actual, err := reader.GetVulnerability(namespace, name); err != nil {
		t.Fatalf("failed to get Vulnerability: %+v", err)
	} else {
		// 检查实际漏洞数量是否与预期相同
		if len(actual) != len(expected) {
			t.Fatalf("unexpected number of vulns: %d", len(actual))
		}
		// 遍历比较实际漏洞和预期漏洞
		for idx := range actual {
			diffs := deep.Equal(expected[idx], actual[idx])
			// 如果存在差异，输出错误信息
			if len(diffs) > 0 {
# 遍历 diffs 列表中的元素，将每个元素输出为错误信息
for _, d := range diffs {
    t.Errorf("Diff: %+v", d)
}
# 结束 for 循环
```

```
# 定义测试函数 TestStore_GetVulnerability_SetVulnerability
func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    # 创建临时数据库文件夹
    dbTempFile := t.TempDir()
    # 创建一个新的存储对象，并检查是否有错误发生
    s, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    # 创建一个包含 v3.Vulnerability 类型元素的额外列表
    extra := []v3.Vulnerability{
        {
            ID:                "my-cve-33333",
            PackageName:       "package-name-2",
            Namespace:         "my-namespace",
            # 其他属性...
        },
        # 其他 v3.Vulnerability 元素...
    }
    # 其他代码...
}
			// 设置版本约束为小于 1.0
			VersionConstraint: "< 1.0",
			// 设置版本格式为语义化版本
			VersionFormat:     "semver",
			// 设置 CPEs 为空字符串数组
			CPEs:              []string{"a-cool-cpe"},
			// 设置相关漏洞引用为两个 CVE
			RelatedVulnerabilities: []v3.VulnerabilityReference{
				{
					ID:        "another-cve",
					Namespace: "nvd",
				},
				{
					ID:        "an-other-cve",
					Namespace: "nvd",
				},
			},
			// 设置修复信息，包括修复版本和状态
			Fix: v3.Fix{
				Versions: []string{"2.0.1"},
				State:    v3.FixedState,
			},
		},
		{
			// 设置另一个漏洞的 ID
			ID:                "my-other-cve-33333",
# 设置包的名称为 "package-name-3"
PackageName:       "package-name-3",
# 设置命名空间为 "my-namespace"
Namespace:         "my-namespace",
# 设置版本约束为 "< 509.2.2"
VersionConstraint: "< 509.2.2",
# 设置版本格式为 "semver"
VersionFormat:     "semver",
# 设置CPEs为空字符串数组
CPEs:              []string{"a-cool-cpe"},
# 设置相关漏洞引用为包含两个VulnerabilityReference对象的数组
RelatedVulnerabilities: []v3.VulnerabilityReference{
    # 第一个VulnerabilityReference对象的ID为 "another-cve"，命名空间为 "nvd"
    {
        ID:        "another-cve",
        Namespace: "nvd",
    },
    # 第二个VulnerabilityReference对象的ID为 "an-other-cve"，命名空间为 "nvd"
    {
        ID:        "an-other-cve",
        Namespace: "nvd",
    },
},
# 设置修复状态为未修复状态
Fix: v3.Fix{
    State: v3.NotFixedState,
},
// 创建一个期望的漏洞切片，包含多个漏洞对象
expected := []v3.Vulnerability{
	// 创建一个漏洞对象
	{
		// 设置漏洞ID
		ID: "my-cve",
		// 设置包名
		PackageName: "package-name",
		// 设置命名空间
		Namespace: "my-namespace",
		// 设置版本约束
		VersionConstraint: "< 1.0",
		// 设置版本格式
		VersionFormat: "semver",
		// 设置CPEs
		CPEs: []string{"a-cool-cpe"},
		// 设置相关漏洞引用
		RelatedVulnerabilities: []v3.VulnerabilityReference{
			// 创建一个漏洞引用对象
			{
				// 设置引用的漏洞ID
				ID: "another-cve",
				// 设置引用的漏洞命名空间
				Namespace: "nvd",
			},
			// 创建另一个漏洞引用对象
			{
				// 设置引用的漏洞ID
				ID: "an-other-cve",
				// 设置引用的漏洞命名空间
				Namespace: "nvd",
			},
		},
		// 设置修复信息
		Fix: v3.Fix{
# 创建一个包含版本号的字符串切片
Versions: []string{"1.0.1"},
# 设置状态为已修复
State:    v3.FixedState,
},
},
{
# 设置漏洞的唯一标识符
ID:                "my-other-cve",
# 设置包的名称
PackageName:       "package-name",
# 设置命名空间
Namespace:         "my-namespace",
# 设置版本约束
VersionConstraint: "< 509.2.2",
# 设置版本格式
VersionFormat:     "semver",
# 创建一个包含CPE的字符串切片
CPEs:              []string{"a-cool-cpe"},
# 创建一个包含相关漏洞引用的VulnerabilityReference切片
RelatedVulnerabilities: []v3.VulnerabilityReference{
    {
        # 设置漏洞的唯一标识符
        ID:        "another-cve",
        # 设置命名空间
        Namespace: "nvd",
    },
    {
        # 设置漏洞的唯一标识符
        ID:        "an-other-cve",
        # 设置命名空间
        Namespace: "nvd",
    },
# 创建一个包含漏洞信息的切片
expected := []v3.Vulnerability{
    {
        ID: "CVE-2021-1234",
        Description: "This is a vulnerability description",
        Fix: v3.Fix{
            Versions: []string{"4.0.5"},
            State:    v3.FixedState,
        },
    },
}

# 将额外的漏洞信息添加到切片中
total := append(expected, extra...)

# 将漏洞信息添加到存储中
if err = s.AddVulnerability(total...); err != nil {
    t.Fatalf("failed to set Vulnerability: %+v", err)
}

# 查询所有的漏洞信息
var allEntries []model.VulnerabilityModel
s.(*store).db.Find(&allEntries)

# 检查查询到的漏洞信息数量是否和预期的数量相同
if len(allEntries) != len(total) {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}
// 断言漏洞读取器的功能是否符合预期
func assertVulnerabilityReader(t *testing.T, s v3.VulnerabilityMetadataStoreReader, namespace, packageName string, expected []v3.VulnerabilityMetadata) {
	// 断言获取的漏洞元数据是否符合预期
	assertVulnerabilityMetadataReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)
}

// 断言漏洞元数据读取器的功能是否符合预期
func assertVulnerabilityMetadataReader(t *testing.T, reader v3.VulnerabilityMetadataStoreReader, id, namespace string, expected v3.VulnerabilityMetadata) {
	// 如果获取漏洞元数据时出现错误，输出错误信息
	if actual, err := reader.GetVulnerabilityMetadata(id, namespace); err != nil {
		t.Fatalf("failed to get metadata: %+v", err)
	// 如果获取的漏洞元数据为空，输出错误信息
	} else if actual == nil {
		t.Fatalf("no metadata returned for id=%q namespace=%q", id, namespace)
	} else {
		// 对实际和预期的 CVSS 元数据进行排序
		sortMetadataCvss(actual.Cvss)
		sortMetadataCvss(expected.Cvss)

		// 确保它们都有相同数量的 CVSS 条目，以防止后续断言时出现错误
		assert.Len(t, expected.Cvss, len(actual.Cvss))
		// 遍历 CVSS 条目，断言其向量、版本和度量是否符合预期
		for idx, actualCvss := range actual.Cvss {
			assert.Equal(t, actualCvss.Vector, expected.Cvss[idx].Vector)
			assert.Equal(t, actualCvss.Version, expected.Cvss[idx].Version)
			assert.Equal(t, actualCvss.Metrics, expected.Cvss[idx].Metrics)
		}
	}
}
// 将actualCvss.VendorMetadata转换为JSON格式的字节流
actualVendor, err := json.Marshal(actualCvss.VendorMetadata)
// 如果转换过程中出现错误，输出错误信息
if err != nil {
    t.Errorf("unable to marshal vendor metadata: %q", err)
}
// 将expected.Cvss[idx].VendorMetadata转换为JSON格式的字节流
expectedVendor, err := json.Marshal(expected.Cvss[idx].VendorMetadata)
// 如果转换过程中出现错误，输出错误信息
if err != nil {
    t.Errorf("unable to marshal vendor metadata: %q", err)
}
// 比较两个JSON格式的字节流是否相等
assert.Equal(t, string(actualVendor), string(expectedVendor))

// 将expected.Cvss和actual.Cvss字段置为nil，因为在此时已经验证了Cvss字段
expected.Cvss = nil
actual.Cvss = nil
// 比较expected和actual是否相等
assert.Equal(t, &expected, actual)
// 对元数据的 CVSS 分数进行排序
func sortMetadataCvss(cvss []v3.Cvss) {
	// 使用 sort.Slice 函数对 cvss 切片进行排序
	sort.Slice(cvss, func(i, j int) bool {
		// 首先按照 Vector 进行排序
		if cvss[i].Vector > cvss[j].Vector {
			return true
		}
		if cvss[i].Vector < cvss[j].Vector {
			return false
		}
		// 如果 Vector 相同，则尝试按照 BaseScore 进行排序
		return cvss[i].Metrics.BaseScore < cvss[j].Metrics.BaseScore
	})
}

// CustomMetadata 实际上是一个空操作，它的值并不具有实际意义，主要用于确保任何类型都可以被存储然后在这些测试用例中进行断言，其中使用了自定义供应商的 CVSS 分数
type CustomMetadata struct {
	SuperScore string
}
// 定义一个结构体，包含一个名为Vendor的字符串字段
type Store struct {
	Vendor     string
}

// 测试函数，用于测试GetVulnerabilityMetadata和SetVulnerabilityMetadata方法
func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
	// 创建临时文件夹用于数据库
	dbTempFile := t.TempDir()

	// 创建一个新的存储对象
	s, err := New(dbTempFile, true)
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}

	// 定义一个VulnerabilityMetadata切片
	total := []v3.VulnerabilityMetadata{
		{
			ID:           "my-cve",
			RecordSource: "record-source",
			Namespace:    "namespace",
			Severity:     "pretty bad",
			URLs:         []string{"https://ancho.re"},
			Description:  "best description ever",
			Cvss: []v3.Cvss{
# 创建一个包含厂商元数据、版本、指标和向量的自定义元数据对象
{
    # 厂商元数据
    VendorMetadata: CustomMetadata{
        Vendor:     "redhat",  # 厂商名称
        SuperScore: "1000",    # 超级分数
    },
    # 版本
    Version: "2.0",
    # 指标
    Metrics: v3.NewCvssMetrics(
        1.1,  # 基础分数
        2.2,  # 严重程度
        3.3,  # 攻击向量
    ),
    # 向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--NOT",
},
{
    # 版本
    Version: "3.0",
    # 指标
    Metrics: v3.NewCvssMetrics(
        1.3,  # 基础分数
        2.1,  # 严重程度
        3.2,  # 攻击向量
    ),
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-other-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",      # 命名空间
    Severity:     "pretty bad",      # 严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",      # 漏洞描述
    Cvss: []v3.Cvss{  # CVSS评分
        {
            Version: "2.0",  # CVSS版本
            Metrics: v3.NewCvssMetrics(  # CVSS指标
                4.1,  # 基础分数
                5.2,  # 攻击向量
                6.3,  # 攻击复杂性
            ),
        },
    },
},
# 创建一个包含漏洞元数据的切片
var total = []model.VulnerabilityMetadataModel{
		{
			// 版本号为 2.0，使用 v2 包中的 CvssMetrics 方法创建漏洞评分指标
			Version: "2.0",
			Metrics: v2.CvssMetrics(
				3.5, // 基础评分
				4.2, // 攻击向量
				2.8, // 攻击复杂度
			),
			// 漏洞向量
			Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
		},
		{
			// 版本号为 3.0，使用 v3 包中的 CvssMetrics 方法创建漏洞评分指标
			Version: "3.0",
			Metrics: v3.NewCvssMetrics(
				1.4, // 基础评分
				2.5, // 攻击向量
				3.6, // 攻击复杂度
			),
			// 漏洞向量
			Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
		},
	},
}

// 将漏洞元数据添加到系统中
if err = s.AddVulnerabilityMetadata(total...); err != nil {
	t.Fatalf("failed to set metadata: %+v", err)
}

// 创建一个空的漏洞元数据切片
var allEntries []model.VulnerabilityMetadataModel
// 从数据库中查找所有条目并将结果存储在allEntries中
s.(*store).db.Find(&allEntries)
// 检查allEntries的长度是否与total的长度相等，如果不相等则输出错误信息
if len(allEntries) != len(total) {
	t.Fatalf("unexpected number of entries: %d", len(allEntries))
}

}

// 测试Store_MergeVulnerabilityMetadata函数
func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		name     string
		add      []v3.VulnerabilityMetadata
		expected v3.VulnerabilityMetadata
		err      bool
	}{
		{
			name: "go-case",
			add: []v3.VulnerabilityMetadata{
				{
					ID:           "my-cve",
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
Cvss: []v3.Cvss{
    # 设置第一个 CVSS 版本为 "2.0"，指标为 (4.1, 5.2, 6.3)，向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
    {
        Version: "2.0",
        Metrics: v3.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 设置第二个 CVSS 版本为 "3.0"，指标为 (1.4, 2.5, 3.6)
    {
        Version: "3.0",
        Metrics: v3.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
# 定义一个包含漏洞元数据的结构体
expected: v3.VulnerabilityMetadata{
    # 指定漏洞的ID
    ID: "my-cve",
    # 指定记录来源
    RecordSource: "record-source",
    # 指定命名空间
    Namespace: "namespace",
    # 指定漏洞严重程度
    Severity: "pretty bad",
    # 指定漏洞相关的URL
    URLs: []string{"https://ancho.re"},
    # 指定漏洞描述
    Description: "worst description ever",
    # 指定CVSS评分
    Cvss: []v3.Cvss{
        {
            # 指定CVSS版本
            Version: "2.0",
            # 指定CVSS指标
            Metrics: v3.NewCvssMetrics(
                4.1, # 指定基本评分
                5.2, # 指定严重性评分
                6.3, # 指定可利用性评分
# 创建一个包含漏洞元数据的切片
add: []v3.VulnerabilityMetadata{
    # 添加一个漏洞元数据对象
    {
        # 设置漏洞ID
        ID: "my-cve",
        # 设置漏洞描述
        Description: "This is my custom CVE",
        # 设置漏洞链接
        Links: []string{"http://example.com/cve/my-cve"},
    },
},
# 定义一个结构体，包含记录来源、命名空间、严重程度和 URL 列表
Record{
    RecordSource: "record-source",
    Namespace:    "namespace",
    Severity:     "pretty bad",
    URLs:         []string{"https://ancho.re"},
},
# 定义另一个结构体，包含记录 ID、记录来源、命名空间、严重程度和 URL 列表
{
    ID:           "my-cve",
    RecordSource: "record-source",
    Namespace:    "namespace",
    Severity:     "pretty bad",
    URLs:         []string{"https://google.com"},
},
# 定义另一个结构体，包含记录 ID、记录来源、命名空间、严重程度和 URL 列表
{
    ID:           "my-cve",
    RecordSource: "record-source",
    Namespace:    "namespace",
    Severity:     "pretty bad",
    URLs:         []string{"https://yahoo.com"},
},
# 定义一个期望的漏洞元数据对象，包括ID、记录来源、命名空间、严重程度、URLs和Cvss
expected: v3.VulnerabilityMetadata{
    ID:           "my-cve",
    RecordSource: "record-source",
    Namespace:    "namespace",
    Severity:     "pretty bad",
    URLs:         []string{"https://ancho.re", "https://google.com", "https://yahoo.com"},
    Cvss:         []v3.Cvss{},
},
# 定义一个测试用的漏洞元数据对象，包括ID、记录来源、命名空间、严重程度和URLs
{
    name: "bad-severity",
    add: []v3.VulnerabilityMetadata{
        {
            ID:           "my-cve",
            RecordSource: "record-source",
            Namespace:    "namespace",
            Severity:     "pretty bad",
            URLs:         []string{"https://ancho.re"},
        },
        {
# 定义一个结构体，包含漏洞的相关信息
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 漏洞的记录来源
    Namespace:    "namespace",  # 漏洞的命名空间
    Severity:     "meh, push that for next tuesday...",  # 漏洞的严重程度
    URLs:         []string{"https://redhat.com"},  # 漏洞相关的URL
},
# 定义一个结构体，包含漏洞的相关信息
{
    name: "mismatch-description",  # 漏洞的名称
    err:  true,  # 是否有错误
    add: []v3.VulnerabilityMetadata{  # 添加漏洞的元数据
        {
            ID:           "my-cve",  # 漏洞的唯一标识符
            RecordSource: "record-source",  # 漏洞的记录来源
            Namespace:    "namespace",  # 漏洞的命名空间
            Severity:     "pretty bad",  # 漏洞的严重程度
            URLs:         []string{"https://ancho.re"},  # 漏洞相关的URL
        }
    }
}
# 描述：最棒的描述
# Cvss：包含多个v3.Cvss对象的数组
# 第一个v3.Cvss对象
{
    # 版本号
    Version: "2.0",
    # 指标
    Metrics: v3.NewCvssMetrics(
        # 基础分数
        4.1,
        # 攻击向量
        5.2,
        # 攻击复杂度
        6.3,
    ),
    # 向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
},
# 第二个v3.Cvss对象
{
    # 版本号
    Version: "3.0",
    # 指标
    Metrics: v3.NewCvssMetrics(
        # 基础分数
        1.4,
        # 攻击向量
        2.5,
        # 攻击复杂度
        3.6,
    ),
    # 向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 创建一个包含 CVE 信息的结构体
{
    ID:           "my-cve",  # CVE ID
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 描述
    Cvss: []v3.Cvss{  # CVE 的 CVSS 评分
        {
            Version: "2.0",  # CVSS 版本
            Metrics: v3.NewCvssMetrics(  # CVSS 评分指标
                4.1,  # 基础评分
                5.2,  # 环境评分
                6.3,  # 基础评分
            ),
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量描述
        },
        {
            # 另一个 CVE 的 CVSS 评分
# 设置版本号为 "3.0"
Version: "3.0",
# 创建一个新的 CVSS 指标对象，设置基本分数为 1.4，攻击复杂度为 2.5，影响范围为 3.6
Metrics: v3.NewCvssMetrics(
    1.4,
    2.5,
    3.6,
),
# 设置向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
# 结束当前的 VulnerabilityMetadata 对象
},
# 结束当前的测试用例
},
# 开始下一个测试用例，名称为 "mismatch-cvss2"
{
name: "mismatch-cvss2",
err:  false,
# 添加一个新的 VulnerabilityMetadata 对象
add: []v3.VulnerabilityMetadata{
    {
    # 设置漏洞 ID 为 "my-cve"
    ID:           "my-cve",
    # 设置记录来源为 "record-source"
    RecordSource: "record-source",
    # 设置命名空间为 "namespace"
    Namespace:    "namespace",
# 定义一个结构体字段 Severity，表示严重程度为 "pretty bad"
# 定义一个结构体字段 URLs，表示URL为空
# 定义一个结构体字段 Description，表示描述为 "best description ever"
# 定义一个结构体字段 Cvss，表示CVSS评分信息
#   - 定义一个Cvss结构体，包含Version、Metrics和Vector字段
#       - Version字段为 "2.0"
#       - Metrics字段为包含4.1、5.2、6.3的CvssMetrics结构体
#       - Vector字段为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
#   - 定义一个Cvss结构体，包含Version、Metrics字段
#       - Version字段为 "3.0"
#       - Metrics字段为包含1.4、2.5、3.6的CvssMetrics结构体
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 漏洞严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "best description ever",  # 漏洞描述
    Cvss: []v3.Cvss{  # CVSS评分
        {
            Version: "2.0",  # CVSS版本
            Metrics: v3.NewCvssMetrics(  # CVSS指标
                4.1,  # 基本评分
                5.2,  # 环境评分
                6.3,  # 基本评分
            ),
            Vector: "AV:P--VERY",  # CVSS向量
        },
    },
},
						},
						{
							// 版本号为3.0，使用v3.NewCvssMetrics创建CVSS指标
							Version: "3.0",
							Metrics: v3.NewCvssMetrics(
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
			// 预期的漏洞元数据
			expected: v3.VulnerabilityMetadata{
				ID:           "my-cve", // 漏洞ID
				RecordSource: "record-source", // 记录来源
				Namespace:    "namespace", // 命名空间
				Severity:     "pretty bad", // 严重程度
				URLs:         []string{"https://ancho.re"}, // 相关URL
				Description:  "best description ever", // 描述
# 创建一个包含多个CVSS对象的列表
Cvss: []v3.Cvss{
    # 创建第一个CVSS对象
    {
        # 设置CVSS版本为2.0
        Version: "2.0",
        # 设置CVSS指标
        Metrics: v3.NewCvssMetrics(
            4.1,    # 基础分数
            5.2,    # 攻击向量
            6.3,    # 攻击复杂性
        ),
        # 设置CVSS向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 创建第二个CVSS对象
    {
        # 设置CVSS版本为3.0
        Version: "3.0",
        # 设置CVSS指标
        Metrics: v3.NewCvssMetrics(
            1.4,    # 基础分数
            2.5,    # 攻击向量
            3.6,    # 攻击复杂性
        ),
        # 设置CVSS向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
    # 继续添加其他CVSS对象
    {
        ...
    },
    ...
},
# 设置版本号为 "2.0"
Version: "2.0",
# 创建一个新的 CVSS 指标对象，传入基本分数、影响分数和可利用性分数
Metrics: v3.NewCvssMetrics(
    4.1,
    5.2,
    6.3,
),
# 设置向量为 "AV:P--VERY"
Vector: "AV:P--VERY",
# 结束 CVSS 指标对象的设置
},
},
},
# 设置漏洞名称为 "mismatch-cvss3"
{
name: "mismatch-cvss3",
# 设置错误标志为 false
err:  false,
# 添加一个新的漏洞元数据对象，传入漏洞 ID、记录来源、命名空间和严重程度
add: []v3.VulnerabilityMetadata{
    {
    ID:           "my-cve",
    RecordSource: "record-source",
    Namespace:    "namespace",
    Severity:     "pretty bad",
# 创建一个包含 URL 和描述的结构体
URLs:         []string{"https://ancho.re"},
Description:  "best description ever",
# 创建一个包含 CVSS 评分的结构体数组
Cvss: []v3.Cvss{
    # 创建一个 CVSS 2.0 版本的评分结构体
    {
        Version: "2.0",
        # 创建一个包含 CVSS 2.0 版本评分指标的结构体
        Metrics: v3.NewCvssMetrics(
            4.1,
            5.2,
            6.3,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 创建一个 CVSS 3.0 版本的评分结构体
    {
        Version: "3.0",
        # 创建一个包含 CVSS 3.0 版本评分指标的结构体
        Metrics: v3.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
# 创建一个包含 CVE 信息的数据结构
{
    ID:           "my-cve",  # CVE ID
    RecordSource: "record-source",  # 记录来源
    Namespace:    "namespace",  # 命名空间
    Severity:     "pretty bad",  # 严重程度
    URLs:         []string{"https://ancho.re"},  # 相关链接
    Description:  "best description ever",  # 描述
    Cvss: []v3.Cvss{  # CVE 的 CVSS 评分
        {
            Version: "2.0",  # CVSS 版本
            Metrics: v3.NewCvssMetrics(  # CVSS 评分指标
                4.1,  # 基础评分
                5.2,  # 攻击向量
                6.3,  # 攻击复杂度
            ),
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量描述
        },
# 创建一个包含漏洞信息的数据结构
{
    # 漏洞版本号
    Version: "3.0",
    # 使用 v3.NewCvssMetrics 创建漏洞的 CVSS 指标
    Metrics: v3.NewCvssMetrics(
        # 基础指标
        1.4,
        # 攻击向量指标
        0,
        # 机密性影响指标
        3.6,
    ),
    # 漏洞向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 期望的漏洞元数据
expected: v3.VulnerabilityMetadata{
    # 漏洞 ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 严重程度
    Severity: "pretty bad",
    # 相关 URL
    URLs: []string{"https://ancho.re"},
    # 漏洞描述
    Description: "best description ever",
    # 漏洞的 CVSS 指标
    Cvss: []v3.Cvss{
# 创建一个包含不同版本的 CVSS 数据的列表
{
    # 版本号为 2.0
    Version: "2.0",
    # 使用 v3.NewCvssMetrics() 创建一个包含 CVSS 指标的对象
    Metrics: v3.NewCvssMetrics(
        # 设置不同的指标值
        4.1,
        5.2,
        6.3,
    ),
    # 设置 CVSS 向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
},
{
    # 版本号为 3.0
    Version: "3.0",
    # 使用 v3.NewCvssMetrics() 创建一个包含 CVSS 指标的对象
    Metrics: v3.NewCvssMetrics(
        # 设置不同的指标值
        1.4,
        2.5,
        3.6,
    ),
    # 设置 CVSS 向量
    Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
{
    # 版本号为 3.0
# 创建一个新的 CVSS 指标对象，传入相应的参数
Metrics: v3.NewCvssMetrics(
    1.4,    # 基础分数
    0,      # 攻击向量
    3.6,    # 攻击复杂度
),
# 设置 CVSS 指标对象的向量
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
# 遍历测试用例
for _, test := range tests {
    # 使用测试名称创建一个子测试
    t.Run(test.name, func(t *testing.T) {
        # 创建临时目录
        dbTempDir := t.TempDir()
        # 创建一个新的存储对象
        s, err := New(dbTempDir, true)
        # 如果创建存储对象时出现错误，打印错误信息并终止测试
        if err != nil {
            t.Fatalf("could not create store: %+v", err)
        }
// 按顺序添加每个元数据
var theErr error
for _, metadata := range test.add {
    // 添加漏洞元数据
    err = s.AddVulnerabilityMetadata(metadata)
    if err != nil {
        theErr = err
        break
    }
}

if test.err && theErr == nil {
    // 如果期望出现错误但没有出现，则测试失败
    t.Fatalf("expected error but did not get one")
} else if !test.err && theErr != nil {
    // 如果不期望出现错误但出现了，则测试失败
    t.Fatalf("expected no error but got one: %+v", theErr)
} else if test.err && theErr != nil {
    // 如果期望出现错误且出现了，则测试通过
    // 测试通过后直接返回，不再执行后续代码
    return
}

// 确保只有一个条目
// 创建一个空的VulnerabilityMetadataModel切片
var allEntries []model.VulnerabilityMetadataModel
// 从数据库中查找所有的VulnerabilityMetadataModel并存储到allEntries切片中
s.(*store).db.Find(&allEntries)
// 如果allEntries的长度不等于1，则输出错误信息
if len(allEntries) != 1 {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}

// 获取预期的元数据对象
if actual, err := s.GetVulnerabilityMetadata(test.expected.ID, test.expected.Namespace); err != nil {
    // 如果获取元数据对象失败，则输出错误信息
    t.Fatalf("failed to get metadata: %+v", err)
} else {
    // 比较预期的元数据对象和实际获取的元数据对象
    diffs := deep.Equal(&test.expected, actual)
    // 如果存在差异，则输出差异信息
    if len(diffs) > 0 {
        for _, d := range diffs {
            t.Errorf("Diff: %+v", d)
        }
    }
}
// 定义一个测试函数，用于测试从元数据中提取 CVSS 分数
func TestCvssScoresInMetadata(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		name     string
		add      []v3.VulnerabilityMetadata
		expected v3.VulnerabilityMetadata
	}{
		{
			name: "append-cvss",
			// 添加漏洞元数据
			add: []v3.VulnerabilityMetadata{
				{
					ID:           "my-cve",
					RecordSource: "record-source",
					Namespace:    "namespace",
					Severity:     "pretty bad",
					URLs:         []string{"https://ancho.re"},
					Description:  "worst description ever",
					// 添加 CVSS 评分
					Cvss: []v3.Cvss{
						{
							Version: "2.0",
# 创建一个新的 CVSS 指标对象，传入基本指标值
Metrics: v3.NewCvssMetrics(
    4.1,  # 基本指标：基础得分
    5.2,  # 基本指标：攻击向量
    6.3,  # 基本指标：攻击复杂性
),
# 设置 CVSS 向量
Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
# 定义一个包含多个元素的数组
# 数组的元素包括 1.4, 2.5, 3.6
# Vector 是一个字符串，表示漏洞的特性
# expected 是一个 v3.VulnerabilityMetadata 类型的对象
# expected 对象包含 ID, RecordSource, Namespace, Severity, URLs, Description, Cvss 等属性
# Cvss 属性是一个包含 v3.Cvss 对象的数组
# 每个 v3.Cvss 对象包含 Version 和 Metrics 属性
# Metrics 属性是一个 v3.CvssMetrics 对象
# 定义一个包含多个元素的列表，每个元素是一个字典
[
    {
        # 字典中的键值对
        Version: "2.0",
        Metrics: v2.NewCvssMetrics(
            # 调用 v2 包中的 NewCvssMetrics 函数，传入参数
            4.1,
            5.2,
            6.3,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    {
        Version: "3.0",
        Metrics: v3.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
]
# 添加一个漏洞元数据到列表中
add: []v3.VulnerabilityMetadata{
    # 漏洞的唯一标识符
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 漏洞的严重程度
    Severity: "pretty bad",
    # 相关的 URL 列表
    URLs: []string{"https://ancho.re"},
    # 漏洞的描述
    Description: "worst description ever",
    # 漏洞的 CVSS 评分
    Cvss: []v3.Cvss{
        {
            # CVSS 版本
            Version: "2.0",
            # CVSS 评分指标
            Metrics: v3.NewCvssMetrics(
                4.1,
                5.2,
                6.3,
            ),
            # CVSS 向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
    },
},
# 创建一个包含漏洞信息的结构体
{
    # 漏洞ID
    ID:           "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace:    "namespace",
    # 漏洞严重程度
    Severity:     "pretty bad",
    # 相关URL
    URLs:         []string{"https://ancho.re"},
    # 漏洞描述
    Description:  "worst description ever",
    # CVSS评分
    Cvss: []v3.Cvss{
        {
            # CVSS版本
            Version: "2.0",
            # CVSS指标
            Metrics: v3.NewCvssMetrics(
                4.1,
                5.2,
                6.3,
            ),
            # CVSS向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
            # 供应商元数据
            VendorMetadata: CustomMetadata{
                # 超级分数
                SuperScore: "100",
                # 供应商
                Vendor:     "debian",
            },
        }
    }
}
# 定义一个包含漏洞元数据的结构体
expected: v3.VulnerabilityMetadata{
    # 指定漏洞的ID
    ID: "my-cve",
    # 指定记录来源
    RecordSource: "record-source",
    # 指定命名空间
    Namespace: "namespace",
    # 指定漏洞的严重程度
    Severity: "pretty bad",
    # 指定漏洞相关的URL
    URLs: []string{"https://ancho.re"},
    # 指定漏洞的描述
    Description: "worst description ever",
    # 指定漏洞的CVSS评分
    Cvss: []v3.Cvss{
        {
            # 指定CVSS版本
            Version: "2.0",
            # 指定CVSS评分的度量
            Metrics: v3.NewCvssMetrics(
                4.1,
                5.2,
                6.3,
            ),
            # 指定CVSS向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
					},
					{
						// 设置版本号为 "2.0"
						Version: "2.0",
						// 创建新的 CVSS 指标对象，设置相应的值
						Metrics: v3.NewCvssMetrics(
							4.1,
							5.2,
							6.3,
						),
						// 设置向量为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
						Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
						// 设置供应商元数据
						VendorMetadata: CustomMetadata{
							// 设置超级分数为 "100"
							SuperScore: "100",
							// 设置供应商为 "debian"
							Vendor:     "debian",
						},
					},
				},
			},
		},
		// 设置名称为 "avoids-duplicate-cvss" 的漏洞元数据
		{
			name: "avoids-duplicate-cvss",
			// 添加漏洞元数据到数组中
			add: []v3.VulnerabilityMetadata{
# 创建一个包含漏洞信息的结构体
{
    # 漏洞ID
    ID: "my-cve",
    # 记录来源
    RecordSource: "record-source",
    # 命名空间
    Namespace: "namespace",
    # 漏洞严重程度
    Severity: "pretty bad",
    # 相关URL
    URLs: []string{"https://ancho.re"},
    # 漏洞描述
    Description: "worst description ever",
    # CVSS评分
    Cvss: []v3.Cvss{
        {
            # CVSS版本
            Version: "3.0",
            # CVSS指标
            Metrics: v3.NewCvssMetrics(
                1.4,
                2.5,
                3.6,
            ),
            # CVSS向量
            Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
        },
    },
},
# 设置漏洞的唯一标识符
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
Description:  "worst description ever",
# 设置漏洞的 CVSS 评分
Cvss: []v3.Cvss{
    {
        # 设置 CVSS 版本
        Version: "3.0",
        # 设置 CVSS 的度量指标
        Metrics: v3.NewCvssMetrics(
            1.4,
            2.5,
            3.6,
        ),
        # 设置 CVSS 的向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
# 设置 CVE 的唯一标识符
ID:           "my-cve",
# 设置记录来源
RecordSource: "record-source",
# 设置命名空间
Namespace:    "namespace",
# 设置严重程度
Severity:     "pretty bad",
# 设置相关 URL
URLs:         []string{"https://ancho.re"},
# 设置描述信息
Description:  "worst description ever",
# 设置 CVE 的 CVSS 评分
Cvss: []v3.Cvss{
    {
        # 设置 CVSS 版本
        Version: "3.0",
        # 设置 CVSS 评分指标
        Metrics: v3.NewCvssMetrics(
            1.4,    # 基础评分
            2.5,    # 攻击向量
            3.6,    # 攻击复杂性
        ),
        # 设置 CVSS 向量
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
# 遍历测试用例列表
for _, test := range tests {
    # 使用测试名称创建子测试
    t.Run(test.name, func(t *testing.T) {
        # 在临时目录创建数据库临时目录
        dbTempDir := t.TempDir()

        # 创建新的数据库实例
        s, err := New(dbTempDir, true)
        if err != nil {
            t.Fatalf("could not create s: %+v", err)
        }

        # 依次添加每个元数据
        for _, metadata := range test.add {
            err = s.AddVulnerabilityMetadata(metadata)
            if err != nil {
                t.Fatalf("unable to s vulnerability metadata: %+v", err)
            }
        }

        # 确保数据库中只有一个条目
        var allEntries []model.VulnerabilityMetadataModel
        s.(*store).db.Find(&allEntries)
    }
}
// 检查所有条目的数量是否不等于1，如果不等于1则输出错误信息
if len(allEntries) != 1 {
    t.Fatalf("unexpected number of entries: %d", len(allEntries))
}

// 调用 assertVulnerabilityMetadataReader 函数，验证预期的漏洞元数据读取器
assertVulnerabilityMetadataReader(t, s, test.expected.ID, test.expected.Namespace, test.expected)
```

```
// 测试 DiffStore 函数
func Test_DiffStore(t *testing.T) {
    // 初始化数据库临时文件夹
    dbTempFile := t.TempDir()

    // 创建新的存储实例 s1
    s1, err := New(dbTempFile, true)
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 重新初始化数据库临时文件夹
    dbTempFile = t.TempDir()

    // 创建新的存储实例 s2
    s2, err := New(dbTempFile, true)
}
	// 如果发生错误，输出错误信息并终止测试
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}

	// 创建基础漏洞列表
	baseVulns := []v3.Vulnerability{
		// 第一个漏洞信息
		{
			Namespace:         "github:python", // 命名空间
			ID:                "CVE-123-4567",   // 漏洞ID
			PackageName:       "pypi:requests",  // 包名
			VersionConstraint: "< 2.0 >= 1.29", // 版本约束
			CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"}, // CPE（通用漏洞披露）条目
		},
		// 第二个漏洞信息
		{
			Namespace:         "github:python",
			ID:                "CVE-123-4567",
			PackageName:       "pypi:requests",
			VersionConstraint: "< 3.0 >= 2.17",
			CPEs:              []string{"cpe:2.3:pypi:requests:*:*:*:*:*:*"},
		},
		{
# 设置命名空间为 "npm"，漏洞ID为 "CVE-123-7654"，包名为 "npm:axios"
# 版本约束为 "< 3.0 >= 2.17"，CPEs为空列表
# 修复状态为未知
{
    Namespace:         "npm",
    ID:                "CVE-123-7654",
    PackageName:       "npm:axios",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
    Fix: v3.Fix{
        State: v3.UnknownFixState,
    },
},
# 设置命名空间为 "nuget"，漏洞ID为 "GHSA-****-******"，包名为 "nuget:net"
# 版本约束为 "< 3.0 >= 2.17"，CPEs为空列表
# 修复状态为未知
{
    Namespace:         "nuget",
    ID:                "GHSA-****-******",
    PackageName:       "nuget:net",
    VersionConstraint: "< 3.0 >= 2.17",
    CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
    Fix: v3.Fix{
        State: v3.UnknownFixState,
    },
},
# 其他类似的设置
		// 设置命名空间为 "hex"
		Namespace:         "hex",
		// 设置漏洞 ID
		ID:                "GHSA-^^^^-^^^^^^",
		// 设置包名为 "hex:esbuild"
		PackageName:       "hex:esbuild",
		// 设置版本约束为 "< 3.0 >= 2.17"
		VersionConstraint: "< 3.0 >= 2.17",
		// 设置 CPEs 为空数组
		CPEs:              []string{"cpe:2.3:hex:esbuild:*:*:*:*:*:*"},
	},
}
// 创建基础元数据数组
baseMetadata := []v3.VulnerabilityMetadata{
	{
		// 设置命名空间为 "nuget"
		Namespace:  "nuget",
		// 设置漏洞 ID
		ID:         "GHSA-****-******",
		// 设置数据源为 "nvd"
		DataSource: "nvd",
	},
}
// 创建目标漏洞数组
targetVulns := []v3.Vulnerability{
	{
		// 设置命名空间为 "github:python"
		Namespace:         "github:python",
		// 设置漏洞 ID
		ID:                "CVE-123-4567",
		// 设置包名为 "pypi:requests"
		PackageName:       "pypi:requests",
		// 设置版本约束为 "< 2.0 >= 1.29"
		VersionConstraint: "< 2.0 >= 1.29",
		{
			// 定义漏洞所属的命名空间
			Namespace:         "github:go",
			// 定义漏洞的唯一标识符
			ID:                "GHSA-....-....",
			// 定义受影响的软件包名称
			PackageName:       "hashicorp:nomad",
			// 定义受影响的软件包版本约束
			VersionConstraint: "< 3.0 >= 2.17",
			// 定义受影响的 CPE（通用漏洞和暴露）标识符
			CPEs:              []string{"cpe:2.3:golang:hashicorp:nomad:*:*:*:*:*"},
		},
		{
			// 定义漏洞所属的命名空间
			Namespace:         "github:go",
			// 定义漏洞的唯一标识符
			ID:                "GHSA-....-....",
			// 定义受影响的软件包名称
			PackageName:       "hashicorp:n",
			// 定义受影响的软件包版本约束
			VersionConstraint: "< 2.0 >= 1.17",
			// 定义受影响的 CPE（通用漏洞和暴露）标识符
			CPEs:              []string{"cpe:2.3:golang:hashicorp:n:*:*:*:*:*"},
		},
		{
			// 定义漏洞所属的命名空间
			Namespace:         "npm",
			// 定义漏洞的唯一标识符
			ID:                "CVE-123-7654",
			// 定义受影响的软件包名称
			PackageName:       "npm:axios",
```

# 设置版本约束为小于3.0且大于等于2.17
VersionConstraint: "< 3.0 >= 2.17",
# 设置CPEs为空字符串数组
CPEs:              []string{"cpe:2.3:npm:axios:*:*:*:*:*:*"},
# 设置修复状态为不修复
Fix: v3.Fix{
    State: v3.WontFixState,
},
# 设置命名空间为nuget
Namespace:         "nuget",
# 设置ID
ID:                "GHSA-****-******",
# 设置包名
PackageName:       "nuget:net",
# 设置版本约束为小于3.0且大于等于2.17
VersionConstraint: "< 3.0 >= 2.17",
# 设置CPEs为字符串数组
CPEs:              []string{"cpe:2.3:nuget:net:*:*:*:*:*:*"},
# 设置修复状态为未知
Fix: v3.Fix{
    State: v3.UnknownFixState,
},
# 设置期望的差异为改变
expectedDiffs := []v3.Diff{
    Reason:    v3.DiffChanged,
		{
			# 漏洞的唯一标识符
			ID:        "CVE-123-4567",
			# 漏洞所属的命名空间
			Namespace: "github:python",
			# 受影响的软件包列表
			Packages:  []string{"pypi:requests"},
		},
		{
			# 漏洞变更的原因
			Reason:    v3.DiffChanged,
			# 漏洞的唯一标识符
			ID:        "CVE-123-7654",
			# 漏洞所属的命名空间
			Namespace: "npm",
			# 受影响的软件包列表
			Packages:  []string{"npm:axios"},
		},
		{
			# 漏洞变更的原因
			Reason:    v3.DiffRemoved,
			# 漏洞的唯一标识符
			ID:        "GHSA-****-******",
			# 漏洞所属的命名空间
			Namespace: "nuget",
			# 受影响的软件包列表
			Packages:  []string{"nuget:net"},
		},
		{
			# 漏洞变更的原因
			Reason:    v3.DiffAdded,
			# 漏洞的唯一标识符
			ID:        "GHSA-....-....",
			# 漏洞所属的命名空间
			Namespace: "github:go",
		}
# 创建一个包含漏洞信息的切片，每个漏洞包含原因、ID、命名空间和包列表
baseVulns := []v3.Vulnerability{
	{
		Reason:    v3.DiffAdded,  # 漏洞添加的原因
		ID:        "GHSA-^^^^-^^^^^^",  # 漏洞的唯一标识符
		Namespace: "hashicorp",  # 漏洞所属的命名空间
		Packages:  []string{"hashicorp:n", "hashicorp:nomad"},  # 受影响的包列表
	},
	{
		Reason:    v3.DiffRemoved,  # 漏洞移除的原因
		ID:        "GHSA-^^^^-^^^^^^",  # 漏洞的唯一标识符
		Namespace: "hex",  # 漏洞所属的命名空间
		Packages:  []string{"hex:esbuild"},  # 受影响的包列表
	},
}

# 将baseVulns中的漏洞添加到s1中
for _, vuln := range baseVulns {
	s1.AddVulnerability(vuln)
}

# 将targetVulns中的漏洞添加到s2中
for _, vuln := range targetVulns {
	s2.AddVulnerability(vuln)
}

# 将baseMetadata中的元数据添加到s1中
for _, meta := range baseMetadata {
	s1.AddVulnerabilityMetadata(meta)
}
	// 调用 s1 的 DiffStore 方法，传入 s2 作为参数，返回结果和错误
	result, err := s1.DiffStore(s2)

	// 对结果进行排序，按照 ID 的大小进行稳定排序
	sort.SliceStable(*result, func(i, j int) bool {
		return (*result)[i].ID < (*result)[j].ID
	})
	// 对每个结果中的 Packages 列表进行排序
	for i := range *result {
		sort.Strings((*result)[i].Packages)
	}

	// 断言，验证错误是否为空
	assert.NoError(t, err)
	// 断言，验证结果是否与期望的差异相等
	assert.Equal(t, expectedDiffs, *result)
}
```