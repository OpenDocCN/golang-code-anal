# `v2ray-core\infra\control\config.go`

```
package control

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "io"  // 导入 io 包，用于进行 I/O 操作
    "io/ioutil"  // 导入 ioutil 包，用于读取文件内容
    "os"  // 导入 os 包，提供操作系统函数
    "strings"  // 导入 strings 包，用于处理字符串

    "github.com/golang/protobuf/proto"  // 导入 protobuf 包，用于处理 Protocol Buffers 数据
    "v2ray.com/core/common"  // 导入 common 包，V2Ray 核心通用功能
    "v2ray.com/core/infra/conf"  // 导入 conf 包，V2Ray 核心配置
    "v2ray.com/core/infra/conf/serial"  // 导入 serial 包，V2Ray 核心配置序列化
)

// ConfigCommand is the json to pb convert struct
type ConfigCommand struct{}  // 定义 ConfigCommand 结构体

// Name for cmd usage
func (c *ConfigCommand) Name() string {  // 定义 Name 方法，返回命令名称
    return "config"
}

// Description for help usage
func (c *ConfigCommand) Description() Description {  // 定义 Description 方法，返回命令描述
    return Description{  // 返回 Description 结构体
        Short: "merge multiple json config",  // 简短描述
        Usage: []string{"v2ctl config config.json c1.json c2.json <url>.json"},  // 用法示例
    }
}

// Execute real work here.
func (c *ConfigCommand) Execute(args []string) error {  // 定义 Execute 方法，执行实际操作
    if len(args) < 1 {  // 如果参数列表为空
        return newError("empty config list")  // 返回错误信息
    }

    conf := &conf.Config{}  // 创建 Config 对象
    for _, arg := range args {  // 遍历参数列表
        ctllog.Println("Read config: ", arg)  // 打印日志
        r, err := c.LoadArg(arg)  // 调用 LoadArg 方法加载参数
        common.Must(err)  // 检查错误
        c, err := serial.DecodeJSONConfig(r)  // 解码 JSON 配置
        if err != nil {  // 如果出现错误
            ctllog.Fatalln(err)  // 打印错误日志
        }
        conf.Override(c, arg)  // 覆盖配置
    }

    pbConfig, err := conf.Build()  // 构建配置
    if err != nil {  // 如果出现错误
        return err  // 返回错误信息
    }

    bytesConfig, err := proto.Marshal(pbConfig)  // 将配置序列化为字节流
    if err != nil {  // 如果出现错误
        return newError("failed to marshal proto config").Base(err)  // 返回错误信息
    }

    if _, err := os.Stdout.Write(bytesConfig); err != nil {  // 将字节流写入标准输出
        return newError("failed to write proto config").Base(err)  // 返回错误信息
    }

    return nil  // 返回空值
}

// LoadArg loads one arg, maybe an remote url, or local file path
func (c *ConfigCommand) LoadArg(arg string) (out io.Reader, err error) {  // 定义 LoadArg 方法，加载参数
    var data []byte  // 定义字节流变量
    if strings.HasPrefix(arg, "http://") || strings.HasPrefix(arg, "https://") {  // 如果参数以 "http://" 或 "https://" 开头
        data, err = FetchHTTPContent(arg)  // 获取远程 HTTP 内容
    } else if arg == "stdin:" {  // 如果参数为 "stdin:"
        data, err = ioutil.ReadAll(os.Stdin)  // 从标准输入读取内容
    } else {  // 否则
        data, err = ioutil.ReadFile(arg)  // 读取文件内容
    }

    if err != nil {  // 如果出现错误
        return  // 返回空值
    }
    out = bytes.NewBuffer(data)  // 创建字节流缓冲区
    return  // 返回结果
}

func init() {  // 初始化函数
    # 注册 ConfigCommand 命令，并确保注册成功
    common.Must(RegisterCommand(&ConfigCommand{}))
# 闭合前面的函数定义
```