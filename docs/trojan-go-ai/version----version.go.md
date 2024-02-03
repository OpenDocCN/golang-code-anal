# `trojan-go\version\version.go`

```go
package version

import (
    "flag"  // 导入 flag 包，用于解析命令行参数
    "fmt"   // 导入 fmt 包，用于格式化输出
    "runtime"   // 导入 runtime 包，用于获取 Go 程序运行时的信息

    "github.com/p4gefau1t/trojan-go/common"   // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/constant" // 导入自定义包 constant
    "github.com/p4gefau1t/trojan-go/option"   // 导入自定义包 option
)

type versionOption struct {
    flag *bool   // 定义 versionOption 结构体，包含一个布尔型指针字段
}

func (*versionOption) Name() string {
    return "version"   // 返回字符串 "version"
}

func (*versionOption) Priority() int {
    return 10   // 返回整数 10
}

func (c *versionOption) Handle() error {
    if *c.flag {   // 如果布尔型指针字段的值为 true
        fmt.Println("Trojan-Go", constant.Version)   // 打印 Trojan-Go 版本信息
        fmt.Println("Go Version:", runtime.Version())   // 打印 Go 版本信息
        fmt.Println("OS/Arch:", runtime.GOOS+"/"+runtime.GOARCH)   // 打印操作系统和架构信息
        fmt.Println("Git Commit:", constant.Commit)   // 打印 Git 提交信息
        fmt.Println("")   // 打印空行
        fmt.Println("Developed by PageFault (p4gefau1t)")   // 打印开发者信息
        fmt.Println("Licensed under GNU General Public License version 3")   // 打印许可证信息
        fmt.Println("GitHub Repository:\thttps://github.com/p4gefau1t/trojan-go")   // 打印 GitHub 仓库地址
        fmt.Println("Trojan-Go Documents:\thttps://p4gefau1t.github.io/trojan-go/")   // 打印 Trojan-Go 文档地址
        return nil   // 返回空值
    }
    return common.NewError("not set")   // 返回 common 包中的错误信息
}

func init() {
    option.RegisterHandler(&versionOption{   // 注册 versionOption 结构体
        flag: flag.Bool("version", false, "Display version and help info"),   // 使用 flag 包创建一个布尔型指针字段
    })
}
```