# `v2ray-core\proxy\blackhole\config_test.go`

```
package blackhole_test

import (
    "bufio"  // 导入 bufio 包，提供了缓冲读写功能
    "net/http"  // 导入 net/http 包，提供了 HTTP 客户端和服务端的实现
    "testing"  // 导入 testing 包，提供了测试框架

    "v2ray.com/core/common"  // 导入 common 包，v2ray 核心通用功能
    "v2ray.com/core/common/buf"  // 导入 buf 包，v2ray 核心缓冲功能
    . "v2ray.com/core/proxy/blackhole"  // 导入 blackhole 包，并将其所有公开的符号导入当前包的命名空间

)

func TestHTTPResponse(t *testing.T) {
    buffer := buf.New()  // 创建一个新的缓冲区

    httpResponse := new(HTTPResponse)  // 创建一个新的 HTTPResponse 对象
    httpResponse.WriteTo(buf.NewWriter(buffer))  // 将 HTTPResponse 对象写入缓冲区

    reader := bufio.NewReader(buffer)  // 创建一个新的缓冲读取器
    response, err := http.ReadResponse(reader, nil)  // 从缓冲读取器中读取 HTTP 响应
    common.Must(err)  // 如果发生错误，立即终止程序并打印错误信息
    if response.StatusCode != 403 {  // 如果响应状态码不是 403
        t.Error("expected status code 403, but got ", response.StatusCode)  // 输出错误信息
    }
}
```