# `trojan-go\proxy\custom\custom.go`

```go
package custom

import (
    "context"
    "strings"

    "gopkg.in/yaml.v3"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/proxy"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

func convert(i interface{}) interface{} {
    // 定义一个函数，用于将接口类型转换为另一个接口类型
    switch x := i.(type) {
    case map[interface{}]interface{}:
        // 如果接口类型是 map[interface{}]interface{}，则进行转换
        m2 := map[string]interface{}{}
        for k, v := range x {
            m2[k.(string)] = convert(v)
        }
        return m2
    case []interface{}:
        // 如果接口类型是 []interface{}，则进行转换
        for i, v := range x {
            x[i] = convert(v)
        }
    }
    return i
}

func buildNodes(ctx context.Context, nodeConfigList []NodeConfig) (map[string]*proxy.Node, error) {
    // 构建节点的函数
    nodes := make(map[string]*proxy.Node)
    for _, nodeCfg := range nodeConfigList {
        // 遍历节点配置列表
        nodeCfg.Protocol = strings.ToUpper(nodeCfg.Protocol)
        // 将节点协议名转换为大写
        if _, err := tunnel.GetTunnel(nodeCfg.Protocol); err != nil {
            return nil, common.NewError("invalid protocol name:" + nodeCfg.Protocol)
        }
        data, err := yaml.Marshal(nodeCfg.Config)
        // 将节点配置转换为 YAML 格式的数据
        common.Must(err)
        nodeContext, err := config.WithYAMLConfig(ctx, data)
        // 使用 YAML 格式的数据创建节点上下文
        if err != nil {
            return nil, common.NewError("failed to parse config data for " + nodeCfg.Tag + " with protocol" + nodeCfg.Protocol).Base(err)
        }
        node := &proxy.Node{
            Name:    nodeCfg.Protocol,
            Next:    make(map[string]*proxy.Node),
            Context: nodeContext,
        }
        nodes[nodeCfg.Tag] = node
        // 将节点添加到节点列表中
    }
    return nodes, nil
}

func init() {
    // 初始化函数
    // 这里可以添加一些初始化逻辑
}
```