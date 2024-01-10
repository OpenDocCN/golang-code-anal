# `v2ray-core\proxy\freedom\config.go`

```
# 定义 Config 结构体的 useIP 方法，用于判断是否使用 IP 地址
func (c *Config) useIP() bool {
    # 返回判断条件，判断是否使用 IP 地址
    return c.DomainStrategy == Config_USE_IP || c.DomainStrategy == Config_USE_IP4 || c.DomainStrategy == Config_USE_IP6
}
```