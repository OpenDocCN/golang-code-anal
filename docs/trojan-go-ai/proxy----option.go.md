# `trojan-go\proxy\option.go`

```go
package proxy

import (
    "bufio"  // 导入 bufio 包，提供了缓冲 I/O 的功能
    "flag"   // 导入 flag 包，用于命令行参数的解析
    "fmt"    // 导入 fmt 包，提供了格式化 I/O 的功能
    "io/ioutil"  // 导入 ioutil 包，提供了 I/O 实用函数
    "os"     // 导入 os 包，提供了操作系统函数
    "runtime"  // 导入 runtime 包，提供了运行时的信息、操作函数
    "strings"  // 导入 strings 包，提供了操作字符串的函数

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/constant"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/option"  // 导入自定义包
)

type Option struct {
    path *string  // 定义 Option 结构体的 path 字段
}

func (o *Option) Name() string {
    return Name  // 返回 Name 字段的值
}

func detectAndReadConfig(file string) ([]byte, bool, error) {
    isJSON := false  // 初始化 isJSON 变量为 false
    switch {
    case strings.HasSuffix(file, ".json"):  // 如果文件名以 .json 结尾
        isJSON = true  // 将 isJSON 设置为 true
    case strings.HasSuffix(file, ".yaml"), strings.HasSuffix(file, ".yml"):  // 如果文件名以 .yaml 或 .yml 结尾
        isJSON = false  // 将 isJSON 设置为 false
    default:
        log.Fatalf("unsupported config format: %s. use .yaml or .json instead.", file)  // 输出错误信息并终止程序
    }

    data, err := ioutil.ReadFile(file)  // 读取文件内容
    if err != nil {
        return nil, false, err  // 如果读取文件出错，返回 nil、false 和错误信息
    }
    return data, isJSON, nil  // 返回文件内容、isJSON 和 nil
}

func (o *Option) Handle() error {
    defaultConfigPath := []string{  // 定义默认的配置文件路径列表
        "config.json",
        "config.yml",
        "config.yaml",
    }

    isJSON := false  // 初始化 isJSON 变量为 false
    var data []byte  // 声明 data 变量为字节切片
    var err error  // 声明 err 变量为错误类型

    switch *o.path {  // 根据 o.path 的值进行判断
    case "":
        log.Warn("no specified config file, use default path to detect config file")  // 输出警告信息
        for _, file := range defaultConfigPath {  // 遍历默认配置文件路径列表
            log.Warn("try to load config from default path:", file)  // 输出警告信息
            data, isJSON, err = detectAndReadConfig(file)  // 调用 detectAndReadConfig 函数读取配置文件
            if err != nil {
                log.Warn(err)  // 输出警告信息
                continue  // 继续下一次循环
            }
            break  // 跳出循环
        }
    default:
        data, isJSON, err = detectAndReadConfig(*o.path)  // 调用 detectAndReadConfig 函数读取指定路径的配置文件
        if err != nil {
            log.Fatal(err)  // 输出错误信息并终止程序
        }
    }

    if data != nil {  // 如果读取到了配置文件内容
        log.Info("trojan-go", constant.Version, "initializing")  // 输出信息
        proxy, err := NewProxyFromConfigData(data, isJSON)  // 根据配置文件内容创建代理
        if err != nil {
            log.Fatal(err)  // 输出错误信息并终止程序
        }
        err = proxy.Run()  // 运行代理
        if err != nil {
            log.Fatal(err)  // 输出错误信息并终止程序
        }
    }

    log.Fatal("no valid config")  // 输出错误信息并终止程序
    return nil  // 返回 nil
}
# 定义 Option 结构体的 Priority 方法，返回固定值 -1
func (o *Option) Priority() int {
    return -1
}

# 初始化函数，注册 Option 和 StdinOption 对象的处理器
func init() {
    option.RegisterHandler(&Option{
        path: flag.String("config", "", "Trojan-Go config filename (.yaml/.yml/.json)"),
    })
    option.RegisterHandler(&StdinOption{
        format:       flag.String("stdin-format", "disabled", "Read from standard input (yaml/json)"),
        suppressHint: flag.Bool("stdin-suppress-hint", false, "Suppress hint text"),
    })
}

# 定义 StdinOption 结构体
type StdinOption struct {
    format       *string
    suppressHint *bool
}

# 定义 StdinOption 结构体的 Name 方法，返回特定字符串
func (o *StdinOption) Name() string {
    return Name + "_STDIN"
}

# 定义 StdinOption 结构体的 Handle 方法，处理从标准输入读取配置的逻辑
func (o *StdinOption) Handle() error {
    # 判断输入格式是否为 JSON
    isJSON, e := o.isFormatJson()
    if e != nil {
        return e
    }

    # 打印提示信息
    if o.suppressHint == nil || !*o.suppressHint {
        fmt.Printf("Trojan-Go %s (%s/%s)\n", constant.Version, runtime.GOOS, runtime.GOARCH)
        if isJSON {
            fmt.Println("Reading JSON configuration from stdin.")
        } else {
            fmt.Println("Reading YAML configuration from stdin.")
        }
    }

    # 从标准输入读取数据
    data, e := ioutil.ReadAll(bufio.NewReader(os.Stdin))
    if e != nil {
        log.Fatalf("Failed to read from stdin: %s", e.Error())
    }

    # 根据读取的数据创建代理对象
    proxy, err := NewProxyFromConfigData(data, isJSON)
    if err != nil {
        log.Fatal(err)
    }
    # 运行代理
    err = proxy.Run()
    if err != nil {
        log.Fatal(err)
    }

    return nil
}

# 定义 StdinOption 结构体的 Priority 方法，返回固定值 0
func (o *StdinOption) Priority() int {
    return 0
}

# 定义 StdinOption 结构体的 isFormatJson 方法，判断输入格式是否为 JSON
func (o *StdinOption) isFormatJson() (isJson bool, e error) {
    if o.format == nil {
        return false, common.NewError("format specifier is nil")
    }
    if *o.format == "disabled" {
        return false, common.NewError("reading from stdin is disabled")
    }
    return strings.ToLower(*o.format) == "json", nil
}
```