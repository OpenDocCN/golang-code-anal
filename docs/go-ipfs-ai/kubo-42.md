# go-ipfs 源码解析 42

# `docs/examples/kubo-as-a-library/main.go`

这段代码是一个名为 "main" 的包，它包含了在 Kubernetes 中管理多重签名（multi-signature）和文件系统的工具。

首先，它导入了以下依赖项：

- "context"
- "flag"
- "fmt"
- "io"
- "log"
- "os"
- "path/filepath"
- "strings"
- "sync"

然后，它导入了以下从 "github.com/ipfs/boxo/coreiface" 和 "github.com/ipfs/boxo/files" 包中导入的功能：

- "icore"
- "files"

接下来，它导入了一个名为 "ma" 的第三方库，并定义了一个 "ma.Transport" 类型，这个类型可能用于管理多重签名。

然后，它导入了一些用于核心 Kubernetes API 的库，如 "github.com/ipfs/kubo/config" 和 "github.com/ipfs/kubo/core"。

接着，它导入了一个名为 "libp2p" 的库，用于在 Kubernetes 中管理多重签名。

然后，它导入了一个名为 "ma/transport" 的库，这个库可能用于管理多重签名。

接下来，它导入了一个名为 "path/filepath" 的库，用于管理文件系统路径。

最后，它导入了一个名为 "fmt/print" 的库，用于格式化输入/输出。

综上所述，这段代码的作用是定义了一个在 Kubernetes 中管理多重签名和文件系统的工具。


```go
package main

import (
	"context"
	"flag"
	"fmt"
	"io"
	"log"
	"os"
	"path/filepath"
	"strings"
	"sync"

	icore "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/files"
	"github.com/ipfs/boxo/path"
	ma "github.com/multiformats/go-multiaddr"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/coreapi"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/plugin/loader" // This package is needed so that all the preloaded plugins are loaded automatically
	"github.com/ipfs/kubo/repo/fsrepo"
	"github.com/libp2p/go-libp2p/core/peer"
)

```

这段代码是一个名为 `setupPlugins` 的函数，它负责设置 IPFS（InterPlanetary File System）仓库的插件。具体来说，它做了以下几件事情：

1. 如果 `externalPluginsPath` 目录不存在，则输出一个错误并返回。
2. 加载所有可用的外部插件，并将它们初始化。
3. 如果初始化过程中出现错误，则输出一个错误并返回。
4. 将加载的外部插件注入到 IPFS 仓库中。

这段代码的作用是设置 IPFS 仓库的插件，以便在初始化时能够使用预加载的和外部插件。


```go
/// ------ Setting up the IPFS Repo

func setupPlugins(externalPluginsPath string) error {
	// Load any external plugins if available on externalPluginsPath
	plugins, err := loader.NewPluginLoader(filepath.Join(externalPluginsPath, "plugins"))
	if err != nil {
		return fmt.Errorf("error loading plugins: %s", err)
	}

	// Load preloaded and external plugins
	if err := plugins.Initialize(); err != nil {
		return fmt.Errorf("error initializing plugins: %s", err)
	}

	if err := plugins.Inject(); err != nil {
		return fmt.Errorf("error initializing plugins: %s", err)
	}

	return nil
}

```

The function `initRepo` takes a `repoPath` and a `cfg` object from a `config.go` file and returns them if all initialization tasks are successful or an error occurs. Here's the code for the `initRepo` function:
go
func initRepo(repoPath string, cfg config.Configuration) error {
	// Get the temp directory to ensure the existence of it if it's not present
	tempDir, err := ioutil.TempDir()
	if err != nil {
		return err
	}

	// Create a config with default options and a 2048 bit key
	cfg, err := config.Init(io.Discard, 2048)
	if err != nil {
		return err
	}

	// When creating the repository, you can define custom settings on the repository, such as enabling experimental
	// features (See experimental-features.md) or customizing the gateway endpoint.
	// To do such things, you should modify the variable `cfg`. For example:
	if *flagExp {
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-filestore
		cfg.Experimental.FilestoreEnabled = true
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-urlstore
		cfg.Experimental.UrlstoreEnabled = true
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-p2p
		cfg.Experimental.Libp2pStreamMounting = true
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#p2p-http-proxy
		cfg.Experimental.P2pHttpProxy = true
		// See also: https://github.com/ipfs/kubo/blob/master/docs/config.md
		// And: https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md
	}

	// Create the repo with the config
	err = fsrepo.Init(repoPath, cfg)
	if err != nil {
		return err
	}

	// Return the temp directory and the nil value if the initialization was successful.
	return tempDir, nil
}

The function has a parameter of type `string` and a parameter of type `config.Configuration


```go
func createTempRepo() (string, error) {
	repoPath, err := os.MkdirTemp("", "ipfs-shell")
	if err != nil {
		return "", fmt.Errorf("failed to get temp dir: %s", err)
	}

	// Create a config with default options and a 2048 bit key
	cfg, err := config.Init(io.Discard, 2048)
	if err != nil {
		return "", err
	}

	// When creating the repository, you can define custom settings on the repository, such as enabling experimental
	// features (See experimental-features.md) or customizing the gateway endpoint.
	// To do such things, you should modify the variable `cfg`. For example:
	if *flagExp {
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-filestore
		cfg.Experimental.FilestoreEnabled = true
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-urlstore
		cfg.Experimental.UrlstoreEnabled = true
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-p2p
		cfg.Experimental.Libp2pStreamMounting = true
		// https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#p2p-http-proxy
		cfg.Experimental.P2pHttpProxy = true
		// See also: https://github.com/ipfs/kubo/blob/master/docs/config.md
		// And: https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md
	}

	// Create the repo with the config
	err = fsrepo.Init(repoPath, cfg)
	if err != nil {
		return "", fmt.Errorf("failed to init ephemeral node: %s", err)
	}

	return repoPath, nil
}

```

这段代码定义了一个名为 createNode 的函数，该函数创建一个 IPFS（InterPlanetary File System）节点并返回其核心 API。IPFS 是一种点对点分布式文件系统，可以在全球范围内高速、安全地共享文件。

函数的作用是打开一个给定的 IPFS 仓库，创建一个新的节点并设置节点的选项。然后函数将返回一个代表新创建的节点的核心 API，或者错误地返回一个空 pointer。

以下是函数的更详细的解释：

1. 函数参数：两个参数，一个是 IPFS 仓库的路径，另一个是仓库的名称。这两个参数都是字符串类型，将作为函数的参数传递给函数实现。
2. 函数实现：

a. 函数首先使用 fsrepo.Open() 函数打开 IPFS 仓库。这个函数是 IPFS 库中的一个命令行工具，用于在本地目录下创建 IPFS 节点。调用 fsrepo.Open() 时，需要提供 IPFS 仓库的路径和名称作为参数。如果调用成功，函数将返回一个 fs.Node 类型的变量，表示新的 IPFS 节点。如果调用失败，函数将返回一个 error。

b. 函数接下来设置新节点的选项。

c. 选项设置新节点成为一个完整的 DHT（Distributed Hash Table）节点，这意味着它既可以用于获取数据，也可以用于存储数据。设置选项时，使用了 libp2p.DHTOption 和 libp2p.DHTClientOption。这些选项定义了新节点应该采取的 DHT 策略。具体来说，使用了 libp2p.DHTOption 时，新节点将尝试成为 DHT 服务器，并存储和获取 DHT 数据。而使用 libp2p.DHTClientOption 时，新节点将仅作为客户端接入 DHT 网络，并从其他节点获取数据。

d. 选项还设置了新节点应该连接的 IPFS 仓库。这个选项通过 fsrepo.Open() 函数实现，并且与选项设置的 IPFS 仓库的名称相同。
3. 函数返回：新创建的 IPFS 节点的核心 API，或者错误。


```go
/// ------ Spawning the node

