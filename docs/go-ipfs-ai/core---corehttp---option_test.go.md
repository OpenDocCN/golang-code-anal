# `kubo\core\corehttp\option_test.go`

```go
package corehttp

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"   // 导入 io 包，用于实现 I/O 操作
    "net/http"  // 导入 net/http 包，用于 HTTP 客户端和服务端的实现
    "net/http/httptest"  // 导入 net/http/httptest 包，用于 HTTP 测试服务器的实现
    "testing"  // 导入 testing 包，用于编写测试函数

    version "github.com/ipfs/kubo"  // 导入版本包
)

type testcasecheckversion struct {
    userAgent    string  // 用户代理
    uri          string  // 统一资源标识符
    shouldHandle bool    // 是否应该处理
    responseBody string  // 响应体
    responseCode int     // 响应状态码
}

func (tc testcasecheckversion) body() string {
    if !tc.shouldHandle && tc.responseBody == "" {
        return fmt.Sprintf("%s (%s != %s)\n", errAPIVersionMismatch, version.ApiVersion, tc.userAgent)  // 格式化输出错误信息
    }

    return tc.responseBody  // 返回响应体
}

func TestCheckVersionOption(t *testing.T) {
    tcs := []testcasecheckversion{  // 定义测试用例切片
        {"/go-ipfs/0.1/", APIPath + "/test/", false, "", http.StatusBadRequest},  // 测试用例1
        {"/go-ipfs/0.1/", APIPath + "/version", true, "check!", http.StatusOK},  // 测试用例2
        {version.ApiVersion, APIPath + "/test", true, "check!", http.StatusOK},  // 测试用例3
        {"Mozilla Firefox/no go-ipfs node", APIPath + "/test", true, "check!", http.StatusOK},  // 测试用例4
        {"/go-ipfs/0.1/", "/webui", true, "check!", http.StatusOK},  // 测试用例5
    }
    # 遍历测试用例切片
    for _, tc := range tcs {
        # 打印测试用例
        t.Logf("%#v", tc)
        # 创建一个新的 HTTP 请求
        r := httptest.NewRequest(http.MethodPost, tc.uri, nil)
        # 添加 User-Agent 到请求头部，使用旧版本，应该失败
        r.Header.Add("User-Agent", tc.userAgent) // old version, should fail

        # 初始化一个标志变量
        called := false
        # 创建一个新的 HTTP 服务路由
        root := http.NewServeMux()
        # 调用 CheckVersionOption 函数，获取处理器和错误信息
        mux, err := CheckVersionOption()(nil, nil, root)
        # 如果有错误，记录错误并终止测试
        if err != nil {
            t.Fatal(err)
        }

        # 注册处理函数到服务路由
        mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            # 标记处理函数被调用
            called = true
            # 如果不应该处理，则记录错误
            if !tc.shouldHandle {
                t.Error("handler was called even though version didn't match")
            }
            # 向响应写入数据
            if _, err := io.WriteString(w, "check!"); err != nil {
                t.Error(err)
            }
        })

        # 创建一个新的响应记录器
        w := httptest.NewRecorder()

        # 调用服务路由处理 HTTP 请求
        root.ServeHTTP(w, r)

        # 如果应该处理但未被调用，则记录错误
        if tc.shouldHandle && !called {
            t.Error("handler wasn't called even though it should have")
        }

        # 检查响应状态码是否符合预期
        if w.Code != tc.responseCode {
            t.Errorf("expected code %d but got %d", tc.responseCode, w.Code)
        }

        # 检查响应体内容是否符合预期
        if w.Body.String() != tc.body() {
            t.Errorf("expected error message %q, got %q", tc.body(), w.Body.String())
        }
    }
# 闭合前面的函数定义
```