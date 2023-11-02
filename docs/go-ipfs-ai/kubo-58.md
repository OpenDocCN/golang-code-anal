# go-ipfs 源码解析 58

# `/opt/kubo/test/cli/harness/nodes.go`

该代码定义了一个名为“harness”的包，该包包含一个名为“Nodes”的类型，该类型表示一维节点切片，以及一个名为“Init”的函数，用于初始化节点。

具体来说，该包通过导入“github.com/ipfs/kubo/test/cli/testutils”和“github.com/multiformats/go-multiaddr”的包来支持Kubernetes风格的节点。然后定义了一个“Nodes”类型，该类型包含多个“Node”类型，每个“Node”类型都是一个指向“Node”对象的指针，其中包含一个“Init”函数，用于初始化节点。

通过创建一个名为“harness”的包，可以很方便地创建一个包含多个节点的“Nodes”实例，并使用其中的“Init”函数来初始化节点。这使得代码可以更轻松地与Kubernetes风格的节点进行交互，并支持对节点群组的操作。


```
package harness

import (
	"sync"

	. "github.com/ipfs/kubo/test/cli/testutils"
	"github.com/multiformats/go-multiaddr"
	"golang.org/x/sync/errgroup"
)

// Nodes is a collection of Kubo nodes along with operations on groups of nodes.
type Nodes []*Node

func (n Nodes) Init(args ...string) Nodes {
	ForEachPar(n, func(node *Node) { node.Init(args...) })
	return n
}

```

这是一个使用Go标准库中的errgroup、fmt和sync包实现的函数，其作用是接收一个整数类型的参数n，并对其中的每个节点执行一个给定的函数f。

具体来说，该函数的行为如下：

1. 创建一个名为“group”的errgroup对象。
2. 通过循环遍历参数n中的每个节点，并为每个节点执行给定的函数f。
3. 在函数内部执行给定的函数f，该函数接收一个名为“node”的参数，然后返回一个 nil 值。
4. 在函数内部执行一个名为“group.Go”的函数，该函数执行一个阻塞操作，使用给定的函数f作为阻塞操作的入口，并返回一个 nil 值。
5. 如果执行过程中出现错误，函数将输出该错误并停止执行。
6. 最后，函数等待名为“group.Wait”的函数的输出，如果输出为非空，则表示所有节点都成功执行了给定的函数f，否则将输出错误并停止执行。


```
func (n Nodes) ForEachPar(f func(*Node)) {
	group := &errgroup.Group{}
	for _, node := range n {
		node := node
		group.Go(func() error {
			f(node)
			return nil
		})
	}
	err := group.Wait()
	if err != nil {
		panic(err)
	}
}

```

该函数 `Connect` 接受一个整数参数 `n`，表示网络中的节点数量。函数内部使用一个 `sync.WaitGroup` 来等待所有节点的连接完成。

对于每个节点 `i`，它遍历整个网络中的所有节点 `j`，如果当前节点 `i` 与另一个节点 `j` 相同，则跳过该节点。否则，当前节点 `i` 将与另一个节点 `j` 进行连接操作，通过调用 `Connect` 函数完成。

函数内部使用一个 `defer` 关键字来通知 `WaitGroup` 组中的所有节点在函数完成时执行一个操作，即 `node.Connect` 函数返回结果。

函数最终返回网络中的节点数量。


```
func (n Nodes) Connect() Nodes {
	wg := sync.WaitGroup{}
	for i, node := range n {
		for j, otherNode := range n {
			if i == j {
				continue
			}
			node := node
			otherNode := otherNode
			wg.Add(1)
			go func() {
				defer wg.Done()
				node.Connect(otherNode)
			}()
		}
	}
	wg.Wait()
	for _, node := range n {
		firstPeer := node.Peers()[0]
		if _, err := firstPeer.ValueForProtocol(multiaddr.P_P2P); err != nil {
			log.Panicf("unexpected state for node %d with peer ID %s: %s", node.ID, node.PeerID(), err)
		}
	}
	return n
}

```

这两位代码定义了两个函数：`StartDaemons` 和 `StopDaemons`。函数接收一个字符串参数 `args`，然后分别传递给 `func` 函数。

`StartDaemons` 函数接收一个 `Nodes` 类型的参数 `n` 和多个字符串参数 `args`。函数内部使用 `ForEachPar` 函数来遍历 `n` 个参数 `args`，并调用 `func` 函数，传入参数 `node` 和 `args`。

`StopDaemons` 函数与 `StartDaemons` 函数类似，只是传递给 `func` 函数的参数类型从 `Nodes` 变成了 `Nodes` 和 `Nodes`。函数内部使用 `ForEachPar` 函数来遍历 `n` 个参数 `args`，并调用 `func` 函数，传入参数 `node`。

这两个函数的具体实现没有其他代码，所以无法判断它们的行为。


```
func (n Nodes) StartDaemons(args ...string) Nodes {
	ForEachPar(n, func(node *Node) { node.StartDaemon(args...) })
	return n
}

func (n Nodes) StopDaemons() Nodes {
	ForEachPar(n, func(node *Node) { node.StopDaemon() })
	return n
}

```

# `/opt/kubo/test/cli/harness/peering.go`

这段代码定义了一个名为 `Peering` 的结构体，用于表示两个主机之间的对等连接。该结构体包含两个整数字段 `From` 和 `To`，分别表示两个主机之间的网络接口。

该代码导入了自定义的 `Peering` 结构体类型，以及从 `github.com/ipfs/kubo/config` 包中导入的 `config` 类型。

该 `config.Peering` 函数定义了一个函数，该函数的输入参数为 `config.Peering` 和 `config.Peering`.函数返回一个默认的 `config.Peering` 类型的实例，即两条网络接口之间的对等连接。

该 `config.Peering` 函数的输入参数 `config.Peering` 是一个字符串，指定了对等连接的名称。如果该名称不存在，则该函数将创建一个新的对等连接。

该 `config.Peering` 函数的输出是一个 `config.Peering` 类型的实例，它包含了两个字段 `from` 和 `to`，分别表示两个主机之间的网络接口。


```
package harness

import (
	"fmt"
	"math/rand"
	"testing"

	"github.com/ipfs/kubo/config"
)

type Peering struct {
	From int
	To   int
}

```

该代码定义了两个函数，以及它们的参数和返回值类型。

函数1 `NewRandPort()` 返回一个整数类型的随机整数，然后将其转换为一个随机端口号，并将其返回。

函数2 `CreatePeerNodes()` 用于创建一个 `PeerNodes` 类型，该类型表示网络中的对等节点。该函数接受一个整数参数 `n`，表示网络中的节点数量，以及一个 `Peerings` 类型的参数 `peerings`，其中每个 `Peerings` 都包含一个对等节点组的配置。函数内部使用一个 `NewT` 函数创建一个 `Harness` 类型的实例，然后使用 `NewNodes` 函数创建 `PeerNodes` 类型的节点，并将这些节点初始化为 `CreateConfig` 函数设置，其中 `config.Routing.Type` 被设置为 `none`，`addresses.Swarm` 属性设置为一个 IP 地址，然后使用 `CreatePeerWith` 函数将节点与对等节点组进行对等连接。最后，函数返回一个 `Harness` 类型的实例和一个包含 `PeerNodes` 类型节点的 `Nodes` 类型的返回值。


```
func NewRandPort() int {
	n := rand.Int()
	return 3000 + (n % 1000)
}

func CreatePeerNodes(t *testing.T, n int, peerings []Peering) (*Harness, Nodes) {
	h := NewT(t)
	nodes := h.NewNodes(n).Init()
	nodes.ForEachPar(func(node *Node) {
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Routing.Type = config.NewOptionalString("none")
			cfg.Addresses.Swarm = []string{fmt.Sprintf("/ip4/127.0.0.1/tcp/%d", NewRandPort())}
		})
	})

	for _, peering := range peerings {
		nodes[peering.From].PeerWith(nodes[peering.To])
	}

	return h, nodes
}

```

# `/opt/kubo/test/cli/harness/run.go`

这段代码定义了一个名为“ harness”的包，其中定义了一个名为“Runner”的类，用于运行子进程和汇总输出。

在“Runner”定义中，使用了一个名为“Runnner”的函数，它接收一个“Runner”类型的参数，这个参数定义了如何运行子进程和汇总输出。具体来说，这个函数使用“fmt”包中的“print”函数将参数的“Runner”字段输出，使用“io”包中的“exec”函数运行子进程，并使用“os”包中的“exec”函数获取子进程的输出。此外，“Runner”还定义了一个“Env”字段，用于存储运行时环境变量，以及一个“Dir”字段，用于指定子进程的执行目录。最后，“Runner”还定义了一个“Verbose”字段，表示在运行子进程时输出更多的信息。

整个包的作用是提供一个可以运行子进程并汇总输出的工具，可以方便地运行程序并获取其输出。


```
package harness

import (
	"fmt"
	"io"
	"os/exec"
	"strings"
)

// Runner is a process runner which can run subprocesses and aggregate output.
type Runner struct {
	Env     map[string]string
	Dir     string
	Verbose bool
}

```

