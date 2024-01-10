# `v2ray-core\common\protocol\http\sniff_test.go`

```
package http_test

import (
    "testing"

    . "v2ray.com/core/common/protocol/http"
)

func TestHTTPHeaders(t *testing.T) {
    cases := []struct {
        input  string   // HTTP 请求内容
        domain string   // 请求的域名
        err    bool     // 是否有错误
    }{
        {
            input: `GET /tutorials/other/top-20-mysql-best-practices/ HTTP/1.1
Host: net.tutsplus.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5 (.NET CLR 3.5.30729)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Cookie: PHPSESSID=r2t5uvjq435r4q7ib3vtdjq120
Pragma: no-cache
Cache-Control: no-cache`,  // HTTP GET 请求内容
            domain: "net.tutsplus.com",  // 请求的域名
        },
        {
            input: `POST /foo.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5 (.NET CLR 3.5.30729)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://localhost/test.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 43
 
first_name=John&last_name=Doe&action=Submit`,  // HTTP POST 请求内容
            domain: "localhost",  // 请求的域名
        },
        {
            input: `X /foo.php HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5 (.NET CLR 3.5.30729)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://localhost/test.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 43
`,  // 错误的 HTTP 请求内容
            domain: "localhost",  // 请求的域名
        },
    }
}
# 定义测试用例数组，包含不同的输入和期望输出
cases := []struct {
    input  string  # 输入的 HTTP 请求内容
    domain string  # 期望的域名
    err    bool    # 是否期望出现错误
}{
    {
        input:  `GET /index.html HTTP/1.1
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5 (.NET CLR 3.5.30729)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://localhost/test.php
Host: localhost
first_name=John&last_name=Doe&action=Submit`,
        domain: "",  # 期望的域名为空
        err:    true,  # 期望出现错误
    },
    {
        input: `GET /foo.php HTTP/1.1
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5 (.NET CLR 3.5.30729)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://localhost/test.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 43
Host: localhost
first_name=John&last_name=Doe&action=Submit`,
        domain: "",  # 期望的域名为空
        err:    true,  # 期望出现错误
    },
    {
        input:  `GET /tutorials/other/top-20-mysql-best-practices/ HTTP/1.1`,
        domain: "",  # 期望的域名为空
        err:    true,  # 期望出现错误
    },
}

# 遍历测试用例数组
for _, test := range cases {
    # 调用 SniffHTTP 函数，解析输入的 HTTP 请求内容
    header, err := SniffHTTP([]byte(test.input))
    # 如果期望出现错误
    if test.err {
        # 如果实际没有出现错误，则输出错误信息
        if err == nil {
            t.Errorf("Expect error but nil, in test: %v", test)
        }
    } else {
        # 如果期望没有出现错误
        if err != nil {
            # 如果实际出现了错误，则输出错误信息
            t.Errorf("Expect no error but actually %s in test %v", err.Error(), test)
        }
        # 如果解析出的域名与期望的域名不一致，则输出错误信息
        if header.Domain() != test.domain {
            t.Error("expected domain ", test.domain, " but got ", header.Domain())
        }
    }
}
```