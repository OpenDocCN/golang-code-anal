# `v2ray-core\infra\control\command.go`

```
package control

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "log"  // 导入 log 包，用于日志记录
    "os"   // 导入 os 包，提供对操作系统功能的访问
    "strings"  // 导入 strings 包，提供对字符串的操作
)

type Description struct {
    Short string   // 描述结构体，包含 Short 字段，用于存储简短描述
    Usage []string  // 描述结构体，包含 Usage 字段，用于存储用法说明
}

type Command interface {
    Name() string  // 定义接口 Command，包含 Name 方法，用于返回命令名称
    Description() Description  // 定义接口 Command，包含 Description 方法，用于返回命令描述
    Execute(args []string) error  // 定义接口 Command，包含 Execute 方法，用于执行命令
}

var (
    commandRegistry = make(map[string]Command)  // 声明并初始化命令注册表，用于存储命令名称和对应的命令对象
    ctllog          = log.New(os.Stderr, "v2ctl> ", 0)  // 创建日志记录器，输出到标准错误，前缀为 "v2ctl> "
)

func RegisterCommand(cmd Command) error {
    entry := strings.ToLower(cmd.Name())  // 将命令名称转换为小写
    if entry == "" {
        return newError("empty command name")  // 如果命令名称为空，返回错误
    }
    commandRegistry[entry] = cmd  // 将命令注册到命令注册表中
    return nil
}

func GetCommand(name string) Command {
    cmd, found := commandRegistry[name]  // 从命令注册表中获取指定名称的命令
    if !found {
        return nil  // 如果未找到命令，返回空
    }
    return cmd  // 返回找到的命令
}

type hiddenCommand interface {
    Hidden() bool  // 定义隐藏命令接口，包含 Hidden 方法，用于判断命令是否隐藏
}

func PrintUsage() {
    for name, cmd := range commandRegistry {  // 遍历命令注册表
        if _, ok := cmd.(hiddenCommand); ok {  // 判断命令是否实现了隐藏命令接口
            continue  // 如果是隐藏命令，继续下一次循环
        }
        fmt.Println("   ", name, "\t\t\t", cmd.Description())  // 输出命令名称和描述
    }
}
```