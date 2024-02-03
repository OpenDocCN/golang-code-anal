# `trojan-go\url\share_link.go`

```go
package url

import (
    "errors" // 引入 errors 包，用于创建错误
    "fmt" // 引入 fmt 包，用于格式化输出
    neturl "net/url" // 引入 net/url 包，并重命名为 neturl，用于处理 URL 相关操作
    "strconv" // 引入 strconv 包，用于字符串和基本数据类型之间的转换
    "strings" // 引入 strings 包，用于处理字符串操作
)

const (
    ShareInfoTypeOriginal  = "original" // 定义常量 ShareInfoTypeOriginal，值为 "original"
    ShareInfoTypeWebSocket = "ws" // 定义常量 ShareInfoTypeWebSocket，值为 "ws"
)

var validTypes = map[string]struct{}{ // 定义 validTypes 变量，值为包含两个元素的 map
    ShareInfoTypeOriginal:  {}, // 将 ShareInfoTypeOriginal 添加到 map 中
    ShareInfoTypeWebSocket: {}, // 将 ShareInfoTypeWebSocket 添加到 map 中
}

var validEncryptionProviders = map[string]struct{}{ // 定义 validEncryptionProviders 变量，值为包含两个元素的 map
    "ss":   {}, // 将 "ss" 添加到 map 中
    "none": {}, // 将 "none" 添加到 map 中
}

var validSSEncryptionMap = map[string]struct{}{ // 定义 validSSEncryptionMap 变量，值为包含三个元素的 map
    "aes-128-gcm":            {}, // 将 "aes-128-gcm" 添加到 map 中
    "aes-256-gcm":            {}, // 将 "aes-256-gcm" 添加到 map 中
    "chacha20-ietf-poly1305": {}, // 将 "chacha20-ietf-poly1305" 添加到 map 中
}

type ShareInfo struct { // 定义结构体 ShareInfo
    TrojanHost     string // 节点 IP / 域名
    Port           uint16 // 节点端口
    TrojanPassword string // Trojan 密码

    SNI  string // SNI
    Type string // 类型
    Host string // HTTP Host Header

    Path       string // WebSocket / H2 Path
    Encryption string // 额外加密
    Plugin     string // 插件设定

    Description string // 节点说明
}

func NewShareInfoFromURL(shareLink string) (info ShareInfo, e error) { // 定义函数 NewShareInfoFromURL，接收一个字符串参数，返回 ShareInfo 和 error
    // share link must be valid url
    parse, e := neturl.Parse(shareLink) // 解析分享链接，返回 URL 对象和错误
    if e != nil { // 如果有错误
        e = fmt.Errorf("invalid url: %s", e.Error()) // 格式化错误信息
        return // 返回错误
    }

    // share link must have `trojan-go://` scheme
    if parse.Scheme != "trojan-go" { // 如果 scheme 不是 "trojan-go"
        e = errors.New("url does not have a trojan-go:// scheme") // 创建错误信息
        return // 返回错误
    }

    // password
    if info.TrojanPassword = parse.User.Username(); info.TrojanPassword == "" { // 获取用户名作为密码，如果密码为空
        e = errors.New("no password specified") // 创建错误信息
        return // 返回错误
    } else if _, hasPassword := parse.User.Password(); hasPassword { // 如果密码存在
        e = errors.New("password possibly missing percentage encoding for colon") // 创建错误信息
        return // 返回错误
    }

    // trojanHost: not empty & strip [] from IPv6 addresses
    if info.TrojanHost = parse.Hostname(); info.TrojanHost == "" { // 获取主机名，如果主机名为空
        e = errors.New("host is empty") // 创建错误信息
        return // 返回错误
    }

    // port
    if info.Port, e = handleTrojanPort(parse.Port()); e != nil { // 处理端口
        return // 返回错误
    }

    // strictly parse the query
    // 解析 URL 查询参数
    query, e := neturl.ParseQuery(parse.RawQuery)
    // 如果解析出错，则返回
    if e != nil {
        return
    }

    // sni
    // 如果查询参数中没有 SNI，则使用默认的 TrojanHost
    if SNIs, ok := query["sni"]; !ok {
        info.SNI = info.TrojanHost
    } else if len(SNIs) > 1 {
        e = errors.New("multiple SNIs")
        return
    } else if info.SNI = SNIs[0]; info.SNI == "" {
        e = errors.New("empty SNI")
        return
    }

    // type
    // 如果查询参数中没有类型，则使用默认的 ShareInfoTypeOriginal
    if types, ok := query["type"]; !ok {
        info.Type = ShareInfoTypeOriginal
    } else if len(types) > 1 {
        e = errors.New("multiple transport types")
        return
    } else if info.Type = types[0]; info.Type == "" {
        e = errors.New("empty transport type")
        return
    } else if _, ok := validTypes[info.Type]; !ok {
        e = fmt.Errorf("unknown transport type: %s", info.Type)
        return
    }

    // host
    // 如果查询参数中没有主机，则使用默认的 TrojanHost
    if hosts, ok := query["host"]; !ok {
        info.Host = info.TrojanHost
    } else if len(hosts) > 1 {
        e = errors.New("multiple hosts")
        return
    } else if info.Host = hosts[0]; info.Host == "" {
        e = errors.New("empty host")
        return
    }

    // path
    // 如果类型为 WebSocket，则查询参数中必须包含路径
    if info.Type == ShareInfoTypeWebSocket {
        if paths, ok := query["path"]; !ok {
            e = errors.New("path is required in websocket")
            return
        } else if len(paths) > 1 {
            e = errors.New("multiple paths")
            return
        } else if info.Path = paths[0]; info.Path == "" {
            e = errors.New("empty path")
            return
        }

        // 路径必须以斜杠开头
        if !strings.HasPrefix(info.Path, "/") {
            e = errors.New("path must start with /")
            return
        }
    }

    // encryption
    // 如果查询参数中没有加密字段，则不进行加密
    if encryptionArr, ok := query["encryption"]; !ok {
        // no encryption. that's okay.
    } else if len(encryptionArr) > 1 {
        e = errors.New("multiple encryption fields")
        return
    // 如果加密信息为空，则返回错误
    } else if info.Encryption = encryptionArr[0]; info.Encryption == "" {
        e = errors.New("empty encryption")
        return
    } else {
        // 根据分号拆分加密信息，获取加密提供者名称
        encryptionParts := strings.SplitN(info.Encryption, ";", 2)
        encryptionProviderName := encryptionParts[0]

        // 检查加密提供者名称是否在支持的列表中
        if _, ok := validEncryptionProviders[encryptionProviderName]; !ok {
            e = fmt.Errorf("unsupported encryption provider name: %s", encryptionProviderName)
            return
        }

        var encryptionParams string
        // 如果加密信息包含参数，则获取参数
        if len(encryptionParts) >= 2 {
            encryptionParams = encryptionParts[1]
        }

        // 如果加密提供者是 "ss"，则进行特定处理
        if encryptionProviderName == "ss" {
            ssParams := strings.SplitN(encryptionParams, ":", 2)
            // 检查是否缺少 ss 密码
            if len(ssParams) < 2 {
                e = errors.New("missing ss password")
                return
            }

            ssMethod, ssPassword := ssParams[0], ssParams[1]
            // 检查 ss 加密方法是否在支持的列表中
            if _, ok := validSSEncryptionMap[ssMethod]; !ok {
                e = fmt.Errorf("unsupported ss method: %s", ssMethod)
                return
            }

            // 检查 ss 密码是否为空
            if ssPassword == "" {
                e = errors.New("ss password cannot be empty")
                return
            }
        }
    }

    // 处理插件信息
    if plugins, ok := query["plugin"]; !ok {
        // 如果没有插件信息，则不做处理
    } else if len(plugins) > 1 {
        e = errors.New("multiple plugins")
        return
    } else if info.Plugin = plugins[0]; info.Plugin == "" {
        e = errors.New("empty plugin")
        return
    }

    // 将描述信息设置为解析后的片段
    info.Description = parse.Fragment

    return
# 定义处理特洛伊木马端口的函数，接受一个字符串参数，返回一个无符号16位整数和一个错误
func handleTrojanPort(p string) (port uint16, e error) {
    # 如果参数为空字符串，则返回443和nil
    if p == "" {
        return 443, nil
    }

    # 尝试将参数解析为整数，如果出现错误则返回
    portParsed, e := strconv.Atoi(p)
    if e != nil {
        return
    }

    # 如果解析后的端口数值小于1或大于65535，则返回错误
    if portParsed < 1 || portParsed > 65535 {
        e = fmt.Errorf("invalid port %d", portParsed)
        return
    }

    # 将解析后的端口数值转换为无符号16位整数并返回
    port = uint16(portParsed)
    return
}
```