// Creates an IPFS node and returns its coreAPI.
func createNode(ctx context.Context, repoPath string) (*core.IpfsNode, error) {
	// Open the repo
	repo, err := fsrepo.Open(repoPath)
	if err != nil {
		return nil, err
	}

	// Construct the node

	nodeOptions := &core.BuildCfg{
		Online:  true,
		Routing: libp2p.DHTOption, // This option sets the node to be a full DHT node (both fetching and storing DHT Records)
		// Routing: libp2p.DHTClientOption, // This option sets the node to be a client DHT node (only fetching records)
		Repo: repo,
	}

	return core.NewNode(ctx, nodeOptions)
}

```

这段代码定义了一个名为spawnEphemeral的函数，它创建了一个临时的只用于当前运行的节点，并返回该节点以及用于访问该节点的IPFS节点。

函数的主要作用是创建一个临时的Temporary Repo，然后创建一个节点，并将其与API一起返回。函数使用了Once同步机制，以确保在每一次调用该函数时都只创建一次节点和API。如果任何错误发生，函数将返回一个非空 error。

函数的实现可以分为两个部分：

1. 加载插件

函数内部调用了setupPlugins函数，该函数的作用是加载所有的依赖插件。由于在代码中没有提供具体的插件列表，因此无法确定setupPlugins函数会加载哪些插件。

2. 创建临时repo

函数内部创建了一个Temporary Repo，并返回其创建的路径。如果创建Temporary Repo时出现错误，函数将返回一个错误。

3. 创建节点

函数内部创建了一个节点，并将其存储在名为node的变量中。然后，函数使用createNode函数将该节点与API一起返回。

4. 返回API和节点

最后，函数使用coreapi.NewCoreAPI函数返回一个名为api的API对象，该对象用于与Core API进行通信。然后，函数使用do...return语句返回已经创建的API和节点，如果曾经出现错误，则返回一个非空 error。


```go
var loadPluginsOnce sync.Once

// Spawns a node to be used just for this run (i.e. creates a tmp repo).
func spawnEphemeral(ctx context.Context) (icore.CoreAPI, *core.IpfsNode, error) {
	var onceErr error
	loadPluginsOnce.Do(func() {
		onceErr = setupPlugins("")
	})
	if onceErr != nil {
		return nil, nil, onceErr
	}

	// Create a Temporary Repo
	repoPath, err := createTempRepo()
	if err != nil {
		return nil, nil, fmt.Errorf("failed to create temp repo: %s", err)
	}

	node, err := createNode(ctx, repoPath)
	if err != nil {
		return nil, nil, err
	}

	api, err := coreapi.NewCoreAPI(node)

	return api, node, err
}

```

此代码定义了一个名为 `connectToPeers` 的函数，接收一个上下文上下文 `ctx`、一个 `ipfs.CoreAPI` 实例 `peers` 以及一个字符串数组 `peers`。

函数的作用是连接到给定的端点（由 `peers` 参数提供），并在连接成功后将连接信息存储在 `peerInfos`  map 中，以便后续使用。

函数的实现包括以下步骤：

1. 初始化 `wg` 和 `peerInfos` 变量。
2. 遍历 `peers` 数组，对于每个端点 `addrStr`：
a. 尝试创建一个 `ma.Multiaddr` 实例 `addr`：
b. 如果 `addr` 失败，立即返回一个错误。
c. 如果 `addr` 成功，创建一个 `peer.AddrInfo` 实例 `pii`：
d. 如果 `pii` 已经存在于 `peerInfos` map 中，检查 `ok` 变量：
   - 如果 `ok`，表示 `pii` 已经被初始化，则直接返回。
   - 如果 `ok` 条件不满足，则需要将 `pi` 设置为新的 `peer.AddrInfo` 实例，并将 `peerInfos` 中的 `pii` 更新为新的 `pi`。
   - 将 `pi` 的 `Addrs` 字段设置为 `pii.Addrs`：
3. 创建一个 `wg.WaitGroup`，等待所有 `peerInfos` 连接操作完成。
4. 对于每个 `peerInfo *peer.AddrInfo`：
a. 创建一个 `ipfs.Swarm` 实例 `swarm`：
b. 使用 `connect` 方法连接到 `swarm` 和 `peerInfo`：
c. 如果连接成功，设置 `defer` 变量为 `wg.Done`，以确保 `wg` 等待组完成。
5. 函数返回一个 `nil`，表示没有错误。


```go
func connectToPeers(ctx context.Context, ipfs icore.CoreAPI, peers []string) error {
	var wg sync.WaitGroup
	peerInfos := make(map[peer.ID]*peer.AddrInfo, len(peers))
	for _, addrStr := range peers {
		addr, err := ma.NewMultiaddr(addrStr)
		if err != nil {
			return err
		}
		pii, err := peer.AddrInfoFromP2pAddr(addr)
		if err != nil {
			return err
		}
		pi, ok := peerInfos[pii.ID]
		if !ok {
			pi = &peer.AddrInfo{ID: pii.ID}
			peerInfos[pi.ID] = pi
		}
		pi.Addrs = append(pi.Addrs, pii.Addrs...)
	}

	wg.Add(len(peerInfos))
	for _, peerInfo := range peerInfos {
		go func(peerInfo *peer.AddrInfo) {
			defer wg.Done()
			err := ipfs.Swarm().Connect(ctx, *peerInfo)
			if err != nil {
				log.Printf("failed to connect to %s: %s", peerInfo.ID, err)
			}
		}(peerInfo)
	}
	wg.Wait()
	return nil
}

```

这段代码定义了一个名为 `getUnixfsNode` 的函数，接受一个路径参数 `path`。它的作用是返回一个 `files.Node` 类型的对象，或者是 `nil` 类型的错误。

函数首先调用一个名为 `os.Stat` 的函数，并将其返回值存储在 `st` 变量中。然后，它调用一个名为 `files.NewSerialFile` 的函数，并将其设置为输入路径为 `path`，以创建一个输入文件。如果设置过程中出现错误，函数将返回 `nil` 并捕获它。

最后，函数返回输入文件 `f` 和一个错误对象 `err`。


```go
func getUnixfsNode(path string) (files.Node, error) {
	st, err := os.Stat(path)
	if err != nil {
		return nil, err
	}

	f, err := files.NewSerialFile(path, false, st)
	if err != nil {
		return nil, err
	}

	return f, nil
}

/// -------

```

Hello! How can I assist you today?


```go
var flagExp = flag.Bool("experimental", false, "enable experimental features")

