# `trojan-go\test\scenario\custom_test.go`

```go
package scenario

import (
    "fmt"
    "testing"

    "github.com/p4gefau1t/trojan-go/common"
    _ "github.com/p4gefau1t/trojan-go/proxy/custom"
    "github.com/p4gefau1t/trojan-go/test/util"
)

func TestCustom1(t *testing.T) {
    // 为服务器和 SOCKS 代理选择一个空闲端口
    serverPort := common.PickPort("tcp", "127.0.0.1")
    socksPort := common.PickPort("tcp", "127.0.0.1")
    // 生成客户端配置数据
    clientData := fmt.Sprintf(`
run-type: custom

inbound:
  node:
    - protocol: adapter
      tag: adapter
      config:
        local-addr: 127.0.0.1
        local-port: %d
    - protocol: socks
      tag: socks
      config:
        local-addr: 127.0.0.1
        local-port: %d
  path:
    -
      - adapter
      - socks

outbound:
  node:
    - protocol: transport
      tag: transport
      config:
        remote-addr: 127.0.0.1
        remote-port: %d

    - protocol: tls
      tag: tls
      config:
        ssl:
          sni: localhost
          key: server.key
          cert: server.crt

    - protocol: trojan
      tag: trojan
      config:
        password:
          - "12345678"

  path:
    - 
      - transport
      - tls
      - trojan

`, socksPort, socksPort, serverPort)
    // 生成服务器配置数据
    serverData := fmt.Sprintf(`
run-type: custom

inbound:
  node:
    - protocol: transport
      tag: transport
      config:
        local-addr: 127.0.0.1
        local-port: %d
        remote-addr: 127.0.0.1
        remote-port: %s

    - protocol: tls
      tag: tls
      config:
        ssl:
          sni: localhost
          key: server.key
          cert: server.crt

    - protocol: trojan
      tag: trojan
      config:
        disable-http-check: true
        password:
          - "12345678"

    - protocol: mux
      tag: mux

    - protocol: simplesocks
      tag: simplesocks
     

  path:
    - 
      - transport
      - tls
      - trojan
    - 
      - transport
      - tls
      - trojan
      - mux
      - simplesocks

outbound:
  node:
    - protocol: freedom
      tag: freedom

  path:
    - 
      - freedom
    // 为服务器和客户端选择一个空闲的端口
    serverPort := common.PickPort("tcp", "127.0.0.1")
    socksPort := common.PickPort("tcp", "127.0.0.1")
    // 格式化客户端数据，包括自定义运行类型、日志级别、入站和出站节点配置
    clientData := fmt.Sprintf(`
run-type: custom
log-level: 0

inbound:
  node:
    - protocol: adapter
      tag: adapter
      config:
        local-addr: 127.0.0.1
        local-port: %d
    - protocol: socks
      tag: socks
      config:
        local-addr: 127.0.0.1
        local-port: %d
  path:
    -
      - adapter
      - socks

outbound:
  node:
    - protocol: transport
      tag: transport
      config:
        remote-addr: 127.0.0.1
        remote-port: %d

    - protocol: tls
      tag: tls
      config:
        ssl:
          sni: localhost
          key: server.key
          cert: server.crt

    - protocol: trojan
      tag: trojan
      config:
        password:
          - "12345678"

    - protocol: shadowsocks
      tag: shadowsocks
      config:
        remote-addr: 127.0.0.1
        remote-port: 80
        shadowsocks:
          enabled: true
          password: "12345678"

    - protocol: websocket
      tag: websocket
      config:
        websocket:
          host: localhost
          path: /ws

  path:
    - 
      - transport
      - tls
      - websocket
      - shadowsocks 
      - trojan

`, socksPort, socksPort, serverPort)
    // 格式化服务器数据，包括自定义运行类型、日志级别、入站和出站节点配置
    serverData := fmt.Sprintf(`
run-type: custom
log-level: 0

inbound:
  node:
    - protocol: transport
      tag: transport
      config:
        local-addr: 127.0.0.1
        local-port: %d
        remote-addr: 127.0.0.1
        remote-port: %s

    - protocol: tls
      tag: tls
      config:
        ssl:
          sni: localhost
          key: server.key
          cert: server.crt

    - protocol: trojan
      tag: trojan
      config:
        disable-http-check: true
        password:
          - "12345678"
    # 配置 Trojan 协议
    - protocol: trojan
      tag: trojan2
      config:
        # 禁用 HTTP 检查
        disable-http-check: true
        # 设置密码
        password:
          - "12345678"
    
    # 配置 WebSocket 协议
    - protocol: websocket
      tag: websocket
      config:
        websocket:
          # 启用 WebSocket
          enabled: true
          # 设置主机
          host: localhost
          # 设置路径
          path: /ws
    
    # 配置 Mux 协议
    - protocol: mux
      tag: mux
    
    # 配置 SimpleSocks 协议
    - protocol: simplesocks
      tag: simplesocks
    
    # 配置 Shadowsocks 协议
    - protocol: shadowsocks
      tag: shadowsocks
      config:
        # 设置远程地址
        remote-addr: 127.0.0.1
        # 设置远程端口
        remote-port: 80
        shadowsocks:
          # 启用 Shadowsocks
          enabled: true
          # 设置密码
          password: "12345678"
    
    # 配置另一个 Shadowsocks 协议
    - protocol: shadowsocks
      tag: shadowsocks2
      config:
        # 设置远程地址
        remote-addr: 127.0.0.1
        # 设置远程端口
        remote-port: 80
        shadowsocks:
          # 启用 Shadowsocks
          enabled: true
          # 设置密码
          password: "12345678"
    
    # 配置路径
    path:
      - 
        # 设置传输层安全协议和协议顺序
        - transport
        - tls
        - shadowsocks 
        - trojan
      - 
        # 设置传输层安全协议和协议顺序
        - transport
        - tls
        - websocket
        - shadowsocks2
        - trojan2
      - 
        # 设置传输层安全协议和协议顺序
        - transport
        - tls
        - shadowsocks
        - trojan
        - mux
        - simplesocks
# 定义出站流量的配置，使用 freedom 协议，并设置标签为 freedom
outbound:
  node:
    - protocol: freedom
      tag: freedom

  # 设置路径为 freedom
  path:
    - 
      - freedom

# 使用 serverPort 和 util.HTTPPort 替换代码中的占位符
`, serverPort, util.HTTPPort)

# 如果客户端和服务器端数据不匹配，则测试失败
if !CheckClientServer(clientData, serverData, socksPort) {
    t.Fail()
}
```