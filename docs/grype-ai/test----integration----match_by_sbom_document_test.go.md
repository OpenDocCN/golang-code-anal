# `grype\test\integration\match_by_sbom_document_test.go`

```
package integration
// 导入所需的包

import (
	"fmt"
	"testing"

	"github.com/google/go-cmp/cmp" // 导入用于比较的包
	"github.com/google/go-cmp/cmp/cmpopts" // 导入用于比较选项的包
	"github.com/scylladb/go-set/strset" // 导入用于字符串集合的包
	"github.com/stretchr/testify/assert" // 导入用于断言的包
	"github.com/stretchr/testify/require" // 导入用于要求的包

	"github.com/anchore/grype/grype" // 导入 grype 包
	"github.com/anchore/grype/grype/db" // 导入 grype 数据库包
	"github.com/anchore/grype/grype/match" // 导入 grype 匹配包
	"github.com/anchore/grype/grype/pkg" // 导入 grype 包
	"github.com/anchore/grype/grype/store" // 导入 grype 存储包
	"github.com/anchore/syft/syft/source" // 导入 syft 源包
)
# 定义测试函数 TestMatchBySBOMDocument，用于测试根据 SBOM 文档进行匹配
func TestMatchBySBOMDocument(t *testing.T) {
    # 定义测试用例
	tests := []struct {
		name            string           # 测试用例名称
		fixture         string           # 测试用例的数据文件路径
		expectedIDs     []string         # 期望的匹配结果 ID 列表
		expectedDetails []match.Detail   # 期望的匹配结果详情列表
	}{
		{
			name:        "single KB package",   # 测试用例名称
			fixture:     "test-fixtures/sbom/syft-sbom-with-kb-packages.json",   # 测试用例的数据文件路径
			expectedIDs: []string{"CVE-2016-3333"},   # 期望的匹配结果 ID 列表
			expectedDetails: []match.Detail{   # 期望的匹配结果详情列表
				{
					Type: match.ExactDirectMatch,   # 匹配类型为精确直接匹配
					SearchedBy: map[string]interface{   # 匹配时使用的搜索条件
						"distro": map[string]string{   # 操作系统信息
							"type":    "windows",   # 操作系统类型为 Windows
							"version": "10816",   # 操作系统版本为 10816
						},
						"namespace": "msrc:distro:windows:10816",   # 命名空间为 msrc:distro:windows:10816
// 创建一个包含名称和版本的映射
"package": map[string]string{
    "name":    "10816",
    "version": "3200970",
},
// 创建一个包含版本约束和漏洞ID的映射
Found: map[string]interface{}{
    "versionConstraint": "3200970 || 878787 || base (kb)",
    "vulnerabilityID":   "CVE-2016-3333",
},
// 设置匹配器为 MsrcMatcher
Matcher:    match.MsrcMatcher,
// 设置置信度为 1
Confidence: 1,
```
``` 
// 创建一个未知包类型的测试用例
name:        "unknown package type",
// 设置测试用例的数据文件路径
fixture:     "test-fixtures/sbom/syft-sbom-with-unknown-packages.json",
// 期望的漏洞ID列表
expectedIDs: []string{"CVE-bogus-my-package-2-idris"},
// 期望的匹配细节列表
expectedDetails: []match.Detail{
# 定义一个类型为 ExactDirectMatch 的匹配对象
Type: match.ExactDirectMatch,

# 根据语言和命名空间进行搜索
SearchedBy: map[string]interface{}{
    "language":  "idris",
    "namespace": "github:language:idris",
    "package":   map[string]string{"name": "my-package", "version": "1.0.5"},
},

# 找到的匹配结果，包括版本约束和漏洞ID
Found: map[string]interface{}{
    "versionConstraint": "< 2.0 (unknown)",
    "vulnerabilityID":   "CVE-bogus-my-package-2-idris",
},

# 使用 StockMatcher 进行匹配
Matcher:    match.StockMatcher,

# 置信度为 1
Confidence: 1,
```

```
# 遍历测试用例
for _, test := range tests {
    # 对每个测试用例运行测试
    t.Run(test.name, func(t *testing.T) {
        # 创建一个模拟的数据库存储对象
        mkStr := newMockDbStore()
// 使用给定的漏洞提供者名称创建漏洞提供者对象
vp, err := db.NewVulnerabilityProvider(mkStr)
// 确保没有错误发生
require.NoError(t, err)
// 使用给定的漏洞元数据提供者名称创建漏洞元数据提供者对象
mp := db.NewVulnerabilityMetadataProvider(mkStr)
// 使用给定的匹配排除提供者名称创建匹配排除提供者对象
ep := db.NewMatchExclusionProvider(mkStr)
// 创建存储对象，包括漏洞提供者、元数据提供者和排除提供者
str := store.Store{
    Provider:          vp,
    MetadataProvider:  mp,
    ExclusionProvider: ep,
}
// 在给定的存储对象中查找漏洞匹配
matches, _, _, err := grype.FindVulnerabilities(str, fmt.Sprintf("sbom:%s", test.fixture), source.SquashedScope, nil)
// 确保没有错误发生
assert.NoError(t, err)
// 创建一个空的漏洞详情列表
details := make([]match.Detail, 0)
// 创建一个新的字符串集合
ids := strset.New()
// 遍历漏洞匹配结果，将详情添加到详情列表中，将漏洞ID添加到ID集合中
for _, m := range matches.Sorted() {
    details = append(details, m.Details...)
    ids.Add(m.Vulnerability.ID)
}
// 确保详情列表的长度与预期的详情长度相同
require.Len(t, details, len(test.expectedDetails))
# 创建比较选项列表，忽略 pkg.Package 结构体中的 "Locations" 字段
cmpOpts := []cmp.Option{
    cmpopts.IgnoreFields(pkg.Package{}, "Locations"),
}

# 遍历预期的详细信息列表，比较预期的详细信息和实际的详细信息，使用比较选项列表进行比较
for i := range test.expectedDetails {
    if d := cmp.Diff(test.expectedDetails[i], details[i], cmpOpts...); d != "" {
        # 如果有差异，输出差异信息
        t.Errorf("unexpected match details (-want +got):\n%s", d)
    }
}

# 断言实际的 ID 列表与预期的 ID 列表相匹配
assert.ElementsMatch(t, test.expectedIDs, ids.List())
```