func main() {
	flag.Parse()

	/// --- Part I: Getting a IPFS node running

	fmt.Println("-- Getting an IPFS node running -- ")

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// Spawn a local peer using a temporary path, for testing purposes
	ipfsA, nodeA, err := spawnEphemeral(ctx)
	if err != nil {
		panic(fmt.Errorf("failed to spawn peer node: %s", err))
	}

	peerCidFile, err := ipfsA.Unixfs().Add(ctx,
		files.NewBytesFile([]byte("hello from ipfs 101 in Kubo")))
	if err != nil {
		panic(fmt.Errorf("could not add File: %s", err))
	}

	fmt.Printf("Added file to peer with CID %s\n", peerCidFile.String())

	// Spawn a node using a temporary path, creating a temporary repo for the run
	fmt.Println("Spawning Kubo node on a temporary repo")
	ipfsB, _, err := spawnEphemeral(ctx)
	if err != nil {
		panic(fmt.Errorf("failed to spawn ephemeral node: %s", err))
	}

	fmt.Println("IPFS node is running")

	/// --- Part II: Adding a file and a directory to IPFS

	fmt.Println("\n-- Adding and getting back files & directories --")

	inputBasePath := "../example-folder/"
	inputPathFile := inputBasePath + "ipfs.paper.draft3.pdf"
	inputPathDirectory := inputBasePath + "test-dir"

	someFile, err := getUnixfsNode(inputPathFile)
	if err != nil {
		panic(fmt.Errorf("could not get File: %s", err))
	}

	cidFile, err := ipfsB.Unixfs().Add(ctx, someFile)
	if err != nil {
		panic(fmt.Errorf("could not add File: %s", err))
	}

	fmt.Printf("Added file to IPFS with CID %s\n", cidFile.String())

	someDirectory, err := getUnixfsNode(inputPathDirectory)
	if err != nil {
		panic(fmt.Errorf("could not get File: %s", err))
	}

	cidDirectory, err := ipfsB.Unixfs().Add(ctx, someDirectory)
	if err != nil {
		panic(fmt.Errorf("could not add Directory: %s", err))
	}

	fmt.Printf("Added directory to IPFS with CID %s\n", cidDirectory.String())

	/// --- Part III: Getting the file and directory you added back

	outputBasePath, err := os.MkdirTemp("", "example")
	if err != nil {
		panic(fmt.Errorf("could not create output dir (%v)", err))
	}
	fmt.Printf("output folder: %s\n", outputBasePath)
	outputPathFile := outputBasePath + strings.Split(cidFile.String(), "/")[2]
	outputPathDirectory := outputBasePath + strings.Split(cidDirectory.String(), "/")[2]

	rootNodeFile, err := ipfsB.Unixfs().Get(ctx, cidFile)
	if err != nil {
		panic(fmt.Errorf("could not get file with CID: %s", err))
	}

	err = files.WriteTo(rootNodeFile, outputPathFile)
	if err != nil {
		panic(fmt.Errorf("could not write out the fetched CID: %s", err))
	}

	fmt.Printf("got file back from IPFS (IPFS path: %s) and wrote it to %s\n", cidFile.String(), outputPathFile)

	rootNodeDirectory, err := ipfsB.Unixfs().Get(ctx, cidDirectory)
	if err != nil {
		panic(fmt.Errorf("could not get file with CID: %s", err))
	}

	err = files.WriteTo(rootNodeDirectory, outputPathDirectory)
	if err != nil {
		panic(fmt.Errorf("could not write out the fetched CID: %s", err))
	}

	fmt.Printf("Got directory back from IPFS (IPFS path: %s) and wrote it to %s\n", cidDirectory.String(), outputPathDirectory)

	/// --- Part IV: Getting a file from the IPFS Network

	fmt.Println("\n-- Going to connect to a few nodes in the Network as bootstrappers --")

	peerMa := fmt.Sprintf("/ip4/127.0.0.1/udp/4010/p2p/%s", nodeA.Identity.String())

	bootstrapNodes := []string{
		// IPFS Bootstrapper nodes.
		// "/dnsaddr/bootstrap.libp2p.io/p2p/QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN",
		// "/dnsaddr/bootstrap.libp2p.io/p2p/QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa",
		// "/dnsaddr/bootstrap.libp2p.io/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb",
		// "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",

		// IPFS Cluster Pinning nodes
		// "/ip4/138.201.67.219/tcp/4001/p2p/QmUd6zHcbkbcs7SMxwLs48qZVX3vpcM8errYS7xEczwRMA",
		// "/ip4/138.201.67.219/udp/4001/quic/p2p/QmUd6zHcbkbcs7SMxwLs48qZVX3vpcM8errYS7xEczwRMA",
		// "/ip4/138.201.67.220/tcp/4001/p2p/QmNSYxZAiJHeLdkBg38roksAR9So7Y5eojks1yjEcUtZ7i",
		// "/ip4/138.201.67.220/udp/4001/quic/p2p/QmNSYxZAiJHeLdkBg38roksAR9So7Y5eojks1yjEcUtZ7i",
		// "/ip4/138.201.68.74/tcp/4001/p2p/QmdnXwLrC8p1ueiq2Qya8joNvk3TVVDAut7PrikmZwubtR",
		// "/ip4/138.201.68.74/udp/4001/quic/p2p/QmdnXwLrC8p1ueiq2Qya8joNvk3TVVDAut7PrikmZwubtR",
		// "/ip4/94.130.135.167/tcp/4001/p2p/QmUEMvxS2e7iDrereVYc5SWPauXPyNwxcy9BXZrC1QTcHE",
		// "/ip4/94.130.135.167/udp/4001/quic/p2p/QmUEMvxS2e7iDrereVYc5SWPauXPyNwxcy9BXZrC1QTcHE",

		// You can add more nodes here, for example, another IPFS node you might have running locally, mine was:
		// "/ip4/127.0.0.1/tcp/4010/p2p/QmZp2fhDLxjYue2RiUvLwT9MWdnbDxam32qYFnGmxZDh5L",
		// "/ip4/127.0.0.1/udp/4010/quic/p2p/QmZp2fhDLxjYue2RiUvLwT9MWdnbDxam32qYFnGmxZDh5L",
		peerMa,
	}

	go func() {
		err := connectToPeers(ctx, ipfsB, bootstrapNodes)
		if err != nil {
			log.Printf("failed connect to peers: %s", err)
		}
	}()

	exampleCIDStr := peerCidFile.RootCid().String()

	fmt.Printf("Fetching a file from the network with CID %s\n", exampleCIDStr)
	outputPath := outputBasePath + exampleCIDStr
	testCID := path.FromCid(peerCidFile.RootCid())

	rootNode, err := ipfsB.Unixfs().Get(ctx, testCID)
	if err != nil {
		panic(fmt.Errorf("could not get file with CID: %s", err))
	}

	err = files.WriteTo(rootNode, outputPath)
	if err != nil {
		panic(fmt.Errorf("could not write out the fetched CID: %s", err))
	}

	fmt.Printf("Wrote the file to %s\n", outputPath)

	fmt.Println("\nAll done! You just finalized your first tutorial on how to use Kubo as a library")
}

```

# `docs/examples/kubo-as-a-library/main_test.go`

这段代码是一个 Go 语言编写的测试用例，主要作用是测试一个名为 "main.go" 的 Go 语言文件是否能够正确运行。

具体来说，代码首先导入了 "os/exec" 和 "testing" 两个包，然后定义了一个名为 "TestExample" 的函数，该函数使用了 "exec.Command" 函数来运行一个名为 "go run main.go" 的命令，并将命令的输出赋值给变量 "out" 和 "err"。

接下来，函数使用 "strings" 包中的 "contains" 函数来检查 "out" 变量中是否有字符串 "All done!"。如果是，则说明 example 运行成功，否则会输出一个错误信息。

最后，函数使用了 "testing.T" 类型的 "Test" 函数来运行测试，该函数会在测试开始时执行 "TestExample" 函数，如果函数输出的结果与 "example did not run successfully" 的预期不符，则会执行 "t.Errorf" 函数并输出错误信息。


```go
package main

import (
	"os/exec"
	"strings"
	"testing"
)

func TestExample(t *testing.T) {
	out, err := exec.Command("go", "run", "main.go").Output()
	if err != nil {
		t.Fatalf("running example (%v)", err)
	}
	if !strings.Contains(string(out), "All done!") {
		t.Errorf("example did not run successfully")
	}
}

