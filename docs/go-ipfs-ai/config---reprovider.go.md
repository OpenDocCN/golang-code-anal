# `kubo\config\reprovider.go`

```
# 定义了默认的重新提供者间隔时间，为 22 小时
DefaultReproviderInterval = time.Hour * 22 // https://github.com/ipfs/kubo/pull/9326
# 定义了默认的重新提供者策略为 "all"
DefaultReproviderStrategy = "all"

# 定义了重新提供者结构体
type Reprovider struct {
    # 重新提供本地存储对象到网络的时间间隔，可选字段
    Interval *OptionalDuration `json:",omitempty"`
    # 声明要公布的键的策略，可选字段
    Strategy *OptionalString   `json:",omitempty"`
}
```