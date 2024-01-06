# `grype\grype\db\v2\store\store_test.go`

```
package store

import (
	"testing"
	"time"

	"github.com/go-test/deep"  // 导入深度比较包

	v2 "github.com/anchore/grype/grype/db/v2"  // 导入版本2的包
	"github.com/anchore/grype/grype/db/v2/store/model"  // 导入存储模型包
)

func assertIDReader(t *testing.T, reader v2.IDReader, expected v2.ID) {  // 定义一个断言函数，用于测试IDReader
	t.Helper()  // 标记该函数是测试辅助函数
	if actual, err := reader.GetID(); err != nil {  // 获取实际的ID
		t.Fatalf("failed to get ID: %+v", err)  // 如果获取失败，输出错误信息
	} else {
		diffs := deep.Equal(&expected, actual)  // 使用深度比较函数比较期望的ID和实际的ID
		if len(diffs) > 0 {  // 如果存在差异
			for _, d := range diffs {  // 遍历差异
		// 如果出现错误，使用 Errorf 方法记录错误信息
		t.Errorf("Diff: %+v", d)
	}

	// 遍历测试用例
	for _, tc := range testCases {
		// 如果测试用例的期望值和实际值不相等
		if d := cmp.Diff(tc.expected, tc.actual); d != "" {
			// 使用 Errorf 方法记录差异信息
			t.Errorf("Diff: %+v", d)
		}
	}
}

// 定义测试函数 TestStore_GetID_SetID
func TestStore_GetID_SetID(t *testing.T) {
	// 创建临时文件夹作为数据库存储路径
	dbTempFile := t.TempDir()

	// 创建一个新的存储对象
	s, err := New(dbTempFile, true)
	// 如果创建过程中出现错误，记录错误信息并终止测试
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}

	// 创建一个期望的 ID 对象
	expected := v2.ID{
		BuildTimestamp: time.Now().UTC(),
		SchemaVersion:  2,
	}

	// 将期望的 ID 对象存储到数据库中
	if err = s.SetID(expected); err != nil {
		// 如果设置 ID 失败，输出错误信息
		t.Fatalf("failed to set ID: %+v", err)
	}

	// 断言 ID 读取器
	assertIDReader(t, s, expected)

}

// 断言漏洞读取器
func assertVulnerabilityReader(t *testing.T, reader v2.VulnerabilityStoreReader, namespace, name string, expected []v2.Vulnerability) {
	// 获取漏洞信息，如果出错，输出错误信息
	if actual, err := reader.GetVulnerability(namespace, name); err != nil {
		t.Fatalf("failed to get Vulnerability: %+v", err)
	} else {
		// 比较实际漏洞数量和期望漏洞数量
		if len(actual) != len(expected) {
			t.Fatalf("unexpected number of vulns: %d", len(actual))
		}

		// 遍历比较每个漏洞的差异
		for idx := range actual {
			// 比较期望漏洞和实际漏洞的差异
			diffs := deep.Equal(expected[idx], actual[idx])
			if len(diffs) > 0 {
				// 输出差异信息
				for _, d := range diffs {
					t.Errorf("Diff: %+v", d)
func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    // 创建临时文件夹作为数据库存储路径
    dbTempFile := t.TempDir()
    // 创建一个新的存储对象
    s, err := New(dbTempFile, true)
    // 如果创建存储对象出错，则打印错误信息
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 创建一个漏洞信息数组
    extra := []v2.Vulnerability{
        {
            ID:                   "my-cve-33333",
            RecordSource:         "record-source",
            PackageName:          "package-name-2",
            Namespace:            "my-namespace",
            VersionConstraint:    "< 1.0",
            // 其他漏洞信息...
        },
        // 其他漏洞信息...
    }
		// 设置版本格式为语义化版本
		VersionFormat:        "semver",
		// 设置CPEs为空字符串数组
		CPEs:                 []string{"a-cool-cpe"},
		// 设置代理漏洞为另一个CVE和另一个CVE
		ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
		// 设置修复版本为"2.0.1"
		FixedInVersion:       "2.0.1",
	},
	{
		// 设置ID为"my-other-cve-33333"
		ID:                   "my-other-cve-33333",
		// 设置记录来源为"record-source"
		RecordSource:         "record-source",
		// 设置包名为"package-name-3"
		PackageName:          "package-name-3",
		// 设置命名空间为"my-namespace"
		Namespace:            "my-namespace",
		// 设置版本约束为"< 509.2.2"
		VersionConstraint:    "< 509.2.2",
		// 设置版本格式为语义化版本
		VersionFormat:        "semver",
		// 设置CPEs为"a-cool-cpe"的字符串数组
		CPEs:                 []string{"a-cool-cpe"},
		// 设置代理漏洞为另一个CVE和另一个CVE
		ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
	},

	expected := []v2.Vulnerability{
		{
			// 设置ID为"my-cve"
			ID:                   "my-cve",
			// 记录来源
			RecordSource:         "record-source",
			// 包名
			PackageName:          "package-name",
			// 命名空间
			Namespace:            "my-namespace",
			// 版本约束
			VersionConstraint:    "< 1.0",
			// 版本格式
			VersionFormat:        "semver",
			// CPEs
			CPEs:                 []string{"a-cool-cpe"},
			// 代理漏洞
			ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
			// 修复版本
			FixedInVersion:       "1.0.1",
		},
		{
			// ID
			ID:                   "my-other-cve",
			// 记录来源
			RecordSource:         "record-source",
			// 包名
			PackageName:          "package-name",
			// 命名空间
			Namespace:            "my-namespace",
			// 版本约束
			VersionConstraint:    "< 509.2.2",
			// 版本格式
			VersionFormat:        "semver",
			// CPEs
			CPEs:                 []string{"a-cool-cpe"},
			// 代理漏洞
			ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
			// 修复版本
			FixedInVersion:       "4.0.5",
		},
	}

	// 将 expected 和 extra 合并成一个新的切片 total
	total := append(expected, extra...)

	// 将合并后的切片作为参数传递给 AddVulnerability 方法，如果出现错误则输出错误信息
	if err = s.AddVulnerability(total...); err != nil {
		t.Fatalf("failed to set Vulnerability: %+v", err)
	}

	// 创建一个空的 VulnerabilityModel 切片 allEntries，然后从数据库中查找所有的 VulnerabilityModel 并存储到 allEntries 中
	var allEntries []model.VulnerabilityModel
	s.(*store).db.Find(&allEntries)

	// 检查 allEntries 的长度是否与 total 的长度相等，如果不相等则输出错误信息
	if len(allEntries) != len(total) {
		t.Fatalf("unexpected number of entries: %d", len(allEntries))
	}

	// 调用 assertVulnerabilityReader 方法，对比 reader 中的数据与 expected 中的数据是否一致
	assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)

}

// 断言 VulnerabilityMetadataStoreReader 的实现是否正确
func assertVulnerabilityMetadataReader(t *testing.T, reader v2.VulnerabilityMetadataStoreReader, id, recordSource string, expected v2.VulnerabilityMetadata) {
	// 通过 reader 获取指定 id 和 recordSource 的 VulnerabilityMetadata，如果出现错误则输出错误信息
	if actual, err := reader.GetVulnerabilityMetadata(id, recordSource); err != nil {
		// 如果获取元数据失败，则输出错误信息
		t.Fatalf("failed to get metadata: %+v", err)
	} else {
		// 比较预期值和实际值的差异
		diffs := deep.Equal(&expected, actual)
		// 如果存在差异，则输出差异信息
		if len(diffs) > 0 {
			for _, d := range diffs {
				t.Errorf("Diff: %+v", d)
			}
		}
	}
}

func TestStore_GetVulnerabilityMetadata_SetVulnerabilityMetadata(t *testing.T) {
	// 创建临时数据库文件
	dbTempFile := t.TempDir()
	// 创建存储对象
	s, err := New(dbTempFile, true)
	// 如果创建存储对象失败，则输出错误信息
	if err != nil {
		t.Fatalf("could not create store: %+v", err)
	}
```