```

# Use Kubo (go-ipfs) as a library to spawn a node and add a file

> Note: if you are trying to customize or extend Kubo, you should read the [Customizing Kubo](../../customizing.md) doc

By the end of this tutorial, you will learn how to:

- Spawn an IPFS node that runs in process (no separate daemon process)
- Create an IPFS repo
- Add files and directories to IPFS
- Retrieve those files and directories using ``cat`` and ``get``
- Connect to other nodes in the network
- Retrieve a file that only exists on the network
- The difference between a node in DHT client mode and full DHT mode

All of this using only golang!

In order to complete this tutorial, you will need:
- golang installed on your machine. See how at https://golang.org/doc/install
- git installed on your machine (so that go can download the repo and the necessary dependencies). See how at https://git-scm.com/downloads
- IPFS Desktop (for convenience) installed and running on your machine. See how at https://github.com/ipfs-shipyard/ipfs-desktop#ipfs-desktop


**Disclaimer**: The example code is quite large (more than 300 lines of code) and it has been a great way to understand the scope of the [go-ipfs Core API](https://godoc.org/github.com/ipfs/interface-go-ipfs-core), and how it can be improved to further the user experience. You can expect to be able to come back to this example in the future and see how the number of lines of code have decreased and how the example have become simpler, making other go-ipfs programs simpler as well.

## Getting started

**Note:** Make sure you have [![](https://img.shields.io/badge/go-%3E%3D1.13.0-blue.svg?style=flat-square)](https://golang.org/dl/) installed.

Download Kubo and jump into the example folder:

```goconsole
$ git clone https://github.com/ipfs/kubo.git
$ cd kubo/docs/examples/kubo-as-a-library
```

## Running the example as-is

To run the example, simply do:

```goconsole
$ go run main.go
```

You should see the following as output:

```go
-- Getting an IPFS node running --
Spawning Kubo node on a temporary repo
IPFS node is running

-- Adding and getting back files & directories --
Added file to IPFS with CID /ipfs/QmV9tSDx9UiPeWExXEeH6aoDvmihvx6jD5eLb4jbTaKGps
Added directory to IPFS with CID /ipfs/QmdQdu1fkaAUokmkfpWrmPHK78F9Eo9K2nnuWuizUjmhyn
Got file back from IPFS (IPFS path: /ipfs/QmV9tSDx9UiPeWExXEeH6aoDvmihvx6jD5eLb4jbTaKGps) and wrote it to ./example-folder/QmV9tSDx9UiPeWExXEeH6aoDvmihvx6jD5eLb4jbTaKGps
Got directory back from IPFS (IPFS path: /ipfs/QmdQdu1fkaAUokmkfpWrmPHK78F9Eo9K2nnuWuizUjmhyn) and wrote it to ./example-folder/QmdQdu1fkaAUokmkfpWrmPHK78F9Eo9K2nnuWuizUjmhyn

-- Going to connect to a few nodes in the Network as bootstrappers --
Fetching a file from the network with CID QmUaoioqU7bxezBQZkUcgcSyokatMY71sxsALxQmRRrHrj
Wrote the file to ./example-folder/QmUaoioqU7bxezBQZkUcgcSyokatMY71sxsALxQmRRrHrj

All done! You just finalized your first tutorial on how to use Kubo as a library
```

## Understanding the example

In this example, we add a file and a directory with files; we get them back from IPFS; and then we use the IPFS network to fetch a file that we didn't have in our machines before.

Each section below has links to lines of code in the file [main.go](./main.go). The code itself will have comments explaining what is happening for you.

### The `func main() {}`

The [main function](./main.go#L202-L331) is where the magic starts, and it is the best place to follow the path of what is happening in the tutorial.

### Part 1: Getting an IPFS node running

To get [get a node running](./main.go#L218-L223) as an [ephemeral node](./main.go#L114-L128) (that will cease to exist when the run ends), you will need to:

- [Prepare and set up the plugins](./main.go#L30-L47)
- [Create an IPFS repo](./main.go#L49-L68)
- [Construct the IPFS node instance itself](./main.go#L72-L96)

As soon as you construct the IPFS node instance, the node will be running.

### Part 2: Adding a file and a directory to IPFS

- [Prepare the file to be added to IPFS](./main.go#L166-L184)
- [Add the file to IPFS](./main.go#L240-L243)
- [Prepare the directory to be added to IPFS](./main.go#L186-L198)
- [Add the directory to IPFS](./main.go#L252-L255)

### Part 3: Getting the file and directory you added back

- [Get the file back](./main.go#L265-L268)
- [Write the file to your local filesystem](./main.go#L270-L273)
- [Get the directory back](./main.go#L277-L280)
- [Write the directory to your local filesystem](./main.go#L282-L285)

### Part 4: Getting a file from the IPFS network

- [Connect to nodes in the network](./main.go#L293-L310)
- [Get the file from the network](./main.go#L318-L321)
- [Write the file to your local filesystem](./main.go#L323-L326)

### Bonus: Spawn a daemon on your existing IPFS repo (on the default path ~/.ipfs)

As a bonus, you can also find lines that show you how to spawn a node over your default path (~/.ipfs) in case you had already started a node there before. To try it:

- [Comment these lines](./main.go#L219-L223)
- [Uncomment these lines](./main.go#L209-L216)

## Voilá! You are now a Kubo hacker

You've learned how to spawn a Kubo node using the Core API. There are many more [methods to experiment next](https://godoc.org/github.com/ipfs/interface-go-ipfs-core). Happy hacking!


# IPFS & Reverse HTTP Proxies

When run in production environments, go-ipfs should generally be run behind a
reverse HTTP proxy (usually NGINX). You may need a reverse proxy to:

* Load balance requests across multiple go-ipfs daemons.
* Cache responses.
* Buffer requests, only releasing them to go-ipfs when complete. This can help
  protect go-ipfs from the
  [slowloris](https://en.wikipedia.org/wiki/Slowloris_(computer_security)
  attack.
* Block content.
* Rate limit and timeout requests.
* Apply QoS rules (e.g., prioritize traffic for certain important IPFS resources).

This document contains a collection of tips, tricks, and pitfalls when running a
go-ipfs node behind a reverse HTTP proxy.

**WARNING:** Due to
[nginx#1293](https://trac.nginx.org/nginx/ticket/1293)/[go-ipfs#6402](https://github.com/ipfs/go-ipfs/issues/6402),
parts of the go-ipfs API will not work correctly behind an NGINX reverse proxy
as go-ipfs starts sending back a response before it finishes reading the request
body. The gateway itself is unaffected.

## Peering

Go-ipfs gateways behind a single load balancing reverse proxy should use the
[peering](../config.md#peering) subsystem to peer with each other. That way, as
long as one go-ipfs daemon has the content being requested, the others will be
able to serve it.

# Garbage Collection

Gateways rarely store content permanently. However, running garbage collection
can slow down a go-ipfs node significantly. If you've noticed this issue in
production, consider "garbage collecting" by resetting the go-ipfs repo whenever
you run out of space, instead of garbage collecting.

1. Initialize your gateways repo to some known-good state (possibly pre-seeding
   it with some content, a config, etc.).
2. When you start running low on space, for each load-balanced go-ipfs node:
    1. Use the nginx API to set one of the upstream go-ipfs node's to "down".
    2. Wait a minute to let go-ipfs finish processing any in-progress requests
      (or the short-lived ones, at least).
    3. Take the go-ipfs node down.
    4. Rollback the go-ipfs repo to the seed state.
    5. Restart the go-ipfs daemon.
    6. Update the nginx config, removing the "down" status from the node.

This will effectively "garbage collect" without actually running the garbage
collector.

# Content Blocking

TODO:

* Filtering requests
* Checking the X-IPFS-Path header in responses to filter again after resolving.

# Subdomain Gateway

TODO: Reverse proxies and the subdomain gateway.

# Load balancing

TODO: discuss load balancing based on the CID versus the source IP.


# `fuse/ipns/common.go`

这段代码定义了一个名为"ipns"的包，它用于管理IPFS网络中的节点和目录。具体来说，它实现了以下功能：

1. 初始化一个名为"ipns记录"的目录，如果已经存在则直接跳过；如果不存在，创建一个空的目录并将其作为根目录。
2. 将给定键的IPFS节点设置为根目录。
3. 将根目录下的所有目录和子目录的IPFS节点类型设置为"public"，以便其他节点可以通过轻量级节点的节点名称发现它们。
4. 创建一个名为"出版者"的IPFS节点，用于将编辑或覆盖其他节点。
5. 使用给定的键在IPFS网络中发布一个路径。

这里使用了多个Go标准库中的库：

- ipld-unixfs：用于管理IPFS网络中的节点和目录。
- namesys：用于管理IPFS网络中的节点和目录，以及创建和管理名称。
- kubo：用于管理IPFS网络中的节点和目录，并支持Kubernetes风格的命名空间。
- libp2p：用于实现IPFS网络中的加密和认证。
- go-libp2p：作为libp2p的Go包装器，提供了创建IPFS节点的方法。
- ipns-pubsub：实现了出版者和订阅者之间的通信。

通过这些库，ipns实现了通过名称空间对IPFS节点进行编辑和发布，使得IPFS网络更加灵活和易于使用。


```go
package ipns

