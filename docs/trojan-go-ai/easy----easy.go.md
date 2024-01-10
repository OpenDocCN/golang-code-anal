# `trojan-go\easy\easy.go`

```
package easy

import (
    "encoding/json"  // 导入 JSON 编解码包
    "flag"  // 导入命令行参数解析包
    "net"  // 导入网络通信包
    "strconv"  // 导入字符串转换包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义的 common 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义的 log 包
    "github.com/p4gefau1t/trojan-go/option"  // 导入自定义的 option 包
    "github.com/p4gefau1t/trojan-go/proxy"  // 导入自定义的 proxy 包
)

type easy struct {
    server   *bool  // 服务器模式标志指针
    client   *bool  // 客户端模式标志指针
    password *string  // 密码指针
    local    *string  // 本地地址指针
    remote   *string  // 远程地址指针
    cert     *string  // 证书文件路径指针
    key      *string  // 密钥文件路径指针
}

type ClientConfig struct {
    RunType    string   `json:"run_type"`  // 运行类型
    LocalAddr  string   `json:"local_addr"`  // 本地地址
    LocalPort  int      `json:"local_port"`  // 本地端口
    RemoteAddr string   `json:"remote_addr"`  // 远程地址
    RemotePort int      `json:"remote_port"`  // 远程端口
    Password   []string `json:"password"`  // 密码列表
}

type TLS struct {
    SNI  string `json:"sni"`  // SNI
    Cert string `json:"cert"`  // 证书文件路径
    Key  string `json:"key"`  // 密钥文件路径
}

type ServerConfig struct {
    RunType    string   `json:"run_type"`  // 运行类型
    LocalAddr  string   `json:"local_addr"`  // 本地地址
    LocalPort  int      `json:"local_port"`  // 本地端口
    RemoteAddr string   `json:"remote_addr"`  // 远程地址
    RemotePort int      `json:"remote_port"`  // 远程端口
    Password   []string `json:"password"`  // 密码列表
    TLS        `json:"ssl"`  // TLS 配置
}

func (o *easy) Name() string {
    return "easy"  // 返回名称为 "easy"
}

func (o *easy) Handle() error {
    if !*o.server && !*o.client {  // 如果既不是服务器模式也不是客户端模式
        return common.NewError("empty")  // 返回一个自定义错误
    }
    if *o.password == "" {  // 如果密码为空
        log.Fatal("empty password is not allowed")  // 记录致命错误日志
    }
    log.Info("easy mode enabled, trojan-go will NOT use the config file")  // 记录信息日志，启用 easy 模式，trojan-go 将不使用配置文件
}
    # 如果客户端不为空
    if *o.client {
        # 如果本地地址为空
        if *o.local == "" {
            # 输出警告信息并使用默认地址
            log.Warn("client local addr is unspecified, using 127.0.0.1:1080")
            *o.local = "127.0.0.1:1080"
        }
        # 分割本地地址的主机和端口
        localHost, localPortStr, err := net.SplitHostPort(*o.local)
        # 如果出现错误，输出错误信息并终止程序
        if err != nil {
            log.Fatal(common.NewError("invalid local addr format:" + *o.local).Base(err))
        }
        # 分割远程地址的主机和端口
        remoteHost, remotePortStr, err := net.SplitHostPort(*o.remote)
        # 如果出现错误，输出错误信息并终止程序
        if err != nil {
            log.Fatal(common.NewError("invalid remote addr format:" + *o.remote).Base(err))
        }
        # 将本地端口转换为整数
        localPort, err := strconv.Atoi(localPortStr)
        # 如果出现错误，输出错误信息并终止程序
        if err != nil {
            log.Fatal(err)
        }
        # 将远程端口转换为整数
        remotePort, err := strconv.Atoi(remotePortStr)
        # 如果出现错误，输出错误信息并终止程序
        if err != nil {
            log.Fatal(err)
        }
        # 创建客户端配置对象
        clientConfig := ClientConfig{
            RunType:    "client",
            LocalAddr:  localHost,
            LocalPort:  localPort,
            RemoteAddr: remoteHost,
            RemotePort: remotePort,
            Password: []string{
                *o.password,
            },
        }
        # 将客户端配置对象转换为 JSON 格式
        clientConfigJSON, err := json.Marshal(&clientConfig)
        # 如果出现错误，终止程序
        common.Must(err)
        # 输出生成的配置信息
        log.Info("generated config:")
        log.Info(string(clientConfigJSON))
        # 根据配置信息创建代理对象
        proxy, err := proxy.NewProxyFromConfigData(clientConfigJSON, true)
        # 如果出现错误，终止程序
        if err != nil {
            log.Fatal(err)
        }
        # 运行代理
        if err := proxy.Run(); err != nil {
            log.Fatal(err)
        }
    }
    } else if *o.server {  # 如果 o.server 为真
        if *o.remote == "" {  # 如果 o.remote 为空
            log.Warn("server remote addr is unspecified, using 127.0.0.1:80")  # 记录警告日志，服务器远程地址未指定，使用默认地址 127.0.0.1:80
            *o.remote = "127.0.0.1:80"  # 设置 o.remote 为默认地址 127.0.0.1:80
        }
        if *o.local == "" {  # 如果 o.local 为空
            log.Warn("server local addr is unspecified, using 0.0.0.0:443")  # 记录警告日志，服务器本地地址未指定，使用默认地址 0.0.0.0:443
            *o.local = "0.0.0.0:443"  # 设置 o.local 为默认地址 0.0.0.0:443
        }
        localHost, localPortStr, err := net.SplitHostPort(*o.local)  # 使用 net.SplitHostPort() 函数分割 o.local 的主机和端口
        if err != nil {  # 如果出现错误
            log.Fatal(common.NewError("invalid local addr format:" + *o.local).Base(err))  # 记录致命错误日志，本地地址格式无效，并包含错误信息
        }
        remoteHost, remotePortStr, err := net.SplitHostPort(*o.remote)  # 使用 net.SplitHostPort() 函数分割 o.remote 的主机和端口
        if err != nil {  # 如果出现错误
            log.Fatal(common.NewError("invalid remote addr format:" + *o.remote).Base(err))  # 记录致命错误日志，远程地址格式无效，并包含错误信息
        }
        localPort, err := strconv.Atoi(localPortStr)  # 将 localPortStr 转换为整数
        if err != nil:  # 如果出现错误
            log.Fatal(err)  # 记录致命错误日志，并包含错误信息
        }
        remotePort, err := strconv.Atoi(remotePortStr)  # 将 remotePortStr 转换为整数
        if err != nil:  # 如果出现错误
            log.Fatal(err)  # 记录致命错误日志，并包含错误信息
        }
        serverConfig := ServerConfig{  # 创建 ServerConfig 结构体对象
            RunType:    "server",  # 设置 RunType 字段为 "server"
            LocalAddr:  localHost,  # 设置 LocalAddr 字段为 localHost
            LocalPort:  localPort,  # 设置 LocalPort 字段为 localPort
            RemoteAddr: remoteHost,  # 设置 RemoteAddr 字段为 remoteHost
            RemotePort: remotePort,  # 设置 RemotePort 字段为 remotePort
            Password: []string{  # 设置 Password 字段为包含 o.password 的字符串数组
                *o.password,
            },
            TLS: TLS{  # 创建 TLS 结构体对象
                Cert: *o.cert,  # 设置 Cert 字段为 o.cert
                Key:  *o.key,  # 设置 Key 字段为 o.key
            },
        }
        serverConfigJSON, err := json.Marshal(&serverConfig)  # 将 serverConfig 转换为 JSON 格式
        common.Must(err)  # 如果出现错误，立即终止程序
        log.Info("generated json config:")  # 记录信息日志，生成的 JSON 配置
        log.Info(string(serverConfigJSON))  # 记录信息日志，生成的 JSON 配置内容
        proxy, err := proxy.NewProxyFromConfigData(serverConfigJSON, true)  # 使用配置数据创建代理对象
        if err != nil {  # 如果出现错误
            log.Fatal(err)  # 记录致命错误日志，并包含错误信息
        }
        if err := proxy.Run(); err != nil {  # 如果代理运行出现错误
            log.Fatal(err)  # 记录致命错误日志，并包含错误信息
        }
    }
    return nil  # 返回空值
# 定义 easy 结构体的 Priority 方法，返回优先级为 50
func (o *easy) Priority() int {
    return 50
}

# 初始化函数，在程序启动时注册 easy 结构体的实例
func init() {
    # 注册 easy 结构体的实例，包括 server、client、password、remote、local、key、cert 等字段
    option.RegisterHandler(&easy{
        server:   flag.Bool("server", false, "Run a trojan-go server"),
        client:   flag.Bool("client", false, "Run a trojan-go client"),
        password: flag.String("password", "", "Password for authentication"),
        remote:   flag.String("remote", "", "Remote address, e.g. 127.0.0.1:12345"),
        local:    flag.String("local", "", "Local address, e.g. 127.0.0.1:12345"),
        key:      flag.String("key", "server.key", "Key of the server"),
        cert:     flag.String("cert", "server.crt", "Certificates of the server"),
    })
}
```