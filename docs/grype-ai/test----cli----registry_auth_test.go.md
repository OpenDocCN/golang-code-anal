# `grype\test\cli\registry_auth_test.go`

```
package cli

import (
	"strings"  // 导入字符串操作包
	"testing"  // 导入测试包
)

func TestRegistryAuth(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例结构
		name       string  // 测试用例名称
		args       []string  // 测试用例参数
		env        map[string]string  // 测试用例环境变量
		assertions []traitAssertion  // 测试用例断言
	}{
		{
			name: "fallback to keychain",  // 测试用例名称
			args: []string{"-vv", "registry:localhost:5000/something:latest"},  // 测试用例参数
			assertions: []traitAssertion{  // 测试用例断言
				assertInOutput("source=OciRegistry"),  // 断言输出中包含指定字符串
				assertInOutput("localhost:5000/something:latest"),  // 断言输出中包含指定字符串
# 断言输出中是否包含特定信息，用于测试
assertInOutput(`no registry credentials configured for "localhost:5000", using the default keychain`),
# 测试用例名称
name: "use creds",
# 测试参数
args: []string{"-vv", "registry:localhost:5000/something:latest"},
# 测试环境变量
env: map[string]string{
    "GRYPE_REGISTRY_AUTH_AUTHORITY": "localhost:5000",
    "GRYPE_REGISTRY_AUTH_USERNAME":  "username",
    "GRYPE_REGISTRY_AUTH_PASSWORD":  "password",
},
# 断言输出中是否包含特定信息，用于测试
assertions: []traitAssertion{
    assertInOutput("source=OciRegistry"),
    assertInOutput("localhost:5000/something:latest"),
    assertInOutput(`using basic auth for registry "localhost:5000"`),
},
# 测试用例名称
name: "use token",
# 测试参数
args: []string{"-vv", "registry:localhost:5000/something:latest"},
		{
			// 设置环境变量，指定 GRYPE_REGISTRY_AUTH_AUTHORITY 和 GRYPE_REGISTRY_AUTH_TOKEN
			env: map[string]string{
				"GRYPE_REGISTRY_AUTH_AUTHORITY": "localhost:5000",
				"GRYPE_REGISTRY_AUTH_TOKEN":     "my-token",
			},
			// 断言，检查输出中是否包含指定的内容
			assertions: []traitAssertion{
				assertInOutput("source=OciRegistry"),
				assertInOutput("localhost:5000/something:latest"),
				assertInOutput(`using token for registry "localhost:5000"`),
			},
		},
		{
			// 测试不提供足够信息时是否回退到密钥链
			name: "not enough info fallsback to keychain",
			// 设置命令行参数
			args: []string{"-vv", "registry:localhost:5000/something:latest"},
			// 设置环境变量，指定 GRYPE_REGISTRY_AUTH_AUTHORITY
			env: map[string]string{
				"GRYPE_REGISTRY_AUTH_AUTHORITY": "localhost:5000",
			},
			// 断言，检查输出中是否包含指定的内容
			assertions: []traitAssertion{
				assertInOutput("source=OciRegistry"),
				assertInOutput("localhost:5000/something:latest"),
				assertInOutput(`no registry credentials configured for "localhost:5000", using the default keychain`),
			},
```
		},
		},
		{
			# 测试允许不安全的 HTTP 标志
			name: "allows insecure http flag",
			# 设置参数
			args: []string{"-vv", "registry:localhost:5000/something:latest"},
			# 设置环境变量
			env: map[string]string{
				"GRYPE_REGISTRY_INSECURE_USE_HTTP": "true",
			},
			# 断言
			assertions: []traitAssertion{
				assertInOutput("insecure-use-http: true"),
			},
		},
		{
			# 使用 TLS 配置
			name: "use tls configuration",
			# 设置参数
			args: []string{"-vvv", "registry:localhost:5000/something:latest"},
			# 设置环境变量
			env: map[string]string{
				"GRYPE_REGISTRY_AUTH_TLS_CERT": "place.crt",
				"GRYPE_REGISTRY_AUTH_TLS_KEY":  "place.key",
			},
			# 断言
			assertions: []traitAssertion{
# 断言输出中是否包含特定的文本
assertInOutput("using custom TLS credentials from"),

# 定义测试用例
tests := []struct {
    name        string   // 测试用例名称
    env         []string // 测试环境变量
    args        []string // 测试参数
    assertions  []func(t *testing.T, stdout, stderr string, exitCode int) // 断言函数列表
}{
    {
        // 第一个测试用例
        name: "Test1",
        env: []string{"ENV_VAR=1"}, // 设置环境变量
        args: []string{"arg1", "arg2"}, // 设置参数
        assertions: []func(t *testing.T, stdout, stderr string, exitCode int){
            // 断言函数列表
            func(t *testing.T, stdout, stderr string, exitCode int) {
                // 断言函数
                // 检查 stdout, stderr, exitCode 是否符合预期
            },
        },
    },
}

# 遍历测试用例
for _, test := range tests {
    // 运行测试用例
    t.Run(test.name, func(t *testing.T) {
        // 运行测试命令，获取标准输出、标准错误和进程状态
        cmd, stdout, stderr := runGrype(t, test.env, test.args...)
        // 遍历断言函数列表，对标准输出、标准错误和退出码进行断言
        for _, traitAssertionFn := range test.assertions {
            traitAssertionFn(t, stdout, stderr, cmd.ProcessState.ExitCode())
        }
        // 如果测试失败，打印相关信息
        if t.Failed() {
            t.Log("STDOUT:\n", stdout)
            t.Log("STDERR:\n", stderr)
            t.Log("COMMAND:", strings.Join(cmd.Args, " "))
        }
    })
}
```