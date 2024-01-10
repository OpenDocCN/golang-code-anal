# `v2ray-core\transport\internet\headers\http\resp.go`

```
# 定义一个名为resp400的指针类型的ResponseConfig结构体变量，表示HTTP 400错误响应
var resp400 = &ResponseConfig{
    # 设置HTTP版本号为1.1
    Version: &Version{
        Value: "1.1",
    },
    # 设置状态码为400，原因为Bad Request
    Status: &Status{
        Code:   "400",
        Reason: "Bad Request",
    },
    # 设置响应头信息
    Header: []*Header{
        {
            Name:  "Connection",
            Value: []string{"close"},
        },
        {
            Name:  "Cache-Control",
            Value: []string{"private"},
        },
        {
            Name:  "Content-Length",
            Value: []string{"0"},
        },
    },
}

# 定义一个名为resp404的指针类型的ResponseConfig结构体变量，表示HTTP 404错误响应
var resp404 = &ResponseConfig{
    # 设置HTTP版本号为1.1
    Version: &Version{
        Value: "1.1",
    },
    # 设置状态码为404，原因为Not Found
    Status: &Status{
        Code:   "404",
        Reason: "Not Found",
    },
    # 设置响应头信息
    Header: []*Header{
        {
            Name:  "Connection",
            Value: []string{"close"},
        },
        {
            Name:  "Cache-Control",
            Value: []string{"private"},
        },
        {
            Name:  "Content-Length",
            Value: []string{"0"},
        },
    },
}
```