这段代码定义了一个名为 `RunRequest` 的结构体类型，它包含了在执行命令前需要设置的一些选项，以及要运行的命令以及运行该命令的方法。

`type` 表示定义了一个名为 `RunRequest` 的类型。

`var` 在定义了 `RunFuncStart` 变量后，它定义了一个名为 `RunFunc` 的函数类型，它指定了如何开始执行 `RunFunc` 中定义的命令。

`type` 再次定义了一个名为 `RunRequest` 的结构体类型，其中包含了一个名为 `CmdOpts` 的选项切片，它定义了在 `RunFunc` 中需要设置的命令选项。

`var` 接下来定义了一个名为 `RunFuncStart` 的变量，它的类型为 `(*exec.Cmd).Start`，表示它是一个指向 `exec.Cmd` 类型的指针，并且可以使用 `(*exec.Cmd).Start` 的语法来获取该指针指向的 `exec.Cmd` 对象的 `Start` 方法。

`type` 最后定义了一个名为 `RunRequest` 的结构体类型，其中包含了一个名为 `Verbose` 的布尔选项，表示是否在运行命令前设置 `--verbose` 选项。


```
type (
	CmdOpt  func(*exec.Cmd)
	RunFunc func(*exec.Cmd) error
)

var RunFuncStart = (*exec.Cmd).Start

type RunRequest struct {
	Path string
	Args []string
	// Options that are applied to the exec.Cmd just before running it
	CmdOpts []CmdOpt
	// Function to use to run the command.
	// If not specified, defaults to cmd.Run
	RunFunc func(*exec.Cmd) error
	Verbose bool
}

```

该代码定义了一个名为 `RunResult` 的结构体，其中包含以下字段：

* `Stdout`：一个指向 `Buffer` 类型的指针，用于获取标准输出（通常是命令行输出）。
* `Stderr`：一个指向 `Buffer` 类型的指针，用于获取标准错误（通常是命令行输出）。
* `Err`：一个指向 `error` 类型的指针，用于获取错误。
* `ExitErr`：一个指向 `exec.ExitError` 类型的指针，用于获取执行结果的错误。
* `Cmd`：一个指向 `exec.Cmd` 类型的指针，用于获取执行 `CMD` 命令的输出。

该结构体还包含两个方法：

* `ExitCode()`：返回 `Cmd` 的 `ProcessState.ExitCode()` 字段，表示执行结果的退出代码（通常是一个数字，如 0 或 1）。
* `environToMap()`：接受一个 `[]string` 类型的参数，将其环境变量映射到一个字符串上。这个方法将 `environ` 环境变量中的每个环境变量映射到它们的字符串表示形式，如 `"OS=windows"` 和 `"PATH=/path/to/your/app/bin:ittees=linux"`。

该代码的上下文和使用示例：

该代码定义了一个 `RunResult` 结构体，用于在命令行中运行 `CMD` 命令并获取其输出。通过组合 `Stdout` 和 `Stderr` 字段，可以获取到命令行输出的详细信息。通过调用 `ExitCode()` 方法，可以获取到执行结果的退出代码。通过调用 `environToMap()` 方法，可以将 `environ` 环境变量中的每个环境变量映射到一个字符串上，这对于某些程序在需要使用 environment 变量时，可以使用 `os.Getenv()` 函数获取环境变量，但需要注意的是该方法仅适用于需要获取命令行输出（如 Linux、macOS）的程序。


```
type RunResult struct {
	Stdout  *Buffer
	Stderr  *Buffer
	Err     error
	ExitErr *exec.ExitError
	Cmd     *exec.Cmd
}

func (r *RunResult) ExitCode() int {
	return r.Cmd.ProcessState.ExitCode()
}

func environToMap(environ []string) map[string]string {
	m := map[string]string{}
	for _, e := range environ {
		kv := strings.Split(e, "=")
		// Skip environment variables that start with =
		// These can occur in Windows https://github.com/golang/go/issues/61956
		if kv[0] == "" {
			continue
		}
		m[kv[0]] = kv[1]
	}
	return m
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `r` 的 Runner 对象和一个名为 `req` 的 RunRequest 对象作为参数。

函数的作用是运行一个命令，并返回其结果。具体来说，它执行以下操作：

1. 创建一个执行命令的实例 `cmd`，并设置其标准输出和标准错误为 `stdout` 和 `stderr`, respectively。
2. 将 RunRequest 中的路径和参数复制到 `cmd` 的环境变量中，根据请求中的环境变量，将它们添加到 `cmd` 的环境变量中。
3. 遍历请求中的命令选项，并将它们传递给 `cmd`。
4. 如果请求中定义了 `runFunc`，则使用它来运行命令。
5. 如果出现错误，记录到 `log` 中，并返回结果 `RunResult` 类型的数据。

函数的实现是非常通用的，因为它假设 Runner 对象中定义了 `runFunc` 函数，它接收一个 RunRequest 对象并执行命令。如果请求中定义了其他的命令选项或函数，可以传递给 `runFunc` 进行处理。


```
func (r *Runner) Run(req RunRequest) *RunResult {
	cmd := exec.Command(req.Path, req.Args...)
	stdout := &Buffer{}
	stderr := &Buffer{}
	cmd.Stdout = stdout
	cmd.Stderr = stderr
	cmd.Dir = r.Dir

	for k, v := range r.Env {
		cmd.Env = append(cmd.Env, fmt.Sprintf("%s=%s", k, v))
	}

	for _, o := range req.CmdOpts {
		o(cmd)
	}

	if req.RunFunc == nil {
		req.RunFunc = (*exec.Cmd).Run
	}

	log.Debugf("running %v", cmd.Args)

	err := req.RunFunc(cmd)

	result := RunResult{
		Stdout: stdout,
		Stderr: stderr,
		Cmd:    cmd,
		Err:    err,
	}

	if exitErr, ok := err.(*exec.ExitError); ok {
		result.ExitErr = exitErr
	}

	return &result
}

```

这段代码定义了一个名为MustRun的函数，该函数接受一个名为RunRequest的接口，代表一个测试套件和运行命令。函数内部执行MustRun函数的run命令，如果执行失败，则获取失败的结果并输出到控制台。

函数内部还有一个名为AssertNoError的函数，该函数接收一个已经解析好的RunResult类型的变量，代表已经执行成功的结果。函数先检查结果是否为nil，如果是，则代表运行成功，不需要做其他处理。否则，函数检查结果的错误是否为nil，如果是，则输出错误信息，否则继续执行。

MustRun函数的作用是确保RunRequest的运行成功并输出失败的结果，以便在测试执行过程中进行日志记录和调试。AssertNoError函数用于在结果解析后对结果进行处理，进一步提高代码的可读性和可维护性。


```
// MustRun runs the command and fails the test if the command fails.
func (r *Runner) MustRun(req RunRequest) *RunResult {
	result := r.Run(req)
	r.AssertNoError(result)
	return result
}

func (r *Runner) AssertNoError(result *RunResult) {
	if result.ExitErr != nil {
		log.Panicf("'%s' returned error, code: %d, err: %s\nstdout:%s\nstderr:%s\n",
			result.Cmd.Args, result.ExitErr.ExitCode(), result.ExitErr.Error(), result.Stdout.String(), result.Stderr.String())
	}
	if result.Err != nil {
		log.Panicf("unable to run %s: %s", result.Cmd.Path, result.Err)
	}
}

```

这两函数的作用是封装 fcntl 函数 runWithEnv 和 runWithPath，通过在函数内部创建一个执行上下文环境 map，然后遍历 fcntl 命令的参数 Env 字段，将传入的参数 key 和 value 分别赋值给 map 的对应键。

具体来说，RunWithEnv 函数接收一个 Env 参数，然后遍历参数列表，将每个参数 key 和 value 分别设置为传入的参数，然后将它们字符串拼接为一个新的环境字符串，并将其添加到 Env 数组中。这样，当 fcntl 命令运行时，使用 fcntl.RunWithEnv 函数时，可以保证所有传递给 fcntl 的参数都会被添加到 Env 数组中。

而 RunWithPath 函数，则接收一个路径参数，并创建一个新的执行上下文环境。它遍历 fcntl 命令的参数 Env 字段，然后将每个参数 key 提取出来，如果该参数是 "PATH"，就使用指定的路径替换 Env 数组中相应的条目，最后将修改后的 Env 数组添加到 NewEnv 数组中。最后，将 NewEnv 数组中的所有字符串按照 "=" 字符串分隔并连接，并将其添加到 Env 数组中。这样，当 fcntl 命令运行时，使用 fcntl.RunWithPath 函数时，可以保证所有传递给 fcntl 的参数都将被添加到 Env 数组中，无论这些参数是本地文件还是系统路径。


```
func RunWithEnv(env map[string]string) CmdOpt {
	return func(cmd *exec.Cmd) {
		for k, v := range env {
			cmd.Env = append(cmd.Env, fmt.Sprintf("%s=%s", k, v))
		}
	}
}

