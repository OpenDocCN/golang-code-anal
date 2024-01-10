# `v2ray-core\app\reverse\reverse.go`

```
// +build !confonly

package reverse

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"  // 导入 context 包，用于处理上下文
    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    "v2ray.com/core/common/errors"  // 导入 v2ray 核心错误处理包
    "v2ray.com/core/common/net"  // 导入 v2ray 核心网络包
    "v2ray.com/core/features/outbound"  // 导入 v2ray 核心出站特性包
    "v2ray.com/core/features/routing"  // 导入 v2ray 核心路由特性包
)

const (
    internalDomain = "reverse.internal.v2ray.com"  // 定义内部域名常量
)

func isDomain(dest net.Destination, domain string) bool {
    return dest.Address.Family().IsDomain() && dest.Address.Domain() == domain  // 判断目标地址是否为域名，并且域名与指定的域名相同
}

func isInternalDomain(dest net.Destination) bool {
    return isDomain(dest, internalDomain)  // 判断目标地址是否为内部域名
}

func init() {
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        r := new(Reverse)  // 创建 Reverse 对象
        if err := core.RequireFeatures(ctx, func(d routing.Dispatcher, om outbound.Manager) error {
            return r.Init(config.(*Config), d, om)  // 初始化 Reverse 对象
        }); err != nil {
            return nil, err
        }
        return r, nil
    }))
}

type Reverse struct {
    bridges []*Bridge  // Reverse 对象包含的 Bridge 切片
    portals []*Portal  // Reverse 对象包含的 Portal 切片
}

func (r *Reverse) Init(config *Config, d routing.Dispatcher, ohm outbound.Manager) error {
    for _, bConfig := range config.BridgeConfig {
        b, err := NewBridge(bConfig, d)  // 根据配置创建 Bridge 对象
        if err != nil {
            return err
        }
        r.bridges = append(r.bridges, b)  // 将创建的 Bridge 对象添加到 Reverse 对象的 bridges 切片中
    }

    for _, pConfig := range config.PortalConfig {
        p, err := NewPortal(pConfig, ohm)  // 根据配置创建 Portal 对象
        if err != nil {
            return err
        }
        r.portals = append(r.portals, p)  // 将创建的 Portal 对象添加到 Reverse 对象的 portals 切片中
    }

    return nil
}

func (r *Reverse) Type() interface{} {
    return (*Reverse)(nil)  // 返回 Reverse 对象的类型
}

func (r *Reverse) Start() error {
    for _, b := range r.bridges {
        if err := b.Start(); err != nil {
            return err  // 启动每个 Bridge 对象
        }
    }

    for _, p := range r.portals {
        if err := p.Start(); err != nil {
            return err  // 启动每个 Portal 对象
        }
    }

    return nil
}

func (r *Reverse) Close() error {
    # 定义一个错误切片，用于存储关闭操作可能产生的错误
    var errs []error
    # 遍历桥接器切片，依次关闭每个桥接器，并将可能产生的错误添加到errs切片中
    for _, b := range r.bridges:
        errs = append(errs, b.Close())
    # 遍历传送门切片，依次关闭每个传送门，并将可能产生的错误添加到errs切片中
    for _, p := range r.portals:
        errs = append(errs, p.Close())
    # 将errs切片中的所有错误合并为一个错误返回
    return errors.Combine(errs...)
# 闭合前面的函数定义
```