import (
	"context"

	ft "github.com/ipfs/boxo/ipld/unixfs"
	"github.com/ipfs/boxo/namesys"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/kubo/core"
	ci "github.com/libp2p/go-libp2p/core/crypto"
)

// InitializeKeyspace sets the ipns record for the given key to
// point to an empty directory.
func InitializeKeyspace(n *core.IpfsNode, key ci.PrivKey) error {
	ctx, cancel := context.WithCancel(n.Context())
	defer cancel()

	emptyDir := ft.EmptyDirNode()

	err := n.Pinning.Pin(ctx, emptyDir, false)
	if err != nil {
		return err
	}

	err = n.Pinning.Flush(ctx)
	if err != nil {
		return err
	}

	pub := namesys.NewIPNSPublisher(n.Routing, n.Repo.Datastore())

	return pub.Publish(ctx, key, path.FromCid(emptyDir.Cid()))
}

```

# `fuse/ipns/ipns_test.go`

这段代码是一个 Go 语言编写的工具链，用于构建 Go 语言项目。它通过结合几个工具链（ipns、fuse、coreapi、fstest、u、ci）来确保构建过程的顺利进行。ipns 是用于构建 IPNS（InterPlanetary System）系统的工具，fuse 是用于创建自定义 FUSE（Filesystem in Userspace）系统的工具，coreapi 是用于与 IPFS（InterPlanetary System）系统的 API 进行交互的工具，而 fstest 是用于测试工具。

具体来说，这段代码的作用是确保在构建 Go 语言项目时，所有必要的构建工具都已经被安装并且配置正确。它通过运行一系列的测试来检测是否支持一些特定的工具链，如 ipns、fuse、coreapi，并在测试成功后构建项目。如果其中任何一个测试失败，工具链将会被清除并重新安装。


```go
//go:build !nofuse && !openbsd && !netbsd && !plan9
// +build !nofuse,!openbsd,!netbsd,!plan9

package ipns

import (
	"bytes"
	"context"
	"fmt"
	"io"
	mrand "math/rand"
	"os"
	"sync"
	"testing"

	"bazil.org/fuse"

	core "github.com/ipfs/kubo/core"
	coreapi "github.com/ipfs/kubo/core/coreapi"

	fstest "bazil.org/fuse/fs/fstestutil"
	u "github.com/ipfs/boxo/util"
	racedet "github.com/ipfs/go-detect-race"
	ci "github.com/libp2p/go-libp2p-testing/ci"
)

```

这两段代码的功能是：在名为 "maybeSkipFuseTests" 的函数中进行一些测试用例，以验证 FUSE（文件系统子系统，File System Subsystem） 功能是否正常运行。

具体来说：

1. 第一段代码，定义了一个名为 "func maybeSkipFuseTests" 的函数，该函数接收一个名为 "t" 的测试断言值（testing.T）。函数的作用是判断 FUSE 是否处于关闭状态，如果是，则执行接下来的代码，否则不执行。

2. 第二段代码，定义了一个名为 "func randBytes" 的函数，该函数接收一个名为 "size" 的整数参数。函数的作用是从一个名为 "u" 的缓冲区（Buffer）中生成随机大小字节，并返回生成的字节数组。函数使用了一个名为 "io.ReadFull" 的 io 函数和一个名为 "u.NewTimeSeededRand" 的新生成随机数种子函数作为实现。


```go
func maybeSkipFuseTests(t *testing.T) {
	if ci.NoFuse() {
		t.Skip("Skipping FUSE tests")
	}
}

func randBytes(size int) []byte {
	b := make([]byte, size)
	_, err := io.ReadFull(u.NewTimeSeededRand(), b)
	if err != nil {
		panic(err)
	}
	return b
}

```

这两段代码都尝试实现了文件操作。

`func mkdir(t *testing.T, path string) {
	err := os.Mkdir(path, os.ModeDir)
	if err != nil {
		t.Fatal(err)
	}
}`

这段代码定义了一个名为 `mkdir` 的函数，接受两个参数 `t` 和 `path`。函数的作用是在指定的路径下创建一个新目录。如果目录已经存在，函数将抛出错误。

`func writeFileOrFail(t *testing.T, size int, path string) []byte {
	data, err := writeFile(size, path)
	if err != nil {
		t.Fatal(err)
	}
	return data
}`

这段代码定义了一个名为 `writeFileOrFail` 的函数，接受两个参数 `t` 和 `size` 和 `path`。函数的作用是 writingFile 函数的补充版本，可以同时成功写入文件并返回数据。函数抛出错误，当调用时，如果没有错误发生，则表示文件已成功写入。


```go
func mkdir(t *testing.T, path string) {
	err := os.Mkdir(path, os.ModeDir)
	if err != nil {
		t.Fatal(err)
	}
}

func writeFileOrFail(t *testing.T, size int, path string) []byte {
	data, err := writeFile(size, path)
	if err != nil {
		t.Fatal(err)
	}
	return data
}

```

这两段代码都是Go语言中的函数，它们的目的是分别实现“writeFile”和“verifyFile”函数。

1. “writeFile”函数接收两个参数，一个是大小（int类型），另一个是文件路径（string类型）。它从随机大小字节数组中生成字节数组，并将字节数组中的数据写入到指定的文件路径中。函数的返回值是字节数组和错误对象。如果调用此函数时出现错误，函数将返回其错误对象。

2. “verifyFile”函数接收两个参数，一个是文件路径（string类型），另一个是预期的数据（[]byte类型）。它从文件中读取数据并将其存储在变量“isData”中。如果从文件中读取的数据与预期的数据不匹配，函数将打印错误并返回错误对象。

“writeFile”函数主要用于在需要将随机生成的字节数据写入到指定文件时提供一种简单的方法。它确保了文件的大小和权限正确设置。但需要注意的是，该函数在写入数据之前需要提供文件系统访问权限，这意味着它不能用于从具有执行权限的进程写入数据。

“verifyFile”函数用于验证从文件中读取的数据是否与预期数据相等。它使用os.ReadFile函数从文件中读取数据，并将其与一个已知的预期数据切片进行比较。如果读取的数据与预期数据不匹配，函数将打印错误并返回错误对象。


```go
func writeFile(size int, path string) ([]byte, error) {
	data := randBytes(size)
	err := os.WriteFile(path, data, 0o666)
	return data, err
}

