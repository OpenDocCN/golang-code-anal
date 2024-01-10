# `trojan-go\proxy\stack.go`

```
package proxy

import (
    "context"  // 导入上下文包

    "github.com/p4gefau1t/trojan-go/log"  // 导入日志包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入隧道包
)

type Node struct {
    Name       string  // 节点名称
    Next       map[string]*Node  // 下一个节点的映射
    IsEndpoint bool  // 是否为终端节点
    context.Context  // 上下文
    tunnel.Server  // 服务器隧道
    tunnel.Client  // 客户端隧道
}

func (n *Node) BuildNext(name string) *Node {
    if next, found := n.Next[name]; found {  // 如果找到下一个节点，则返回
        return next
    }
    t, err := tunnel.GetTunnel(name)  // 获取隧道
    if err != nil {
        log.Fatal(err)  // 如果出错，则记录日志并退出
    }
    s, err := t.NewServer(n.Context, n.Server)  // 创建新的服务器
    if err != nil {
        log.Fatal(err)  // 如果出错，则记录日志并退出
    }
    newNode := &Node{  // 创建新节点
        Name:    name,
        Next:    make(map[string]*Node),
        Context: n.Context,
        Server:  s,
    }
    n.Next[name] = newNode  // 将新节点添加到下一个节点映射中
    return newNode
}

func (n *Node) LinkNextNode(next *Node) *Node {
    if next, found := n.Next[next.Name]; found {  // 如果找到下一个节点，则返回
        return next
    }
    n.Next[next.Name] = next  // 将下一个节点添加到下一个节点映射中
    t, err := tunnel.GetTunnel(next.Name)  // 获取隧道
    if err != nil {
        log.Fatal(err)  // 如果出错，则记录日志并退出
    }
    s, err := t.NewServer(next.Context, n.Server)  // 创建新的服务器，子节点的上下文已经初始化
    if err != nil {
        log.Fatal(err)  // 如果出错，则记录日志并退出
    }
    next.Server = s  // 设置子节点的服务器
    return next
}

func FindAllEndpoints(root *Node) []tunnel.Server {
    list := make([]tunnel.Server, 0)  // 创建服务器列表
    if root.IsEndpoint || len(root.Next) == 0 {  // 如果是终端节点或者没有下一个节点
        list = append(list, root.Server)  // 将当前节点的服务器添加到列表中
    }
    for _, next := range root.Next {  // 遍历下一个节点
        list = append(list, FindAllEndpoints(next)...)  // 递归查找所有终端节点，并添加到列表中
    }
    return list  // 返回服务器列表
}

// CreateClientStack create client tunnel stacks from lists
func CreateClientStack(ctx context.Context, clientStack []string) (tunnel.Client, error) {
    var client tunnel.Client  // 客户端隧道
    for _, name := range clientStack {  // 遍历客户端隧道列表
        t, err := tunnel.GetTunnel(name)  // 获取隧道
        if err != nil {
            return nil, err  // 如果出错，则返回空和错误
        }
        client, err = t.NewClient(ctx, client)  // 创建新的客户端
        if err != nil {
            return nil, err  // 如果出错，则返回空和错误
        }
    }
    return client, nil  // 返回客户端和空错误
}
// 从给定的服务器堆栈列表创建服务器隧道堆栈
func CreateServerStack(ctx context.Context, serverStack []string) (tunnel.Server, error) {
    // 声明一个服务器变量
    var server tunnel.Server
    // 遍历服务器堆栈列表
    for _, name := range serverStack {
        // 获取指定名称的隧道对象
        t, err := tunnel.GetTunnel(name)
        // 如果出现错误，返回 nil 和错误
        if err != nil {
            return nil, err
        }
        // 使用隧道对象创建新的服务器，并返回服务器和错误
        server, err = t.NewServer(ctx, server)
        // 如果出现错误，返回 nil 和错误
        if err != nil {
            return nil, err
        }
    }
    // 返回服务器对象和 nil 错误
    return server, nil
}
```