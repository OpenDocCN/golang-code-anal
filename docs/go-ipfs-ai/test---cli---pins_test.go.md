# `kubo\test\cli\pins_test.go`

```go
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/go-cid"  // 导入 go-cid 包，用于处理 IPFS Content Identifier
    "github.com/ipfs/kubo/test/cli/harness"  // 导入 harness 包，用于测试 CLI
    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入 testutils 包，用于测试工具
    "github.com/stretchr/testify/assert"  // 导入 assert 包，用于编写测试断言
    "github.com/stretchr/testify/require"  // 导入 require 包，用于编写测试断言
)

type testPinsArgs struct {
    runDaemon bool  // 定义一个结构体，包含 runDaemon、pinArg、lsArg、baseArg 四个字段
    pinArg    string
    lsArg     string
    baseArg   string
}

func testPins(t *testing.T, args testPinsArgs) {  // 定义一个测试函数 testPins
    })
}

func testPinsErrorReporting(t *testing.T, args testPinsArgs) {  // 定义一个测试函数 testPinsErrorReporting
    t.Run(fmt.Sprintf("test pins error reporting with args=%+v", args), func(t *testing.T) {  // 使用 t.Run 运行测试，并格式化输出测试参数
        t.Parallel()  // 标记测试函数可以并行执行
        node := harness.NewT(t).NewNode().Init()  // 创建一个测试节点
        if args.runDaemon {  // 如果 runDaemon 为真
            node.StartDaemon("--offline")  // 启动守护进程
        }
        randomCID := "Qme8uX5n9hn15pw9p6WcVKoziyyC9LXv4LEgvsmKMULjnV"  // 定义一个随机的 CID
        res := node.RunIPFS(StrCat("pin", "add", args.pinArg, randomCID)...)  // 运行 IPFS 命令，将随机 CID 添加到 pinArg 中
        assert.NotEqual(t, 0, res.ExitErr.ExitCode())  // 断言结果不等于 0
        assert.Contains(t, res.Stderr.String(), "ipld: could not find")  // 断言结果包含指定字符串
    })
}

func testPinDAG(t *testing.T, args testPinsArgs) {  // 定义一个测试函数 testPinDAG
    t.Run(fmt.Sprintf("test pin DAG with args=%+v", args), func(t *testing.T) {  // 使用 t.Run 运行测试，并格式化输出测试参数
        t.Parallel()  // 标记测试函数可以并行执行
        h := harness.NewT(t)  // 创建一个测试节点
        node := h.NewNode().Init()  // 初始化一个新的节点
        if args.runDaemon {  // 如果 runDaemon 为真
            node.StartDaemon("--offline")  // 启动守护进程
        }
        bytes := RandomBytes(1 << 20)  // 生成 1MB 的随机字节
        tmpFile := h.WriteToTemp(string(bytes))  // 将随机字节写入临时文件
        cid := node.IPFS(StrCat("add", args.pinArg, "--pin=false", "-q", tmpFile)...).Stdout.Trimmed()  // 将临时文件添加到 IPFS，并获取 CID

        node.IPFS("pin", "add", "--recursive=true", cid)  // 递归添加 CID
        node.IPFS("pin", "rm", cid)  // 移除 CID

        // remove part of the DAG
        part := node.IPFS("refs", cid).Stdout.Lines()[0]  // 获取 DAG 的一部分
        node.IPFS("block", "rm", part)  // 移除 DAG 的一部分

        res := node.RunIPFS("pin", "add", "--recursive=true", cid)  // 运行 IPFS 命令，递归添加 CID
        assert.NotEqual(t, 0, res)  // 断言结果不等于 0
        assert.Contains(t, res.Stderr.String(), "ipld: could not find")  // 断言结果包含指定字符串
    })
}

func testPinProgress(t *testing.T, args testPinsArgs) {  // 定义一个测试函数 testPinProgress
    # 运行测试用例，输出测试进度和参数信息
    t.Run(fmt.Sprintf("test pin progress with args=%+v", args), func(t *testing.T) {
        # 标记该测试用例可以并行执行
        t.Parallel()
        # 创建测试工具实例
        h := harness.NewT(t)
        # 初始化节点
        node := h.NewNode().Init()

        # 如果参数中包含运行守护进程的标志
        if args.runDaemon:
            # 启动 IPFS 守护进程，离线模式
            node.StartDaemon("--offline")

        # 生成 1 MiB 大小的随机字节流
        bytes := RandomBytes(1 << 20) // 1 MiB
        # 将随机字节流写入临时文件
        tmpFile := h.WriteToTemp(string(bytes))
        # 将临时文件添加到 IPFS，并返回 CID
        cid := node.IPFS(StrCat("add", args.pinArg, "--pin=false", "-q", tmpFile)...).Stdout.Trimmed()

        # 执行带有进度显示的 IPFS pin 操作
        res := node.RunIPFS("pin", "add", "--progress", cid)
        # 断言操作执行无错误
        node.Runner.AssertNoError(res)

        # 断言结果中包含特定信息
        assert.Contains(t, res.Stderr.String(), " 5 nodes")
    })
func TestPins(t *testing.T) {
    // 测试用例：测试在没有守护进程运行时进行固定
    t.Parallel()
    t.Run("test pinning without daemon running", func(t *testing.T) {
        // 子测试用例：测试在没有守护进程运行时进行固定
        t.Parallel()
        // 调用测试函数，测试固定时的错误报告
        testPinsErrorReporting(t, testPinsArgs{})
        // 调用测试函数，测试固定时的错误报告，带有进度参数
        testPinsErrorReporting(t, testPinsArgs{pinArg: "--progress"})
        // 调用测试函数，测试固定DAG
        testPinDAG(t, testPinsArgs{})
        // 调用测试函数，测试固定DAG，带有原始叶子参数
        testPinDAG(t, testPinsArgs{pinArg: "--raw-leaves"})
        // 调用测试函数，测试固定进度
        testPinProgress(t, testPinsArgs{})
        // 调用测试函数，测试固定
        testPins(t, testPinsArgs{})
        // 调用测试函数，测试固定，带有进度参数
        testPins(t, testPinsArgs{pinArg: "--progress"})
        // 调用测试函数，测试固定，带有进度和ls参数
        testPins(t, testPinsArgs{pinArg: "--progress", lsArg: "--stream"})
        // 调用测试函数，测试固定，带有base参数
        testPins(t, testPinsArgs{baseArg: "--cid-base=base32"})
        // 调用测试函数，测试固定，带有ls和base参数
        testPins(t, testPinsArgs{lsArg: "--stream", baseArg: "--cid-base=base32"})
    })

    t.Run("test pinning with daemon running without network", func(t *testing.T) {
        // 子测试用例：测试在守护进程运行但没有网络时进行固定
        t.Parallel()
        // 调用测试函数，测试固定时的错误报告，运行守护进程
        testPinsErrorReporting(t, testPinsArgs{runDaemon: true})
        // 调用测试函数，测试固定时的错误报告，带有进度参数，运行守护进程
        testPinsErrorReporting(t, testPinsArgs{runDaemon: true, pinArg: "--progress"})
        // 调用测试函数，测试固定DAG，运行守护进程
        testPinDAG(t, testPinsArgs{runDaemon: true})
        // 调用测试函数，测试固定DAG，带有原始叶子参数，运行守护进程
        testPinDAG(t, testPinsArgs{runDaemon: true, pinArg: "--raw-leaves"})
        // 调用测试函数，测试固定进度，运行守护进程
        testPinProgress(t, testPinsArgs{runDaemon: true})
        // 调用测试函数，测试固定，运行守护进程
        testPins(t, testPinsArgs{runDaemon: true})
        // 调用测试函数，测试固定，带有进度参数，运行守护进程
        testPins(t, testPinsArgs{runDaemon: true, pinArg: "--progress"})
        // 调用测试函数，测试固定，带有进度和ls参数，运行守护进程
        testPins(t, testPinsArgs{runDaemon: true, pinArg: "--progress", lsArg: "--stream"})
        // 调用测试函数，测试固定，带有base参数，运行守护进程
        testPins(t, testPinsArgs{runDaemon: true, baseArg: "--cid-base=base32"})
        // 调用测试函数，测试固定，带有ls和base参数，运行守护进程
        testPins(t, testPinsArgs{runDaemon: true, lsArg: "--stream", baseArg: "--cid-base=base32"})
    })
}
    # 运行测试用例，测试带有名称的 CLI 文本输出的固定功能
    t.Run("test pinning with names cli text output", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()

        # 创建新的 IPFS 节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 向 IPFS 添加随机字符串并不固定
        cidAStr := node.IPFSAddStr(RandomStr(1000), "--pin=false")
        cidBStr := node.IPFSAddStr(RandomStr(1000), "--pin=false")

        # 对 cidAStr 执行 pin 操作，并命名为 testPin
        _ = node.IPFS("pin", "add", "--name", "testPin", cidAStr)

        # 构建输出字符串
        outARegular := cidAStr + " recursive"
        outADetailed := outARegular + " testPin"
        outBRegular := cidBStr + " recursive"
        outBDetailed := outBRegular + " testPin"

        # 定义一个函数，用于执行 pin ls 命令并返回结果
        pinLs := func(args ...string) []string {
            return strings.Split(node.IPFS(StrCat("pin", "ls", args)...).Stdout.Trimmed(), "\n")
        }

        # 执行 pin ls 命令并检查输出
        lsOut := pinLs("-t=recursive")
        require.Contains(t, lsOut, outARegular)
        require.NotContains(t, lsOut, outADetailed)

        # 执行带名称的 pin ls 命令并检查输出
        lsOut = pinLs("-t=recursive", "--names")
        require.Contains(t, lsOut, outADetailed)
        require.NotContains(t, lsOut, outARegular)

        # 更新 cidAStr 的 pin 状态为 cidBStr
        _ = node.IPFS("pin", "update", cidAStr, cidBStr)
        # 执行带名称的 pin ls 命令并检查输出
        lsOut = pinLs("-t=recursive", "--names")
        require.Contains(t, lsOut, outBDetailed)
        require.NotContains(t, lsOut, outADetailed)
    })

    # JSON that is also the wire format of /api/v0
    # 运行测试，测试带有名称的 JSON 输出的固定功能
    t.Run("test pinning with names json output", func(t *testing.T) {
        t.Parallel()

        # 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 生成随机字符串并将其添加到 IPFS，不固定
        cidAStr := node.IPFSAddStr(RandomStr(1000), "--pin=false")
        cidBStr := node.IPFSAddStr(RandomStr(1000), "--pin=false")

        # 将 cidAStr 添加到固定中，命名为 testPinJson
        _ = node.IPFS("pin", "add", "--name", "testPinJson", cidAStr)

        # 准备输出字符串
        outARegular := `"` + cidAStr + `":{"Type":"recursive"`
        outADetailed := outARegular + `,"Name":"testPinJson"`
        outBRegular := `"` + cidBStr + `":{"Type":"recursive"`
        outBDetailed := outBRegular + `,"Name":"testPinJson`

        # 定义一个函数，用于执行 pin ls 命令并返回结果
        pinLs := func(args ...string) string {
            return node.IPFS(StrCat("pin", "ls", "--enc=json", args)...).Stdout.Trimmed()
        }

        # 执行 pin ls 命令，检查输出是否包含 outARegular，不包含 outADetailed
        lsOut := pinLs("-t=recursive")
        require.Contains(t, lsOut, outARegular)
        require.NotContains(t, lsOut, outADetailed)

        # 执行 pin ls 命令，检查输出是否包含 outADetailed
        lsOut = pinLs("-t=recursive", "--names")
        require.Contains(t, lsOut, outADetailed)

        # 将 cidAStr 更新为 cidBStr
        _ = node.IPFS("pin", "update", cidAStr, cidBStr)
        # 执行 pin ls 命令，检查输出是否包含 outBDetailed
        lsOut = pinLs("-t=recursive", "--names")
        require.Contains(t, lsOut, outBDetailed)
    })
# 闭合前面的函数定义
```