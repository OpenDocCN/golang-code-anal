# `kubo\config\gateway.go`

```go
package config

const (
    // DefaultInlineDNSLink specifies the default value for InlineDNSLink
    DefaultInlineDNSLink         = false
    // DefaultDeserializedResponses specifies the default value for DeserializedResponses
    DefaultDeserializedResponses = true
    // DefaultDisableHTMLErrors specifies the default value for DisableHTMLErrors
    DefaultDisableHTMLErrors     = false
    // DefaultExposeRoutingAPI specifies the default value for ExposeRoutingAPI
    DefaultExposeRoutingAPI      = false
)

type GatewaySpec struct {
    // Paths is explicit list of path prefixes that should be handled by
    // this gateway. Example: `["/ipfs", "/ipns", "/api"]`
    Paths []string

    // UseSubdomains indicates whether or not this gateway uses subdomains
    // for IPFS resources instead of paths. That is: http://CID.ipfs.GATEWAY/...
    //
    // If this flag is set, any /ipns/$id and/or /ipfs/$id paths in Paths
    // will be permanently redirected to http://$id.[ipns|ipfs].$gateway/.
    //
    // We do not support using both paths and subdomains for a single domain
    // for security reasons (Origin isolation).
    UseSubdomains bool

    // NoDNSLink configures this gateway to _not_ resolve DNSLink for the FQDN
    // provided in `Host` HTTP header.
    NoDNSLink bool

    // InlineDNSLink configures this gateway to always inline DNSLink names
    // (FQDN) into a single DNS label in order to interop with wildcard TLS certs
    // and Origin per CID isolation provided by rules like https://publicsuffix.org
    InlineDNSLink Flag

    // DeserializedResponses configures this gateway to respond to deserialized
    // responses. Disabling this option enables a Trustless Gateway, as per:
    // https://specs.ipfs.tech/http-gateways/trustless-gateway/.
    DeserializedResponses Flag
}

// Gateway contains options for the HTTP gateway server.
type Gateway struct {
    // HTTPHeaders configures the headers that should be returned by this
    // gateway.
    HTTPHeaders map[string][]string // HTTP headers to return with the gateway

    // RootRedirect is the path to which requests to `/` on this gateway
    // should be redirected.
    RootRedirect string

    // REMOVED: modern replacement tracked in https://github.com/ipfs/specs/issues/375
    Writable Flag `json:",omitempty"`
}
    // PathPrefixes被移除：https://github.com/ipfs/go-ipfs/issues/7702
    PathPrefixes []string

    // FIXME: 尚未实现：https://github.com/ipfs/kubo/issues/8059
    APICommands []string

    // NoFetch配置网关在响应请求时不获取块。
    NoFetch bool

    // NoDNSLink配置网关在响应具有“Host”HTTP标头中的值的请求时不执行DNS TXT记录查找。
    // 可以在PublicGateways中针对每个FQDN覆盖此标志。
    NoDNSLink bool

    // DeserializedResponses配置此网关以响应反序列化的请求。禁用此选项将启用一个无信任的网关，如下所示：
    // https://specs.ipfs.tech/http-gateways/trustless-gateway/。可以在PublicGateways中针对每个FQDN覆盖此选项。
    DeserializedResponses Flag

    // DisableHTMLErrors禁用当发生错误时的漂亮HTML页面。相反，将发送一个带有原始错误消息的“text/plain”页面。
    DisableHTMLErrors Flag

    // PublicGateways配置已知公共网关的行为。
    // 每个键都是完全合格的域名（FQDN）。
    PublicGateways map[string]*GatewaySpec

    // ExposeRoutingAPI配置网关端口以在/routing/v1（https://specs.ipfs.tech/routing/http-routing-v1/）上公开路由系统作为HTTP API。
    ExposeRoutingAPI Flag
# 闭合前面的函数定义
```