func RunWithPath(path string) CmdOpt {
	return func(cmd *exec.Cmd) {
		var newEnv []string
		for _, env := range cmd.Env {
			e := strings.Split(env, "=")
			if e[0] == "PATH" {
				paths := strings.Split(e[1], ":")
				paths = append(paths, path)
				e[1] = strings.Join(paths, ":")
				fmt.Printf("path: %s\n", strings.Join(e, "="))
			}
			newEnv = append(newEnv, strings.Join(e, "="))
		}
		cmd.Env = newEnv
	}
}

```

这段代码定义了三个名为RunWithStdin的函数，它们的参数都是一个通过stdin(读取输入)和/或stdout(输出)的io.Reader或io.Writer类型。

函数RunWithStdin创建了一个名为RunWithStdin的函数，它接收一个执行命令的执行上下文，然后将执行上下文的stdin设置为读取的io.Reader类型。

函数RunWithStdinStr创建了一个名为RunWithStdinStr的函数，它接收一个字符串参数，并将其作为io.Reader类型传递给RunWithStdin函数。

函数RunWithStdout创建了一个名为RunWithStdout的函数，它接收一个io.Writer类型参数，并将其作为执行命令的stdout设置。

总结起来，这段代码定义了三个接受io.Reader或io.Writer类型的函数，它们分别将执行命令的输入或输出设置为Reader或Writer类型。


```
func RunWithStdin(reader io.Reader) CmdOpt {
	return func(cmd *exec.Cmd) {
		cmd.Stdin = reader
	}
}

func RunWithStdinStr(s string) CmdOpt {
	return RunWithStdin(strings.NewReader(s))
}

func RunWithStdout(writer io.Writer) CmdOpt {
	return func(cmd *exec.Cmd) {
		cmd.Stdout = io.MultiWriter(writer, cmd.Stdout)
	}
}

```

这段代码定义了一个名为 "RunWithStderr" 的函数，它接收一个 "writer" 参数和一个 "cmd Opt" 类型的参数 "cmd"。

函数的作用是接受一个执行命令 "cmd"，并将其传递给 "func" 函数。在 "func" 函数内部，创建了一个接受 "cmd" 和 "writer" 参数的 "CmdOpt" 类型的临时函数。

在 "func" 函数内的 "cmd Opt" 函数中，通过 "io.MultiWriter" 类型将 "writer" 和 "cmd.Stdout" 组合在一起，将 "stdout" 字段也写入了 "writer" 中。这使得 "RunWithStderr" 函数可以将 "stdout" 输出到 "writer"。

因此，这段代码的作用是将 "stdout" 的输出同时写入 "writer"。


```
func RunWithStderr(writer io.Writer) CmdOpt {
	return func(cmd *exec.Cmd) {
		cmd.Stderr = io.MultiWriter(writer, cmd.Stdout)
	}
}

```

# `/opt/kubo/test/cli/testutils/asserts.go`

这段代码定义了一个名为 "testutils" 的包，其中包含一个名为 "AssertStringContainsOneOf" 的函数。

函数接受三个参数：

1. t: 代表当前测试客户端的断言值。
2. str: 代表要测试的字符串。
3. ss: 是一个 slice 变量，其中包含多个字符串参数。

函数的作用是检验给定的字符串是否包含给定的任意一个子字符串，然后输出相应的错误消息。

函数具体实现如下：

1. 初始化 ss 变量，包含一个空字符串和多个子字符串。
2. 遍历 ss 中的每个子字符串，使用 strings.Contains() 函数检验给定的字符串是否包含当前子字符串。
3. 如果当前字符串包含给定的子字符串，则直接返回，否则创建一个新的错误消息并将其附加给 t。
4. 函数最后使用 t.Errorf() 函数输出错误消息。


```
package testutils

import (
	"strings"
	"testing"
)

func AssertStringContainsOneOf(t *testing.T, str string, ss ...string) {
	for _, s := range ss {
		if strings.Contains(str, s) {
			return
		}
	}
	t.Errorf("%q does not contain one of %v", str, ss)
}

```

# `/opt/kubo/test/cli/testutils/cids.go`

这段代码定义了一个名为`testutils`的包，以及其中两个常量`CIDWelcomeDocs`和`CIDEmptyDir`。

`testutils`包的作用可能是用于测试某些功能的库，它可能包含了与测试相关的常量、函数或者结构体等。

`CIDWelcomeDocs`和`CIDEmptyDir`是该包中的两个常量，它们分别表示欢迎文件的ID和空目录的ID。这些常量可能在测试中用于某些用例的定义或者数据结构之中。


```
package testutils

const (
	CIDWelcomeDocs = "QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc"
	CIDEmptyDir    = "QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn"
)

```

# `/opt/kubo/test/cli/testutils/files.go`

这段代码定义了一个名为 `MustOpen` 的函数，它接受一个字符串参数 `name`，然后返回一个指向文件的 `*os.File` 类型。

函数的作用是帮助用户在操作系统中打开一个文件，如果文件不存在，函数会抛出错误并打印错误信息。函数使用 `os.Open` 函数打开文件，如果打开失败，函数会打印错误信息，并返回 ` nil`，即没有返回值。

函数的实现中，首先导入了一些依赖库，包括 `log`、`os` 和 `path/filepath`。然后定义了 `MustOpen` 函数，函数内部使用 `os.Open` 函数打开一个文件，并返回该文件的 `*os.File` 类型。函数的参数是一个字符串 `name`，表示要打开的文件的名称。函数内部先调用 `os.Open` 函数打开文件，如果文件不存在，则使用 `log.Printf` 函数抛出错误信息，并返回 ` nil`。如果文件存在，函数返回该文件的 `*os.File` 类型，即一个指向文件的引用。


```
package testutils

import (
	"log"
	"os"
	"path/filepath"
)

func MustOpen(name string) *os.File {
	f, err := os.Open(name)
	if err != nil {
		log.Panicf("opening %s: %s", name, err)
	}
	return f
}

```

这段代码定义了一个名为 `FindUp` 的函数，用于在指定的目录中搜索文件。函数接受两个参数，一个是文件名（`name`），另一个是目录名（`dir`）。函数在开始搜索之前先检查一下目录是否存在文件，如果不存在文件，则返回一个空字符串。

函数内部首先从 `dir` 目录开始递归搜索，如果搜索失败，函数会输出一个空字符串并调用 `panic` 函数。如果搜索成功，函数会继续搜索目录下的子目录。对于每个子目录，函数会先获取该目录下的所有文件信息，然后搜索文件名是否与 `name` 相等。如果是，函数返回该文件的完整路径。如果不是，函数会继续递归进入该子目录并继续搜索。

如果函数在搜索过程中遇到目录为空或非递归目录等情况，函数会循环退出搜索并返回一个空字符串。


```
// Searches for a file in a dir, then the parent dir, etc.
// If the file is not found, an empty string is returned.
func FindUp(name, dir string) string {
	curDir := dir
	for {
		entries, err := os.ReadDir(curDir)
		if err != nil {
			panic(err)
		}
		for _, e := range entries {
			if name == e.Name() {
				return filepath.Join(curDir, name)
			}
		}
		newDir := filepath.Dir(curDir)
		if newDir == curDir {
			return ""
		}
		curDir = newDir
	}
}

```

# `/opt/kubo/test/cli/testutils/floats.go`

这段代码定义了一个名为 `FloatTruncate` 的函数，它的作用是截取一个浮点数的指定位数并将结果四舍五入为浮点数。

函数接收两个参数：一个浮点数 `value` 和一个整数 `decimalPlaces`，它们分别表示要截取的位数和舍入的位数。函数内部首先定义了一个浮点数 `pow`，它的值是 1.0，然后定义一个循环 `for`，该循环的 `i` 变量从 0 到 `decimalPlaces-1`（向下取整）进行迭代。每次迭代中，将 `pow` 的值设置为之前的值乘以 10.0，即将小数点向右移动一位。这样，在循环结束后，浮点数 `pow` 将包含足够的小数位，可以进行四舍五入将浮点数截取到指定位数的精度。

接下来，在循环的外部，将 `value` 乘以 `pow`，再将结果除以 `pow`，最后将得到的结果赋值给输入的 `value`，这样就可以截取浮点数的指定位数并将其四舍五入为浮点数。


```
package testutils

func FloatTruncate(value float64, decimalPlaces int) float64 {
	pow := 1.0
	for i := 0; i < decimalPlaces; i++ {
		pow *= 10.0
	}
	return float64(int(value*pow)) / pow
}

