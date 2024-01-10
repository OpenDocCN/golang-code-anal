# `kubo\test\api-startup\main.go`

```
package main

import (
    "fmt"  // 导入格式化包，用于格式化输出
    "log"  // 导入日志包，用于记录日志
    "net/http"  // 导入网络包，用于发送 HTTP 请求
    "sync"  // 导入同步包，用于实现并发控制
    "time"  // 导入时间包，用于处理时间相关操作
)

func main() {
    when := make(chan time.Time, 2)  // 创建一个带缓冲的通道，用于存放时间
    var wg sync.WaitGroup  // 创建一个同步等待组
    wg.Add(2)  // 增加等待组的计数器

    for _, port := range []string{"5001", "8080"} {  // 遍历端口列表
        go func(port string) {  // 启动一个 goroutine
            defer wg.Done()  // 减少等待组的计数器
            for {  // 无限循环
                r, err := http.Get(fmt.Sprintf("http://127.0.0.1:%s", port))  // 发送 HTTP GET 请求
                if err != nil {  // 如果请求出错
                    continue  // 继续下一次循环
                }
                t := time.Now()  // 获取当前时间
                when <- t  // 将当前时间发送到通道中
                log.Println(port, t, r.StatusCode)  // 记录端口、时间和状态码
                break  // 跳出循环
            }
        }(port)  // 传入端口参数
    }
    wg.Wait()  // 等待所有 goroutine 完成

    first := <-when  // 从通道中取出第一个时间
    second := <-when  // 从通道中取出第二个时间
    log.Println(second.Sub(first))  // 记录第二个时间减去第一个时间的差值
}
```