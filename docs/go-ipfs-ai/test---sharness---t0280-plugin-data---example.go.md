# `kubo\test\sharness\t0280-plugin-data\example.go`

```go
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"   // 导入 os 包，用于访问操作系统功能

    "github.com/ipfs/kubo/plugin"  // 导入第三方包 github.com/ipfs/kubo/plugin
)

var Plugins = []plugin.Plugin{  // 定义变量 Plugins，类型为 plugin.Plugin 切片
    &testPlugin{},  // 初始化 testPlugin 结构体，并添加到 Plugins 中
}

var _ = Plugins // used

type testPlugin struct{}  // 定义 testPlugin 结构体

func (*testPlugin) Name() string {  // 实现 testPlugin 结构体的 Name 方法
    return "test-plugin"  // 返回字符串 "test-plugin"
}

func (*testPlugin) Version() string {  // 实现 testPlugin 结构体的 Version 方法
    return "0.1.0"  // 返回字符串 "0.1.0"
}

func (*testPlugin) Init(env *plugin.Environment) error {  // 实现 testPlugin 结构体的 Init 方法
    fmt.Fprintf(os.Stderr, "testplugin %s\n", env.Repo)  // 格式化输出环境的 Repo 到标准错误输出
    fmt.Fprintf(os.Stderr, "testplugin %v\n", env.Config)  // 格式化输出环境的 Config 到标准错误输出
    return nil  // 返回空错误
}
```