```

# `/opt/kubo/test/cli/testutils/json.go`

这段代码定义了一个名为 `testutils` 的包，其中包含了一些用于测试的实用工具函数。

1. `import "encoding/json"` 导入了一个名为 `encoding/json` 的库，它提供了 JSON 解析和序列化的功能。

2. `type JSONObj map[string]interface{}` 定义了一个名为 `JSONObj` 的类型，它将 `map` 键的类型声明为 `interface{}`，即可以代表任何类型的接口。

3. `func ToJSONStr(m JSONObj) string` 函数将 `m` 参数中的 `JSONObj` 对象序列化为 JSON 字符串，并返回该字符串。

4. `b, err := json.Marshal(m)` 将 `m` 参数打包为一个 JSON 字符串 `b`，然后使用 `json.Marshal` 函数将 `m` 对象序列化为 JSON 字符串。如果序列化过程中出现错误，函数将输出错误并返回 `nil`。

5. `return string(b)` 函数返回生成的 JSON 字符串，作为输入参数传递给 `ToJSONStr` 函数。

6. `ToJSONStr` 函数的实现没有具体的功能，只是一个简单的包裹，将 `m` 对象序列化为 JSON 字符串，然后返回该字符串。


```
package testutils

import "encoding/json"

type JSONObj map[string]interface{}

func ToJSONStr(m JSONObj) string {
	b, err := json.Marshal(m)
	if err != nil {
		panic(err)
	}
	return string(b)
}

```

# `/opt/kubo/test/cli/testutils/random.go`

这段代码定义了两个名为 "RandomBytes" 和 "RandomStr" 的函数，以及它们的作用范围。

函数 "RandomBytes" 的作用是生成一个指定长度的字节数组。它通过调用 "crypto/rand" 包中的 "rand" 函数来生成一个随机字节数组，并将结果赋值给一个名为 "bytes" 的变量。函数使用 "make" 函数创建一个长度为 "n" 的字节数组，并在调用 "rand.Read" 函数时将生成的字节填充到数组中。如果 "rand.Read" 函数返回的值不包含在 "0" 和 "1" 之间，函数将抛出异常并返回。

函数 "RandomStr" 的作用是生成一个指定长度的字符串。它与 "RandomBytes" 函数类似，只是使用的不是字节数组，而是将生成的字节转换为字符串。它使用 "string" 函数将生成的字节转换为字符串，并将结果返回。

函数 "RandomBytes" 和 "RandomStr" 的作用范围都是在 "testutils" 包中，可能用于测试其他函数或包。


```
package testutils

import "crypto/rand"

func RandomBytes(n int) []byte {
	bytes := make([]byte, n)
	_, err := rand.Read(bytes)
	if err != nil {
		panic(err)
	}
	return bytes
}

func RandomStr(n int) string {
	return string(RandomBytes(n))
}

```

# `/opt/kubo/test/cli/testutils/random_files.go`

该代码定义了一个名为 `testutils` 的包，其中包含了一些通用的功能和工具方法。

该包导入了 `fmt`、`io`、`math/rand`、`os`、`path` 和 `time` 包，这些包分别用于格式化字符串、输入/输出、随机数生成、操作系统路径操作以及时间计算等功能。

该包中定义了两个变量 `AlphabetEasy` 和 `AlphabetHard`，它们都定义了一系列的字母字符。其中，`AlphabetEasy` 的字符串是 `abcdefghijklmnopqrstuvwxyz01234567890-_`，而 `AlphabetHard` 的字符串则包含了 `ABCDEFGHIJKLMNOPQRSTUVWXYZ` 和 `01234567890!@#$%^&*()-_+= ;.,<>'\"[]{}()`。这些字符串可以分别用于测试代码中需要用到的字母或字符。

该包还定义了一个名为 `generateUniqueString` 的函数，它会从 `AlphabetEasy` 和 `AlphabetHard` 中随机选择一个字符串，并返回该字符串。

该包最后还定义了一个名为 `generateLicenseText` 的函数，它会使用 `generateUniqueString` 函数生成一段随机的许可证文本字符串，并输出该字符串。


```
package testutils

import (
	"fmt"
	"io"
	"math/rand"
	"os"
	"path"
	"time"
)

var (
	AlphabetEasy = []rune("abcdefghijklmnopqrstuvwxyz01234567890-_")
	AlphabetHard = []rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ01234567890!@#$%^&*()-_+= ;.,<>'\"[]{}() ")
)

```

该代码定义了一个名为 `RandFiles` 的结构体，用于表示一个文件随机化工具。

该结构体包含以下字段：

- `Rand`：一个随机数生成器实例。
- `FileSize`：每个文件的二进制大小。
- `FilenameSize`：每个文件的文本大小。
- `Alphabet`：用于文件和目录名称的字符集。
- `FanoutDepth`：目录层次结构深度。
- `FanoutFiles`：每个目录下的文件数量。
- `FanoutDirs`：每个目录下的子目录数量。
- `RandomSize`：是否随机化文件大小。
- `RandomFanout`：是否随机化目录数。

该结构体的 `NewRandFiles` 函数创建了一个新的 `RandFiles` 实例，并返回其指针。

该函数的参数包括：

- `rand.New()`：用于生成新的随机数。
- `time.Now().UnixNano()`：获取当前时间的纳秒级 Unix 表示。
- `int(rand.NextInt())`：生成一个介于 0 和 1 之间的随机整数。
- `rand.Rand()`：将生成的随机数用于 `rand.NewRand()` 的生成。

新创建的 `RandFiles` 实例的默认值如下：

- `Rand`:`rand.New(rand.Now().UnixNano())`。
- `FileSize`:`4096`。
- `FilenameSize`:`16`。
- `Alphabet`:`abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`。
- `FanoutDepth`:`2`。
- `FanoutFiles`:`5`。
- `FanoutDirs`:`5`。
- `RandomSize`:`true`。
- `RandomFanout`:`true`。

该结构体的 `Randomize` 函数可以通过在 `RandFiles` 实例上设置 `RandomSize` 和 `RandomFanout` 来显式地随机化文件大小和目录数。


```
type RandFiles struct {
	Rand         *rand.Rand
	FileSize     int // the size per file.
	FilenameSize int
	Alphabet     []rune // for filenames

	FanoutDepth int // how deep the hierarchy goes
	FanoutFiles int // how many files per dir
	FanoutDirs  int // how many dirs per dir

	RandomSize   bool // randomize file sizes
	RandomFanout bool // randomize fanout numbers
}

func NewRandFiles() *RandFiles {
	return &RandFiles{
		Rand:         rand.New(rand.NewSource(time.Now().UnixNano())),
		FileSize:     4096,
		FilenameSize: 16,
		Alphabet:     AlphabetEasy,
		FanoutDepth:  2,
		FanoutDirs:   5,
		FanoutFiles:  10,
		RandomSize:   true,
	}
}

```

该函数的作用是创建一个随机文件的生成器，接收一个根目录 `root` 和一个深度 `depth`，然后按照指定的规则创建 `numfiles` 个随机文件，并填充这些文件。

具体来说，函数首先判断 `randomFanout` 是否为 `true`，如果是，就随机生成 `numFiles` 个文件，如果不是，则根据 `numFiles` 确定要生成的文件数量。接着，函数循环遍历生成的文件，对于每个文件，如果调用 `WriteRandomFile` 时出现错误，就返回错误，否则成功。

接下来，函数判断深度是否小于等于 `randomFanout` 生成的目录数量，如果是，就随机生成 `numDirectories` 个目录，如果不是，则随机生成 `numDirectories` 个目录。然后，函数循环遍历生成的目录，如果调用 `WriteRandomDir` 时出现错误，就返回错误，否则成功。

最终，函数返回 `nil` 表示没有错误。


```
func (r *RandFiles) WriteRandomFiles(root string, depth int) error {
	numfiles := r.FanoutFiles
	if r.RandomFanout {
		numfiles = rand.Intn(r.FanoutFiles) + 1
	}

	for i := 0; i < numfiles; i++ {
		if err := r.WriteRandomFile(root); err != nil {
			return err
		}
	}

	if depth+1 <= r.FanoutDepth {
		numdirs := r.FanoutDirs
		if r.RandomFanout {
			numdirs = r.Rand.Intn(numdirs) + 1
		}

		for i := 0; i < numdirs; i++ {
			if err := r.WriteRandomDir(root, depth+1); err != nil {
				return err
			}
		}
	}

	return nil
}

```

这两函数的作用如下：

1. `RandomFilename`函数的作用是生成一个指定长度的随机字符串，并将其返回。具体实现过程如下：
a. 创建一个长度为`length`的字符数组`b`；
b. 遍历`b`数组，从下标`i`开始，使用`r.Alphabet`数组中的元素，并将`r.Rand.Intn(len(r.Alphabet))`生成的随机整数赋值给`b[i]`；
c. 最后，将`b`数组中的所有元素拼接成随机字符串，并返回。
2. `WriteRandomFile`函数的作用是生成一个指定长度的随机文件，并将其写入到指定目录`root`中，如果指定`RandomSize`为`true`，则随机生成的文件长度为`rand.Intn(r.FileSize)`，否则随机文件长度为`r.FileSize`。具体实现过程如下：
a. 根据`r.RandomSize`的值，生成指定长度的随机文件；
b. 使用`os.Create`函数创建一个新文件`f`；
c. 使用`io.CopyN`函数将`r.Rand`和`r.FileSize`字节数据分别写入到文件中，写入的字节数根据`r.FileSize`与`r.RandomSize`的较小值决定；
d. 最后，使用`f.Close`函数关闭写入到文件中。


