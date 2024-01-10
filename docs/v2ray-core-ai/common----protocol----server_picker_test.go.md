# `v2ray-core\common\protocol\server_picker_test.go`

```
package protocol_test

import (
    "testing"
    "time"

    "v2ray.com/core/common/net"
    . "v2ray.com/core/common/protocol"
)

func TestServerList(t *testing.T) {
    // 创建一个新的服务器列表
    list := NewServerList()
    // 向服务器列表中添加一个服务器规格，使用本地IP和端口1，并且始终有效
    list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(1)), AlwaysValid()))
    // 检查服务器列表的大小是否为1
    if list.Size() != 1 {
        t.Error("list size: ", list.Size())
    }
    // 向服务器列表中添加一个服务器规格，使用本地IP和端口2，并且在当前时间之前有效
    list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(2)), BeforeTime(time.Now().Add(time.Second))))
    // 检查服务器列表的大小是否为2
    if list.Size() != 2 {
        t.Error("list.size: ", list.Size())
    }

    // 获取索引为1的服务器
    server := list.GetServer(1)
    // 检查服务器的端口是否为2
    if server.Destination().Port != 2 {
        t.Error("server: ", server.Destination())
    }
    // 等待2秒
    time.Sleep(2 * time.Second)
    // 再次获取索引为1的服务器
    server = list.GetServer(1)
    // 检查服务器是否为空
    if server != nil {
        t.Error("server: ", server)
    }

    // 获取索引为0的服务器
    server = list.GetServer(0)
    // 检查服务器的端口是否为1
    if server.Destination().Port != 1 {
        t.Error("server: ", server.Destination())
    }
}

func TestServerPicker(t *testing.T) {
    // 创建一个新的服务器列表
    list := NewServerList()
    // 向服务器列表中添加三个服务器规格，分别使用本地IP和端口1、2、3，并且在当前时间之前有效
    list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(1)), AlwaysValid()))
    list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(2)), BeforeTime(time.Now().Add(time.Second))))
    list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(3)), BeforeTime(time.Now().Add(time.Second))))

    // 创建一个新的轮询服务器选择器，使用上述的服务器列表
    picker := NewRoundRobinServerPicker(list)
    // 选择一个服务器
    server := picker.PickServer()
    // 检查服务器的端口是否为1
    if server.Destination().Port != 1 {
        t.Error("server: ", server.Destination())
    }
    // 再次选择一个服务器
    server = picker.PickServer()
    // 检查服务器的端口是否为2
    if server.Destination().Port != 2 {
        t.Error("server: ", server.Destination())
    }
    // 再次选择一个服务器
    server = picker.PickServer()
    // 检查服务器的端口是否为3
    if server.Destination().Port != 3 {
        t.Error("server: ", server.Destination())
    }
    // 再次选择一个服务器
    server = picker.PickServer()
    // 检查服务器的端口是否为1
    if server.Destination().Port != 1 {
        t.Error("server: ", server.Destination())
    }

    // 等待2秒
    time.Sleep(2 * time.Second)
}
    # 从服务器选择器中选择一个服务器
    server = picker.PickServer()
    # 如果选择的服务器目的地端口不是1，则输出错误信息
    if server.Destination().Port != 1:
        t.Error("server: ", server.Destination())
    # 重新选择一个服务器
    server = picker.PickServer()
    # 如果选择的服务器目的地端口不是1，则输出错误信息
    if server.Destination().Port != 1:
        t.Error("server: ", server.Destination())
# 闭合前面的函数定义
```