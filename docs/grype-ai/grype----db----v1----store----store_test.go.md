# `grype\grype\db\v1\store\store_test.go`

```
package store

import (
	"testing"  // 导入测试包
	"time"  // 导入时间包

	"github.com/go-test/deep"  // 导入深度比较包

	v1 "github.com/anchore/grype/grype/db/v1"  // 导入版本1的数据库包
	"github.com/anchore/grype/grype/db/v1/store/model"  // 导入模型包
)

func assertIDReader(t *testing.T, reader v1.IDReader, expected v1.ID) {  // 定义一个断言函数，用于测试IDReader
	t.Helper()  // 标记该函数是测试辅助函数
	if actual, err := reader.GetID(); err != nil {  // 获取实际的ID值
		t.Fatalf("failed to get ID: %+v", err)  // 如果获取失败，则输出错误信息
	} else {
		diffs := deep.Equal(&expected, actual)  // 比较预期值和实际值
		if len(diffs) > 0 {  // 如果有差异
			for _, d := range diffs {  // 遍历差异
		// 如果测试失败，输出差异信息
		t.Errorf("Diff: %+v", d)
	}

	// 测试存储器的 GetID 和 SetID 方法
func TestStore_GetID_SetID(t *testing.T) {
	// 创建临时文件夹作为数据库存储路径
	dbTempFile := t.TempDir()

	// 创建一个新的存储器实例
	s, err := New(dbTempFile, true)
	if err != nil {
		// 如果创建失败，输出错误信息
		t.Fatalf("could not create store: %+v", err)
	}

	// 创建预期的 ID 对象
	expected := v1.ID{
		BuildTimestamp: time.Now().UTC(),
		SchemaVersion:  2,
	}

	// 将预期的 ID 对象存储到存储器中
	if err = s.SetID(expected); err != nil {
		// 如果设置 ID 失败，输出错误信息
		t.Fatalf("failed to set ID: %+v", err)
	}

	// 断言 ID 读取器的行为是否符合预期
	assertIDReader(t, s, expected)

}

// 断言漏洞读取器的行为是否符合预期
func assertVulnerabilityReader(t *testing.T, reader v1.VulnerabilityStoreReader, namespace, name string, expected []v1.Vulnerability) {
	// 如果获取漏洞失败，输出错误信息
	if actual, err := reader.GetVulnerability(namespace, name); err != nil {
		t.Fatalf("failed to get Vulnerability: %+v", err)
	} else {
		// 如果实际漏洞数量与预期不符，输出错误信息
		if len(actual) != len(expected) {
			t.Fatalf("unexpected number of vulns: %d", len(actual))
		}

		// 遍历比较每个漏洞的差异
		for idx := range actual {
			// 比较预期漏洞和实际漏洞的差异
			diffs := deep.Equal(expected[idx], actual[idx])
			// 如果存在差异，输出差异信息
			if len(diffs) > 0 {
				for _, d := range diffs {
					t.Errorf("Diff: %+v", d)
// 定义测试函数，用于测试 GetVulnerability 和 SetVulnerability 方法
func TestStore_GetVulnerability_SetVulnerability(t *testing.T) {
    // 创建临时文件夹作为数据库存储路径
    dbTempFile := t.TempDir()
    // 创建新的存储对象
    s, err := New(dbTempFile, true)
    // 如果创建存储对象出错，则打印错误信息并终止测试
    if err != nil {
        t.Fatalf("could not create store: %+v", err)
    }

    // 创建一个漏洞对象数组
    extra := []v1.Vulnerability{
        {
            ID:                   "my-cve-33333",
            RecordSource:         "record-source",
            PackageName:          "package-name-2",
            Namespace:            "my-namespace",
            VersionConstraint:    "< 1.0",
            // 其他漏洞属性...
        },
        // 其他漏洞对象...
    }
		// 设置版本格式为语义化版本
		VersionFormat:        "semver",
		// 设置CPEs为空字符串数组
		CPEs:                 []string{"a-cool-cpe"},
		// 设置代理漏洞为另外两个CVE
		ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
		// 设置修复版本为"2.0.1"
		FixedInVersion:       "2.0.1",
	},
	{
		// 设置CVE ID为"my-other-cve-33333"
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
		// 设置CPEs为"a-cool-cpe"字符串数组
		CPEs:                 []string{"a-cool-cpe"},
		// 设置代理漏洞为另外两个CVE
		ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},
	},
}

// 设置期望的漏洞列表
expected := []v1.Vulnerability{
	{
		// 设置CVE ID为"my-cve"
		ID:                   "my-cve",
			RecordSource:         "record-source",  // 记录来源
			PackageName:          "package-name",   // 包名
			Namespace:            "my-namespace",   // 命名空间
			VersionConstraint:    "< 1.0",          // 版本约束
			VersionFormat:        "semver",         // 版本格式
			CPEs:                 []string{"a-cool-cpe"},  // CPE（通用平台漏洞披露）条目
			ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},  // 代理漏洞
			FixedInVersion:       "1.0.1",          // 修复版本
		},
		{
			ID:                   "my-other-cve",   // 漏洞ID
			RecordSource:         "record-source",  // 记录来源
			PackageName:          "package-name",   // 包名
			Namespace:            "my-namespace",   // 命名空间
			VersionConstraint:    "< 509.2.2",      // 版本约束
			VersionFormat:        "semver",         // 版本格式
			CPEs:                 []string{"a-cool-cpe"},  // CPE（通用平台漏洞披露）条目
			ProxyVulnerabilities: []string{"another-cve", "an-other-cve"},  // 代理漏洞
			FixedInVersion:       "4.0.5",          // 修复版本
		},
```

	}

	// 将 expected 和 extra 合并成一个新的切片 total
	total := append(expected, extra...)

	// 将合并后的切片作为参数传递给 AddVulnerability 方法，并检查是否有错误
	if err = s.AddVulnerability(total...); err != nil {
		t.Fatalf("failed to set Vulnerability: %+v", err)
	}

	// 创建一个空的 VulnerabilityModel 切片 allEntries，并从数据库中查找所有的记录
	var allEntries []model.VulnerabilityModel
	s.(*store).db.Find(&allEntries)

	// 检查数据库中的记录数量是否与合并后的切片 total 的长度相等
	if len(allEntries) != len(total) {
		t.Fatalf("unexpected number of entries: %d", len(allEntries))
	}

	// 调用 assertVulnerabilityReader 方法，检查读取的结果是否符合预期
	assertVulnerabilityReader(t, s, expected[0].Namespace, expected[0].PackageName, expected)

}

