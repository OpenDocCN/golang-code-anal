# `kubo\core\commands\pin\remotepin_test.go`

```go
package pin

import (
    "testing"
)

func TestNormalizeEndpoint(t *testing.T) {
    cases := []struct {  // 定义测试用例结构体切片
        in  string         // 输入的字符串
        err string         // 期望的错误信息
        out string         // 期望的输出字符串
    }{
        {
            in:  "https://1.example.com",  // 输入字符串
            err: "",                        // 期望的错误信息
            out: "https://1.example.com",   // 期望的输出字符串
        },
        {
            in:  "https://2.example.com/",  // 输入字符串
            err: "",                        // 期望的错误信息
            out: "https://2.example.com",   // 期望的输出字符串
        },
        {
            in:  "https://3.example.com/pins/",  // 输入字符串
            err: "service endpoint should be provided without the /pins suffix",  // 期望的错误信息
            out: "",                            // 期望的输出字符串
        },
        {
            in:  "https://4.example.com/pins",  // 输入字符串
            err: "service endpoint should be provided without the /pins suffix",  // 期望的错误信息
            out: "",                            // 期望的输出字符串
        },
        {
            in:  "https://5.example.com/./some//nonsense/../path/../path/",  // 输入字符串
            err: "",                                                        // 期望的错误信息
            out: "https://5.example.com/some/path",                         // 期望的输出字符串
        },
        {
            in:  "https://6.example.com/endpoint/?query=val",  // 输入字符串
            err: "service endpoint should be provided without any query parameters",  // 期望的错误信息
            out: "",                                                    // 期望的输出字符串
        },
        {
            in:  "http://192.168.0.5:45000/",  // 输入字符串
            err: "",                            // 期望的错误信息
            out: "http://192.168.0.5:45000",   // 期望的输出字符串
        },
        {
            in:  "foo://4.example.com/pins",  // 输入字符串
            err: "service endpoint must be a valid HTTP URL",  // 期望的错误信息
            out: "",                                        // 期望的输出字符串
        },
    }

    for _, tc := range cases {  // 遍历测试用例
        out, err := normalizeEndpoint(tc.in)  // 调用函数进行测试
        if err != nil && tc.err != err.Error() {  // 判断错误信息是否符合预期
            t.Errorf("unexpected error for %q: expected %q; got %q", tc.in, tc.err, err)  // 输出错误信息
            continue
        }
        if out != tc.out {  // 判断输出是否符合预期
            t.Errorf("unexpected endpoint for %q: expected %q; got %q", tc.in, tc.out, out)  // 输出错误信息
            continue
        }
    }
}
```