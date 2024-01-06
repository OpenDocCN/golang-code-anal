# `grype\internal\bus\bus.go`

```
# 定义一个名为bus的包
package bus

# 导入名为github.com/wagoodman/go-partybus的包
import "github.com/wagoodman/go-partybus"

# 声明一个名为publisher的partybus.Publisher类型的变量
var publisher partybus.Publisher

# 定义一个名为Set的函数，接受一个partybus.Publisher类型的参数，并将其赋值给全局变量publisher
func Set(p partybus.Publisher) {
    publisher = p
}

# 定义一个名为Publish的函数，接受一个partybus.Event类型的参数
# 如果publisher不为空，则调用其Publish方法发布事件
func Publish(event partybus.Event) {
    if publisher != nil {
        publisher.Publish(event)
    }
}
```