// 创建一个包含漏洞元数据的切片
total := []v2.VulnerabilityMetadata{
	// 创建一个漏洞元数据对象
	{
		// 设置漏洞ID
		ID: "my-cve",
		// 设置记录来源
		RecordSource: "record-source",
		// 设置漏洞严重程度
		Severity: "pretty bad",
		// 设置漏洞链接
		Links: []string{"https://ancho.re"},
		// 设置漏洞描述
		Description: "best description ever",
		// 创建并设置CVSS V2对象
		CvssV2: &v2.Cvss{
			BaseScore:           1.1,
			ExploitabilityScore: 2.2,
			ImpactScore:         3.3,
			Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--NOT",
		},
		// 创建并设置CVSS V3对象
		CvssV3: &v2.Cvss{
			BaseScore:           1.3,
			ExploitabilityScore: 2.1,
			ImpactScore:         3.2,
			Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--NICE",
		},
		// 继续添加其他漏洞元数据对象...
	},
	// 继续添加其他漏洞元数据对象...
}
		},
		{
			// CVE的唯一标识符
			ID:           "my-other-cve",
			// 记录来源
			RecordSource: "record-source",
			// 严重程度
			Severity:     "pretty bad",
			// 相关链接
			Links:        []string{"https://ancho.re"},
			// 描述
			Description:  "worst description ever",
			// CVE的CVSS V2评分
			CvssV2: &v2.Cvss{
				// 基础分
				BaseScore:           4.1,
				// 可利用性分数
				ExploitabilityScore: 5.2,
				// 影响分数
				ImpactScore:         6.3,
				// 向量描述
				Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
			},
			// CVE的CVSS V3评分
			CvssV3: &v2.Cvss{
				// 基础分
				BaseScore:           1.4,
				// 可利用性分数
				ExploitabilityScore: 2.5,
				// 影响分数
				ImpactScore:         3.6,
				// 向量描述
				Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
			},
		},
	}

	// 将漏洞元数据添加到存储中
	if err = s.AddVulnerabilityMetadata(total...); err != nil {
		t.Fatalf("failed to set metadata: %+v", err)
	}

	// 查询所有的漏洞元数据
	var allEntries []model.VulnerabilityMetadataModel
	s.(*store).db.Find(&allEntries)
	// 检查查询结果是否和添加的漏洞元数据数量一致
	if len(allEntries) != len(total) {
		t.Fatalf("unexpected number of entries: %d", len(allEntries))
	}

}

