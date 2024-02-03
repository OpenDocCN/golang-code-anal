# `v2ray-core\common\log\logger_test.go`

```go
package log_test

import (
    "io/ioutil"  // 导入用于读取文件的包
    "os"  // 导入操作系统相关的包
    "strings"  // 导入字符串处理相关的包
    "testing"  // 导入测试相关的包
    "time"  // 导入时间相关的包

    "v2ray.com/core/common"  // 导入自定义包
    "v2ray.com/core/common/buf"  // 导入自定义包
    . "v2ray.com/core/common/log"  // 导入自定义包并使用其中的所有函数
)

func TestFileLogger(t *testing.T) {
    f, err := ioutil.TempFile("", "vtest")  // 创建临时文件
    common.Must(err)  // 检查错误
    path := f.Name()  // 获取临时文件的路径
    common.Must(f.Close())  // 关闭文件

    creator, err := CreateFileLogWriter(path)  // 创建文件日志写入器
    common.Must(err)  // 检查错误

    handler := NewLogger(creator)  // 创建新的日志处理器
    handler.Handle(&GeneralMessage{Content: "Test Log"})  // 处理一般消息
    time.Sleep(2 * time.Second)  // 休眠2秒

    common.Must(common.Close(handler))  // 关闭日志处理器

    f, err = os.Open(path)  // 打开文件
    common.Must(err)  // 检查错误
    defer f.Close() // nolint: errcheck  // 延迟关闭文件，忽略错误

    b, err := buf.ReadAllToBytes(f)  // 读取文件内容到字节流
    common.Must(err)  // 检查错误
    if !strings.Contains(string(b), "Test Log") {  // 检查文件内容是否包含特定字符串
        t.Fatal("Expect log text contains 'Test Log', but actually: ", string(b))  // 输出错误信息
    }
}
```