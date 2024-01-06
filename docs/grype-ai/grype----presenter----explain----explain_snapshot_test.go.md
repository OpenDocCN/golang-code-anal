# `grype\grype\presenter\explain\explain_snapshot_test.go`

```
package explain_test

import (
	"bytes"  // 导入 bytes 包，用于操作字节
	"encoding/json"  // 导入 json 包，用于处理 JSON 数据
	"os"  // 导入 os 包，用于操作操作系统功能
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/gkampitakis/go-snaps/snaps"  // 导入第三方包 go-snaps/snaps
	"github.com/stretchr/testify/require"  // 导入第三方包 testify/require

	"github.com/anchore/grype/grype/presenter/explain"  // 导入 anchore/grype/grype/presenter/explain 包
	"github.com/anchore/grype/grype/presenter/models"  // 导入 anchore/grype/grype/presenter/models 包
)

func TestExplainSnapshot(t *testing.T) {
	// load sample json
	testCases := []struct {
		name             string  // 定义结构体字段 name，表示测试用例名称
		fixture          string  // 定义结构体字段 fixture，表示测试用例的样本数据文件名
		# 定义一个包含漏洞ID的字符串切片
		vulnerabilityIDs []string
	}{
		# 第一个测试数据
		{
			# 测试名称
			name:             "keycloak-CVE-2020-12413",
			# 测试数据文件路径
			fixture:          "./test-fixtures/keycloak-test.json",
			# 漏洞ID列表
			vulnerabilityIDs: []string{"CVE-2020-12413"},
		},
		# 第二个测试数据
		{
			# 测试名称
			name:             "chainguard-ruby-CVE-2023-28755",
			# 测试数据文件路径
			fixture:          "test-fixtures/chainguard-ruby-test.json",
			# 漏洞ID列表
			vulnerabilityIDs: []string{"CVE-2023-28755"},
		},
		# 第三个测试数据
		{
			# 测试名称
			name: "test a GHSA",
			# 测试数据文件创建方式的注释
			/*
				fixture created by:
				Saving output of
				grype anchore/test_images@sha256:10008791acbc5866de04108746a02a0c4029ce3a4400a9b3dad45d7f2245f9da -o json
				Then filtering matches to relevant ones:
				jq -c '.matches[]' | rg -e GHSA-cfh5-3ghh-wfjx -e CVE-2014-3577 | jq -s .
			*/
		},
		*/
		// 定义测试用例
		testCases := []struct {
			// 测试用例名称
			name:             string
			// 测试用例的数据文件路径
			fixture:          "test-fixtures/ghsa-test.json",
			// 漏洞ID列表
			vulnerabilityIDs: []string{"GHSA-cfh5-3ghh-wfjx"},
		},
		{
			// 测试用例名称
			name:             "test a CVE alias of a GHSA",
			// 测试用例的数据文件路径
			fixture:          "test-fixtures/ghsa-test.json",
			// 漏洞ID列表
			vulnerabilityIDs: []string{"CVE-2014-3577"},
		}
	}
	// 遍历测试用例
	for _, tc := range testCases {
		// 在测试中运行每个测试用例
		t.Run(tc.name, func(t *testing.T) {
			// 打开测试数据文件
			r, err := os.Open(tc.fixture)
			// 确保没有错误发生
			require.NoError(t, err)

			// 解析为 models.Document 对象
			doc := models.Document{}
			decoder := json.NewDecoder(r)
			err = decoder.Decode(&doc)
			// 确保没有错误发生
			require.NoError(t, err)
// 创建一个新的字节缓冲区，用于存储解释的结果
w := bytes.NewBufferString("")
// 创建一个漏洞解释器，将解释结果写入到字节缓冲区中
explainer := explain.NewVulnerabilityExplainer(w, &doc)
// 调用ExplainByID方法，根据给定的漏洞ID解释漏洞
err = explainer.ExplainByID(tc.vulnerabilityIDs)
// 断言结果是否没有错误
require.NoError(t, err)
// 断言输出结果是否符合预期
snaps.MatchSnapshot(t, w.String())
```