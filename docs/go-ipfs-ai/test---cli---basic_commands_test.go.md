# `kubo\test\cli\basic_commands_test.go`

```
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "regexp"  // 导入 regexp 包，用于正则表达式匹配
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/blang/semver/v4"  // 导入 semver 包，用于处理语义化版本
    "github.com/ipfs/kubo/test/cli/harness"  // 导入 harness 包，用于测试框架
    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入 testutils 包，用于测试工具
    "github.com/stretchr/testify/assert"  // 导入 assert 包，用于断言
    gomod "golang.org/x/mod/module"  // 导入 gomod 包，用于处理模块
)

var versionRegexp = regexp.MustCompile(`^ipfs version (.+)$`)  // 定义一个正则表达式，用于匹配版本号

func parseVersionOutput(s string) semver.Version {
    versString := versionRegexp.FindStringSubmatch(s)[1]  // 使用正则表达式匹配版本号
    v, err := semver.Parse(versString)  // 解析版本号为 semver.Version 对象
    if err != nil {
        panic(err)  // 如果解析出错，则抛出异常
    }
    return v  // 返回解析后的版本号对象
}

func TestCurDirIsWritable(t *testing.T) {
    t.Parallel()  // 声明测试函数可以并行执行
    h := harness.NewT(t)  // 创建测试框架对象
    h.WriteFile("test.txt", "It works!")  // 在当前目录下写入文件
}

func TestIPFSVersionCommandMatchesFlag(t *testing.T) {
    t.Parallel()  // 声明测试函数可以并行执行
    node := harness.NewT(t).NewNode()  // 创建新的节点对象
    commandVersionStr := node.IPFS("version").Stdout.String()  // 执行 IPFS 命令获取版本信息
    commandVersionStr = strings.TrimSpace(commandVersionStr)  // 去除字符串两端的空白字符
    commandVersion := parseVersionOutput(commandVersionStr)  // 解析命令输出的版本信息

    flagVersionStr := node.IPFS("--version").Stdout.String()  // 执行 IPFS 带参数的版本命令获取版本信息
    flagVersionStr = strings.TrimSpace(flagVersionStr)  // 去除字符串两端的空白字符
    flagVersion := parseVersionOutput(flagVersionStr)  // 解析带参数命令输出的版本信息

    assert.Equal(t, commandVersion, flagVersion)  // 断言两个版本信息是否相等
}

func TestIPFSVersionAll(t *testing.T) {
    t.Parallel()  // 声明测试函数可以并行执行
    node := harness.NewT(t).NewNode()  // 创建新的节点对象
    res := node.IPFS("version", "--all").Stdout.String()  // 执行 IPFS 带参数的版本命令获取版本信息
    res = strings.TrimSpace(res)  // 去除字符串两端的空白字符
    assert.Contains(t, res, "Kubo version")  // 断言版本信息中包含 "Kubo version"
    assert.Contains(t, res, "Repo version")  // 断言版本信息中包含 "Repo version"
    assert.Contains(t, res, "System version")  // 断言版本信息中包含 "System version"
    assert.Contains(t, res, "Golang version")  // 断言版本信息中包含 "Golang version"
}

func TestIPFSVersionDeps(t *testing.T) {
    t.Parallel()  // 声明测试函数可以并行执行
    node := harness.NewT(t).NewNode()  // 创建新的节点对象
    res := node.IPFS("version", "deps").Stdout.String()  // 执行 IPFS 带参数的版本命令获取依赖信息
    res = strings.TrimSpace(res)  // 去除字符串两端的空白字符
    lines := SplitLines(res)  // 将字符串按行分割成数组

    assert.Equal(t, "github.com/ipfs/kubo@(devel)", lines[0])  // 断言依赖信息的第一行是否符合预期
}
    # 遍历除第一个元素外的所有行
    for _, depLine := range lines[1:] {
        # 以 " => " 为分隔符，将每行内容分割成两部分
        split := strings.Split(depLine, " => ")
        # 遍历分割后的每个模块版本
        for _, moduleVersion := range split {
            # 以 "@" 为分隔符，将模块和版本分割开
            splitModVers := strings.Split(moduleVersion, "@")
            # 获取模块路径
            modPath := splitModVers[0]
            # 获取模块版本
            modVers := splitModVers[1]
            # 断言模块版本检查无误，否则输出错误信息
            assert.NoError(t, gomod.Check(modPath, modVers), "path: %s, version: %s", modPath, modVers)
        }
    }
# 测试IPFS命令
func TestIPFSCommands(t *testing.T):
    # 并行测试
    t.Parallel()
    # 创建新的节点
    node := harness.NewT(t).NewNode()
    # 获取IPFS命令
    cmds := node.IPFSCommands()
    # 断言IPFS命令中包含"ipfs add"
    assert.Contains(t, cmds, "ipfs add")
    # 断言IPFS命令中包含"ipfs daemon"
    assert.Contains(t, cmds, "ipfs daemon")
    # 断言IPFS命令中包含"ipfs update"
    assert.Contains(t, cmds, "ipfs update")

# 测试所有子命令是否接受帮助
func TestAllSubcommandsAcceptHelp(t *testing.T):
    # 并行测试
    t.Parallel()
    # 创建新的节点
    node := harness.NewT(t).NewNode()
    # 遍历IPFS命令
    for _, cmd := range node.IPFSCommands():
        # 复制cmd
        cmd := cmd
        # 运行子测试
        t.Run(fmt.Sprintf("command %q accepts help", cmd), func(t *testing.T):
            # 并行测试
            t.Parallel()
            # 分割命令
            splitCmd := strings.Split(cmd, " ")[1:]
            # 执行IPFS帮助命令
            node.IPFS(StrCat("help", splitCmd)...)
            # 执行IPFS帮助命令
            node.IPFS(StrCat(splitCmd, "--help")...)

# 测试帮助文本中是否提到所有根命令
func TestAllRootCommandsAreMentionedInHelpText(t *testing.T):
    # 并行测试
    t.Parallel()
    # 创建新的节点
    node := harness.NewT(t).NewNode()
    # 获取IPFS命令
    cmds := node.IPFSCommands()
    # 创建根命令列表
    var rootCmds []string
    # 遍历IPFS命令
    for _, cmd := range cmds:
        # 分割命令
        splitCmd := strings.Split(cmd, " ")
        # 如果命令长度为2
        if len(splitCmd) == 2:
            # 将根命令添加到列表中
            rootCmds = append(rootCmds, splitCmd[1])

    # 一些基本命令不会出现在帮助消息中
    # 但我们默认要求它们出现在帮助消息中，这样我们就必须有意地排除它们
    notInHelp := map[string]bool{
        "object":   true,
        "shutdown": true,
    }

    # 去除帮助消息中的空白字符
    helpMsg := strings.TrimSpace(node.IPFS("--help").Stdout.String())
    # 遍历根命令
    for _, rootCmd := range rootCmds:
        # 如果根命令不在排除列表中
        if _, ok := notInHelp[rootCmd]; ok:
            # 继续下一次循环
            continue
        # 断言帮助消息中包含根命令
        assert.Contains(t, helpMsg, fmt.Sprintf("  %s", rootCmd))

# 测试命令文档的宽度
func TestCommandDocsWidth(t *testing.T):
    # 并行测试
    t.Parallel()
    # 创建新的节点
    node := harness.NewT(t).NewNode()

    # 要求新命令明确选择更长的行
    # 遍历节点的 IPFS 命令列表
    for _, cmd := range node.IPFSCommands() {
        # 检查命令是否在允许列表中，如果是则跳过
        if _, ok := allowList[cmd]; ok {
            continue
        }
        # 使用测试框架运行命令是否符合文档的宽度限制
        t.Run(fmt.Sprintf("command %q conforms to docs width limit", cmd), func(t *testing.T) {
            # 将命令按空格分割
            splitCmd := strings.Split(cmd, " ")
            # 执行 IPFS 命令并获取输出
            resStr := node.IPFS(StrCat(splitCmd[1:], "--help")...)
            # 去除输出字符串两端的空白字符
            res := strings.TrimSpace(resStr.Stdout.String())
            # 遍历输出的每一行
            for _, line := range SplitLines(res) {
                # 断言每行的长度小于等于 80，否则输出错误信息
                assert.LessOrEqualf(t, len(line), 80, "expected width %d < 80 for %q", len(line), cmd)
            }
        })
    }
# 测试当传递错误标志时，所有命令都失败
func TestAllCommandsFailWhenPassedBadFlag(t *testing.T) {
    # 启用并行测试
    t.Parallel()
    # 创建一个新的节点
    node := harness.NewT(t).NewNode()

    # 遍历节点的IPFS命令
    for _, cmd := range node.IPFSCommands() {
        # 对每个命令运行子测试
        t.Run(fmt.Sprintf("command %q fails when passed a bad flag", cmd), func(t *testing.T) {
            # 将命令拆分成字符串数组
            splitCmd := strings.Split(cmd, " ")
            # 运行IPFS命令并传递错误标志
            res := node.RunIPFS(StrCat(splitCmd, "--badflag")...)
            # 断言命令的退出码为1
            assert.Equal(t, 1, res.Cmd.ProcessState.ExitCode())
        })
    }
}

# 测试命令的标志
func TestCommandsFlags(t *testing.T) {
    # 启用并行测试
    t.Parallel()
    # 创建一个新的节点
    node := harness.NewT(t).NewNode()
    # 获取IPFS命令的标志并存储为字符串
    resStr := node.IPFS("commands", "--flags").Stdout.String()
    # 断言结果字符串包含特定的命令和标志
    assert.Contains(t, resStr, "ipfs pin add --recursive / ipfs pin add -r")
    assert.Contains(t, resStr, "ipfs id --format / ipfs id -f")
    assert.Contains(t, resStr, "ipfs repo gc --quiet / ipfs repo gc -q")
}
```