// 定义一个函数，用于断言读取的漏洞元数据是否符合预期
func assertVulnerabilityMetadataReader(t *testing.T, reader v1.VulnerabilityMetadataStoreReader, id, recordSource string, expected v1.VulnerabilityMetadata) {
	// 调用 GetVulnerabilityMetadata 方法，获取指定 id 和 recordSource 的漏洞元数据，并检查是否有错误
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
total := []v1.VulnerabilityMetadata{
    // 添加漏洞的ID
    ID:           "my-cve",
    // 添加记录来源
    RecordSource: "record-source",
    // 添加漏洞的严重程度
    Severity:     "pretty bad",
    // 添加漏洞相关链接
    Links:        []string{"https://ancho.re"},
    // 添加漏洞描述
    Description:  "best description ever",
    // 创建并添加CVSS V2对象
    CvssV2: &v1.Cvss{
        // 添加基础分数
        BaseScore:           1.1,
        // 添加可利用性分数
        ExploitabilityScore: 2.2,
        // 添加影响分数
        ImpactScore:         3.3,
        // 添加向量
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--NOT",
    },
    // 创建并添加CVSS V3对象
    CvssV3: &v1.Cvss{
        // 添加基础分数
        BaseScore:           1.3,
        // 添加可利用性分数
        ExploitabilityScore: 2.1,
        // 添加影响分数
        ImpactScore:         3.2,
        // 添加向量
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--NICE",
    },
    // 继续添加其他漏洞元数据...
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
			CvssV2: &v1.Cvss{
				BaseScore:           4.1,
				ExploitabilityScore: 5.2,
				ImpactScore:         6.3,
				Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
			},
			// CVE的CVSS V3评分
			CvssV3: &v1.Cvss{
				BaseScore:           1.4,
				ExploitabilityScore: 2.5,
				ImpactScore:         3.6,
				Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
			},
		},
	}

	// 将漏洞元数据添加到存储中
	if err = s.AddVulnerabilityMetadata(total...); err != nil {
		t.Fatalf("failed to set metadata: %+v", err)
	}

	// 获取所有漏洞元数据条目
	var allEntries []model.VulnerabilityMetadataModel
	s.(*store).db.Find(&allEntries)
	// 检查获取的条目数量是否与期望的数量相同
	if len(allEntries) != len(total) {
		t.Fatalf("unexpected number of entries: %d", len(allEntries))
	}

}