func verifyFile(t *testing.T, path string, wantData []byte) {
	isData, err := os.ReadFile(path)
	if err != nil {
		t.Fatal(err)
	}
	if len(isData) != len(wantData) {
		t.Fatal("Data not equal - length check failed")
	}
	if !bytes.Equal(isData, wantData) {
		t.Fatal("Data not equal")
	}
}

```

这两段代码定义了两个函数，作用如下：

1. `checkExists` 函数的作用是检查给定的路径是否存在于操作系统中。它使用了 `os.Stat` 函数来获取路径的文件状态信息 (如果路径不存在，函数会崩溃并返回一个错误)。如果路径存在，函数会尝试使用 `os.Exit` 函数来退出，如果失败了，函数会崩溃并返回一个错误。
2. `closeMount` 函数的作用是关闭已经挂载的文件系统。它接收一个 `mountWrap` 类型的参数，并使用 `if err := recover(); err != nil` 代码来捕获任何 panics that may occur during the process. 如果 panics occur，函数会崩溃并记录错误信息。然后，函数关闭挂载的文件系统，并调用 `mnt.Close()` 函数关闭已经挂载的文件系统。


```go
func checkExists(t *testing.T, path string) {
	_, err := os.Stat(path)
	if err != nil {
		t.Fatal(err)
	}
}

func closeMount(mnt *mountWrap) {
	if err := recover(); err != nil {
		log.Error("Recovered panic")
		log.Error(err)
	}
	mnt.Close()
}

```

该代码定义了一个名为 mountWrap 的结构体，该结构体包含一个 fstest.Mount 类型的指针和一个 FileSystem 类型的指针。

在 Close() 函数中，关闭了当前挂载的文件系统，并关闭了测试中使用的 fuse 功能。

在 setupIpnsTest() 函数中，初始化了一个 IpfsNode，并调用了一系列 Ipfs 功能，包括初始化 Keyspace 和创建文件系统。在创建文件系统时，可能需要关闭之前的操作，以确保只读或只写。

最后，在 IpfsNode 上使用了 mount 函数，并返回了一个指向 fstest.Mount 类型和 fs 类型对象的引用。


```go
type mountWrap struct {
	*fstest.Mount
	Fs *FileSystem
}

func (m *mountWrap) Close() error {
	m.Fs.Destroy()
	m.Mount.Close()
	return nil
}

func setupIpnsTest(t *testing.T, node *core.IpfsNode) (*core.IpfsNode, *mountWrap) {
	t.Helper()
	maybeSkipFuseTests(t)

	var err error
	if node == nil {
		node, err = core.NewNode(context.Background(), &core.BuildCfg{})
		if err != nil {
			t.Fatal(err)
		}

		err = InitializeKeyspace(node, node.PrivateKey)
		if err != nil {
			t.Fatal(err)
		}
	}

	coreAPI, err := coreapi.NewCoreAPI(node)
	if err != nil {
		t.Fatal(err)
	}

	fs, err := NewFileSystem(node.Context(), coreAPI, "", "")
	if err != nil {
		t.Fatal(err)
	}
	mnt, err := fstest.MountedT(t, fs, nil)
	if err == fuse.ErrOSXFUSENotFound {
		t.Skip(err)
	}
	if err != nil {
		t.Fatalf("error mounting at temporary directory: %v", err)
	}

	return node, &mountWrap{
		Mount: mnt,
		Fs:    fs,
	}
}

```

这段代码是一个名为 "TestIpnsLocalLink" 的函数，是用来测试本地链接的。它接受一个测试框架 "testing.T" 作为参数。

以下是代码的作用：

1. 设置 Ipns 测试的上下文，如果没有提供上下文，则使用默认的上下文。
2. 创建一个名为 "local" 的目录，并将它作为 Ipns 测试的目标目录。
3. 使用 os.Readlink 函数读取本地链接，并存储在变量 "linksto" 中。
4. 检查 "linksto" 是否等于 "nd.Identity.String()"。
5. 如果 "linksto" 不等于 "nd.Identity.String()"，则输出错误并退出测试。

由于上下文和测试函数都没有提供更多的信息，因此无法提供更多有关代码如何工作的背景。


```go
func TestIpnsLocalLink(t *testing.T) {
	nd, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()
	name := mnt.Dir + "/local"

	checkExists(t, name)

	linksto, err := os.Readlink(name)
	if err != nil {
		t.Fatal(err)
	}

	if linksto != nd.Identity.String() {
		t.Fatal("Link invalid")
	}
}

```

这段代码的作用是测试 IpnsBasicIO 函数的正确性。IpnsBasicIO 函数用于在本地测试文件系统操作。

具体来说，这段代码会首先创建一个名为 "local/testfile" 的文件，并向其中写入 10 字节的数据。然后，它将文件读取回并将其存储在名为 "rbuf" 的缓冲区中。

接下来，它再次创建一个名为 "local/testfile" 的文件，并将其读取回并将其存储在名为 "rbuf2" 的缓冲区中。

最后，它比较 "rbuf" 和 "rbuf2" 中的数据是否相等。如果不相等，它就会输出 "Incorrect Read!" 错误并跳过当前测试。否则，它将不会输出任何错误信息，而是继续执行之前的操作。


```go
// Test writing a file and reading it back.
func TestIpnsBasicIO(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	nd, mnt := setupIpnsTest(t, nil)
	defer closeMount(mnt)

	fname := mnt.Dir + "/local/testfile"
	data := writeFileOrFail(t, 10, fname)

	rbuf, err := os.ReadFile(fname)
	if err != nil {
		t.Fatal(err)
	}

	if !bytes.Equal(rbuf, data) {
		t.Fatal("Incorrect Read!")
	}

	fname2 := mnt.Dir + "/" + nd.Identity.String() + "/testfile"
	rbuf, err = os.ReadFile(fname2)
	if err != nil {
		t.Fatal(err)
	}

	if !bytes.Equal(rbuf, data) {
		t.Fatal("Incorrect Read!")
	}
}

```

这段代码的主要目的是测试文件在挂载IPNS过程中的持久性。它分为两个部分：

1. 设置IPNS环境并进行设置测试的函数。这个部分的作用是在IPNS中创建一个测试文件，并尝试在第一次挂载后和第二次挂载后分别读取该文件的内容。如果文件数据在两次挂载之间发生变化，那么测试就会失败。

2. 读取文件并比较内容以检查文件数据是否在两次挂载之间发生变化的具体实现。

具体来说，代码首先创建一个名为"/local/atestfile"的文件，并向其中写入127个字节的数据。然后，代码挂载IPNS并关闭了之前打开的文件。接着，代码再次尝试挂载IPNS并打开之前创建的文件。如果IPNS在两次挂载之间成功读取并写入了文件数据，那么它们的 contents应该相同。否则，如果它们的内容不同，那么测试就会失败。


```go
// Test to make sure file changes persist over mounts of ipns.
func TestFilePersistence(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	node, mnt := setupIpnsTest(t, nil)

	fname := "/local/atestfile"
	data := writeFileOrFail(t, 127, mnt.Dir+fname)

	mnt.Close()

	t.Log("Closed, opening new fs")
	_, mnt = setupIpnsTest(t, node)
	defer mnt.Close()

	rbuf, err := os.ReadFile(mnt.Dir + fname)
	if err != nil {
		t.Fatal(err)
	}

	if !bytes.Equal(rbuf, data) {
		t.Fatalf("File data changed between mounts! sizes differ: %d != %d", len(data), len(rbuf))
	}
}

