# `grype\cmd\grype\cli\commands\update_test.go`

```
package commands
// 导入命令包

import (
	"net/http"
	"net/http/httptest"
	"testing"

	hashiVersion "github.com/anchore/go-version"
	"github.com/anchore/grype/cmd/grype/internal"
)
// 导入所需的包

func TestIsUpdateAvailable(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		name          string
		buildVersion  string
		latestVersion string
		code          int
		isAvailable   bool
		newVersion    string
		err           bool
		// 定义测试用例的字段
# 定义一个包含不同情况的测试用例的列表
[
    {
        name:          "equal",          # 测试用例名称
        buildVersion:  "1.0.0",          # 当前版本号
        latestVersion: "1.0.0",          # 最新版本号
        code:          200,              # 状态码
        isAvailable:   false,            # 是否有更新可用
        newVersion:    "",               # 新版本号
        err:           false,            # 是否有错误
    },
    {
        name:          "hasUpdate",      # 测试用例名称
        buildVersion:  "1.0.0",          # 当前版本号
        latestVersion: "1.2.0",          # 最新版本号
        code:          200,              # 状态码
        isAvailable:   true,             # 是否有更新可用
        newVersion:    "1.2.0",          # 新版本号
        err:           false,            # 是否有错误
    },
    {
        ...
    }
]
# 定义一个名为 "aheadOfLatest" 的更新对象，包括更新名称、构建版本、最新版本、状态码、是否可用、新版本和错误标志
{
    name:          "aheadOfLatest",
    buildVersion:  "1.2.0",
    latestVersion: "1.0.0",
    code:          200,
    isAvailable:   false,
    newVersion:    "",
    err:           false,
},
# 定义一个名为 "EmptyUpdate" 的更新对象，包括更新名称、构建版本、最新版本、状态码、是否可用、新版本和错误标志
{
    name:          "EmptyUpdate",
    buildVersion:  "1.0.0",
    latestVersion: "",
    code:          200,
    isAvailable:   false,
    newVersion:    "",
    err:           true,
},
# 定义一个名为 "GarbageUpdate" 的更新对象，包括更新名称、构建版本
{
    name:          "GarbageUpdate",
    buildVersion:  "1.0.0",
		{
			# 最新版本号
			latestVersion: "hdfjksdhfhkj",
			# 状态码
			code:          200,
			# 是否可用
			isAvailable:   false,
			# 新版本号
			newVersion:    "",
			# 是否出错
			err:           true,
		},
		{
			# 名称
			name:          "BadUpdate",
			# 构建版本号
			buildVersion:  "1.0.0",
			# 最新版本号
			latestVersion: "1.0.",
			# 状态码
			code:          500,
			# 是否可用
			isAvailable:   false,
			# 新版本号
			newVersion:    "",
			# 是否出错
			err:           true,
		},
		{
			# 名称
			name:          "NoBuildVersion",
			# 构建版本号
			buildVersion:  internal.NotProvided,
			# 最新版本号
			latestVersion: "1.0.0",
			# 状态码
			code:          200,
// 初始化测试数据结构，包括是否可用、新版本号、错误标志等
isAvailable:   false, // 是否可用标志，初始为false
newVersion:    "",    // 新版本号，初始为空字符串
err:           false,  // 错误标志，初始为false

// 初始化测试数据结构，包括名称、构建版本、最新版本、状态码、是否可用、新版本号、错误标志等
{
    name:          "BadUpdateValidVersion", // 测试名称
    buildVersion:  "1.0.0",                // 构建版本号
    latestVersion: "2.0.0",                // 最新版本号
    code:          404,                     // 状态码
    isAvailable:   false,                   // 是否可用标志，初始为false
    newVersion:    "",                      // 新版本号，初始为空字符串
    err:           true,                    // 错误标志，初始为true
}

// 遍历测试数据
for _, test := range tests {
    t.Run(test.name, func(t *testing.T) {
        // 设置模拟数据
        // 本地...
        version := test.buildVersion
// 创建一个新的 ServeMux 对象，用于处理 HTTP 请求
handler := http.NewServeMux()

// 为 latestAppVersionURL.path 路径注册一个处理函数，返回测试数据
handler.HandleFunc(latestAppVersionURL.path, func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(test.code) // 设置响应状态码
    _, _ = w.Write([]byte(test.latestVersion)) // 写入响应数据
})

// 创建一个模拟的 HTTP 服务器，使用上面创建的 handler 处理请求
mockSrv := httptest.NewServer(handler)

// 更新 latestAppVersionURL.host 为模拟服务器的 URL
latestAppVersionURL.host = mockSrv.URL

// 延迟关闭模拟服务器的操作，确保在函数返回前关闭服务器
defer mockSrv.Close()

// 调用 isUpdateAvailable 函数，检查是否有新版本可用
isAvailable, newVersion, err := isUpdateAvailable(version)

// 检查错误情况
if err != nil && !test.err {
    t.Fatalf("got error but expected none: %+v", err)
} else if err == nil && test.err {
    t.Fatalf("expected error but got none")
}

// 检查新版本是否符合预期
if newVersion != test.newVersion {
    t.Errorf("unexpected NEW version: %+v", newVersion)
}
# 定义测试函数，用于测试 isAvailable 是否符合预期
func TestFetchLatestApplicationVersion(t *testing.T) {
    # 定义测试用例
    tests := []struct {
        name     string   # 测试用例名称
        response string   # 模拟的响应数据
        code     int      # 模拟的状态码
        err      bool     # 是否预期出错
        expected *hashiVersion.Version  # 期望的版本号
    }{
        {
            name:     "gocase",   # 测试用例名称
            response: "1.0.0",    # 模拟的响应数据
		{
			# 设置状态码为 200
			code:     200,
			# 设置期望的版本号为 1.0.0
			expected: hashiVersion.Must(hashiVersion.NewVersion("1.0.0")),
		},
		{
			# 设置名称为 "garbage"
			name:     "garbage",
			# 设置响应内容为 "garbage"
			response: "garbage",
			# 设置状态码为 200
			code:     200,
			# 设置期望的版本号为 nil
			expected: nil,
			# 设置出现错误为 true
			err:      true,
		},
		{
			# 设置名称为 "http 500"
			name:     "http 500",
			# 设置响应内容为 "1.0.0"
			response: "1.0.0",
			# 设置状态码为 500
			code:     500,
			# 设置期望的版本号为 nil
			expected: nil,
			# 设置出现错误为 true
			err:      true,
		},
		{
			# 设置名称为 "http 404"
			name:     "http 404",
			# 设置响应内容为 "1.0.0"
# 定义一个包含不同测试情况的列表
test_cases = [
    {
        # 测试名称
        name: "not found",
        # 响应内容
        response: "not found",
        # 响应状态码
        code: 404,
        # 期望结果
        expected: nil,
        # 是否出现错误
        err: true,
    },
    {
        name: "empty",
        response: "",
        code: 200,
        expected: nil,
        err: true,
    },
    {
        name: "too long",
        response: "this is really long this is really long this is really long this is really long this is really long this is really long this is really long this is really long ",
        code: 200,
        expected: nil,
        err: true,
    },
]
		for _, test := range tests {
		// 遍历测试用例
		t.Run(test.name, func(t *testing.T) {
			// 使用测试用例的名称创建子测试
			// 设置模拟服务器
			handler := http.NewServeMux()
			// 设置处理最新应用版本URL的处理函数
			handler.HandleFunc(latestAppVersionURL.path, func(w http.ResponseWriter, r *http.Request) {
				// 返回测试用例指定的状态码和响应数据
				w.WriteHeader(test.code)
				_, _ = w.Write([]byte(test.response))
			})
			// 创建模拟服务器
			mockSrv := httptest.NewServer(handler)
			// 更新最新应用版本URL的主机部分为模拟服务器的URL
			latestAppVersionURL.host = mockSrv.URL
			// 延迟关闭模拟服务器
			defer mockSrv.Close()

			// 获取最新应用版本并检查错误
			actual, err := fetchLatestApplicationVersion()
			// 如果有错误且测试用例不期望有错误，则输出错误信息
			if err != nil && !test.err {
				t.Fatalf("got error but expected none: %+v", err)
			// 如果没有错误且测试用例期望有错误，则输出错误信息
			} else if err == nil && test.err {
				t.Fatalf("expected error but got none")
			}

			// 如果有错误
			if err != nil {
				return
			}
            # 如果遇到空的返回语句，则直接返回，结束当前函数的执行
			if actual.String() != test.expected.String() {
                # 如果实际值和期望值不相等，则输出错误信息
				t.Errorf("unexpected version: %+v", actual.String())
			}
		})
	}
    # 结束当前测试用例的执行
}
```