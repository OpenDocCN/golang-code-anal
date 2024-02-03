# `kubo\core\commands\commands_test.go`

```go
package commands

import (
    "strings"  // 导入字符串操作包
    "testing"  // 导入测试包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令包
)

func collectPaths(prefix string, cmd *cmds.Command, out map[string]struct{}) {
    for name, sub := range cmd.Subcommands {  // 遍历命令的子命令
        path := prefix + "/" + name  // 构建子命令的路径
        out[path] = struct{}{}  // 将路径添加到输出集合中
        collectPaths(path, sub, out)  // 递归调用，继续收集子命令的路径
    }
}

func TestROCommands(t *testing.T) {
    list := []string{  // 定义命令路径列表
        "/block",
        "/block/get",
        // ... 其他命令路径
        "/version",
    }

    cmdSet := make(map[string]struct{})  // 创建空的命令集合
    collectPaths("", RootRO, cmdSet)  // 收集只读命令的路径

    for _, path := range list {  // 遍历命令路径列表
        if _, ok := cmdSet[path]; !ok {  // 如果路径不在命令集合中
            t.Errorf("%q not in result", path)  // 输出错误信息
        } else {
            delete(cmdSet, path)  // 从命令集合中删除该路径
        }
    }

    for path := range cmdSet {  // 遍历剩余的命令集合
        t.Errorf("%q in result but shouldn't be", path)  // 输出错误信息
    }

    for _, path := range list {  // 遍历命令路径列表
        path = path[1:] // remove leading slash  // 去掉路径开头的斜杠
        split := strings.Split(path, "/")  // 按斜杠分割路径
        sub, err := RootRO.Get(split)  // 获取子命令
        if err != nil {  // 如果有错误
            t.Errorf("error getting subcommand %q: %v", path, err)  // 输出错误信息
        } else if sub == nil {  // 如果子命令为空
            t.Errorf("subcommand %q is nil even though there was no error", path)  // 输出错误信息
        }
    }
}

func TestCommands(t *testing.T) {
    }

    cmdSet := make(map[string]struct{})  // 创建空的命令集合
    collectPaths("", Root, cmdSet)  // 收集命令的路径
    # 遍历列表中的路径
    for _, path := range list:
        # 如果路径不在cmdSet中，则输出错误信息
        if _, ok := cmdSet[path]; !ok:
            t.Errorf("%q not in result", path)
        # 否则，从cmdSet中删除该路径
        else:
            delete(cmdSet, path)

    # 遍历cmdSet中的路径
    for path := range cmdSet:
        # 输出错误信息，表示路径在结果中但不应该存在
        t.Errorf("%q in result but shouldn't be", path)

    # 遍历列表中的路径
    for _, path := range list:
        # 去掉路径的开头斜杠
        path = path[1:] // remove leading slash
        # 以斜杠为分隔符，将路径分割成子路径
        split := strings.Split(path, "/")
        # 从Root中获取子命令
        sub, err := Root.Get(split)
        # 如果出现错误，则输出错误信息
        if err != nil:
            t.Errorf("error getting subcommand %q: %v", path, err)
        # 如果子命令为空且没有错误，则输出错误信息
        else if sub == nil:
            t.Errorf("subcommand %q is nil even though there was no error", path)
# 闭合前面的函数定义
```