```

这段代码是一个名为 "TestMultipleDirs" 的测试函数，用于测试在不同目录下是否存在文件。它使用了一位代表测试的上下文信息 "t"，以及一个名为 "node" 的测试代理对象。

首先，函数会调用 "setupIpnsTest" 函数来设置 Ipns 测试环境。然后，它会创建一个名为 "dir1" 的顶级目录。

接下来，函数会输出一个顶级目录已经创建的事实。然后，它会在 "dir1" 目录下创建一个名为 "file1" 的文件，并尝试写入一些内容。

接着，函数会输出一个名为 "file1" 的文件的 exist 状态，如果文件已存在，则使用 "writeFileOrFail" 函数尝试写入的内容。如果写入成功，函数会使用 "verifyFile" 函数验证文件的内容是否正确。

然后，函数会创建一个名为 "dir2" 的子目录。接着，函数会输出一个名为 "dir2" 的目录是否存在，如果存在，则使用 "writeFileOrFail" 函数尝试写入一些内容。如果写入成功，函数会使用 "verifyFile" 函数验证文件的内容是否正确。

接下来，函数关闭了 "dir2" 目录并重新挂载了 Ipns 测试环境。然后，它关闭了 "node" 对象并重新启动了 Ipns 测试环境。

最后，函数会再次调用 "setupIpnsTest" 函数来关闭 "node" 对象。


```go
func TestMultipleDirs(t *testing.T) {
	node, mnt := setupIpnsTest(t, nil)

	t.Log("make a top level dir")
	dir1 := "/local/test1"
	mkdir(t, mnt.Dir+dir1)

	checkExists(t, mnt.Dir+dir1)

	t.Log("write a file in it")
	data1 := writeFileOrFail(t, 4000, mnt.Dir+dir1+"/file1")

	verifyFile(t, mnt.Dir+dir1+"/file1", data1)

	t.Log("sub directory")
	mkdir(t, mnt.Dir+dir1+"/dir2")

	checkExists(t, mnt.Dir+dir1+"/dir2")

	t.Log("file in that subdirectory")
	data2 := writeFileOrFail(t, 5000, mnt.Dir+dir1+"/dir2/file2")

	verifyFile(t, mnt.Dir+dir1+"/dir2/file2", data2)

	mnt.Close()
	t.Log("closing mount, then restarting")

	_, mnt = setupIpnsTest(t, node)

	checkExists(t, mnt.Dir+dir1)

	verifyFile(t, mnt.Dir+dir1+"/file1", data1)

	verifyFile(t, mnt.Dir+dir1+"/dir2/file2", data2)
	mnt.Close()
}

```

这段代码的作用是测试文件系统是否正确地报告文件大小。它使用了 `testing.short` 函数，因此不会输出详细信息，只会输出一个简单的断言。

具体来说，这段代码包括以下步骤：

1. 设置 `setupIpnsTest` 函数，用于设置 Ipns 测试环境。
2. 创建一个名为 `local/sizecheck` 的目录，并在其中创建一个名为 `local/sizecheck` 的文件。
3. 使用 `writeFileOrFail` 函数向文件 `local/sizecheck` 中写入数据。
4. 使用 `os.Stat` 函数获取文件 `local/sizecheck` 的元数据信息，如文件大小。
5. 比较文件大小和实际数据长度，如果两者不相等，则输出错误信息并跳过当前测试。

这段代码的作用是确保文件系统能够正确地报告文件大小，可以用于测试文件系统功能正常运行。


```go
// Test to make sure the filesystem reports file sizes correctly.
func TestFileSizeReporting(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	_, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()

	fname := mnt.Dir + "/local/sizecheck"
	data := writeFileOrFail(t, 5555, fname)

	finfo, err := os.Stat(fname)
	if err != nil {
		t.Fatal(err)
	}

	if finfo.Size() != int64(len(data)) {
		t.Fatal("Read incorrect size from stat!")
	}
}

```

这段代码是用来测试一个名为“DoubleEntryFailure”的函数。它的作用是验证在给定同一个名字的情况下，是否可以创建多个entry。

具体来说，这段代码包括以下步骤：

1. 设置一个名为“doubleentryfailure”的测试函数；
2. 如果当前函数是测试时间，则跳过；
3. 创建一个名为“/local/thisisadir”的目录，并把它赋值给变量“dname”；
4. 创建一个新的目录，并把当前目录的权限设置为777；
5. 如果出现错误，打印错误信息并跳过；
6. 如果新的目录创建成功，打印错误信息，并等待测试时间结束。

如果测试时间小于设置为short，则会执行步骤3至6，即创建目录并测试它是否可以创建多个具有相同名字的entry。如果实验失败，函数将打印错误信息并退出。


```go
// Test to make sure you can't create multiple entries with the same name.
func TestDoubleEntryFailure(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	_, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()

	dname := mnt.Dir + "/local/thisisadir"
	err := os.Mkdir(dname, 0o777)
	if err != nil {
		t.Fatal(err)
	}

	err = os.Mkdir(dname, 0o777)
	if err == nil {
		t.Fatal("Should have gotten error one creating new directory.")
	}
}

```

这段代码是一个名为 `TestAppendFile` 的函数，它是为测试框架 `testing` 中的 `TestAppendFile` 函数提供实现的。

具体来说，这段代码的作用是测试一个名为 `localhost` 的服务器上，有一个名为 `/local/file` 的文件，向该文件中写入一组字节数据，然后关闭文件并尝试从该文件中读取之前写入的数据，以确保读取的数据与之前写入的数据相同。

以下是代码的步骤：

1. 如果当前测试处于短测试模式（即不使用 `-n` 选项指定测试套件数量），则直接跳过本次测试，不需要执行任何操作。

2. 调用 `setupIpnsTest` 函数，传递一个空指针 `nil`，作为参数。这个函数的作用在下一行中被实现。

3. 创建一个名为 `fname` 的文件，并将其目录设置为 `/local/file`，使用 `writeFileOrFail` 函数将一个大小为 1300 的字节数据写入该文件。

4. 关闭文件并尝试从文件中读取之前写入的数据，使用 `os.OpenFile` 函数打开文件，并使用 `os.O_RDWR` 和 `os.O_APPEND` 选项以只读写的方式打开文件。

5. 创建一个大小为 500 的字节数组 `nudata`。

6. 使用 `fi.Write` 函数向文件中写入 `nudata` 字节数组中的数据。

7. 关闭文件并确保所有写入的数据已写入文件。

8. 如果文件已成功写入数据，则再次打开文件并从文件中读取之前写入的数据，使用 `os.ReadFile` 函数从文件中读取数据。

9. 比较从文件中读取的数据与之前写入的数据是否相同，如果不同，则打印出 "Failed to write enough bytes." 的错误消息，并跳过当前测试。

10. 如果所有操作都成功，则不输出任何信息，并跳过当前测试。


```go
func TestAppendFile(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	_, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()

	fname := mnt.Dir + "/local/file"
	data := writeFileOrFail(t, 1300, fname)

	fi, err := os.OpenFile(fname, os.O_RDWR|os.O_APPEND, 0o666)
	if err != nil {
		t.Fatal(err)
	}

	nudata := randBytes(500)

	n, err := fi.Write(nudata)
	if err != nil {
		t.Fatal(err)
	}
	err = fi.Close()
	if err != nil {
		t.Fatal(err)
	}

	if n != len(nudata) {
		t.Fatal("Failed to write enough bytes.")
	}

	data = append(data, nudata...)

	rbuf, err := os.ReadFile(fname)
	if err != nil {
		t.Fatal(err)
	}
	if !bytes.Equal(rbuf, data) {
		t.Fatal("Data inconsistent!")
	}
}

