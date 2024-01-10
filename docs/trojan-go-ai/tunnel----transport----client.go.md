# `trojan-go\tunnel\transport\client.go`

```
package transport

import (
    "context"  // 导入上下文包
    "os"  // 导入操作系统包
    "os/exec"  // 导入执行外部命令包
    "strconv"  // 导入字符串转换包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入自定义包
)

// Client implements tunnel.Client
type Client struct {
    serverAddress *tunnel.Address  // 定义服务器地址
    cmd           *exec.Cmd  // 定义外部命令
    ctx           context.Context  // 定义上下文
    cancel        context.CancelFunc  // 定义取消函数
    direct        *freedom.Client  // 定义自由客户端
}

func (c *Client) Close() error {
    c.cancel()  // 取消函数
    if c.cmd != nil && c.cmd.Process != nil {
        c.cmd.Process.Kill()  // 终止外部命令进程
    }
    return nil
}

func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    panic("not supported")  // 抛出异常，表示不支持该操作
}

// DialConn implements tunnel.Client. It will ignore the params and directly dial to the remote server
func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := c.direct.DialConn(c.serverAddress, nil)  // 使用自由客户端直接连接远程服务器
    if err != nil {
        return nil, common.NewError("transport failed to connect to remote server").Base(err)  // 如果连接失败，返回错误信息
    }
    return &Conn{
        Conn: conn,  // 返回连接对象
    }, nil
}

// NewClient creates a transport layer client
func NewClient(ctx context.Context, _ tunnel.Client) (*Client, error) {
    cfg := config.FromContext(ctx, Name).(*Config)  // 从上下文中获取配置信息

    var cmd *exec.Cmd  // 定义外部命令
    serverAddress := tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort)  // 创建服务器地址
    # 如果启用了传输插件
    if cfg.TransportPlugin.Enabled:
        # 输出警告信息
        log.Warn("trojan-go will use transport plugin and work in plain text mode")
        # 根据传输插件类型进行不同的处理
        switch cfg.TransportPlugin.Type:
            # 如果是 shadowsocks 类型的传输插件
            case "shadowsocks":
                # 设置插件的主机和端口
                pluginHost := "127.0.0.1"
                pluginPort := common.PickPort("tcp", pluginHost)
                # 将插件的环境变量添加到配置中
                cfg.TransportPlugin.Env = append(
                    cfg.TransportPlugin.Env,
                    "SS_LOCAL_HOST="+pluginHost,
                    "SS_LOCAL_PORT="+strconv.FormatInt(int64(pluginPort), 10),
                    "SS_REMOTE_HOST="+cfg.RemoteHost,
                    "SS_REMOTE_PORT="+strconv.FormatInt(int64(cfg.RemotePort), 10),
                    "SS_PLUGIN_OPTIONS="+cfg.TransportPlugin.Option,
                )
                # 更新远程主机和端口为插件的主机和端口
                cfg.RemoteHost = pluginHost
                cfg.RemotePort = pluginPort
                # 创建插件的服务器地址
                serverAddress = tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort)
                # 输出插件地址信息
                log.Debug("plugin address", serverAddress.String())
                # 输出插件环境信息
                log.Debug("plugin env", cfg.TransportPlugin.Env)

                # 执行传输插件的命令
                cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
                cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
                cmd.Stdout = os.Stdout
                cmd.Stderr = os.Stdout
                cmd.Start()
            # 如果是其他类型的传输插件
            case "other":
                # 执行传输插件的命令
                cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
                cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
                cmd.Stdout = os.Stdout
                cmd.Stderr = os.Stdout
                cmd.Start()
            # 如果是明文类型的传输插件
            case "plaintext":
                # 什么都不做
                // do nothing
            # 如果是其他类型的传输插件
            default:
                # 返回错误信息
                return nil, common.NewError("invalid plugin type: " + cfg.TransportPlugin.Type)
        # 创建直连客户端
        direct, err := freedom.NewClient(ctx, nil)
        # 必须处理错误
        common.Must(err)
        # 创建上下文和取消函数
        ctx, cancel := context.WithCancel(ctx)
        # 创建客户端对象
        client := &Client{
            serverAddress: serverAddress,
            cmd:           cmd,
            ctx:           ctx,
            cancel:        cancel,
            direct:        direct,
    # 返回客户端对象和空的错误对象
    return client, nil
# 闭合前面的函数定义
```