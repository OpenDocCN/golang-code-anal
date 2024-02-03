# `v2ray-core\common\log\log_test.go`

```go
package log_test

import (
    "testing"  // 导入测试包
    "github.com/google/go-cmp/cmp"  // 导入比较包
    "v2ray.com/core/common/log"  // 导入日志包
    "v2ray.com/core/common/net"  // 导入网络包
)

type testLogger struct {
    value string  // 定义测试日志结构体
}

func (l *testLogger) Handle(msg log.Message) {
    l.value = msg.String()  // 实现日志处理方法
}

func TestLogRecord(t *testing.T) {
    var logger testLogger  // 创建测试日志对象
    log.RegisterHandler(&logger)  // 注册日志处理器

    ip := "8.8.8.8"  // 定义 IP 地址
    log.Record(&log.GeneralMessage{  // 记录一条一般消息
        Severity: log.Severity_Error,  // 设置消息级别为错误
        Content:  net.ParseAddress(ip),  // 设置消息内容为解析后的 IP 地址
    })

    if diff := cmp.Diff("[Error] "+ip, logger.value); diff != "" {  // 比较日志输出与预期结果
        t.Error(diff)  // 输出比较结果
    }
}
```