# `grype\cmd\grype\cli\commands\update_test.go`

```
package commands

import (
    "net/http"
    "net/http/httptest"
    "testing"

    hashiVersion "github.com/anchore/go-version"
    "github.com/anchore/grype/cmd/grype/internal"
)

func TestIsUpdateAvailable(t *testing.T) {
    tests := []struct {
        name          string
        buildVersion  string
        latestVersion string
        code          int
        isAvailable   bool
        newVersion    string
        err           bool
    }

    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            // setup mocks
            // local...
            version := test.buildVersion
            // remote...
            handler := http.NewServeMux()
            // 设置处理最新应用版本的 URL
            handler.HandleFunc(latestAppVersionURL.path, func(w http.ResponseWriter, r *http.Request) {
                w.WriteHeader(test.code)
                // 返回最新版本号
                _, _ = w.Write([]byte(test.latestVersion))
            })
            // 创建一个模拟的 HTTP 服务器
            mockSrv := httptest.NewServer(handler)
            // 设置最新应用版本的 URL
            latestAppVersionURL.host = mockSrv.URL
            // 在测试结束后关闭模拟的 HTTP 服务器
            defer mockSrv.Close()

            // 检查是否有更新可用
            isAvailable, newVersion, err := isUpdateAvailable(version)
            // 如果预期有错误但实际没有错误，报错
            if err != nil && !test.err {
                t.Fatalf("got error but expected none: %+v", err)
            // 如果预期没有错误但实际有错误，报错
            } else if err == nil && test.err {
                t.Fatalf("expected error but got none")
            }

            // 检查新版本号是否符合预期
            if newVersion != test.newVersion {
                t.Errorf("unexpected NEW version: %+v", newVersion)
            }

            // 检查是否有更新可用是否符合预期
            if isAvailable != test.isAvailable {
                t.Errorf("unexpected result: %+v", isAvailable)
            }
        })
    }

}

func TestFetchLatestApplicationVersion(t *testing.T) {
    tests := []struct {
        name     string
        response string
        code     int
        err      bool
        expected *hashiVersion.Version
    # 定义一个包含多个测试用例的列表
    {
        # 第一个测试用例
        {
            # 测试用例名称
            name:     "gocase",
            # 期望的响应版本号
            response: "1.0.0",
            # 期望的响应状态码
            code:     200,
            # 期望的版本号哈希值
            expected: hashiVersion.Must(hashiVersion.NewVersion("1.0.0")),
        },
        # 第二个测试用例
        {
            # 测试用例名称
            name:     "garbage",
            # 期望的响应内容为垃圾数据
            response: "garbage",
            # 期望的响应状态码
            code:     200,
            # 期望的版本号哈希值为空
            expected: nil,
            # 期望出现错误
            err:      true,
        },
        # 第三个测试用例
        {
            # 测试用例名称
            name:     "http 500",
            # 期望的响应版本号
            response: "1.0.0",
            # 期望的响应状态码为 500
            code:     500,
            # 期望的版本号哈希值为空
            expected: nil,
            # 期望出现错误
            err:      true,
        },
        # 第四个测试用例
        {
            # 测试用例名称
            name:     "http 404",
            # 期望的响应版本号
            response: "1.0.0",
            # 期望的响应状态码为 404
            code:     404,
            # 期望的版本号哈希值为空
            expected: nil,
            # 期望出现错误
            err:      true,
        },
        # 第五个测试用例
        {
            # 测试用例名称
            name:     "empty",
            # 期望的响应内容为空
            response: "",
            # 期望的响应状态码为 200
            code:     200,
            # 期望的版本号哈希值为空
            expected: nil,
            # 期望出现错误
            err:      true,
        },
        # 第六个测试用例
        {
            # 测试用例名称
            name:     "too long",
            # 期望的响应内容为非常长的字符串
            response: "this is really long this is really long this is really long this is really long this is really long this is really long this is really long this is really long ",
            # 期望的响应状态码为 200
            code:     200,
            # 期望的版本号哈希值为空
            expected: nil,
            # 期望出现错误
            err:      true,
        },
    }
    for _, test := range tests {
        t.Run(test.name, func(t *testing.T) {
            // 遍历测试用例，对每个测试用例运行子测试
            // 设置模拟服务器
            handler := http.NewServeMux()
            // 处理最新应用版本的 URL 请求，返回模拟的响应
            handler.HandleFunc(latestAppVersionURL.path, func(w http.ResponseWriter, r *http.Request) {
                w.WriteHeader(test.code)
                _, _ = w.Write([]byte(test.response))
            })
            // 创建模拟服务器
            mockSrv := httptest.NewServer(handler)
            // 设置最新应用版本的 URL 为模拟服务器的 URL
            latestAppVersionURL.host = mockSrv.URL
            // 延迟关闭模拟服务器
            defer mockSrv.Close()

            // 获取最新应用版本，检查是否有错误
            actual, err := fetchLatestApplicationVersion()
            if err != nil && !test.err {
                // 如果有错误但不应该有错误，则测试失败
                t.Fatalf("got error but expected none: %+v", err)
            } else if err == nil && test.err {
                // 如果没有错误但应该有错误，则测试失败
                t.Fatalf("expected error but got none")
            }

            // 如果有错误，则直接返回，不进行后续的比较
            if err != nil {
                return
            }

            // 比较实际结果和期望结果，如果不一致则测试失败
            if actual.String() != test.expected.String() {
                t.Errorf("unexpected version: %+v", actual.String())
            }
        })
    }
# 闭合前面的函数定义
```