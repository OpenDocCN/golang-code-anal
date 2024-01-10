# `trojan-go\url\option.go`

```
package url

import (
    "encoding/json"  // 导入 JSON 编解码包
    "flag"  // 导入命令行参数解析包
    "net"  // 导入网络操作包
    "strconv"  // 导入字符串转换包
    "strings"  // 导入字符串操作包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/option"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/proxy"  // 导入自定义包
)

const Name = "URL"  // 定义常量 Name 为 "URL"

type Websocket struct {  // 定义结构体 Websocket
    Enabled bool   `json:"enabled"`  // 布尔类型字段 Enabled，用于 JSON 编解码
    Host    string `json:"host"`  // 字符串类型字段 Host，用于 JSON 编解码
    Path    string `json:"path"`  // 字符串类型字段 Path，用于 JSON 编解码
}

type TLS struct {  // 定义结构体 TLS
    SNI string `json:"sni"`  // 字符串类型字段 SNI，用于 JSON 编解码
}

type Shadowsocks struct {  // 定义结构体 Shadowsocks
    Enabled  bool   `json:"enabled"`  // 布尔类型字段 Enabled，用于 JSON 编解码
    Method   string `json:"method"`  // 字符串类型字段 Method，用于 JSON 编解码
    Password string `json:"password"`  // 字符串类型字段 Password，用于 JSON 编解码
}

type Mux struct {  // 定义结构体 Mux
    Enabled bool `json:"enabled"`  // 布尔类型字段 Enabled，用于 JSON 编解码
}

type API struct {  // 定义结构体 API
    Enabled bool   `json:"enabled"`  // 布尔类型字段 Enabled，用于 JSON 编解码
    APIHost string `json:"api_addr"`  // 字符串类型字段 APIHost，用于 JSON 编解码
    APIPort int    `json:"api_port"`  // 整数类型字段 APIPort，用于 JSON 编解码
}

type UrlConfig struct {  // 定义结构体 UrlConfig
    RunType     string   `json:"run_type"`  // 字符串类型字段 RunType，用于 JSON 编解码
    LocalAddr   string   `json:"local_addr"`  // 字符串类型字段 LocalAddr，用于 JSON 编解码
    LocalPort   int      `json:"local_port"`  // 整数类型字段 LocalPort，用于 JSON 编解码
    RemoteAddr  string   `json:"remote_addr"`  // 字符串类型字段 RemoteAddr，用于 JSON 编解码
    RemotePort  int      `json:"remote_port"`  // 整数类型字段 RemotePort，用于 JSON 编解码
    Password    []string `json:"password"`  // 字符串数组类型字段 Password，用于 JSON 编解码
    Websocket   `json:"websocket"`  // 嵌套结构体 Websocket，用于 JSON 编解码
    Shadowsocks `json:"shadowsocks"`  // 嵌套结构体 Shadowsocks，用于 JSON 编解码
    TLS         `json:"ssl"`  // 嵌套结构体 TLS，用于 JSON 编解码
    Mux         `json:"mux"`  // 嵌套结构体 Mux，用于 JSON 编解码
    API         `json:"api"`  // 嵌套结构体 API，用于 JSON 编解码
}

type url struct {  // 定义结构体 url
    url    *string  // 字符串指针类型字段 url
    option *string  // 字符串指针类型字段 option
}

func (u *url) Name() string {  // 定义结构体方法 Name，返回字符串类型
    return Name  // 返回常量 Name
}

func (u *url) Handle() error {  // 定义结构体方法 Handle，返回错误类型
    if u.url == nil || *u.url == "" {  // 判断字段 url 是否为空或者值为空字符串
        return common.NewError("")  // 返回自定义错误
    }
    info, err := NewShareInfoFromURL(*u.url)  // 调用函数 NewShareInfoFromURL，返回值赋给 info 和 err
    if err != nil {  // 判断 err 是否为空
        log.Fatal(err)  // 输出错误日志并终止程序
    }
    wsEnabled := false  // 声明并初始化布尔类型变量 wsEnabled
    if info.Type == ShareInfoTypeWebSocket {  // 判断 info 的 Type 字段是否为 ShareInfoTypeWebSocket
        wsEnabled = true  // 如果是，则将 wsEnabled 设置为 true
    }
    ssEnabled := false  // 声明并初始化布尔类型变量 ssEnabled
    ssPassword := ""  // 声明并初始化字符串类型变量 ssPassword
    ssMethod := ""  // 声明并初始化字符串类型变量 ssMethod
    # 如果加密信息以"ss;"开头，则启用shadowsocks加密
    if strings.HasPrefix(info.Encryption, "ss;"):
        ssEnabled = true
        # 分割加密信息，获取加密方法和密码
        ssConfig := strings.Split(info.Encryption[3:], ":")
        # 如果分割后的长度不为2，则输出错误信息
        if len(ssConfig) != 2:
            log.Fatalf("invalid shadowsocks config: %s", info.Encryption)
        ssMethod = ssConfig[0]
        ssPassword = ssConfig[1]
    
    # 初始化muxEnabled为false
    muxEnabled := false
    # 初始化监听主机为"127.0.0.1"
    listenHost := "127.0.0.1"
    # 初始化监听端口为1080
    listenPort := 1080
    
    # 初始化apiEnabled为false
    apiEnabled := false
    # 初始化API主机为"127.0.0.1"
    apiHost := "127.0.0.1"
    # 初始化API端口为10000
    apiPort := 10000
    
    # 分割选项字符串，遍历每个选项
    options := strings.Split(*u.option, ";")
    for _, o := range options:
        key := ""
        val := ""
        l := strings.Split(o, "=")
        # 如果分割后的长度不为2，则输出错误信息
        if len(l) != 2:
            log.Fatal("option format error, no \"key=value\" pair found:", o)
        key = l[0]
        val = l[1]
        # 根据选项的key进行不同的处理
        switch key:
            # 如果是mux选项，则解析为布尔值
            case "mux":
                muxEnabled, err = strconv.ParseBool(val)
                if err != nil:
                    log.Fatal(err)
            # 如果是listen选项，则解析主机和端口
            case "listen":
                h, p, err := net.SplitHostPort(val)
                if err != nil:
                    log.Fatal(err)
                listenHost = h
                lp, err := strconv.Atoi(p)
                if err != nil:
                    log.Fatal(err)
                listenPort = lp
            # 如果是api选项，则启用API并解析主机和端口
            case "api":
                apiEnabled = true
                h, p, err := net.SplitHostPort(val)
                if err != nil:
                    log.Fatal(err)
                apiHost = h
                lp, err := strconv.Atoi(p)
                if err != nil:
                    log.Fatal(err)
                apiPort = lp
            # 如果是其他选项，则输出错误信息
            default:
                log.Fatal("invalid option", o)
    # 创建 UrlConfig 结构体实例，用于配置客户端连接信息
    config := UrlConfig{
        RunType:    "client",  # 设置运行类型为客户端
        LocalAddr:  listenHost,  # 设置本地地址为监听主机
        LocalPort:  listenPort,  # 设置本地端口为监听端口
        RemoteAddr: info.TrojanHost,  # 设置远程地址为 Trojan 主机地址
        RemotePort: int(info.Port),  # 设置远程端口为 info.Port 的整数值
        Password:   []string{info.TrojanPassword},  # 设置密码为包含 info.TrojanPassword 的字符串切片
        TLS: TLS{  # 配置 TLS
            SNI: info.SNI,  # 设置 SNI 为 info.SNI
        },
        Websocket: Websocket{  # 配置 Websocket
            Enabled: wsEnabled,  # 设置启用状态为 wsEnabled
            Path:    info.Path,  # 设置路径为 info.Path
            Host:    info.Host,  # 设置主机为 info.Host
        },
        Mux: Mux{  # 配置 Mux
            Enabled: muxEnabled,  # 设置启用状态为 muxEnabled
        },
        Shadowsocks: Shadowsocks{  # 配置 Shadowsocks
            Enabled:  ssEnabled,  # 设置启用状态为 ssEnabled
            Password: ssPassword,  # 设置密码为 ssPassword
            Method:   ssMethod,  # 设置加密方法为 ssMethod
        },
        API: API{  # 配置 API
            Enabled: apiEnabled,  # 设置启用状态为 apiEnabled
            APIHost: apiHost,  # 设置 API 主机为 apiHost
            APIPort: apiPort,  # 设置 API 端口为 apiPort
        },
    }
    # 将配置信息转换为 JSON 格式的数据
    data, err := json.Marshal(&config)
    # 如果转换过程中出现错误，则记录错误并退出程序
    if err != nil {
        log.Fatal(err)
    }
    # 记录转换后的 JSON 数据
    log.Debug(string(data))
    # 根据配置信息创建代理客户端
    client, err := proxy.NewProxyFromConfigData(data, true)
    # 如果创建过程中出现错误，则记录错误并退出程序
    if err != nil {
        log.Fatal(err)
    }
    # 运行代理客户端
    return client.Run()
# 定义一个方法，返回优先级为10
func (u *url) Priority() int {
    return 10
}

# 初始化方法，在程序启动时注册处理程序
func init() {
    # 注册处理程序，传入一个url结构体对象
    option.RegisterHandler(&url{
        # 设置url字段，使用flag包获取命令行参数，参数名为"url"，默认值为空字符串，描述为"Setup trojan-go client with a url link"
        url:    flag.String("url", "", "Setup trojan-go client with a url link"),
        # 设置option字段，使用flag包获取命令行参数，参数名为"url-option"，默认值为"mux=true;listen=127.0.0.1:1080"，描述为"URL mode options"
        option: flag.String("url-option", "mux=true;listen=127.0.0.1:1080", "URL mode options"),
    })
}
```