// 测试合并漏洞元数据
func TestStore_MergeVulnerabilityMetadata(t *testing.T) {
	tests := []struct {
		name     string
		add      []v1.VulnerabilityMetadata
		expected v1.VulnerabilityMetadata
		err      bool
# 创建一个包含漏洞元数据的结构体切片
{
    name: "go-case",  # 漏洞名称
    add: []v1.VulnerabilityMetadata{  # 添加漏洞元数据
        {
            ID:           "my-cve",  # 漏洞ID
            RecordSource: "record-source",  # 记录来源
            Severity:     "pretty bad",  # 严重程度
            Links:        []string{"https://ancho.re"},  # 相关链接
            Description:  "worst description ever",  # 描述
            CvssV2: &v1.Cvss{  # CVSS V2评分
                BaseScore:           4.1,  # 基础分数
                ExploitabilityScore: 5.2,  # 可利用性分数
                ImpactScore:         6.3,  # 影响分数
                Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量
            },
            CvssV3: &v1.Cvss{  # CVSS V3评分
                BaseScore:           1.4,  # 基础分数
                ExploitabilityScore: 2.5,  # 可利用性分数
                ImpactScore:         3.6,  # 影响分数
                # 其他CVSS V3评分字段
            }
        }
    }
}
# 定义一个预期的漏洞元数据结构
expected: v1.VulnerabilityMetadata{
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
    CvssV2: &v1.Cvss{
        BaseScore: 4.1,
        ExploitabilityScore: 5.2,
        ImpactScore: 6.3,
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
    },
    # 指定CVSS V3评分
    CvssV3: &v1.Cvss{
        BaseScore: 1.4,
        ExploitabilityScore: 2.5,
        ImpactScore: 3.6,
        Vector: "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
    },
},
# 创建一个名为"merge-links"的漏洞元数据
{
    name: "merge-links",
    add: []v1.VulnerabilityMetadata{
        # 添加一个名为"my-cve"的漏洞元数据
        {
            ID: "my-cve",
            RecordSource: "record-source",
            Severity: "pretty bad",
            Links: []string{"https://ancho.re"},
        },
        # 添加另一个名为"my-cve"的漏洞元数据
        {
            ID: "my-cve",
            RecordSource: "record-source",
            Severity: "pretty bad",
            Links: []string{"https://google.com"},
        },
        {
            # 可以继续添加其他漏洞元数据
        },
    },
},
# 定义一个ID为"my-cve"的漏洞记录
ID:           "my-cve",
# 指定记录来源为"record-source"
RecordSource: "record-source",
# 指定漏洞严重程度为"pretty bad"
Severity:     "pretty bad",
# 指定漏洞相关链接为["https://yahoo.com"]
Links:        []string{"https://yahoo.com"},
# 定义一个测试用例，期望的漏洞记录元数据
expected: v1.VulnerabilityMetadata{
    # 期望的漏洞记录ID为"my-cve"
    ID:           "my-cve",
    # 期望的漏洞记录来源为"record-source"
    RecordSource: "record-source",
    # 期望的漏洞严重程度为"pretty bad"
    Severity:     "pretty bad",
    # 期望的漏洞相关链接为["https://ancho.re", "https://google.com", "https://yahoo.com"]
    Links:        []string{"https://ancho.re", "https://google.com", "https://yahoo.com"},
},
# 定义另一个测试用例，漏洞严重程度为"pretty bad"
name: "bad-severity",
add: []v1.VulnerabilityMetadata{
    {
        # 漏洞记录ID为"my-cve"
        ID:           "my-cve",
        # 漏洞记录来源为"record-source"
        RecordSource: "record-source",
        # 漏洞严重程度为"pretty bad"
        Severity:     "pretty bad",
// 创建一个包含漏洞元数据的切片，每个元数据包含漏洞的ID、记录来源、严重程度和链接
// 第一个元数据
{
    ID:           "my-cve", // 漏洞的ID
    RecordSource: "record-source", // 记录来源
    Severity:     "meh, push that for next tuesday...", // 严重程度
    Links:        []string{"https://redhat.com"}, // 链接
},
// 第二个元数据
{
    ID:           "my-cve", // 漏洞的ID
    RecordSource: "record-source", // 记录来源
    Severity:     "pretty bad", // 严重程度
    Links:        []string{"https://ancho.re"}, // 链接
},
// 设置错误标志为true
err: true,
// 设置名称为"mismatch-description"的错误标志为true
{
    name: "mismatch-description",
    err:  true,
    // 添加一个包含漏洞元数据的切片
    add: []v1.VulnerabilityMetadata{
        {
            ID:           "my-cve", // 漏洞的ID
            RecordSource: "record-source", // 记录来源
            Severity:     "pretty bad", // 严重程度
            Links:        []string{"https://ancho.re"}, // 链接
# 创建一个包含漏洞信息的数据结构
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 漏洞信息的来源
    Severity:     "pretty bad",  # 漏洞的严重程度
    Links:        []string{"https://ancho.re"},  # 相关链接
    Description:  "worst description ever",  # 漏洞描述
    CvssV2: &v1.Cvss{  # CVE 的 CVSS V2 评分
        BaseScore:           4.1,  # 基础分数
        ExploitabilityScore: 5.2,  # 可利用性分数
        ImpactScore:         6.3,  # 影响分数
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量描述
    },
    CvssV3: &v1.Cvss{  # CVE 的 CVSS V3 评分
        BaseScore:           1.4,  # 基础分数
        ExploitabilityScore: 2.5,  # 可利用性分数
        ImpactScore:         3.6,  # 影响分数
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 向量描述
    },
},
# 创建一个包含CVSSv2和CVSSv3评分的VulnerabilityMetadata对象
CvssV2: &v1.Cvss{
    BaseScore:           4.1,  # 设置CVSSv2基础评分
    ExploitabilityScore: 5.2,  # 设置CVSSv2可利用性评分
    ImpactScore:         6.3,  # 设置CVSSv2影响评分
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 设置CVSSv2向量
},
CvssV3: &v1.Cvss{
    BaseScore:           1.4,  # 设置CVSSv3基础评分
    ExploitabilityScore: 2.5,  # 设置CVSSv3可利用性评分
    ImpactScore:         3.6,  # 设置CVSSv3影响评分
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 设置CVSSv3向量
},
# 创建一个包含mismatch-cvss2名称和错误标志的VulnerabilityMetadata对象
name: "mismatch-cvss2",
err:  true,
# 添加一个包含CVSSv2和CVSSv3评分的VulnerabilityMetadata对象到列表中
add: []v1.VulnerabilityMetadata{
    {
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
# 设置 CVE 的 CVSS V2 信息
CvssV2: &v1.Cvss{
    # 设置基础分数
    BaseScore:           4.1,
    # 设置可利用性分数
    ExploitabilityScore: 5.2,
    # 设置影响分数
    ImpactScore:         6.3,
    # 设置向量
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",
},
# 设置 CVE 的 CVSS V3 信息
CvssV3: &v1.Cvss{
    # 设置基础分数
    BaseScore:           1.4,
    # 设置可利用性分数
    ExploitabilityScore: 2.5,
    # 设置影响分数
    ImpactScore:         3.6,
    # 设置向量
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
CvssV2: &v1.Cvss{
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
CvssV3: &v1.Cvss{
    # 设置基础分数为1.4
    BaseScore:           1.4,
    # 设置可利用性分数为2.5
    ExploitabilityScore: 2.5,
    # 设置影响分数为3.6
    ImpactScore:         3.6,
    # 设置向量为"AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD"
    Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",
},
# 定义名称为 "mismatch-cvss3" 的变量，值为 true，表示存在 CVSS3 不匹配的情况
name: "mismatch-cvss3",
err:  true,
# 定义名称为 "add" 的变量，值为一个空的 VulnerabilityMetadata 列表
add: []v1.VulnerabilityMetadata{
    # 定义一个 VulnerabilityMetadata 对象
    {
        ID:           "my-cve",  # 漏洞 ID
        RecordSource: "record-source",  # 记录来源
        Severity:     "pretty bad",  # 严重程度
        Links:        []string{"https://ancho.re"},  # 相关链接
        Description:  "best description ever",  # 描述
        # 定义一个 Cvss 对象，包含 CVSS2 的相关信息
        CvssV2: &v1.Cvss{
            BaseScore:           4.1,  # 基础分数
            ExploitabilityScore: 5.2,  # 可利用性分数
            ImpactScore:         6.3,  # 影响分数
            Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 向量
        },
        # 定义一个 Cvss 对象，包含 CVSS3 的相关信息
        CvssV3: &v1.Cvss{
            BaseScore:           1.4,  # 基础分数
            ExploitabilityScore: 2.5,  # 可利用性分数
            ImpactScore:         3.6,  # 影响分数
            Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 向量
        },
    },
# 创建一个包含漏洞信息的结构体
{
    ID:           "my-cve",  # 漏洞的唯一标识符
    RecordSource: "record-source",  # 漏洞信息的来源
    Severity:     "pretty bad",  # 漏洞的严重程度
    Links:        []string{"https://ancho.re"},  # 相关链接
    Description:  "best description ever",  # 漏洞的描述
    CvssV2: &v1.Cvss{  # 漏洞的 CVSS V2 评分
        BaseScore:           4.1,  # 基础评分
        ExploitabilityScore: 5.2,  # 可利用性评分
        ImpactScore:         6.3,  # 影响评分
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--VERY",  # 评分向量
    },
    CvssV3: &v1.Cvss{  # 漏洞的 CVSS V3 评分
        BaseScore:           1.4,  # 基础评分
        ExploitabilityScore: 0,  # 可利用性评分
        ImpactScore:         3.6,  # 影响评分
        Vector:              "AV:N/AC:L/Au:N/C:P/I:P/A:P--GOOD",  # 评分向量
    },
}
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
			// 如果测试出现错误，打印错误信息并终止测试
			if test.err && theErr == nil {
				t.Fatalf("expected error but did not get one")
			} else if !test.err && theErr != nil {
				// 如果不期望出现错误，但出现了错误，打印错误信息并终止测试
				t.Fatalf("expected no error but got one: %+v", theErr)
			} else if test.err && theErr != nil {
				// 如果期望出现错误，并且出现了错误，测试通过，直接返回
				// test pass...
				return
			}

			// 确保只有一个条目
			// 查询数据库中的所有条目
			var allEntries []model.VulnerabilityMetadataModel
			s.(*store).db.Find(&allEntries)
			// 如果条目数量不等于1，打印错误信息并终止测试
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