# `v2ray-core\app\log\log_test.go`

```
package log_test

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    "github.com/golang/mock/gomock"  // 导入模拟控制包
    "v2ray.com/core/app/log"  // 导入日志包
    "v2ray.com/core/common"  // 导入通用包
    clog "v2ray.com/core/common/log"  // 导入通用日志包
    "v2ray.com/core/testing/mocks"  // 导入模拟包
)

func TestCustomLogHandler(t *testing.T) {
    mockCtl := gomock.NewController(t)  // 创建模拟控制器
    defer mockCtl.Finish()  // 延迟关闭模拟控制器

    var loggedValue []string  // 声明字符串切片变量

    mockHandler := mocks.NewLogHandler(mockCtl)  // 创建模拟日志处理器
    mockHandler.EXPECT().Handle(gomock.Any()).AnyTimes().DoAndReturn(func(msg clog.Message) {  // 设置模拟处理器的期望行为
        loggedValue = append(loggedValue, msg.String())  // 将日志消息字符串追加到切片中
    })

    log.RegisterHandlerCreator(log.LogType_Console, func(lt log.LogType, options log.HandlerCreatorOptions) (clog.Handler, error) {  // 注册日志处理器创建函数
        return mockHandler, nil  // 返回模拟处理器
    })

    logger, err := log.New(context.Background(), &log.Config{  // 创建日志记录器
        ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试级别
        ErrorLogType:  log.LogType_Console,  // 设置错误日志类型为控制台
        AccessLogType: log.LogType_None,  // 设置访问日志类型为无
    })
    common.Must(err)  // 检查错误

    common.Must(logger.Start())  // 启动日志记录器

    clog.Record(&clog.GeneralMessage{  // 记录一般消息
        Severity: clog.Severity_Debug,  // 设置消息严重程度为调试级别
        Content:  "test",  // 设置消息内容为测试
    })

    if len(loggedValue) < 2 {  // 如果记录的日志消息数量小于2
        t.Fatal("expected 2 log messages, but actually ", loggedValue)  // 输出错误信息
    }

    if loggedValue[1] != "[Debug] test" {  // 如果第二条记录的日志消息不符合预期
        t.Fatal("expected '[Debug] test', but actually ", loggedValue[1])  // 输出错误信息
    }

    common.Must(logger.Close())  // 关闭日志记录器
}
```