```
func (r *RandFiles) RandomFilename(length int) string {
	b := make([]rune, length)
	for i := range b {
		b[i] = r.Alphabet[r.Rand.Intn(len(r.Alphabet))]
	}
	return string(b)
}

func (r *RandFiles) WriteRandomFile(root string) error {
	filesize := int64(r.FileSize)
	if r.RandomSize {
		filesize = r.Rand.Int63n(filesize) + 1
	}

	n := rand.Intn(r.FilenameSize-4) + 4
	name := r.RandomFilename(n)
	filepath := path.Join(root, name)
	f, err := os.Create(filepath)
	if err != nil {
		return fmt.Errorf("creating random file: %w", err)
	}

	if _, err := io.CopyN(f, r.Rand, filesize); err != nil {
		return fmt.Errorf("copying random file: %w", err)
	}

	return f.Close()
}

```

该函数的作用是创建一个随机目录，并向其中写入一系列随机文件。

具体来说，函数接收两个参数：一个指向 `RandFiles` 结构体的指针 `r`，以及一个整数 `depth`。函数首先检查 `depth` 是否大于 `r.FanoutDepth`，如果是，就返回一个空字符串（表示无法创建目录）。否则，函数生成一个随机文件名，并将其与 `root` 字段拼接成一个新的目录名称。接着，函数创建一个新目录并设置目录权限为 755。然后，函数调用 `r.WriteRandomFiles` 函数，将该目录中的所有文件写入到随机文件中。最后，函数检查是否出现错误，并将错误信息返回。

该函数的实现符合 Google Rust 编程语言中的 `fmt` 函数的要求，即具有可读性和可理解性。


```
func (r *RandFiles) WriteRandomDir(root string, depth int) error {
	if depth > r.FanoutDepth {
		return nil
	}

	n := rand.Intn(r.FilenameSize-4) + 4
	name := r.RandomFilename(n)
	root = path.Join(root, name)
	if err := os.MkdirAll(root, 0o755); err != nil {
		return fmt.Errorf("creating random dir: %w", err)
	}

	err := r.WriteRandomFiles(root, depth)
	if err != nil {
		return fmt.Errorf("writing random files in random dir: %w", err)
	}
	return nil
}

```

# `/opt/kubo/test/cli/testutils/requires.go`

这段代码定义了两个名为"RequiresDocker"和"RequiresFUSE"的函数，它们的作用是检查测试程序是否使用了Docker和/或FUSE。具体来说，这两个函数会检查当前运行环境是否包含名为"TEST_DOCKER"和"TEST_FUSE"的等环境变量，如果缺少其中任何一个，则会输出一条错误消息并暂停测试程序的执行。

函数内部使用Runtime.Newline()创建一个新的字符串，然后将其附加到测试函数的参数列表中。这样做是为了确保每次调用函数时，都会输出一个新的空行，以便在控制台留下清晰的行间距离，并使得输出更易于阅读。


```
package testutils

import (
	"os"
	"runtime"
	"testing"
)

func RequiresDocker(t *testing.T) {
	if os.Getenv("TEST_DOCKER") != "1" {
		t.SkipNow()
	}
}

func RequiresFUSE(t *testing.T) {
	if os.Getenv("TEST_FUSE") != "1" {
		t.SkipNow()
	}
}

```

这段代码定义了三个名为"Requires"的函数，用于在测试运行时检查环境变量是否设置正确。

第一个函数名为"RequiresExpensive"，其作用是检查是否设置了名为"TEST_EXPENSIVE"的环境变量，如果设置了该变量并且它的值为"1"，则执行t.SkipNow()。换句话说，该函数用于测试是否使用了无用测试代码，以及在测试之前需要进行的一些设置。

第二个函数名为"RequiresPlugins"，其作用是检查是否设置了名为"TEST_PLUGIN"的环境变量，如果设置了该变量并且它的值为"1"，则执行t.SkipNow()。与第一个函数类似，该函数用于测试是否使用了无用测试代码，以及在测试之前需要进行的一些设置。

第三个函数名为"RequiresLinux"，其作用是检查运行时程序的操作系统是否为"linux"，如果不是，则执行t.SkipNow()。该函数同样用于测试是否使用了无用测试代码，以及在测试之前需要进行的一些设置。


```
func RequiresExpensive(t *testing.T) {
	if os.Getenv("TEST_EXPENSIVE") == "1" || testing.Short() {
		t.SkipNow()
	}
}

func RequiresPlugins(t *testing.T) {
	if os.Getenv("TEST_PLUGIN") != "1" {
		t.SkipNow()
	}
}

func RequiresLinux(t *testing.T) {
	if runtime.GOOS != "linux" {
		t.SkipNow()
	}
}

```

# `/opt/kubo/test/cli/testutils/strings.go`

这段代码定义了一个名为`testutils`的包，其中定义了一些`fmt`函数和一个`testutils`常量。

具体来说，这个包通过导入多个外部库来实现对网络协议的功能测试。这些导入的库包括：

- `bufio`库：用于字符串处理和文件输入/输出。
- `net`库：用于网络通信，包括TCP和UDP协议。
- `netip`库：用于IP地址解析和转换。
- `strings`库：用于字符串处理。
- `sync`库：用于同步操作。

此外，这个包定义了一个名为`testutils`的常量，可能用于在测试过程中设置一些全局性的参数或者常量。

由于`fmt`函数在这里没有被定义，所以它们的具体实现不会被输出。而`testutils`常量的具体实现，由于代码没有给出，所以我们也不清楚它具体会被用于什么目的。


```
package testutils

import (
	"bufio"
	"fmt"
	"net"
	"net/netip"
	"net/url"
	"strings"
	"sync"

	"github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
)

```

这段代码定义了一个名为 `StrCat` 的函数，它接受一组字符串或字符串切片作为参数，并将它们全部合并成一个字符串切片。

函数的实现分为两个步骤。首先，它检查每个输入参数 `a` 是否是字符串类型。如果是，函数检查 `a` 是否为空字符串，如果是，函数将不会输出该参数。否则，函数将继续执行。

对于不是字符串类型的输入参数 `a`，函数会尝试将它转换为字符串类型。如果转换成功，函数将尝试将所有其他输入参数附加到结果字符串中。如果转换失败，函数将输出错误信息并崩溃。

函数返回一个空字符串切片，其中包含所有附加到结果上的字符串。


```
// StrCat takes a bunch of strings or string slices
// and concats them all together into one string slice.
// If an arg is not one of those types, this panics.
// If an arg is an empty string, it is dropped.
func StrCat(args ...interface{}) []string {
	res := make([]string, 0)
	for _, a := range args {
		if s, ok := a.(string); ok {
			if s != "" {
				res = append(res, s)
			}
			continue
		}
		if ss, ok := a.([]string); ok {
			for _, s := range ss {
				if s != "" {
					res = append(res, s)
				}
			}
			continue
		}
		panic(fmt.Sprintf("arg '%v' must be a string or string slice, but is '%T'", a, a))
	}
	return res
}

```

这两段代码的功能是用于将字符串 `s` 的前 `previewLength` 个字符进行预览，并在必要时截取预览部分并返回。同时，这两段代码分别实现了两个与 `s` 字符串相关的函数：`PreviewStr` 和 `SplitLines`。

1. `PreviewStr` 函数接收一个字符串参数 `s`，根据传入的 `previewLength` 参数对字符串进行预览。`previewLength` 参数表示要预览的字符数量，如果接收的字符数小于 `previewLength`，则将字符数截取为 `previewLength`。函数返回预览字符串。

2. `SplitLines` 函数接收一个字符串参数 `s`，将其拆分为多个字符串并返回。函数首先使用 `bufio.NewScanner` 实例从 `s` 读取字符，然后使用 `scanner.Scan` 方法循环读取字符。在读取的字符串中，将空格和换行符作为分割符，并将剩余的字符串返回。这样，`SplitLines` 函数可以处理包含换行符的字符串，使其可以被 `bufio.S平均分割`。


```
// PreviewStr returns a preview of s, which is a prefix for logging that avoids dumping a huge string to logs.
func PreviewStr(s string) string {
	suffix := "..."
	previewLength := 10
	if len(s) < previewLength {
		previewLength = len(s)
		suffix = ""
	}
	return s[0:previewLength] + suffix
}

func SplitLines(s string) []string {
	var lines []string
	scanner := bufio.NewScanner(strings.NewReader(s))
	for scanner.Scan() {
		lines = append(lines, scanner.Text())
	}
	return lines
}

```

这段代码定义了一个名为 URLStrToMultiaddr 的函数，它接受一个 URL 字符串参数并返回一个 Multiaddr 类型。

