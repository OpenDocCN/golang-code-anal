# `kubo\config\dns.go`

```go
// DNS 结构体定义了使用自定义解析器的 DNS 解析规则
type DNS struct {
    // Resolvers 是一个 FQDN 到 URL 的映射，用于自定义 DNS 解析
    // 以 `https://` 开头的 URL 表示 DoH 端点
    // 未来可以添加对其他解析器类型的支持
    // https://en.wikipedia.org/wiki/Fully_qualified_domain_name
    // https://en.wikipedia.org/wiki/DNS_over_HTTPS
    //
    // 示例:
    // - ENS 的自定义解析器:          `eth.` → `https://dns.eth.limo/dns-query`
    // - 覆盖默认的操作系统解析器: `.`    → `https://doh.applied-privacy.net/query`
    Resolvers map[string]string
    // MaxCacheTTL 是 DNS 条目在缓存中有效的最长时间
    MaxCacheTTL *OptionalDuration `json:",omitempty"`
}
```