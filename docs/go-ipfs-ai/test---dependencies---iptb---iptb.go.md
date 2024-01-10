# `kubo\test\dependencies\iptb\iptb.go`

```
package main
// 导入所需的包
import (
    "fmt"
    "os"

    cli "github.com/ipfs/iptb/cli"
    testbed "github.com/ipfs/iptb/testbed"

    plugin "github.com/ipfs/iptb-plugins/local"
)

// 初始化函数
func init() {
    // 注册插件
    _, err := testbed.RegisterPlugin(testbed.IptbPlugin{
        From:        "<builtin>",
        NewNode:     plugin.NewNode,
        GetAttrList: plugin.GetAttrList,
        GetAttrDesc: plugin.GetAttrDesc,
        PluginName:  plugin.PluginName,
        BuiltIn:     true,
    }, false)
    // 如果注册出错，抛出异常
    if err != nil {
        panic(err)
    }
}

// 主函数
func main() {
    // 创建命令行接口
    cli := cli.NewCli()
    // 运行命令行接口，如果出错，输出错误信息并退出程序
    if err := cli.Run(os.Args); err != nil {
        fmt.Fprintf(cli.ErrWriter, "%s\n", err)
        os.Exit(1)
    }
}
```