函数首先使用 url.Parse() 函数将传入的 URL 字符串解析为 net.URL 类型，然后使用 netip.ParseAddrPort() 函数将解析出的 Host 字段转换为 net.IPAddr 类型。接着，函数使用 net.TCPAddrFromAddrPort() 函数将解析出的 IPv4 字段转换为 net.TCPAddr 类型。最后，函数使用 manet.FromNetAddr() 函数将解析出的 IPv4 字段转换为 manet.Multiaddr 类型。

如果解析过程中出现错误，函数会输出错误信息并退出。


```
// URLStrToMultiaddr converts a URL string like http://localhost:80 to a multiaddr.
func URLStrToMultiaddr(u string) multiaddr.Multiaddr {
	parsedURL, err := url.Parse(u)
	if err != nil {
		panic(err)
	}
	addrPort, err := netip.ParseAddrPort(parsedURL.Host)
	if err != nil {
		panic(err)
	}
	tcpAddr := net.TCPAddrFromAddrPort(addrPort)
	ma, err := manet.FromNetAddr(tcpAddr)
	if err != nil {
		panic(err)
	}
	return ma
}

```

这段代码定义了一个名为 `ForEachPar` 的函数，其接收一个整数类型参数 `s` 和一个函数类型参数 `f`。函数内部创建一个 `sync.WaitGroup` 对象 `wg`，并等待 `s` 中的每个元素 `x` 完成，然后执行传递给 `f` 的函数 `f(x)`。

具体来说，这段代码的作用是等待 `s` 中的每个元素执行 `f` 中的函数，并在所有元素都完成之后返回。由于 `ForEachPar` 函数使用了 `sync.WaitGroup` 对象，因此它会阻塞当前线程，直到所有元素都完成。如果某个元素出现异常终止程序，那么其他元素也会等待该异常被处理。


```
// ForEachPar invokes f in a new goroutine for each element of s and waits for all to complete.
func ForEachPar[T any](s []T, f func(T)) {
	wg := sync.WaitGroup{}
	wg.Add(len(s))
	for _, x := range s {
		go func(x T) {
			defer wg.Done()
			f(x)
		}(x)
	}
	wg.Wait()
}

```

# `/opt/kubo/test/cli/testutils/pinningservice/pinning.go`

这段代码定义了一个名为 "pinningservice" 的包，其中包含了一些用于与外部服务器进行交互的函数和变量。

该包导入了多个外部库，包括 "encoding/json"、"fmt"、"net/http"、"reflect"、"strconv"、"strings" 和 "sync" 和 "time"。这些库以及该包中定义的函数和变量将在下面被使用。

该包中定义了一些函数，包括：

- "createAndSendPostRequest" 和 "createAndSendHTTPSRequest"，这两个函数均通过调用 "net/http" 中的 "http.Client.NewRequest" 函数来发送 HTTP 请求。不同的是，第一个函数使用 "POST" 方法发送 HTTP 请求，第二个函数使用 "GET" 方法发送 HTTP 请求。

- "createAndSendJSONRequest" 和 "createAndSendJSONResponse"，这两个函数均通过调用 "reflect" 中的 " deepCopy" 函数来复制请求和响应的 JSON 数据。

- "createAndSendStringRequest" 和 "createAndSendStringResponse"，这两个函数均通过调用 "strings" 中的 "Splat" 函数来将字符串内容转换为哈希字符串，并将它们作为参数传递给 "fmt.Printf"。

- "createAndSendUUIDRequest" 和 "createAndSendUUIDResponse"，这两个函数均通过调用 "github.com/google/uuid" 中的 "uuid.New" 函数来生成唯一的 UUID。

- "createAndSendPinRequest" 和 "createAndSendPinResponse"，这两个函数均通过调用 "pinningservice/http/client" 中的 "createPinClientRequest" 和 "createPinClientResponse" 函数来创建和发送 PIN 客户端请求和响应。

- "createAndSendTemplateRequest" 和 "createAndSendTemplateResponse"，这两个函数均通过调用 "strings" 中的 "format" 函数来将模板内容转换为字符串，并将它们作为参数传递给 "fmt.Printf"。

- "createAndSendTemplateUUIDRequest" 和 "createAndSendTemplateUUIDResponse"，这两个函数均通过调用 "createAndSendUUIDRequest" 和 "createAndSendUUIDResponse" 函数来创建和发送模板 UUID 请求和响应。

- "createAndSendTemplateStringRequest" 和 "createAndSendTemplateStringResponse"，这两个函数均通过调用 "createAndSendStringRequest" 和 "createAndSendStringResponse" 函数来创建和发送模板字符串请求和响应。

- "createAndSendPostRequestResponse" 和 "createAndSendHTTPSRequestResponse"，这两个函数均通过调用 "httpprouter" 库中的 "routeHTTP" 函数来路由 HTTP 请求和响应到特定的函数。

- "createAndSendPostRequestResponse" 和 "createAndSendHTTPSRequestResponsePinResponse" 两个函数均通过调用 "createAndSendPostRequestResponse" 函数来创建和发送 HTTP 请求，然后通过调用 "createPinClientRequest" 和 "createPinClientResponse" 函数来获取 PIN 客户端，再通过调用 "fmt.Printf" 函数来将 PIN 客户端发送回客户端，并最终将客户端发送的 HTTP 请求和响应发送回服务器。


```
package pinningservice

import (
	"encoding/json"
	"fmt"
	"net/http"
	"reflect"
	"strconv"
	"strings"
	"sync"
	"time"

	"github.com/google/uuid"
	"github.com/julienschmidt/httprouter"
)

```

这段代码定义了一个名为 `NewRouter` 的函数，用于创建一个 HTTP 路由器。该路由器中定义了五个 HTTP 请求方法，分别是 GET、POST、GET、POST 和 DELETE。

GET 请求方法指向到 `/api/v1/pins` 接口，并且传递了一个 `svc.listPins` 参数，用于返回一些pin列表。

POST 请求方法指向到 `/api/v1/pins` 接口，并且传递了一个 `svc.addPin` 参数，用于将一个新的 pin 添加到系统中。

GET 请求方法指向到 `/api/v1/pins/:requestID` 接口，并且传递了一个 `svc.getPin` 参数和一个 `:requestID` 参数，用于返回指定 requestID 对应的 pin 信息。

POST 请求方法指向到 `/api/v1/pins/:requestID` 接口，并且传递了一个 `svc.replacePin` 参数和一个 `:requestID` 参数，用于替换指定 requestID 对应的 pin 信息。

DELETE 请求方法指向到 `/api/v1/pins/:requestID` 接口，并且传递了一个 `svc.removePin` 参数和一个 `:requestID` 参数，用于从系统中删除指定 requestID 对应的 pin 信息。

该函数还定义了一个名为 `authHandler` 的函数，用于处理身份验证。在 `authHandler` 中，使用了一个传递给路由器的 `authToken` 和一个指向 `PinningService` 类型的 `svc`。`authHandler` 函数的逻辑在路由器创建后开始执行，在处理一个 HTTP 请求时，首先检查请求是否需要身份验证，然后提取出授权头部中的访问令牌，检查它是否与传递给路由器的令牌匹配。如果令牌不正确或者没有传递令牌，则会返回相应的错误信息。如果令牌正确，则会继续执行路由器的业务逻辑，并将结果返回给客户端。


```
func NewRouter(authToken string, svc *PinningService) http.Handler {
	router := httprouter.New()
	router.GET("/api/v1/pins", svc.listPins)
	router.POST("/api/v1/pins", svc.addPin)
	router.GET("/api/v1/pins/:requestID", svc.getPin)
	router.POST("/api/v1/pins/:requestID", svc.replacePin)
	router.DELETE("/api/v1/pins/:requestID", svc.removePin)

	handler := authHandler(authToken, router)

	return handler
}

func authHandler(authToken string, delegate http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		authz := r.Header.Get("Authorization")
		if !strings.HasPrefix(authz, "Bearer ") {
			errResp(w, "invalid authorization token, must start with 'Bearer '", "", http.StatusBadRequest)
			return
		}

		token := strings.TrimPrefix(authz, "Bearer ")
		if token != authToken {
			errResp(w, "access denied", "", http.StatusUnauthorized)
			return
		}

		delegate.ServeHTTP(w, r)
	})
}

```

这段代码定义了一个名为New的函数，该函数返回一个指向名为PinningService的结构体的指针。

PinningService结构体包含一个PinAdded函数，该函数接受两个参数，一个AddPinRequest和一个PinStatus，然后执行相应的操作。

该函数返回的PinningService结构体，包含一个m同步锁，一个PinAdded函数，和一个pins数组，用于保存已经添加的Pin。

该函数的作用是创建一个基本的PinningService实例，该实例可以使用AddPinRequest和PinStatus作为其参数，并在PinAdded函数中执行相应的操作，以保持Pin的状态。通过调用New函数，可以创建一个新的PinningService实例，从而在需要时进行测试。


```
func New() *PinningService {
	return &PinningService{
		PinAdded: func(*AddPinRequest, *PinStatus) {},
	}
}

// PinningService is a basic pinning service that implements the Remote Pinning API, for testing Kubo's integration with remote pinning services.
// Pins are not persisted, they are just kept in-memory, and this provides callbacks for controlling the behavior of the pinning service.
type PinningService struct {
	m sync.Mutex
	// PinAdded is a callback that is invoked after a new pin is added via the API.
	PinAdded func(*AddPinRequest, *PinStatus)
	pins     []*PinStatus
}

```

