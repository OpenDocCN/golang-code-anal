# `trojan-go\url\option_test.go`

```go
package url

import (
    "testing"  // 导入测试包
    "time"     // 导入时间包

    _ "github.com/p4gefau1t/trojan-go/proxy/client"  // 导入代理客户端包
)

func TestUrl_Handle(t *testing.T) {
    urlCases := []string{  // 定义 URL 测试用例数组
        "trojan-go://password@server.com",  // URL 测试用例1
        "trojan-go://password@server.com/?type=ws&host=baidu.com&path=%2fwspath",  // URL 测试用例2
        "trojan-go://password@server.com/?encryption=ss%3baes-256-gcm%3afuckgfw",  // URL 测试用例3
        "trojan-go://password@server.com/?type=ws&host=baidu.com&path=%2fwspath&encryption=ss%3Baes-256-gcm%3Afuckgfw",  // URL 测试用例4
    }
    optionCases := []string{  // 定义选项测试用例数组
        "mux=true;listen=127.0.0.1:0",  // 选项测试用例1
        "mux=false;listen=127.0.0.1:0",  // 选项测试用例2
        "mux=false;listen=127.0.0.1:0;api=127.0.0.1:0",  // 选项测试用例3
    }

    for _, s := range urlCases {  // 遍历 URL 测试用例数组
        for _, option := range optionCases {  // 遍历选项测试用例数组
            s := s  // 复制 URL 测试用例
            option := option  // 复制选项测试用例
            u := &url{  // 创建 URL 对象
                url:    &s,  // 设置 URL 属性
                option: &option,  // 设置选项属性
            }
            u.Name()  // 调用 URL 对象的 Name 方法
            u.Priority()  // 调用 URL 对象的 Priority 方法

            errChan := make(chan error, 1)  // 创建错误通道
            go func() {  // 启动协程
                errChan <- u.Handle()  // 调用 URL 对象的 Handle 方法，并将结果发送到错误通道
            }()

            select {  // 选择通道操作
            case err := <-errChan:  // 如果错误通道有值
                t.Fatal(err)  // 输出错误信息
            case <-time.After(time.Second * 1):  // 如果超时1秒
            }
        }
    }
}
```