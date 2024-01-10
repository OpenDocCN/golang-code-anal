# `kubo\config\api.go`

```
package config

import (
    "encoding/base64"  // 导入 base64 编码包
    "strings"  // 导入字符串处理包
)

const (
    APITag           = "API"  // 定义常量 APITag 为 "API"
    AuthorizationTag = "Authorizations"  // 定义常量 AuthorizationTag 为 "Authorizations"
)

type RPCAuthScope struct {
    AuthSecret   string   // AuthSecret 是用于与 HTTP "Authorization" 头部进行比较的密钥。密钥的格式为 "type:value"。查看文档以获取支持的类型。
    AllowedPaths []string  // AllowedPaths 是允许的 RPC 路径前缀的显式列表。默认情况下，不允许任何路径。["/api/v0"] 公开所有 RPC。
}

type API struct {
    HTTPHeaders     map[string][]string  // HTTPHeaders 是与 API 返回的 HTTP 头部。
    Authorizations  map[string]*RPCAuthScope `json:",omitempty"`  // Authorization 是用于在 API 中进行身份验证的授权的映射。如果映射为空，则 RPC API 对所有人公开。查看文档以获取更多详细信息。
}

// ConvertAuthSecret 将给定格式为 "type:value" 的密钥转换为 HTTP Authorization 头部值。它可以处理 'bearer' 和 'basic' 类型。如果类型存在但未知，则返回空字符串。如果类型不存在，则假定为 'bearer' 类型。
func ConvertAuthSecret(secret string) string {
    if secret == "" {
        return secret
    }

    split := strings.SplitN(secret, ":", 2)
    if len(split) < 2 {
        // 没有前缀：假定为 bearer 令牌。
        return "Bearer " + secret
    }

    if strings.HasPrefix(secret, "basic:") {
        if strings.Contains(split[1], ":") {
            // 假定为 basic:user:password
            return "Basic " + base64.StdEncoding.EncodeToString([]byte(split[1]))
        } else {
            // 假定已经是 base64 编码。
            return "Basic " + split[1]
        }
    } else if strings.HasPrefix(secret, "bearer:") {
        return "Bearer " + split[1]
    }

    // 未知。类型存在，但我们无法处理它。
    return ""
}
```