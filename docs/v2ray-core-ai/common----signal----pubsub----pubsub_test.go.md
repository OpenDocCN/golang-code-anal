# `v2ray-core\common\signal\pubsub\pubsub_test.go`

```
package pubsub_test

import (
    "testing"

    . "v2ray.com/core/common/signal/pubsub"
)

func TestPubsub(t *testing.T) {
    // 创建一个新的发布-订阅服务
    service := NewService()

    // 订阅主题"a"
    sub := service.Subscribe("a")
    // 发布主题"a"的数值1
    service.Publish("a", 1)

    // 从订阅中获取值，并进行比较
    select {
    case v := <-sub.Wait():
        if v != 1 {
            t.Error("expected subscribed value 1, but got ", v)
        }
    default:
        t.Fail()
    }

    // 关闭订阅
    sub.Close()
    // 再次发布主题"a"的数值2
    service.Publish("a", 2)

    // 检查是否收到订阅的值
    select {
    case <-sub.Wait():
        t.Fail()
    default:
    }

    // 清理发布-订阅服务
    service.Cleanup()
}
```