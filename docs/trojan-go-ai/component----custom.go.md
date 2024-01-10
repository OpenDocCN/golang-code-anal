# `trojan-go\component\custom.go`

```
// 根据条件编译标记，选择构建 custom 或 full 版本
// +build custom full

package build

import (
    _ "github.com/p4gefau1t/trojan-go/proxy/custom"  // 导入自定义代理模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/adapter"  // 导入适配器模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/dokodemo"  // 导入dokodemo模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入自由模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/http"  // 导入HTTP模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/mux"  // 导入MUX模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/router"  // 导入路由模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/shadowsocks"  // 导入shadowsocks模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/simplesocks"  // 导入simplesocks模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/socks"  // 导入socks模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/tls"  // 导入TLS模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/tproxy"  // 导入tproxy模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入传输模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/trojan"  // 导入trojan模块
    _ "github.com/p4gefau1t/trojan-go/tunnel/websocket"  // 导入websocket模块
)
```