```

这段代码是用来测试并发写入的。测试中会创建 `nactors` 个不同的文件，每个文件大小为 `fileSize`。然后在每个文件中，尝试写入 `filesPerActor` 个字节的数据。

代码的作用如下：

1. 安装 `testing` 和 `github.com/stretchr/testify/assert` 库。
2. 如果当前目录下有一个名为 `ipns` 的测试目录，则执行 `setupIpnsTest` 函数来设置 `ipns` 测试目录下的环境。
3. 如果当前目录下没有名为 `ipns` 的测试目录，则执行 `createIpnsTestDirectory` 函数来创建一个名为 `ipns` 的测试目录。
4. 设置 `nactors` 为 `4`，以及 `filesPerActor` 为 `400`。
5. 创建一个大小为 `fileSize` 的 `n` 维数组 `data`，其中 `n` 为 `nactors`。
6. 如果当前目录下有一个名为 `racedet.WithRace` 的函数，则执行 `racedet.WithRace` 函数，设置 `nactors` 为 `2`，以及 `filesPerActor` 为 `50`。
7. 开始一个 `sync.WaitGroup` 并等待所有 `go` 函数完成。
8. 遍历每个 `data` 元素，创建一个大小为 `filesPerActor` 的 `n` 维数组 `out`，其中包含写入的文件号。
9. 调用 `writeFile` 函数，将数据写入到 `out` 中，并记录文件号和 `i`。
10. 如果写入失败，则输出错误并跳过当前循环。
11. 循环遍历每个文件数组，调用 `verifyFile` 函数，比较当前 `data` 数组中的元素和 `out` 数组中的元素，并输出结果。


```go
func TestConcurrentWrites(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	_, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()

	nactors := 4
	filesPerActor := 400
	fileSize := 2000

	data := make([][][]byte, nactors)

	if racedet.WithRace() {
		nactors = 2
		filesPerActor = 50
	}

	wg := sync.WaitGroup{}
	for i := 0; i < nactors; i++ {
		data[i] = make([][]byte, filesPerActor)
		wg.Add(1)
		go func(n int) {
			defer wg.Done()
			for j := 0; j < filesPerActor; j++ {
				out, err := writeFile(fileSize, mnt.Dir+fmt.Sprintf("/local/%dFILE%d", n, j))
				if err != nil {
					t.Error(err)
					continue
				}
				data[n][j] = out
			}
		}(i)
	}
	wg.Wait()

	for i := 0; i < nactors; i++ {
		for j := 0; j < filesPerActor; j++ {
			if data[i][j] == nil {
				// Error already reported.
				continue
			}
			verifyFile(t, mnt.Dir+fmt.Sprintf("/local/%dFILE%d", i, j), data[i][j])
		}
	}
}

```

This is a Go program that creates a new directory called `dir` and creates a number of files in it, each with a random name. The program uses the `os` package to create the directory and the `iostream` package to read and write data to the files.

The program is structured as follows:

1. The first three lines initialize the worker variables: `i`, `nfileWorkers`, and `nfiles`.
2. The next five lines check if the directory `dir` already exists. If it does, the program exits with an error. If it doesn't, the program creates the directory and returns.
3. The next three lines initialize the data structures for the files: `files`, `dirs`, and `nfiles`.
4. The next five lines read the names of the files from the `dirs` slice and create a new file with the same name in each directory，除非一个新的文件已经在 `files` 中出现。
5. The next line allocate a workspace for the worker.
6. The workspace accesses the directory and the files in it, and writes data to the new file.
7. The workspace waits for all the files to be processed and returns.
8. Finally, the program reads the data from the new files and checks if it matches what was written. If it doesn't, an error is printed.


```go
func TestFSThrash(t *testing.T) {
	files := make(map[string][]byte)

	if testing.Short() {
		t.SkipNow()
	}
	_, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()

	base := mnt.Dir + "/local"
	dirs := []string{base}
	dirlock := sync.RWMutex{}
	filelock := sync.Mutex{}

	ndirWorkers := 2
	nfileWorkers := 2

	ndirs := 100
	nfiles := 200

	wg := sync.WaitGroup{}

	// Spawn off workers to make directories
	for i := 0; i < ndirWorkers; i++ {
		wg.Add(1)
		go func(worker int) {
			defer wg.Done()
			for j := 0; j < ndirs; j++ {
				dirlock.RLock()
				n := mrand.Intn(len(dirs))
				dir := dirs[n]
				dirlock.RUnlock()

				newDir := fmt.Sprintf("%s/dir%d-%d", dir, worker, j)
				err := os.Mkdir(newDir, os.ModeDir)
				if err != nil {
					t.Error(err)
					continue
				}
				dirlock.Lock()
				dirs = append(dirs, newDir)
				dirlock.Unlock()
			}
		}(i)
	}

	// Spawn off workers to make files
	for i := 0; i < nfileWorkers; i++ {
		wg.Add(1)
		go func(worker int) {
			defer wg.Done()
			for j := 0; j < nfiles; j++ {
				dirlock.RLock()
				n := mrand.Intn(len(dirs))
				dir := dirs[n]
				dirlock.RUnlock()

				newFileName := fmt.Sprintf("%s/file%d-%d", dir, worker, j)

				data, err := writeFile(2000+mrand.Intn(5000), newFileName)
				if err != nil {
					t.Error(err)
					continue
				}
				filelock.Lock()
				files[newFileName] = data
				filelock.Unlock()
			}
		}(i)
	}

	wg.Wait()
	for name, data := range files {
		out, err := os.ReadFile(name)
		if err != nil {
			t.Error(err)
		}

		if !bytes.Equal(data, out) {
			t.Errorf("Data didn't match in %s: expected %v, got %v", name, data, out)
		}
	}
}

```

该测试代码的作用是测试在分布式锁写入文件的情况下，每次写入多少字节可以确保在同一时间有多个进程同时访问到同一个文件，从而实现并发访问。

具体来说，该测试代码使用Linux的`fi`文件系统来创建一个名为`local_file`的文件，并使用`randBytes`函数生成一个大小为1001字节的数据包。然后，该代码使用`fi.Write`函数将数据包中的数据写入到`fi`文件中。为了模拟多个进程同时访问文件，该代码使用一个循环来多次调用`fi.Write`函数，每次调用都传入不同的数据接口（即每次写入的数据字节数不同）。

如果`fi.Write`函数在调用过程中出现错误，例如写入数据时写入的内存不足或者写入数据时出现错误，那么该代码会记录错误并跳过该次测试。如果所有测试都通过，那么测试将通过，并输出测试的名称。否则，测试将失败，并输出错误信息。


```go
// Test writing a medium sized file one byte at a time.
func TestMultiWrite(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}

	_, mnt := setupIpnsTest(t, nil)
	defer mnt.Close()

	fpath := mnt.Dir + "/local/file"
	fi, err := os.Create(fpath)
	if err != nil {
		t.Fatal(err)
	}

	data := randBytes(1001)
	for i := 0; i < len(data); i++ {
		n, err := fi.Write(data[i : i+1])
		if err != nil {
			t.Fatal(err)
		}
		if n != 1 {
			t.Fatal("Somehow wrote the wrong number of bytes! (n != 1)")
		}
	}
	fi.Close()

	rbuf, err := os.ReadFile(fpath)
	if err != nil {
		t.Fatal(err)
	}

	if !bytes.Equal(rbuf, data) {
		t.Fatal("File on disk did not match bytes written")
	}
}

```