// 测试漏洞元数据合并功能
func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
	tests := []struct {
		name     string
		add      []v2.VulnerabilityMetadata
		expected v2.VulnerabilityMetadata
		err      bool
```

# 创建一个包含漏洞元数据的结构体切片
{
    name: "go-case",  # 漏洞名称
    add: []v2.VulnerabilityMetadata{  # 添加漏洞元数据
        {
            ID: "my-cve",  # 漏洞ID
            RecordSource: "record-source",  # 记录来源
            Severity: "pretty bad",  # 漏洞严重程度
            Links: []string{"https://ancho.re"},  # 相关链接
            Description: "worst description ever",  # 漏洞描述
            CvssV2: &v2.Cvss{  # CVSS V2评分
                BaseScore: 4.1,  # 基础分数
                ExploitabilityScore: 5.2,  # 可利用性分数
                ImpactScore: 6.3,  # 影响分数
                Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量
            },
            CvssV3: &v2.Cvss{  # CVSS V3评分
                BaseScore: 1.4,  # 基础分数
                ExploitabilityScore: 2.5,  # 可利用性分数
                ImpactScore: 3.6,  # 影响分数
                # 其他CVSS V3评分字段
            }
        }
        # 其他漏洞元数据
    }
}
# 定义一个预期的漏洞元数据对象
expected: v2.VulnerabilityMetadata{
    # 指定漏洞ID
    ID: "my-cve",
    # 指定记录来源
    RecordSource: "record-source",
    # 指定漏洞严重程度
    Severity: "pretty bad",
    # 指定相关链接
    Links: []string{"https://ancho.re"},
    # 指定漏洞描述
    Description: "worst description ever",
    # 指定CVSS V2评分
    CvssV2: &v2.Cvss{
        BaseScore: 4.1,
        ExploitabilityScore: 5.2,
        ImpactScore: 6.3,
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 指定CVSS V3评分
    CvssV3: &v2.Cvss{
        BaseScore: 1.4,
        ExploitabilityScore: 2.5,
        ImpactScore: 3.6,
# 创建一个名为"merge-links"的漏洞元数据列表
{
    name: "merge-links",
    add: []v2.VulnerabilityMetadata{
        # 添加一个漏洞元数据对象
        {
            ID: "my-cve",
            RecordSource: "record-source",
            Severity: "pretty bad",
            Links: []string{"https://ancho.re"},
        },
        # 添加另一个漏洞元数据对象
        {
            ID: "my-cve",
            RecordSource: "record-source",
            Severity: "pretty bad",
            Links: []string{"https://google.com"},
        },
        {
            # 继续添加其他漏洞元数据对象
# 定义一个结构体字段为"my-cve"的ID
ID:           "my-cve",
# 定义一个结构体字段为"record-source"的RecordSource
RecordSource: "record-source",
# 定义一个结构体字段为"pretty bad"的Severity
Severity:     "pretty bad",
# 定义一个结构体字段为包含"https://yahoo.com"的Links
Links:        []string{"https://yahoo.com"},
# 定义一个结构体字段为"my-cve"的ID
ID:           "my-cve",
# 定义一个结构体字段为"record-source"的RecordSource
RecordSource: "record-source",
# 定义一个结构体字段为"pretty bad"的Severity
Severity:     "pretty bad",
// 创建一个包含漏洞信息的切片，每个漏洞信息包含ID、记录来源、严重程度和链接
// 第一个漏洞信息
{
    ID:           "my-cve",
    RecordSource: "record-source",
    Severity:     "meh, push that for next tuesday...",
    Links:        []string{"https://redhat.com"},
},
// 第二个漏洞信息
{
    ID:           "my-cve",
    RecordSource: "record-source",
    Severity:     "pretty bad",
    Links:        []string{"https://ancho.re"},
}
// 错误标志为 true
err: true,
// 漏洞信息名称为 "mismatch-description"
name: "mismatch-description",
// 错误标志为 true
err:  true,
// 添加漏洞信息到切片
add: []v2.VulnerabilityMetadata{
    {
        ID:           "my-cve",
        RecordSource: "record-source",
        Severity:     "pretty bad",
        Links:        []string{"https://ancho.re"},
    }
}
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 漏洞信息来源
    Severity:     "pretty bad",  # 漏洞严重程度
    Links:        []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 漏洞描述
    # 使用 CVSS v2 对漏洞进行评分
    CvssV2: &v2.Cvss{
        BaseScore:           4.1,  # 基础分数
        ExploitabilityScore: 5.2,  # 可利用性分数
        ImpactScore:         6.3,  # 影响分数
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量描述
    },
    # 使用 CVSS v3 对漏洞进行评分
    CvssV3: &v2.Cvss{
        BaseScore:           1.4,  # 基础分数
        ExploitabilityScore: 2.5,  # 可利用性分数
        ImpactScore:         3.6,  # 影响分数
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 向量描述
    },
},
# 创建一个包含CVSSv2和CVSSv3评分的VulnerabilityMetadata对象
CvssV2: &v2.Cvss{
    BaseScore:           4.1,  # 设置CVSSv2基础评分
    ExploitabilityScore: 5.2,  # 设置CVSSv2可利用性评分
    ImpactScore:         6.3,  # 设置CVSSv2影响评分
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 设置CVSSv2向量
},
CvssV3: &v2.Cvss{
    BaseScore:           1.4,  # 设置CVSSv3基础评分
    ExploitabilityScore: 2.5,  # 设置CVSSv3可利用性评分
    ImpactScore:         3.6,  # 设置CVSSv3影响评分
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 设置CVSSv3向量
},
```
这段代码创建了一个包含CVSSv2和CVSSv3评分的VulnerabilityMetadata对象，并设置了相应的评分和向量。
# 设置 CVE 的 ID
ID:           "my-cve",
# 设置记录来源
RecordSource: "record-source",
# 设置严重程度
Severity:     "pretty bad",
# 设置链接列表
Links:        []string{"https://ancho.re"},
# 设置描述
Description:  "best description ever",
# 设置 CVE 的 CVSS V2 评分
CvssV2: &v2.Cvss{
    BaseScore:           4.1,
    ExploitabilityScore: 5.2,
    ImpactScore:         6.3,
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
},
# 设置 CVE 的 CVSS V3 评分
CvssV3: &v2.Cvss{
    BaseScore:           1.4,
    ExploitabilityScore: 2.5,
    ImpactScore:         3.6,
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 设置下一个 CVE 的 ID
ID:           "my-cve",
# 设置记录来源为"record-source"
RecordSource: "record-source",
# 设置严重程度为"pretty bad"
Severity:     "pretty bad",
# 设置链接列表为["https://ancho.re"]
Links:        []string{"https://ancho.re"},
# 设置描述为"best description ever"
Description:  "best description ever",
# 设置CvssV2对象的属性
CvssV2: &v2.Cvss{
    # 设置基础分数为4.1
    BaseScore:           4.1,
    # 设置可利用性分数为5.2
    ExploitabilityScore: 5.2,
    # 设置影响分数为6.3
    ImpactScore:         6.3,
    # 设置向量为"AV:P--VERY"
    Vector:              "AV:P--VERY",
},
# 设置CvssV3对象的属性
CvssV3: &v2.Cvss{
    # 设置基础分数为1.4
    BaseScore:           1.4,
    # 设置可利用性分数为2.5
    ExploitabilityScore: 2.5,
    # 设置影响分数为3.6
    ImpactScore:         3.6,
    # 设置向量为"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 定义名称为 "mismatch-cvss3" 的错误，标记为错误
name: "mismatch-cvss3",
err:  true,
# 添加一个 VulnerabilityMetadata 对象到 add 列表中
add: []v2.VulnerabilityMetadata{
    {
        ID:           "my-cve",
        RecordSource: "record-source",
        Severity:     "pretty bad",
        Links:        []string{"https://ancho.re"},
        Description:  "best description ever",
        # 创建一个 Cvss 对象，包含 CVSS v2 的相关信息
        CvssV2: &v2.Cvss{
            BaseScore:           4.1,
            ExploitabilityScore: 5.2,
            ImpactScore:         6.3,
            Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
        },
        # 创建一个 Cvss 对象，包含 CVSS v3 的相关信息
        CvssV3: &v2.Cvss{
            BaseScore:           1.4,
            ExploitabilityScore: 2.5,
            ImpactScore:         3.6,
            Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
				},
				},  // 结束第二个嵌套的 map 对象
				{   // 开始新的 map 对象
					ID:           "my-cve",  // 设置 ID 字段值为 "my-cve"
					RecordSource: "record-source",  // 设置 RecordSource 字段值为 "record-source"
					Severity:     "pretty bad",  // 设置 Severity 字段值为 "pretty bad"
					Links:        []string{"https://ancho.re"},  // 设置 Links 字段值为包含一个字符串的数组
					Description:  "best description ever",  // 设置 Description 字段值为 "best description ever"
					CvssV2: &v2.Cvss{  // 设置 CvssV2 字段值为一个指向 v2.Cvss 结构体的指针
						BaseScore:           4.1,  // 设置 BaseScore 字段值为 4.1
						ExploitabilityScore: 5.2,  // 设置 ExploitabilityScore 字段值为 5.2
						ImpactScore:         6.3,  // 设置 ImpactScore 字段值为 6.3
						Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  // 设置 Vector 字段值为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY"
					},
					CvssV3: &v2.Cvss{  // 设置 CvssV3 字段值为一个指向 v2.Cvss 结构体的指针
						BaseScore:           1.4,  // 设置 BaseScore 字段值为 1.4
						ExploitabilityScore: 0,  // 设置 ExploitabilityScore 字段值为 0
						ImpactScore:         3.6,  // 设置 ImpactScore 字段值为 3.6
						Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  // 设置 Vector 字段值为 "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
					},
		},
	},
}

for _, test := range tests {
	// 对每个测试用例进行测试
	t.Run(test.name, func(t *testing.T) {
		// 在临时目录创建数据库
		dbTempDir := t.TempDir()

		// 创建新的存储对象
		s, err := New(dbTempDir, true)
		if err != nil {
			// 如果创建失败，输出错误信息
			t.Fatalf("could not create store: %+v", err)
		}

		// 依次添加每个元数据
		var theErr error
		for _, metadata := range test.add {
			// 添加漏洞元数据
			err = s.AddVulnerabilityMetadata(metadata)
			if err != nil {
				// 如果添加失败，记录错误信息
				theErr = err
			// 如果测试出现错误并且没有预期的错误
			if test.err && theErr == nil {
				t.Fatalf("expected error but did not get one")
			// 如果没有预期的错误但是出现了错误
			} else if !test.err && theErr != nil {
				t.Fatalf("expected no error but got one: %+v", theErr)
			// 如果预期有错误并且出现了错误
			} else if test.err && theErr != nil {
				// 测试通过...
				return
			}

			// 确保只有一个条目
			// 查找所有的条目并存储在allEntries中
			var allEntries []model.VulnerabilityMetadataModel
			s.(*store).db.Find(&allEntries)
			// 如果条目数量不等于1，则测试失败
			if len(allEntries) != 1 {
				t.Fatalf("unexpected number of entries: %d", len(allEntries))
			}
// 获取最终的元数据对象
if actual, err := s.GetVulnerabilityMetadata(test.expected.ID, test.expected.RecordSource); err != nil {
    // 如果获取元数据失败，输出错误信息
    t.Fatalf("failed to get metadata: %+v", err)
} else {
    // 比较预期的元数据和实际获取的元数据
    diffs := deep.Equal(&test.expected, actual)
    // 如果有差异，输出差异信息
    if len(diffs) > 0 {
        for _, d := range diffs {
            t.Errorf("Diff: %+v", d)
        }
    }
}
```