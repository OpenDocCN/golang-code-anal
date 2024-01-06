# `grype\test\cli\cmd_test.go`

```
package cli
// 导入所需的包

import (
	"encoding/json"
	"path/filepath"
	"strings"
	"testing"

	"github.com/stretchr/testify/require"
)
// 导入所需的包

func TestCmd(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		name       string
		args       []string
		env        map[string]string
		assertions []traitAssertion
	}{
		{
			name: "no-args-shows-help",
```
在这段代码中，我们首先导入了所需的包，然后定义了一个测试用例。
// 定义一个空字符串数组作为参数
args: []string{},
// 定义一个traitAssertion数组，用于存储断言
assertions: []traitAssertion{
    // 断言输出中应该包含特定的错误信息
    assertInOutput("an image/directory argument is required"),
    // 断言输出中应该包含特定的帮助描述信息
    assertInOutput("A vulnerability scanner for container images, filesystems, and SBOMs"),
    // 断言返回代码应该是失败的
    assertFailingReturnCode,
},
// 定义一个名为"ensure valid descriptor"的测试用例
{
    name: "ensure valid descriptor",
    // 传入参数为指定的图片和输出格式
    args: []string{getFixtureImage(t, "image-bare"), "-o", "json"},
    // 定义一个traitAssertion数组，用于存储断言
    assertions: []traitAssertion{
        // 断言输出中应该包含特定的app配置块
        assertInOutput(`"check-for-app-update": false`),
        // 断言输出中应该包含特定的db状态块
        assertInOutput(`"db": {`),
        // 断言输出中应该包含特定的db状态块
        assertInOutput(`"built":`),
    },
},
// 定义一个名为"platform-option-wired-up"的测试用例
{
    name: "platform-option-wired-up",
    // 传入参数为指定的平台、输出格式和镜像
    args: []string{"--platform", "arm64", "-o", "json", "registry:busybox:1.31"},
    // 定义一个traitAssertion数组，用于存储断言
    assertions: []traitAssertion{
// 断言输出中包含指定的 linux/arm64 镜像摘要
assertInOutput("sha256:1ee006886991ad4689838d3a288e0dd3fd29b70e276622f16b67a8922831a853"), // linux/arm64 image digest

// 测试应用程序对搜索选项的响应
name: "responds-to-search-options",
args: []string{"--help"},
env: map[string]string{
    "GRYPE_SEARCH_UNINDEXED_ARCHIVES": "true",
    "GRYPE_SEARCH_INDEXED_ARCHIVES":   "false",
    "GRYPE_SEARCH_SCOPE":              "all-layers",
},
assertions: []traitAssertion{
    // 日志中的应用程序配置与我们期望配置的匹配。注意：我们不测试此选项的进一步连接，只测试配置是否响应包目录级别的选项。
    assertInOutput("unindexed-archives: true"),
    assertInOutput("indexed-archives: false"),
    assertInOutput("scope: 'all-layers'"),
},
# 定义测试用例：在失败的情况下，检查输出中的漏洞
{
    name: "vulnerabilities in output on -f with failure",
    # 传入的参数：镜像名称、选项、平台
    args: []string{"registry:busybox:1.31", "-f", "high", "--platform", "linux/amd64"},
    # 断言：检查输出中是否包含指定的漏洞，同时检查返回码是否为失败状态
    assertions: []traitAssertion{
        assertInOutput("CVE-2021-42379"),
        assertFailingReturnCode,
    },
},
# 定义测试用例：忽略漏洞的原因是否在模板中可用
{
    name: "reason for ignored vulnerabilities is available in the template",
    args: []string{
        # 传入的参数：SBOM 文件路径、配置文件路径、输出格式、模板路径
        "sbom:" + filepath.Join("test-fixtures", "test-ignore-reason", "sbom.json"),
        "-c", filepath.Join("test-fixtures", "test-ignore-reason", "config-with-ignore.yaml"),
        "-o", "template",
        "-t", filepath.Join("test-fixtures", "test-ignore-reason", "template-with-ignore-reasons"),
    },
    # 断言：检查输出中是否包含指定的被忽略的漏洞原因，同时检查返回码是否为成功状态
    assertions: []traitAssertion{
        assertInOutput("CVE-2021-42385 (test reason for vulnerability being ignored)"),
        assertSucceedingReturnCode,
    },
},
		},
		{
			// 测试用例名称：ignore-states wired up
			name: "ignore-states wired up",
			// 测试参数：指定输入文件和忽略状态为unknown
			args: []string{"./test-fixtures/sbom-grype-source.json", "--ignore-states", "unknown"},
			// 断言：验证返回码为成功，标准输出中包含指定内容，不包含指定内容
			assertions: []traitAssertion{
				assertSucceedingReturnCode,
				assertRowInStdOut([]string{"Pygments", "2.6.1", "2.7.4", "python", "GHSA-pq64-v7f5-gqh8", "High"}),
				assertNotInOutput("CVE-2014-6052"),
			},
		},
		{
			// 测试用例名称：ignore-states wired up - ignore fixed
			name: "ignore-states wired up - ignore fixed",
			// 测试参数：指定输入文件和忽略状态为fixed
			args: []string{"./test-fixtures/sbom-grype-source.json", "--ignore-states", "fixed"},
			// 断言：验证返回码为成功，标准输出中包含指定内容，不包含指定内容
			assertions: []traitAssertion{
				assertSucceedingReturnCode,
				assertRowInStdOut([]string{"libvncserver", "0.9.9", "apk", "CVE-2014-6052", "High"}),
				assertNotInOutput("GHSA-pq64-v7f5-gqh8"),
			},
		},
		{
// 定义测试用例，包括名称、参数和断言
name: "ignore-states wired up - ignore fixed, show suppressed",
args: []string{"./test-fixtures/sbom-grype-source.json", "--ignore-states", "fixed", "--show-suppressed"},
assertions: []traitAssertion{
    assertSucceedingReturnCode, // 断言命令执行成功
    assertRowInStdOut([]string{"Pygments", "2.6.1", "2.7.4", "python", "GHSA-pq64-v7f5-gqh8", "High", "(suppressed)"}), // 断言标准输出中包含指定内容
},

// 遍历测试用例，执行每个测试
for _, test := range tests {
    t.Run(test.name, func(t *testing.T) {
        // 运行 grype 命令，获取标准输出、标准错误和命令状态
        cmd, stdout, stderr := runGrype(t, test.env, test.args...)
        // 遍历断言函数列表，对标准输出、标准错误和命令状态进行断言
        for _, traitFn := range test.assertions {
            traitFn(t, stdout, stderr, cmd.ProcessState.ExitCode())
        }
        // 如果测试失败，输出标准输出、标准错误和命令
        if t.Failed() {
            t.Log("STDOUT:\n", stdout)
            t.Log("STDERR:\n", stderr)
            t.Log("COMMAND:", strings.Join(cmd.Args, " "))
        }
    }
}
// 定义测试函数，测试描述符名称和版本是否设置正确
func Test_descriptorNameAndVersionSet(t *testing.T) {
    // 运行 Grype，获取输出结果
    _, output, _ := runGrype(t, nil, "-o", "json", getFixtureImage(t, "image-bare"))

    // 解析 JSON 格式的输出结果
    parsed := map[string]any{}
    err := json.Unmarshal([]byte(output), &parsed)
    require.NoError(t, err)

    // 获取描述符信息
    desc, _ := parsed["descriptor"].(map[string]any)
    require.NotNil(t, desc)

    // 获取描述符的名称
    name := desc["name"]
    require.Equal(t, "grype", name)

    // 获取描述符的版本
    version := desc["version"]
    require.NotEmpty(t, version)
}
抱歉，我无法为您提供代码注释，因为您没有提供任何代码。如果您有任何需要帮助的代码，请随时告诉我。我会尽力帮助您。
```