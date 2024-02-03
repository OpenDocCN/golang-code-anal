# `grype\internal\bus\bus.go`

```go
# 定义一个名为bus的包
package bus

# 导入名为go-partybus的包
import "github.com/wagoodman/go-partybus"

# 声明一个名为publisher的变量，类型为partybus.Publisher
var publisher partybus.Publisher

# 定义一个名为Set的函数，参数为p，类型为partybus.Publisher
func Set(p partybus.Publisher) {
    # 将传入的p赋值给全局变量publisher
    publisher = p
}

# 定义一个名为Publish的函数，参数为event，类型为partybus.Event
func Publish(event partybus.Event) {
    # 检查publisher是否为空
    if publisher != nil {
        # 如果不为空，调用publisher的Publish方法，传入event作为参数
        publisher.Publish(event)
    }
}
```