# `grype\cmd\grype\cli\options\registry_test.go`

```
package options

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/stretchr/testify/assert"  // 导入 testify 包，用于断言测试结果

	"github.com/anchore/stereoscope/pkg/image"  // 导入 image 包，用于处理镜像

)

func TestHasNonEmptyCredentials(t *testing.T) {
	tests := []struct {  // 定义测试用例结构体
		username, password, token, cert, key string  // 定义测试用例的输入参数
		expected                             bool  // 定义测试用例的期望输出
	}{

		{
			"", "", "", "", "",  // 设置测试用例的输入参数
			false,  // 设置测试用例的期望输出
# 创建一个包含多个字典的列表
		},
		# 第一个字典，包含键"user"，值为空字符串，其他键值对为空，最后一个值为false
		{
			"user", "", "", "", "",
			false,
		},
		# 第二个字典，包含键为空字符串，值为"pass"，其他键值对为空，最后一个值为false
		{
			"", "pass", "", "", "",
			false,
		},
		# 第三个字典，包含键为空字符串，值为"pass"，键"tok"的值为空字符串，其他键值对为空，最后一个值为true
		{
			"", "pass", "tok", "", "",
			true,
		},
		# 第四个字典，包含键"user"，值为空字符串，键"tok"的值为空字符串，其他键值对为空，最后一个值为true
		{
			"user", "", "tok", "", "",
			true,
		},
		# 第五个字典，包含键为空字符串，值为空字符串，键"tok"的值为空字符串，其他键值对为空，最后一个值为true
		{
			"", "", "tok", "", "",
			true,
		},
		{
			// 用户名、密码、令牌、空字符串、空字符串，true表示有效
			"user", "pass", "tok", "", "",
			true,
		},

		{
			// 用户名、密码、空字符串、空字符串，true表示有效
			"user", "pass", "", "", "",
			true,
		},
		{
			// 空字符串、空字符串、空字符串、证书、密钥，true表示有效
			"", "", "", "cert", "key",
			true,
		},
		{
			// 空字符串、空字符串、空字符串、证书、空字符串，false表示无效
			"", "", "", "cert", "",
			false,
		},
		{
			// 空字符串、空字符串、空字符串、空字符串、密钥
// 定义测试用例
tests := []struct {
    name     string
    input    registry
    expected image.RegistryOptions
}{
    // 第一个测试用例
    {
        name:  "no registry options", // 测试用例名称
        input: registry{}, // 输入的 registry 对象
        expected image.RegistryOptions, // 期望的输出结果
    },
    // 其他测试用例...
}

// 遍历测试用例
for _, test := range tests {
    // 使用测试用例的名称创建子测试
    t.Run(fmt.Sprintf("%+v", test), func(t *testing.T) {
        // 断言测试结果是否符合预期
        assert.Equal(t, test.expected, hasNonEmptyCredentials(test.username, test.password, test.token, test.cert, test.key))
    })
}
# 设置预期的 image.RegistryOptions 结构体，包含空的 Credentials 切片
expected: image.RegistryOptions{
    Credentials: []image.RegistryCredentials{},
},
# 设置预期的 image.RegistryOptions 结构体，包含 InsecureSkipTLSVerify 为 true，以及空的 Credentials 切片
},
{
    name: "set InsecureSkipTLSVerify",
    input: registry{
        InsecureSkipTLSVerify: true,
    },
    expected: image.RegistryOptions{
        InsecureSkipTLSVerify: true,
        Credentials:           []image.RegistryCredentials{},
    },
},
# 设置预期的 image.RegistryOptions 结构体，包含 InsecureUseHTTP 为 true
{
    name: "set InsecureUseHTTP",
    input: registry{
        InsecureUseHTTP: true,
    },
    expected: image.RegistryOptions{
# 设置不安全使用 HTTP 的选项为 true
InsecureUseHTTP: true,
# 设置凭据为空列表
Credentials:     []image.RegistryCredentials{},
```

```
# 设置所有布尔选项为 true
InsecureSkipTLSVerify: true,
InsecureUseHTTP:       true,
# 设置凭据为空列表
Credentials:           []image.RegistryCredentials{},
```

```
# 提供所有的 TLS 配置
CACert:                "ca.crt",
# 设置跳过 TLS 验证
InsecureSkipTLSVerify: true,
# 设置注册表认证信息
Auth: []RegistryCredentials{
    # 设置客户端 TLS 证书和密钥
    {
        TLSCert: "client.crt",
        TLSKey:  "client.key",
    },
},
# 设置预期的注册表选项
expected: image.RegistryOptions{
    # 设置 CA 证书文件或目录
    CAFileOrDir:           "ca.crt",
    # 设置跳过 TLS 验证
    InsecureSkipTLSVerify: true,
    # 设置客户端证书和密钥
    Credentials: []image.RegistryCredentials{
        {
            ClientCert: "client.crt",
            ClientKey:  "client.key",
        },
    },
},
# 遍历测试用例列表
for _, test := range tests:
    # 在测试中运行每个测试用例
    t.Run(test.name, func(t *testing.T):
        # 断言测试结果与预期结果相等
        assert.Equal(t, &test.expected, test.input.ToOptions())
    )
```