这段代码定义了两个数据结构：Pin和PinStatus。

Pin结构体包含以下字段：

* CID：ID字符串，存储该Pin实例的唯一标识符。
* Name：字符串，设置为Pin实例的名称。
* Origins：字符串数组，存储该Pin实例的来源。
* Meta：包含一个map类型的字段，存储与该Pin实例相关的元数据。

PinStatus结构体包含以下字段：

* M：一个基于sync.Mutex类型的变量，用于存储当前正在进行的操作的计数。
* RequestID：字符串，设置为当前正在进行的请求的唯一标识符。
* Status：字符串，设置为Pin实例的当前状态。
* Created：时间.Time类型，设置为Pin实例创建的时间。
* Pin：Pin结构体实例，用于存储当前正在进行的请求的Pin实例。
* Delegates：字符串数组，存储当前正在进行的操作的受让者。
* Info：包含一个map类型的字段，用于存储与当前正在进行的操作相关的元数据。


```
type Pin struct {
	CID     string                 `json:"cid"`
	Name    string                 `json:"name"`
	Origins []string               `json:"origins"`
	Meta    map[string]interface{} `json:"meta"`
}

type PinStatus struct {
	M         sync.Mutex
	RequestID string
	Status    string
	Created   time.Time
	Pin       Pin
	Delegates []string
	Info      map[string]interface{}
}

```

此函数名为 `func (p *PinStatus) MarshalJSON() ([]byte, error)`，它接收一个指向 `PinStatus` 类型变量的指针 `p`，并返回一个 JSON 编码后的 `PinStatus` 对象和可能的错误。

具体来说，函数首先定义了一个名为 `pinStatusJSON` 的结构体，其中包含以下字段：

- `RequestID`：请求 ID，以json:"requestid"` 格式编码。
- `Status`：状态，以 json:"status"` 格式编码。
- `Created`：创建时间，以 json:"created"` 格式编码。
- `Pin`：指针，以 json:"pin"` 格式编码。
- `Delegates`： delegate，以 json:"delegates"` 格式编码。
- `Info`：信息，以 json:"info"` 格式编码。

接着，函数首先对 `p` 所指向的 `PinStatus` 对象进行加锁操作，以防止数据竞争问题。然后，对 `pinStatusJSON` 结构体进行 JSON 编码，并将结果返回。

如果此操作成功，函数将返回以下格式的 JSON 编码后的 `PinStatus` 对象：
json
{
 "requestid": "your_request_id",
 "status": "your_status",
 "created": "your_created_time",
 "pin": "your_pin",
 "delegates": ["your_delegate_1", "your_delegate_2", ...],
 "info": {
   "your_key1": "your_value1",
   "your_key2": "your_value2",
   ...
 }
}

如果此操作失败，函数将返回一个非空错误对象。


```
func (p *PinStatus) MarshalJSON() ([]byte, error) {
	type pinStatusJSON struct {
		RequestID string                 `json:"requestid"`
		Status    string                 `json:"status"`
		Created   time.Time              `json:"created"`
		Pin       Pin                    `json:"pin"`
		Delegates []string               `json:"delegates"`
		Info      map[string]interface{} `json:"info"`
	}
	// lock the pin before marshaling it to protect against data races while marshaling
	p.M.Lock()
	pinJSON := pinStatusJSON{
		RequestID: p.RequestID,
		Status:    p.Status,
		Created:   p.Created,
		Pin:       p.Pin,
		Delegates: p.Delegates,
		Info:      p.Info,
	}
	p.M.Unlock()
	return json.Marshal(pinJSON)
}

```

该函数名为 `Clone()`，其作用是复制一个名为 `PinStatus` 的结构体。

该函数接受一个名为 `p` 的 `PinStatus` 类型的参数，并返回一个新的 `PinStatus` 类型的值。

函数内部将 `p` 的 `RequestID`、`Status`、`Created`、`Pin`、`Delegates` 和 `Info` 字段复制到新的 `PinStatus` 结构体中，并设置其 `MatchExact`、`MatchIExact`、`MatchPartial` 和 `MatchIPartial` 字段为相应的值。

该函数还定义了一系列常量，包括 `matchExact`、`statusQueued`、`statusPinning`、`statusPinned` 和 `statusFailed`。

最后，函数使用了 `timeLayout` 常量来设置格式化时间，格式为 `"YYYY-MM-DD HH:MM:SS.999Z"`。


```
func (p *PinStatus) Clone() PinStatus {
	return PinStatus{
		RequestID: p.RequestID,
		Status:    p.Status,
		Created:   p.Created,
		Pin:       p.Pin,
		Delegates: p.Delegates,
		Info:      p.Info,
	}
}

const (
	matchExact    = "exact"
	matchIExact   = "iexact"
	matchPartial  = "partial"
	matchIPartial = "ipartial"

	statusQueued  = "queued"
	statusPinning = "pinning"
	statusPinned  = "pinned"
	statusFailed  = "failed"

	timeLayout = "2006-01-02T15:04:05.999Z"
)

```

此代码定义了一个名为 `errRespResp` 的函数，接受四个参数：

1. 一个名为 `w` 的 HTTP 响应writer。
2. 一个字符串类型的错误信息 `reason`。
3. 一个字符串类型的详细信息 `details`。
4. 一个整数类型的状态码 `statusCode`。

函数的作用是创建并返回一个错误响应对象 `errResp`。

该函数内部创建了一个名为 `errorObj` 的结构体，它包含一个名为 `Reason` 的字段和一个名为 `Details` 的字段。

接着，该函数创建了一个名为 `errResp` 的结构体，它包含一个名为 `Error` 的字段，该字段是一个 `errorObj` 类型的实例。

然后，该函数使用 `errResp` 的 `Error` 字段将错误信息 `reason` 和 `details` 写入到 HTTP 响应writer对象 `w` 中。

最后，该函数将 `errResp` 对象写入到 HTTP 响应writer对象 `w` 中，并调用 `writeJSON` 函数来写入 JSON 数据。


```
func errResp(w http.ResponseWriter, reason, details string, statusCode int) {
	type errorObj struct {
		Reason  string `json:"reason"`
		Details string `json:"details"`
	}
	type errorResp struct {
		Error errorObj `json:"error"`
	}
	resp := errorResp{
		Error: errorObj{
			Reason:  reason,
			Details: details,
		},
	}
	writeJSON(w, resp, statusCode)
}

```

这段代码定义了一个名为`writeJSON`的函数，接受一个`http.ResponseWriter`作为第一个参数，一个`any`类型的参数作为第二个参数，以及一个整数类型的参数`statusCode`。

函数首先检查传入的`val`参数是否为`nil`，如果是，则设置响应头为`Content-Type: text/plain`，并返回一个错误消息。否则，将`json.Marshal`函数应用于`val`参数，将其转换为JSON字节切片，并设置响应头为`Content-Type: application/json`。

接下来，函数创建一个名为`w`的`http.ResponseWriter`对象，并设置响应状态码为`http.StatusInternalServerError`。然后，函数 writesink 写入包含`val`字节切片的内容到响应流中，并设置`w.Header().Set`设置响应头`Content-Type`。最后，函数调用`w.Write`方法来完成写入操作。

该函数的作用是将`val`参数的JSON字节切片写入到一个HTTP响应中，并设置响应头为`Content-Type: application/json`。


```
func writeJSON(w http.ResponseWriter, val any, statusCode int) {
	b, err := json.Marshal(val)
	if err != nil {
		w.Header().Set("Content-Type", "text/plain")
		errResp(w, fmt.Sprintf("marshaling response: %s", err), "", http.StatusInternalServerError)
		return
	}
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(statusCode)
	_, _ = w.Write(b)
}

type AddPinRequest struct {
	CID     string                 `json:"cid"`
	Name    string                 `json:"name"`
	Origins []string               `json:"origins"`
	Meta    map[string]interface{} `json:"meta"`
}

```

此函数接受一个指向PinningService的指针、一个HTTP请求对象(包括请求体)和一个包含请求参数的哈希映射。

首先，它解码请求参数并将其转换为AddPinRequest类型，然后检查是否有任何错误。如果没有错误，它创建一个PinStatus对象，其中RequestID设置为一个新的UUID,Status设置为队列中的状态，Created设置为当前时间，Pin设置为给定的请求参数。

接下来，它互斥锁住PinningService的m对象，将新的PinAdd操作添加到队列的末尾，然后写入JSON格式的响应并返回。最后，它调用PinningService的PinAdded函数，传递AddReq参数，将新的PinStatus加入队列。


