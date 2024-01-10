# `v2ray-core\common\signal\notifier_test.go`

```
# 声明 signal_test 包
package signal_test

# 导入 testing 包
import (
    "testing"

    # 导入 signal 包，并使用 . 符号将其所有公共函数和变量导入当前包的命名空间
    . "v2ray.com/core/common/signal"
)

# 定义测试函数 TestNotifierSignal
func TestNotifierSignal(t *testing.T) {
    # 创建一个新的 Notifier 对象
    n := NewNotifier()

    # 创建一个等待通知的通道
    w := n.Wait()
    # 发送信号通知
    n.Signal()

    # 使用 select 语句监听通道
    select {
    # 如果 w 通道收到数据
    case <-w:
    # 如果 w 通道没有收到数据
    default:
        # 测试失败
        t.Fail()
    }
}
```