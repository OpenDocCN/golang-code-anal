# `v2ray-core\main\main.go`

```
package main

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "flag" // 导入 flag 包，用于命令行参数解析
    "fmt" // 导入 fmt 包，用于格式化输出
    "io/ioutil" // 导入 ioutil 包，用于读取文件
    "log" // 导入 log 包，用于日志记录
    "os" // 导入 os 包，用于操作系统功能
    "os/signal" // 导入 signal 包，用于信号处理
    "path" // 导入 path 包，用于处理文件路径
    "path/filepath" // 导入 filepath 包，用于处理文件路径
    "runtime" // 导入 runtime 包，用于运行时信息
    "strings" // 导入 strings 包，用于字符串处理
    "syscall" // 导入 syscall 包，用于系统调用

    "v2ray.com/core" // 导入 v2ray 核心包
    "v2ray.com/core/common/cmdarg" // 导入 v2ray 命令行参数包
    "v2ray.com/core/common/platform" // 导入 v2ray 平台包
    _ "v2ray.com/core/main/distro/all" // 导入 v2ray 主包
)

var (
    configFiles cmdarg.Arg // "Config file for V2Ray.", the option is customed type, parse in main
    configDir   string // 配置文件目录
    version     = flag.Bool("version", false, "Show current version of V2Ray.") // 定义 version 命令行参数
    test        = flag.Bool("test", false, "Test config file only, without launching V2Ray server.") // 定义 test 命令行参数
    format      = flag.String("format", "json", "Format of input file.") // 定义 format 命令行参数

    /* We have to do this here because Golang's Test will also need to parse flag, before
     * main func in this file is run.
     */
    _ = func() error { // 匿名函数，用于初始化命令行参数
        flag.Var(&configFiles, "config", "Config file for V2Ray. Multiple assign is accepted (only json). Latter ones overrides the former ones.") // 定义 config 命令行参数
        flag.Var(&configFiles, "c", "Short alias of -config") // 定义 c 命令行参数
        flag.StringVar(&configDir, "confdir", "", "A dir with multiple json config") // 定义 confdir 命令行参数

        return nil
    }()
)

func fileExists(file string) bool { // 判断文件是否存在
    info, err := os.Stat(file) // 获取文件信息
    return err == nil && !info.IsDir() // 返回文件是否存在
}

func dirExists(file string) bool { // 判断目录是否存在
    if file == "" {
        return false
    }
    info, err := os.Stat(file) // 获取文件信息
    return err == nil && info.IsDir() // 返回目录是否存在
}

func readConfDir(dirPath string) { // 读取配置目录
    confs, err := ioutil.ReadDir(dirPath) // 读取目录下的文件
    if err != nil {
        log.Fatalln(err) // 如果出错，记录错误并退出
    }
    for _, f := range confs { // 遍历目录下的文件
        if strings.HasSuffix(f.Name(), ".json") { // 如果文件以 .json 结尾
            configFiles.Set(path.Join(dirPath, f.Name())) // 将文件路径添加到配置文件列表中
        }
    }
}

func getConfigFilePath() (cmdarg.Arg, error) { // 获取配置文件路径
    if dirExists(configDir) { // 如果配置目录存在
        log.Println("Using confdir from arg:", configDir) // 记录使用的配置目录
        readConfDir(configDir) // 读取配置目录下的文件
    } else {
        // 如果环境变量中存在配置目录路径，则使用该路径
        if envConfDir := platform.GetConfDirPath(); dirExists(envConfDir) {
            // 打印日志，使用环境变量中的配置目录路径
            log.Println("Using confdir from env:", envConfDir)
            // 读取配置目录中的配置文件
            readConfDir(envConfDir)
        }
    }

    // 如果配置文件列表不为空，则返回配置文件列表
    if len(configFiles) > 0 {
        return configFiles, nil
    }

    // 获取当前工作目录路径
    if workingDir, err := os.Getwd(); err == nil {
        // 拼接默认配置文件路径
        configFile := filepath.Join(workingDir, "config.json")
        // 如果默认配置文件存在，则返回默认配置文件路径
        if fileExists(configFile) {
            // 打印日志，使用默认配置文件
            log.Println("Using default config: ", configFile)
            return cmdarg.Arg{configFile}, nil
        }
    }

    // 获取平台配置文件路径
    if configFile := platform.GetConfigurationPath(); fileExists(configFile) {
        // 打印日志，使用环境变量中的配置文件
        log.Println("Using config from env: ", configFile)
        return cmdarg.Arg{configFile}, nil
    }

    // 打印日志，使用标准输入的配置
    log.Println("Using config from STDIN")
    return cmdarg.Arg{"stdin:"}, nil
}

func GetConfigFormat() string {
    // 根据输入的格式字符串，返回配置文件的格式
    switch strings.ToLower(*format) {
    case "pb", "protobuf":
        return "protobuf"
    default:
        return "json"
    }
}

func startV2Ray() (core.Server, error) {
    // 获取配置文件路径
    configFiles, err := getConfigFilePath()
    if err != nil {
        return nil, err
    }

    // 加载配置文件
    config, err := core.LoadConfig(GetConfigFormat(), configFiles[0], configFiles)
    if err != nil {
        return nil, newError("failed to read config files: [", configFiles.String(), "]").Base(err)
    }

    // 创建 V2Ray 服务器
    server, err := core.New(config)
    if err != nil {
        return nil, newError("failed to create server").Base(err)
    }

    return server, nil
}

func printVersion() {
    // 打印 V2Ray 版本信息
    version := core.VersionStatement()
    for _, s := range version {
        fmt.Println(s)
    }
}

func main() {

    flag.Parse()

    printVersion()

    if *version {
        return
    }

    // 启动 V2Ray 服务器
    server, err := startV2Ray()
    if err != nil {
        fmt.Println(err)
        // 配置错误。使用特殊值阻止 systemd 重新启动
        os.Exit(23)
    }

    if *test {
        fmt.Println("Configuration OK.")
        os.Exit(0)
    }

    // 启动服务器
    if err := server.Start(); err != nil {
        fmt.Println("Failed to start", err)
        os.Exit(-1)
    }
    defer server.Close()

    // 显式触发 GC 以删除配置加载时产生的垃圾
    runtime.GC()

    {
        // 创建一个接收操作系统信号的通道
        osSignals := make(chan os.Signal, 1)
        // 监听中断信号和 SIGTERM 信号
        signal.Notify(osSignals, os.Interrupt, syscall.SIGTERM)
        // 阻塞等待信号
        <-osSignals
    }
}
```