```
func (p *PinningService) addPin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
	var addReq AddPinRequest
	err := json.NewDecoder(req.Body).Decode(&addReq)
	if err != nil {
		errResp(writer, fmt.Sprintf("unmarshaling req: %s", err), "", http.StatusBadRequest)
		return
	}

	pin := &PinStatus{
		RequestID: uuid.NewString(),
		Status:    statusQueued,
		Created:   time.Now(),
		Pin:       Pin(addReq),
	}

	p.m.Lock()
	p.pins = append(p.pins, pin)
	p.m.Unlock()

	writeJSON(writer, &pin, http.StatusAccepted)
	p.PinAdded(&addReq, pin)
}

```

This is a Go function that handles the HTTP request for a "/pins" endpoint. It takes the request body as an afterString, which is used to determine the freshness of the pin. If the afterString is empty or invalid, the function returns an HTTP error with a given status code.

The function reads the pin data from the request body and checks its freshness. It does this by parsing the afterString to determine the timestamp for the pin data. The timestamp is then compared to the freshness of the pin data in the request body. If the pin data is not fresh, the function returns an HTTP error.

The function also handles the case where the afterString is not present or invalid. In this case, the function returns an HTTP error with a status code appropriate for the error.

The function returns an instance of the struct `ListPinsResponse`, which contains the pin data in the response body.


```
type ListPinsResponse struct {
	Count   int          `json:"count"`
	Results []*PinStatus `json:"results"`
}

func (p *PinningService) listPins(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
	q := req.URL.Query()

	cidStr := q.Get("cid")
	name := q.Get("name")
	match := q.Get("match")
	status := q.Get("status")
	beforeStr := q.Get("before")
	afterStr := q.Get("after")
	limitStr := q.Get("limit")
	metaStr := q.Get("meta")

	if limitStr == "" {
		limitStr = "10"
	}
	limit, err := strconv.Atoi(limitStr)
	if err != nil {
		errResp(writer, fmt.Sprintf("parsing limit: %s", err), "", http.StatusBadRequest)
		return
	}

	var cids []string
	if cidStr != "" {
		cids = strings.Split(cidStr, ",")
	}

	var statuses []string
	if status != "" {
		statuses = strings.Split(status, ",")
	}

	p.m.Lock()
	defer p.m.Unlock()
	var pins []*PinStatus
	for _, pinStatus := range p.pins {
		// clone it so we can immediately release the lock
		pinStatus.M.Lock()
		clonedPS := pinStatus.Clone()
		pinStatus.M.Unlock()

		// cid
		var matchesCID bool
		if len(cids) == 0 {
			matchesCID = true
		} else {
			for _, cid := range cids {
				if cid == clonedPS.Pin.CID {
					matchesCID = true
				}
			}
		}
		if !matchesCID {
			continue
		}

		// name
		if match == "" {
			match = matchExact
		}
		if name != "" {
			switch match {
			case matchExact:
				if name != clonedPS.Pin.Name {
					continue
				}
			case matchIExact:
				if !strings.EqualFold(name, clonedPS.Pin.Name) {
					continue
				}
			case matchPartial:
				if !strings.Contains(clonedPS.Pin.Name, name) {
					continue
				}
			case matchIPartial:
				if !strings.Contains(strings.ToLower(clonedPS.Pin.Name), strings.ToLower(name)) {
					continue
				}
			default:
				errResp(writer, fmt.Sprintf("unknown match %q", match), "", http.StatusBadRequest)
				return
			}
		}

		// status
		var matchesStatus bool
		if len(statuses) == 0 {
			statuses = []string{statusPinned}
		}
		for _, status := range statuses {
			if status == clonedPS.Status {
				matchesStatus = true
			}
		}
		if !matchesStatus {
			continue
		}

		// before
		if beforeStr != "" {
			before, err := time.Parse(timeLayout, beforeStr)
			if err != nil {
				errResp(writer, fmt.Sprintf("parsing before: %s", err), "", http.StatusBadRequest)
				return
			}
			if !clonedPS.Created.Before(before) {
				continue
			}
		}

		// after
		if afterStr != "" {
			after, err := time.Parse(timeLayout, afterStr)
			if err != nil {
				errResp(writer, fmt.Sprintf("parsing before: %s", err), "", http.StatusBadRequest)
				return
			}
			if !clonedPS.Created.After(after) {
				continue
			}
		}

		// meta
		if metaStr != "" {
			meta := map[string]interface{}{}
			err := json.Unmarshal([]byte(metaStr), &meta)
			if err != nil {
				errResp(writer, fmt.Sprintf("parsing meta: %s", err), "", http.StatusBadRequest)
				return
			}
			var matchesMeta bool
			for k, v := range meta {
				pinV, contains := clonedPS.Pin.Meta[k]
				if !contains || !reflect.DeepEqual(pinV, v) {
					matchesMeta = false
					break
				}
			}
			if !matchesMeta {
				continue
			}
		}

		// add the original pin status, not the cloned one
		pins = append(pins, pinStatus)

		if len(pins) == limit {
			break
		}
	}

	out := ListPinsResponse{
		Count:   len(pins),
		Results: pins,
	}
	writeJSON(writer, out, http.StatusOK)
}

```

这两函数分别是 PiningService 的两个重要方法，其作用如下：

1. `getPin` 函数接收一个 HTTP 请求、一个 HTTP 请求参数和 PiningService 的实例，它用于获取具有给定 `requestID` 的逻辑删除点（pin）。该函数首先获取参数中的 `requestID`，然后使用 `p.m.Lock()` 和 `p.m.Unlock()` 操作来获取对该 pin 的锁定状态。接着，该函数遍历存储在 `p.pins` 切片中的所有 pin，检查给定的 `requestID` 是否与任何 pin 的 `RequestID` 相等。如果是，函数将 HTTP 状态码 `http.StatusOK` 写入 `writer`，表示删除该 pin，然后返回。如果在循环中没有找到给定的 `requestID`，函数将 HTTP 状态码 `http.StatusNotFound` 写入 `writer`，并返回。

2. `replacePin` 函数同样接收一个 HTTP 请求、一个 HTTP 请求参数和 PiningService 的实例，它用于将给定 `requestID` 的逻辑删除点（pin）替换为传入的 `replaceReq`。该函数首先获取参数中的 `requestID`，然后使用 `json.NewDecoder(req.Body)` 将请求体中的 `replaceReq` 解码为 `Pin` 类型，并返回。接着，该函数使用 `p.m.Lock()` 和 `p.m.Unlock()` 操作来获取要替换的 pin 的锁定状态。在更新 pin 时，该函数使用 `p.m.Lock()` 锁定 pin，并尝试使用 `decode` 函数从请求中读取更新 `replaceReq`。如果解码过程中出现错误，函数将 HTTP 状态码 `http.StatusBadRequest` 写入 `writer`，并返回。如果更新成功，函数将 HTTP 状态码 `http.StatusAccepted` 写入 `writer`，并返回。


```
func (p *PinningService) getPin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
	requestID := params.ByName("requestID")
	p.m.Lock()
	defer p.m.Unlock()
	for _, pin := range p.pins {
		if pin.RequestID == requestID {
			writeJSON(writer, pin, http.StatusOK)
			return
		}
	}
	errResp(writer, "", "", http.StatusNotFound)
}

func (p *PinningService) replacePin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
	requestID := params.ByName("requestID")

	var replaceReq Pin
	err := json.NewDecoder(req.Body).Decode(&replaceReq)
	if err != nil {
		errResp(writer, fmt.Sprintf("decoding request: %s", err), "", http.StatusBadRequest)
		return
	}

	p.m.Lock()
	defer p.m.Unlock()
	for _, pin := range p.pins {
		if pin.RequestID == requestID {
			pin.M.Lock()
			pin.Pin = replaceReq
			pin.M.Unlock()
			writer.WriteHeader(http.StatusAccepted)
			return
		}
	}
	errResp(writer, "", "", http.StatusNotFound)
}

```

这段代码是一个名为`removePin`的函数，属于名为`PinningService`的`PinningService`类型的变量。它接受三个参数：

1. 一个指向`PinningService`类型变量的`p`参数。
2. 一个指向`http.ResponseWriter`类型变量`writer`，它用于写入响应。
3. 一个包含`httprouter.Params`类型的参数`params`。

函数的作用是移除具有给定`requestID`的`PinningService`实例中的所有挂载。首先，它确保`writer`已经写入`http.ResponseWriter`。然后，它遍历`p.pins`列表，查找具有给定`requestID`的`PinningService`实例。如果找到，它将其移动到`p.pins`的起始位置，并继续遍历。如果遍历完所有`PinningService`实例，则返回`http.StatusNotFound`错误。否则，返回`http.StatusAccepted`。


```
func (p *PinningService) removePin(writer http.ResponseWriter, req *http.Request, params httprouter.Params) {
	requestID := params.ByName("requestID")

	p.m.Lock()
	defer p.m.Unlock()

	for i, pin := range p.pins {
		if pin.RequestID == requestID {
			p.pins = append(p.pins[0:i], p.pins[i+1:]...)
			writer.WriteHeader(http.StatusAccepted)
			return
		}
	}

	errResp(writer, "", "", http.StatusNotFound)
}

```