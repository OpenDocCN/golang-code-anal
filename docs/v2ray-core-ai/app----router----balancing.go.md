# `v2ray-core\app\router\balancing.go`

```
// +build !confonly

package router

import (
    "v2ray.com/core/common/dice"  // 导入随机数生成包
    "v2ray.com/core/features/outbound"  // 导入出站连接管理包
)

type BalancingStrategy interface {  // 定义负载均衡策略接口
    PickOutbound([]string) string  // 选择出站连接的方法
}

type RandomStrategy struct {  // 随机策略结构体
}

func (s *RandomStrategy) PickOutbound(tags []string) string {  // 随机策略选择出站连接的方法
    n := len(tags)  // 获取标签数量
    if n == 0 {  // 如果标签数量为0
        panic("0 tags")  // 抛出异常
    }

    return tags[dice.Roll(n)]  // 返回随机选择的标签
}

type Balancer struct {  // 负载均衡器结构体
    selectors []string  // 标签选择器
    strategy  BalancingStrategy  // 负载均衡策略
    ohm       outbound.Manager  // 出站连接管理器
}

func (b *Balancer) PickOutbound() (string, error) {  // 负载均衡器选择出站连接的方法
    hs, ok := b.ohm.(outbound.HandlerSelector)  // 尝试将出站连接管理器转换为处理程序选择器
    if !ok {  // 如果转换失败
        return "", newError("outbound.Manager is not a HandlerSelector")  // 返回错误
    }
    tags := hs.Select(b.selectors)  // 使用标签选择器选择标签
    if len(tags) == 0 {  // 如果没有可用的标签
        return "", newError("no available outbounds selected")  // 返回错误
    }
    tag := b.strategy.PickOutbound(tags)  // 使用负载均衡策略选择标签
    if tag == "" {  // 如果选择的标签为空
        return "", newError("balancing strategy returns empty tag")  // 返回错误
    }
    return tag, nil  // 返回选择的标签
}
```