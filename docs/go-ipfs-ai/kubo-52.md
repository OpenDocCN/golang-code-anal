# go-ipfs 源码解析 52

# `repo/fsrepo/migrations/ipfsfetcher/ipfsfetcher_test.go`

这段代码是一个名为 "ipfsfetcher" 的包，它的作用是下载一个名为 "example.文本" 的文件，并将其存储在本地磁盘上。

它首先导入了以下外部库：


package ipfsfetcher
import (
  "bufio"
  "bytes"
  "context"
  "fmt"
  "os"
  "path/filepath"
  "testing"
)


然后，它实现了以下功能：


//下载文件并写入缓冲区
func downloadFile(path string, dest bytes.Buffer, info *load.FileInfo) error {
  if err := os.Mkdir(path); err != nil {
     return err
  }
  if err := syscall.Chmod(0775, path); err != nil {
     return err
  }
  if err := syscall.Chdir(path); err != nil {
     return err
  }
  if err := syscall.Fetch(path, "GET", ""); err != nil {
     return err
  }
  if err := os.MemExchange(dest, bytes.NewBufferString(info.Body)); err != nil {
     return err
  }
  return nil
}


这段代码中的 `downloadFile` 函数从远程服务器下载一个文件，并将其写入一个缓冲区 `bytes.Buffer` 中。注意，它使用 `ipfsfetcher/plugin/loader` 和 `ipfsfetcher/repo/fsrepo/migrations` 包来下载文件。

函数的参数包括：

- `path`：文件存储目录，这里是当前工作目录。
- `dest`：目标文件缓冲区。
- `info`：下载的文件信息，包括 `FileInfo` 类型的文件元数据。

函数实现了一个简单的文件下载过程，将下载的文件写入缓冲区并返回。如果下载过程中出现错误，函数返回错误。


```go
package ipfsfetcher

import (
	"bufio"
	"bytes"
	"context"
	"fmt"
	"os"
	"path/filepath"
	"testing"

	"github.com/ipfs/kubo/plugin/loader"
	"github.com/ipfs/kubo/repo/fsrepo/migrations"
)

```

该代码是一个名为 `IpfsFetcher` 的函数，属于一个名为 `testIpfsFetcher` 的测试函数。函数初始化时执行 `setupPlugins` 函数，如果初始化失败，则输出错误并中止执行。如果初始化成功，则执行以下操作：

1. 设置 `IpfsFetcher` 函数的参数，即要使用的版本号。
2. 如果设置失败，执行 `setupPlugins` 函数。
3. 调用 `setupPlugins` 函数。
4. 如果调用 `setupPlugins` 函数成功，设置一个 cancel 事件，并在事件触发时执行 `close` 操作。
5. 调用 `IpfsFetcher` 函数。
6. 调用 `IpfsFetcher` 函数尝试获取 "go-ipfs/versions" 版本，并存储到 `lines` 切片中的第一个元素。
7. 检查 `lines` 切片的长度是否等于 6。
8. 如果 `lines` 切片的长度小于 6，则执行 `t.Fatal` 函数并输出错误信息。
9. 如果 `lines` 切片的长度大于等于 6，则执行 `t.Fatal` 函数并输出错误信息。
10. 如果初始化 `IpfsFetcher` 函数失败，则执行 `t.Fatal` 函数并输出错误信息。
11. 如果通过调用 `IpfsFetcher` 函数成功获取到所有预期数据，则不执行任何操作。


```go
func init() {
	err := setupPlugins()
	if err != nil {
		panic(err)
	}
}

func TestIpfsFetcher(t *testing.T) {
	skipUnlessEpic(t)

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	fetcher := NewIpfsFetcher("", 0, nil, "")
	defer fetcher.Close()

	out, err := fetcher.Fetch(ctx, "go-ipfs/versions")
	if err != nil {
		t.Fatal(err)
	}

	var lines []string
	scan := bufio.NewScanner(bytes.NewReader(out))
	for scan.Scan() {
		lines = append(lines, scan.Text())
	}
	err = scan.Err()
	if err != nil {
		t.Fatal("could not read versions:", err)
	}

	if len(lines) < 6 {
		t.Fatal("do not get all expected data")
	}
	if lines[0] != "v0.3.2" {
		t.Fatal("expected v1.0.0 as first line, got", lines[0])
	}

	// Check not found
	if _, err = fetcher.Fetch(ctx, "/no_such_file"); err == nil {
		t.Fatal("expected error 404")
	}
}

```

这段代码是一个名为 "TestInitIpfsFetcher" 的函数测试，主要作用是测试 IpfsFetcher 类型的功能性。以下是该函数的步骤和逻辑：

1. 创建一个名为 "func TestInitIpfsFetcher(t *testing.T)" 的函数内部，使用 "ctx" 和 "cancel" 上下文，确保函数失败时取消试验。

2. 创建一个 "f" 变量，使用 "NewIpfsFetcher" 函数初始化一个空的 Ipfs 节点，并将其存储在 "ipfsTmpDir" 和 "openErr" 变量中。

3. 使用 "initTempNode" 函数初始化一个空文件的 temp 节点，并将其存储在 "ipfsTmpDir" 和 "openErr" 变量中。

4. 使用 "startTempNode" 函数启动 Ipfs 节点，并将其存储在 "openErr" 变量中。

5. 定义一个名为 "stopFunc" 的函数，作为 "f.ipfsStopFunc" 方法的备份，在 "startTempNode" 函数调用时执行该函数，将其 "stopFuncCalled" 设置为 true，并在 "f.ipfsStopFunc" 函数中执行停止 Ipfs 节点的操作。

6. 使用 "AddrInfo" 函数获取 Ipfs 节点的地址信息，并将其存储在 "addrInfo" 变量中。

7. 如果 "addrInfo.ID" 变量为空，则执行 "t.Error" 函数并输出一条错误消息。

8. 如果 "addrInfo.Addrs" 变量为空，则执行 "t.Error" 函数并输出一条错误消息。

9. 调用 "f.ipfsClose" 函数关闭 Ipfs 节点，如果关闭失败，则执行 "t.Error" 函数并输出一条错误消息。

10. 如果 "stopFunc" 函数已被调用，但 "f.ipfsStopFunc" 还未被调用，则执行 "t.Error" 函数并输出一条错误消息。

11. 调用 "f.ipfsClose" 函数关闭 Ipfs 节点，如果关闭失败，则执行 "t.Error" 函数并输出一条错误消息。

12. 关闭 "f.ipfsStopFunc" 函数，确保所有关闭操作都已停止。


```go
func TestInitIpfsFetcher(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	f := NewIpfsFetcher("", 0, nil, "")
	defer f.Close()

	// Init ipfs repo
	f.ipfsTmpDir, f.openErr = initTempNode(ctx, nil, nil)
	if f.openErr != nil {
		t.Fatalf("failed to initialize ipfs node: %s", f.openErr)
	}

	// Start ipfs node
	f.openErr = f.startTempNode(ctx)
	if f.openErr != nil {
		t.Errorf("failed to start ipfs node: %s", f.openErr)
		return
	}

	var stopFuncCalled bool
	stopFunc := f.ipfsStopFunc
	f.ipfsStopFunc = func() {
		stopFuncCalled = true
		stopFunc()
	}

	addrInfo := f.AddrInfo()
	if string(addrInfo.ID) == "" {
		t.Error("AddInfo ID not set")
	}
	if len(addrInfo.Addrs) == 0 {
		t.Error("AddInfo Addrs not set")
	}
	t.Log("Temp node listening on:", addrInfo.Addrs)

	err := f.Close()
	if err != nil {
		t.Fatalf("failed to close fetcher: %s", err)
	}

	if stopFunc != nil && !stopFuncCalled {
		t.Error("Close did not call stop function")
	}

	err = f.Close()
	if err != nil {
		t.Fatalf("failed to close fetcher 2nd time: %s", err)
	}
}

```

这道题目是关于一个名为 `TestReadIpfsConfig` 的测试函数。函数的作用是测试 IPFS（Inter-Platform File System）的配置文件是否正确。

具体来说，这个测试函数包含以下内容：

1. 定义了一个名为 `testConfig` 的字符串变量，它描述了 IPFS 配置文件的样例。这个样例字符串中包含了以下配置项：

  - `Bootstrap`：描述了 IPFS 使用的 DNS 地址和端口。
  - `Migration`：描述了 IPFS 的迁移策略，包括从本地文件系统到 IPFS 和从 IPFS 到本地文件系统的过程。
  - `Peering`：描述了 IPFS 之间的对等连接。

2. 在函数体内，调用了一个名为 `func TestReadIpfsConfig` 的函数，这个函数内部包含以下内容：

  - `t.Run`：用于运行测试函数。
  - `testConfig`：用于读取 IPFS 配置文件的内容，并打印出来以进行核对。
  - `os.WriteFile`：用于读取 IPFS 配置文件的内容，并将其打印出来。
  - `testing.T`：用于标记测试函数的测试类型。
  - `fmt.Printf`：用于打印输出。

通过这个测试函数，可以确保 IPFS 配置文件的正确性。


```go
func TestReadIpfsConfig(t *testing.T) {
	testConfig := `
{
	"Bootstrap": [
		"/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
		"/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
	],
	"Migration": {
		"DownloadSources": ["IPFS", "HTTP", "127.0.0.1", "https://127.0.1.1"],
		"Keep": "cache"
	},
	"Peering": {
		"Peers": [
			{
				"ID": "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5",
				"Addrs": ["/ip4/127.0.0.1/tcp/4001", "/ip4/127.0.0.1/udp/4001/quic"]
			}
		]
	}
}
```

这段代码的作用是读取两个IPFS配置文件，第一个文件是一个已经定义好的"no_such_dir"配置文件，第二个文件是使用上面读取的"tmp_dir"配置文件中的IPFS配置文件。

首先，代码会读取"no_such_dir"文件中的IPFS配置文件，如果读取成功，会将读取结果存储在"bootstrap"变量中，同时存储在"peers"变量中。

如果上面尝试读取"no_such_dir"文件失败，则会将"bootstrap"变量设置为零，并将"peers"变量设置为零。

接下来，代码会读取"tmp_dir"文件中的IPFS配置文件，如果读取成功，会将读取结果存储在"bootstrap"变量中，同时存储在"peers"变量中。

如果上面尝试读取"tmp_dir"文件失败，则会将"bootstrap"变量设置为零，并将"peers"变量设置为零。

接下来，代码会将上面读取到的两个IPFS配置文件的内容再读取一遍，并检查其中的bootstrap地址和peers数量是否符合预期。如果两个文件读取失败，则会执行容错处理。


```go
`

	noSuchDir := "no_such_dir-5953aa51-1145-4efd-afd1-a069075fcf76"
	bootstrap, peers := readIpfsConfig(&noSuchDir, "")
	if bootstrap != nil {
		t.Error("expected nil bootstrap")
	}
	if peers != nil {
		t.Error("expected nil peers")
	}

	tmpDir := makeConfig(t, testConfig)

	bootstrap, peers = readIpfsConfig(nil, "")
	if bootstrap != nil || peers != nil {
		t.Fatal("expected nil ipfs config items")
	}

	bootstrap, peers = readIpfsConfig(&tmpDir, "")
	if len(bootstrap) != 2 {
		t.Fatal("wrong number of bootstrap addresses")
	}
	if bootstrap[0] != "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt" {
		t.Fatal("wrong bootstrap address")
	}

	if len(peers) != 1 {
		t.Fatal("wrong number of peers")
	}

	peer := peers[0]
	if peer.ID.String() != "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5" {
		t.Errorf("wrong ID for first peer")
	}
	if len(peer.Addrs) != 2 {
		t.Error("wrong number of addrs for first peer")
	}
}

```

该测试代码的作用是测试一个名为 "BadBootstrappingIpfsConfig" 的函数，它属于 "unscaled-测试" 包。具体来说，它会在 IPFS 集群上进行一个bootstrap过程，并验证以下三个设置的值是否正确：

1. "Migration": 设置中的 "DownloadSources" 字段指定 IPFS 集群下载的内容来源，应该包括 HTTP 和 127.0.0.1。
2. "Peering": 设置中的 "Peers" 字段指定 IPFS 集群的对等连接关系，应该包括一个连接到本地 127.0.0.1 的对等客户端，以及一个连接到外部网络的客户端。
3. "Bootstrap": 设置中的 "Bootstrap" 字段指定 IPFS 集群的 bootstrap 过程，它的值为 "unreadable"，即不使用任何现有的配置，然后下载 sources、Keep 和 Migration 设置中定义的文件，最后开始 bootstrap 过程。


```go
func TestBadBootstrappingIpfsConfig(t *testing.T) {
	const configBadBootstrap = `
{
	"Bootstrap": "unreadable",
	"Migration": {
		"DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
		"Keep": "cache"
	},
	"Peering": {
		"Peers": [
			{
				"ID": "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5",
				"Addrs": ["/ip4/127.0.0.1/tcp/4001", "/ip4/127.0.0.1/udp/4001/quic"]
			}
		]
	}
}
```

这段代码的作用是读取一个名为 "ipfsConfig" 的配置文件，其中包含了IPFS(InterPlanetary File System)的配置信息。它将该文件所在的目录 "tmpDir" 创建出来，然后读取并打印出在该目录下发现的IPFS配置信息和连接的节点数量。

如果读取配置文件时出现错误，或者读取的节点数量不正确，那么程序会输出错误信息并退出。

具体来说，代码首先通过调用 `makeConfig` 函数创建一个名为 "tmpDir" 的目录，并且在函数中传入两个参数 `t` 和 `configBadBootstrap`，作为 `MakeConfig` 函数的第一个和第二个参数，分别表示测试时间和一个被认为是 "坏" 的预设网络配置。

接着，代码通过调用 `readIpfsConfig` 函数读取 "ipfsConfig" 文件中的配置信息，并将返回值存储在 `bootstrap` 和 `peers` 变量中。如果读取成功，那么程序会判断 `bootstrap` 是否为 `nil`，如果是，就输出一个错误消息并退出。

接下来，程序会检查 `peers` 数组中包含的节点数量是否为1，如果是，就程序会判断 `peers[0].Addrs` 数组中的元素数量是否为2，如果不是，就输出一个错误消息并退出。

最后，程序会使用 `os.RemoveAll` 函数删除 "tmpDir" 目录中的所有文件和子目录，并退出程序。


```go
`

	tmpDir := makeConfig(t, configBadBootstrap)

	bootstrap, peers := readIpfsConfig(&tmpDir, "")
	if bootstrap != nil {
		t.Fatal("expected nil bootstrap")
	}
	if len(peers) != 1 {
		t.Fatal("wrong number of peers")
	}
	if len(peers[0].Addrs) != 2 {
		t.Error("wrong number of addrs for first peer")
	}
	os.RemoveAll(tmpDir)
}

```

该测试函数的作用是测试 Ipfs 配置文件中的 bad peers 配置是否正确。

具体来说，该函数会创建一个名为 `configBadPeers` 的 Config 对象，其中包含以下内容：


{
 "Bootstrap": [
   "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
   "/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
 ],
 "Migration": {
   "DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
   "Keep": "cache"
 },
 "Peering": "Unreadable-data"
}


然后，该函数会将该配置文件存储到当前工作目录下的 `tmpDir` 目录中，并读取其中的 `配置文件内容`。

接下来，该函数会尝试使用 `readIpfsConfig` 函数从 `tmpDir` 目录中读取 `Ipfs` 配置文件的内容，如果读取成功，则将结果存储在 `bootstrap` 和 `peers` 变量中。

最后，该函数会检查 `bootstrap` 数组是否包含两个元素，以及 `bootstrap` 中的第一个元素是否等于 `"/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt"`。如果所有条件都满足，则表示配置文件中的 bad peers 配置是正确的，否则会输出错误信息并停止执行。


```go
func TestBadPeersIpfsConfig(t *testing.T) {
	const configBadPeers = `
{
	"Bootstrap": [
		"/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
		"/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
	],
	"Migration": {
		"DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
		"Keep": "cache"
	},
	"Peering": "Unreadable-data"
}
`

	tmpDir := makeConfig(t, configBadPeers)

	bootstrap, peers := readIpfsConfig(&tmpDir, "")
	if peers != nil {
		t.Fatal("expected nil peers")
	}
	if len(bootstrap) != 2 {
		t.Fatal("wrong number of bootstrap addresses")
	}
	if bootstrap[0] != "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt" {
		t.Fatal("wrong bootstrap address")
	}
}

```

这段代码定义了一个名为 `makeConfig` 的函数，接受两个参数：`t` 和 `configData` 都是 testing 包中的测试函数断言类型。函数的作用是在 temporary directory（临时目录）中创建一个名为 `config.yaml` 的配置文件，并将 `configData` 中的内容写入该文件中。

函数的具体实现可以分为以下几个步骤：

1. 创建一个名为 `tmpDir` 的临时目录，如果失败则输出错误并退出。
2. 创建一个名为 `config` 的文件，并尝试将其写入 `configData` 中的内容。如果写入失败，则输出错误并退出。
3. 关闭文件。
4. 返回 temporary directory。

通过这段代码，可以实现创建一个临时目录，并将一个或多个测试函数的配置数据存储到其中的某个文件中。


```go
func makeConfig(t *testing.T, configData string) string {
	tmpDir := t.TempDir()

	cfgFile, err := os.Create(filepath.Join(tmpDir, "config"))
	if err != nil {
		t.Fatal(err)
	}
	if _, err = cfgFile.Write([]byte(configData)); err != nil {
		t.Fatal(err)
	}
	if err = cfgFile.Close(); err != nil {
		t.Fatal(err)
	}
	return tmpDir
}

```

这两段代码定义了一个名为skipUnlessEpic的函数和名为setupPlugins的函数。

func skipUnlessEpic(t *testing.T) {
	if os.Getenv("IPFS_EPIC_TEST") == "" {
		t.SkipNow()
	}
}skipUnlessEpic函数是用来在测试中跳过测试用例的。在内置的 skipUnlessEpic 函数中，首先检查是否有在任何测试用例中设置的环境变量中存在 IPFS_EPIC_TEST，如果有，则直接跳过测试用例。否则，不会跳过任何测试用例。

func setupPlugins() error {
	defaultPath, err := migrations.IpfsDir("")
	if err != nil {
		return err
	}

	// Load plugins. This will skip the repo if not available.
	plugins, err := loader.NewPluginLoader(filepath.Join(defaultPath, "plugins"))
	if err != nil {
		return fmt.Errorf("error loading plugins: %w", err)
	}

	if err := plugins.Initialize(); err != nil {
		// Need to ignore errors here because plugins may already be loaded when
		// run from ipfs daemon.
		return fmt.Errorf("error initializing plugins: %w", err)
	}

	if err := plugins.Inject(); err != nil {
		// Need to ignore errors here because plugins may already be loaded when
		// run from ipfs daemon.
		return fmt.Errorf("error injecting plugins: %w", err)
	}

	return nil
}setupPlugins函数是用来设置IPFS插件加载器的函数。这个函数的作用是在 IPFS 插件加载器初始化之后，如果设置的目录不存在，则返回一个错误。

上述函数的主要作用是加载 IPFS 插件并初始化，如果初始化过程中出现错误，则返回一个错误，并跳过当前测试用例。


```go
func skipUnlessEpic(t *testing.T) {
	if os.Getenv("IPFS_EPIC_TEST") == "" {
		t.SkipNow()
	}
}

func setupPlugins() error {
	defaultPath, err := migrations.IpfsDir("")
	if err != nil {
		return err
	}

	// Load plugins. This will skip the repo if not available.
	plugins, err := loader.NewPluginLoader(filepath.Join(defaultPath, "plugins"))
	if err != nil {
		return fmt.Errorf("error loading plugins: %w", err)
	}

	if err := plugins.Initialize(); err != nil {
		// Need to ignore errors here because plugins may already be loaded when
		// run from ipfs daemon.
		return fmt.Errorf("error initializing plugins: %w", err)
	}

	if err := plugins.Inject(); err != nil {
		// Need to ignore errors here because plugins may already be loaded when
		// run from ipfs daemon.
		return fmt.Errorf("error injecting plugins: %w", err)
	}

	return nil
}

```

# `routing/composer.go`

这段代码定义了一个名为 "routing" 的 package，它包含了两个相关的类：`Composer` 和 `Router`. 

`Composer` 类是一个多做者，它通过组合多个路由器实例来提供路由功能。 `Router` 类实现了 `routing.Router` 接口，用于管理路由器实例的创建和注册。 

通过 `import` 语句引入的外部依赖包括： 

* `context` 类型，用于处理异步操作的结果。 
* `errors` 类型，用于处理错误信息。 
* `github.com/hashicorp/go-multierror`，用于处理由于多个错误而导致的上堆栈信息。 
* `github.com/ipfs/go-cid`，用于跨域携带数据。 
* `routinghelpers`，定义了多种路由器助手函数。 
* `github.com/libp2p/go-libp2p-routing-helpers`，定义了多种路由器助手函数。 
* `github.com/multiformats/go-multihash`，定义了多种哈希函数。 

另外，该代码还定义了一个名为 `Composer` 的内部类，它实现了 `Composable` 的接口，用于组合多个路由器实例。


```go
package routing

import (
	"context"
	"errors"

	"github.com/hashicorp/go-multierror"
	"github.com/ipfs/go-cid"
	routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/routing"
	"github.com/multiformats/go-multihash"
)

var (
	_ routinghelpers.ProvideManyRouter = &Composer{}
	_ routing.Routing                  = &Composer{}
)

```

该代码定义了一个名为 Composer 的结构体，它包含以下五个路由：

1. GetValueRouter：该路由用于获取由 Composer 提供的服务器的值。
2. PutValueRouter：该路由用于将由 Composer 提供的服务的值设置为给定 ID 的值。
3. FindPeersRouter：该路由用于查找与给定 ID 类似的服务器。
4. FindProvidersRouter：该路由用于查找与给定 ID 类似的服务器提供者。
5. ProvideRouter：该路由用于设置给定 ID 的服务器的值。

该结构体还包含一个名为 Provide 的方法，用于提供服务器的值。在 Provide 方法中，首先调用 Composer 中的 ProvideRouter 中的 Provide 方法，然后根据需要将值设置为给定 ID 的值，最后返回是否成功调用 Provide 方法。


```go
type Composer struct {
	GetValueRouter      routing.Routing
	PutValueRouter      routing.Routing
	FindPeersRouter     routing.Routing
	FindProvidersRouter routing.Routing
	ProvideRouter       routing.Routing
}

func (c *Composer) Provide(ctx context.Context, cid cid.Cid, provide bool) error {
	log.Debug("composer: calling provide: ", cid)
	err := c.ProvideRouter.Provide(ctx, cid, provide)
	if err != nil {
		log.Debug("composer: calling provide: ", cid, " error: ", err)
	}

	return err
}

```

这段代码定义了一个名为 `ProvideMany` 的函数，接收一个名为 `Composer` 的指针和一个名为 `keys` 的切片（多哈希表）。

函数的作用是调用一个名为 `ProvideRouter` 的路由器（routing helper）实例 `c`，并传递一个名为 `keys` 的切片，该路由器使用 `ProvideManyRouter` 类型，该类型实现了 `routinghelpers.ProvideManyRouter` 接口。

如果 `c.ProvideManyRouter` 实现了一个实现了 `routinghelpers.ProvideManyRouter` 接口的路由器，那么函数将成功调用该路由器，并返回，否则将返回一个错误。

函数的副作用是输出一条日志消息，指出调用 `ProvideMany` 函数时出现的问题，然后返回一个错误。


```go
func (c *Composer) ProvideMany(ctx context.Context, keys []multihash.Multihash) error {
	log.Debug("composer: calling provide many: ", len(keys))
	pmr, ok := c.ProvideRouter.(routinghelpers.ProvideManyRouter)
	if !ok {
		log.Debug("composer: provide many is not implemented on the actual router")
		return nil
	}

	err := pmr.ProvideMany(ctx, keys)
	if err != nil {
		log.Debug("composer: calling provide many error: ", err)
	}

	return err
}

```

该代码定义了两个函数：`Ready()` 和 `FindProvidersAsync()`。这两个函数的功能如下：

1. `Ready()` 函数的作用是检查 `c` 指向的 `Composer` 对象是否处于准备就绪状态，如果 `c` 指向的 `Composer` 对象提供了一个 `ReadyAbleRouter` 类型的实例，则调用该实例的 `Ready()` 方法，并将结果存储在 `pmr` 和 `ready` 变量中，最后返回 `ready` 的值。如果 `c` 指向的 `Composer` 对象没有提供 `ReadyAbleRouter` 类型的实例，则直接返回 `true`。

2. `FindProvidersAsync()` 函数的作用是调用 `c` 指向的 `Composer` 对象的 `FindProvidersRouter.FindProvidersAsync()` 方法，该方法将在 `ctx` 上下文中异步执行，并返回一个 `<peer.AddrInfo>` 类型的通道，用于通知 `Composer` 对象有新提供的提供商。在调用该函数时，将 `cid` 和 `count` 参数传递给 `FindProvidersAsync()` 函数，并将结果存储在 `channel` 变量中，最后返回 `channel` 的通道。


```go
func (c *Composer) Ready() bool {
	log.Debug("composer: calling ready")
	pmr, ok := c.ProvideRouter.(routinghelpers.ReadyAbleRouter)
	if !ok {
		return true
	}

	ready := pmr.Ready()

	log.Debug("composer: calling ready result: ", ready)

	return ready
}

func (c *Composer) FindProvidersAsync(ctx context.Context, cid cid.Cid, count int) <-chan peer.AddrInfo {
	log.Debug("composer: calling findProvidersAsync: ", cid)
	return c.FindProvidersRouter.FindProvidersAsync(ctx, cid, count)
}

```

该代码定义了两个函数，一个是 `func (c *Composer) FindPeer(ctx context.Context, pid peer.ID) (peer.AddrInfo, error)`，另一个是 `func (c *Composer) PutValue(ctx context.Context, key string, val []byte, opts ...routing.Option) error`。

这两个函数的功能如下：

1. `func (c *Composer) FindPeer(ctx context.Context, pid peer.ID) (peer.AddrInfo, error)` 函数接收一个 `peer.ID` 类型的参数 `pid`，它是要查找的邻居的 ID。这个函数返回一个 `peer.AddrInfo` 类型的结果，或者是错误的。

2. `func (c *Composer) PutValue(ctx context.Context, key string, val []byte, opts ...routing.Option) error` 函数接收一个 `string` 类型的参数 `key` 和一个 `[]byte` 类型的参数 `val`。它还接收一个 `[]routing.Option` 类型的参数 `opts`。这个函数返回一个 `error` 类型的结果。

函数调用关系：

从 `func (c *Composer) PutValue(ctx context.Context, key string, val []byte, opts ...routing.Option) error` 函数中调用 `func (c *Composer) FindPeer(ctx context.Context, pid peer.ID) (peer.AddrInfo, error)` 函数，如果 `func (c *Composer) FindPeer(ctx context.Context, pid peer.ID)` 函数返回一个非 `nil` 的结果，则将其作为 `key` 参数传递给 `func (c *Composer) PutValue(ctx context.Context, key string, val []byte, opts ...routing.Option)` 函数，否则返回 `nil`。


```go
func (c *Composer) FindPeer(ctx context.Context, pid peer.ID) (peer.AddrInfo, error) {
	log.Debug("composer: calling findPeer: ", pid)
	addr, err := c.FindPeersRouter.FindPeer(ctx, pid)
	if err != nil {
		log.Debug("composer: calling findPeer error: ", pid, addr.String(), err)
	}
	return addr, err
}

func (c *Composer) PutValue(ctx context.Context, key string, val []byte, opts ...routing.Option) error {
	log.Debug("composer: calling putValue: ", key, len(val))
	err := c.PutValueRouter.PutValue(ctx, key, val, opts...)
	if err != nil {
		log.Debug("composer: calling putValue error: ", key, len(val), err)
	}

	return err
}

```

这两函数接收一个 Composer 类型的参数 c，并接受一个键名 string 和一个或多个选项 opts，然后执行不同的功能。

func (c *Composer) GetValue(ctx context.Context, key string, opts ...routing.Option) ([]byte, error) {
	log.Debug("composer: calling getValue: ", key)
	val, err := c.GetValueRouter.GetValue(ctx, key, opts...)
	if err != nil {
		log.Debug("composer: calling getValue error: ", key, len(val), err)
	}
	return val, err
}

func (c *Composer) SearchValue(ctx context.Context, key string, opts ...routing.Option) (<-chan []byte, error) {
	log.Debug("composer: calling searchValue: ", key)
	ch, err := c.GetValueRouter.SearchValue(ctx, key, opts...)

	// avoid nil channels on implementations not supporting SearchValue method.
	if errors.Is(err, routing.ErrNotFound) && ch == nil {
		out := make(chan []byte)
		close(out)
		return out, err
	}

	if err != nil {
		log.Debug("composer: calling searchValue error: ", key, err)
	}

	return ch, err
}


```go
func (c *Composer) GetValue(ctx context.Context, key string, opts ...routing.Option) ([]byte, error) {
	log.Debug("composer: calling getValue: ", key)
	val, err := c.GetValueRouter.GetValue(ctx, key, opts...)
	if err != nil {
		log.Debug("composer: calling getValue error: ", key, len(val), err)
	}

	return val, err
}

func (c *Composer) SearchValue(ctx context.Context, key string, opts ...routing.Option) (<-chan []byte, error) {
	log.Debug("composer: calling searchValue: ", key)
	ch, err := c.GetValueRouter.SearchValue(ctx, key, opts...)

	// avoid nil channels on implementations not supporting SearchValue method.
	if errors.Is(err, routing.ErrNotFound) && ch == nil {
		out := make(chan []byte)
		close(out)
		return out, err
	}

	if err != nil {
		log.Debug("composer: calling searchValue error: ", key, err)
	}

	return ch, err
}

```

该函数名为 `Bootstrap`，它接收一个名为 `c` 的 `Composer` 对象，并使用 `ctx` 上下文作为参数。函数的主要作用是调用 `c` 对象的 `FindPeersRouter`、`FindProvidersRouter` 和 `GetValueRouter` 方法，并使用它们来执行 `Bootstrap` 操作。

具体来说，该函数首先调用 `c.FindPeersRouter.Bootstrap(ctx)` 方法，该方法将根据设定的路由查找与作曲家相互连接的节点。接着，函数又调用 `c.FindProvidersRouter.Bootstrap(ctx)` 方法，同样根据设定的路由查找与作曲家提供者连接的节点。

然后，函数又调用 `c.GetValueRouter.Bootstrap(ctx)` 方法，根据设定的路由查找与作品相关联的值。接下来，函数调用 `c.ProvideRouter.Bootstrap(ctx)` 方法，根据设定的路由执行与作品关联的操作。

最后，函数使用 `multierror.Append` 函数将上述五个 `Bootstrap` 操作的结果进行合并，并将错误信息添加到结果中。如果执行过程中出现错误，函数将输出错误信息并返回。


```go
func (c *Composer) Bootstrap(ctx context.Context) error {
	log.Debug("composer: calling bootstrap")
	errfp := c.FindPeersRouter.Bootstrap(ctx)
	errfps := c.FindProvidersRouter.Bootstrap(ctx)
	errgv := c.GetValueRouter.Bootstrap(ctx)
	errpv := c.PutValueRouter.Bootstrap(ctx)
	errp := c.ProvideRouter.Bootstrap(ctx)
	err := multierror.Append(errfp, errfps, errgv, errpv, errp)
	if err != nil {
		log.Debug("composer: calling bootstrap error: ", err)
	}
	return err
}

```

# `routing/delegated.go`

这段代码是一个 Go 语言的包，名为 "routing"，它实现了从 IPFS 存储桶中获取数据并路由到本地客户端。它主要通过以下几个步骤实现：

1. 通过 `net/http` 包创建一个 HTTP 客户端，用于与 IPFS 存储桶的交互。
2. 通过 `contentrouter` 包中的 `ContentRouter` 类型，设置路由策略，将获取到的数据路由到具体的 backend 服务。
3. 通过 `libp2p/go-libp2p-kad-dht` 包与 IPFS 存储桶的键值对存储体进行交互，以实现对数据的键值对查询。
4. 通过 `libp2p/go-libp2p-kad-dht/dual` 包，设置dual身份到本地，以在本地选择一个IPFS节点。
5. 通过 `libp2p/go-libp2p-kad-dht/fullrt` 包，设置留空名字为 "fullrt"，以 enable 并行高度并行。
6. 通过 `libp2p/go-libp2p-kad-dht/record` 包，设置一个名为 "record" 的标志，以记录IPFS的元数据。
7. 通过 `libp2p/go-libp2p-routing-helpers` 包中的 `加入路由前缀` 函数，将获取到的数据添加到路由前缀中。
8. 通过 `routinghelpers` 包中的 `应用路由` 函数，将路由前缀应用到路由上，从而实现数据的路由。
9. 最后，将设置好的路由缓存到本地，以便后续请求时可以重新使用。

综上，这段代码的作用是实现从 IPFS 存储桶中获取数据并路由到本地客户端。


```go
package routing

import (
	"context"
	"encoding/base64"
	"errors"
	"fmt"
	"net/http"

	drclient "github.com/ipfs/boxo/routing/http/client"
	"github.com/ipfs/boxo/routing/http/contentrouter"
	"github.com/ipfs/go-datastore"
	logging "github.com/ipfs/go-log"
	version "github.com/ipfs/kubo"
	"github.com/ipfs/kubo/config"
	dht "github.com/libp2p/go-libp2p-kad-dht"
	"github.com/libp2p/go-libp2p-kad-dht/dual"
	"github.com/libp2p/go-libp2p-kad-dht/fullrt"
	record "github.com/libp2p/go-libp2p-record"
	routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"
	ic "github.com/libp2p/go-libp2p/core/crypto"
	host "github.com/libp2p/go-libp2p/core/host"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/routing"
	ma "github.com/multiformats/go-multiaddr"
	"go.opencensus.io/stats/view"
)

```

这段代码定义了一个名为 "routing/delegated" 的日志输出器，用于输出关于路由和守护路由器（也称为代理路由器）的 Delegated 方法的结果。

具体来说，这段代码的作用是创建一个代表所有路由器的方法对象 "finalRouter"，该对象包含所有已知的路由器，并且使用 "Parse" 函数将每个路由器的路由信息注入到 "finalRouter" 对象中。

在 "Parse" 函数中，我们首先检查 "Parse" 函数的参数是否正确，然后使用 "methods" 字段中提供的路由器名称循环遍历所有路由器，并尝试使用 "parse" 函数来创建一个代表该路由器的 "router" 对象。如果 "parse" 函数发生错误，或者尝试创建 "router" 对象的其他部分失败，那么 "Parse" 函数将返回一个空 "nil" 并记录错误。

最后，如果 "Parse" 函数成功创建一个代表所有路由器的 "finalRouter" 对象，那么 "Parse" 函数返回该对象，否则返回一个空对象并记录错误。


```go
var log = logging.Logger("routing/delegated")

func Parse(routers config.Routers, methods config.Methods, extraDHT *ExtraDHTParams, extraHTTP *ExtraHTTPParams) (routing.Routing, error) {
	if err := methods.Check(); err != nil {
		return nil, err
	}

	createdRouters := make(map[string]routing.Routing)
	finalRouter := &Composer{}

	// Create all needed routers from method names
	for mn, m := range methods {
		router, err := parse(make(map[string]bool), createdRouters, m.RouterName, routers, extraDHT, extraHTTP)
		if err != nil {
			return nil, err
		}

		switch mn {
		case config.MethodNamePutIPNS:
			finalRouter.PutValueRouter = router
		case config.MethodNameGetIPNS:
			finalRouter.GetValueRouter = router
		case config.MethodNameFindPeers:
			finalRouter.FindPeersRouter = router
		case config.MethodNameFindProviders:
			finalRouter.FindProvidersRouter = router
		case config.MethodNameProvide:
			finalRouter.ProvideRouter = router
		}

		log.Info("using method ", mn, " with router ", m.RouterName)
	}

	return finalRouter, nil
}

```

This is a Go language function that creates a router and returns it. The router can


```go
func parse(visited map[string]bool,
	createdRouters map[string]routing.Routing,
	routerName string,
	routersCfg config.Routers,
	extraDHT *ExtraDHTParams,
	extraHTTP *ExtraHTTPParams,
) (routing.Routing, error) {
	// check if we already created it
	r, ok := createdRouters[routerName]
	if ok {
		return r, nil
	}

	// check if we are in a dep loop
	if visited[routerName] {
		return nil, fmt.Errorf("dependency loop creating router with name %q", routerName)
	}

	// set node as visited
	visited[routerName] = true

	cfg, ok := routersCfg[routerName]
	if !ok {
		return nil, fmt.Errorf("config for router with name %q not found", routerName)
	}

	var router routing.Routing
	var err error
	switch cfg.Type {
	case config.RouterTypeHTTP:
		router, err = httpRoutingFromConfig(cfg.Router, extraHTTP)
	case config.RouterTypeDHT:
		router, err = dhtRoutingFromConfig(cfg.Router, extraDHT)
	case config.RouterTypeParallel:
		crp := cfg.Parameters.(*config.ComposableRouterParams)
		var pr []*routinghelpers.ParallelRouter
		for _, cr := range crp.Routers {
			ri, err := parse(visited, createdRouters, cr.RouterName, routersCfg, extraDHT, extraHTTP)
			if err != nil {
				return nil, err
			}

			pr = append(pr, &routinghelpers.ParallelRouter{
				Router:                  ri,
				IgnoreError:             cr.IgnoreErrors,
				DoNotWaitForSearchValue: true,
				Timeout:                 cr.Timeout.Duration,
				ExecuteAfter:            cr.ExecuteAfter.WithDefault(0),
			})

		}

		router = routinghelpers.NewComposableParallel(pr)
	case config.RouterTypeSequential:
		crp := cfg.Parameters.(*config.ComposableRouterParams)
		var sr []*routinghelpers.SequentialRouter
		for _, cr := range crp.Routers {
			ri, err := parse(visited, createdRouters, cr.RouterName, routersCfg, extraDHT, extraHTTP)
			if err != nil {
				return nil, err
			}

			sr = append(sr, &routinghelpers.SequentialRouter{
				Router:      ri,
				IgnoreError: cr.IgnoreErrors,
				Timeout:     cr.Timeout.Duration,
			})

		}

		router = routinghelpers.NewComposableSequential(sr)
	default:
		return nil, fmt.Errorf("unknown router type %q", cfg.Type)
	}

	if err != nil {
		return nil, err
	}

	createdRouters[routerName] = router

	log.Info("created router ", routerName, " with params ", cfg.Parameters)

	return router, nil
}

```

这段代码定义了一个名为ExtraHTTPParams的结构体，该结构体包含一个字符串类型的成员PeerID，一个字符串类型的成员Addrs，一个字节数组类型的成员PrivKeyB64，以及一个空字符串类型的成员None。

接着，该代码实现了一个名为httpRoutingFromConfig的函数，该函数接收一个endpoint参数，一个peerID参数，以及一个字符串类型的成员Addrs和一个字符串类型的成员PrivKeyB64。函数首先从配置文件中获取一个HTTP路由的配置信息，然后使用这些信息构建一个HTTP路由实例，并将其返回。

最后，该函数使用httpRoutingFromConfig函数构建一个HTTP路由实例，将构建出的路由实例返回，并且错误处理函数也在该函数中实现。


```go
type ExtraHTTPParams struct {
	PeerID     string
	Addrs      []string
	PrivKeyB64 string
}

func ConstructHTTPRouter(endpoint string, peerID string, addrs []string, privKey string) (routing.Routing, error) {
	return httpRoutingFromConfig(
		config.Router{
			Type: "http",
			Parameters: &config.HTTPRouterParams{
				Endpoint: endpoint,
			},
		},
		&ExtraHTTPParams{
			PeerID:     peerID,
			Addrs:      addrs,
			PrivKeyB64: privKey,
		},
	)
}

```

This appears to be a Go struct that implements the `httpRoutingWrapper` type, which wraps an HTTP client and承袭了 HTTP handlers like `http.Handler`. It appears to handle the configuration of an HTTP provider (e.g. AWS Lambda) by creating an HTTP client, setting up the connection pool for HTTP requests, and registering the routes and endpoints that should be scoped to this provider.

It appears to be using the `drclient` package to handle the HTTP client, which is a依赖 that is part of the Driller API for testing HTTP and HTTPS servers. This package is designed to be used with the `httpd` or `httpsd` daemon as the backend, and provides features like the HTTP/1.1 protocol, streaming I/O, and automatic HTTP/HTTPS request and response handling.

The `httpRoutingWrapper` struct includes fields for the HTTP client, HTTP provider connection pool, and HTTP routes that should be handled by this provider. It also includes a method `TransportClone` which appears to duplicate the functionality of `http.DefaultTransport.Clone()` and append any additional settings to the existing transport object.

It appears that the `createAddrInfo` function is used to set up the IP address and port information for the HTTP provider, and this information is passed to the `drclient` constructor.


```go
func httpRoutingFromConfig(conf config.Router, extraHTTP *ExtraHTTPParams) (routing.Routing, error) {
	params := conf.Parameters.(*config.HTTPRouterParams)
	if params.Endpoint == "" {
		return nil, NewParamNeededErr("Endpoint", conf.Type)
	}

	params.FillDefaults()

	// Increase per-host connection pool since we are making lots of concurrent requests.
	transport := http.DefaultTransport.(*http.Transport).Clone()
	transport.MaxIdleConns = 500
	transport.MaxIdleConnsPerHost = 100

	delegateHTTPClient := &http.Client{
		Transport: &drclient.ResponseBodyLimitedTransport{
			RoundTripper: transport,
			LimitBytes:   1 << 20,
		},
	}

	key, err := decodePrivKey(extraHTTP.PrivKeyB64)
	if err != nil {
		return nil, err
	}

	addrInfo, err := createAddrInfo(extraHTTP.PeerID, extraHTTP.Addrs)
	if err != nil {
		return nil, err
	}

	cli, err := drclient.New(
		params.Endpoint,
		drclient.WithHTTPClient(delegateHTTPClient),
		drclient.WithIdentity(key),
		drclient.WithProviderInfo(addrInfo.ID, addrInfo.Addrs),
		drclient.WithUserAgent(version.GetUserAgentVersion()),
	)
	if err != nil {
		return nil, err
	}

	cr := contentrouter.NewContentRoutingClient(
		cli,
		contentrouter.WithMaxProvideBatchSize(params.MaxProvideBatchSize),
		contentrouter.WithMaxProvideConcurrency(params.MaxProvideConcurrency),
	)

	err = view.Register(drclient.OpenCensusViews...)
	if err != nil {
		return nil, fmt.Errorf("registering HTTP delegated routing views: %w", err)
	}

	return &httpRoutingWrapper{
		ContentRouting:    cr,
		PeerRouting:       cr,
		ValueStore:        cr,
		ProvideManyRouter: cr,
	}, nil
}

```

这两段代码都实现了一些关于 `ic` 包的函数，具体解释如下：

1. `decodePrivKey` 函数的作用是解码一个字节字符串（keyB64）为私钥（私钥是一个 `ic.PrivKey` 类型）。首先，它使用 `base64.StdEncoding` 的 `DecodeString` 函数将 keyB64 解码为具体的字节切片（可以用于将字符串转换为字节）。然后，如果解码过程中出现错误，它将返回一个非空错误对象（error）。最后，它使用 `ic.UnmarshalPrivateKey` 的函数从字节切片中还原出私钥。

2. `createAddrInfo` 函数的作用是在一个 `peerID` 和多个 `addrs` 的基础上生成一个 `peer.AddrInfo` 对象。首先，它使用 `peer.Decode` 函数将 `peerID` 解码为具体的 `peer.ID` 类型。然后，它创建一个 `ma.Multiaddr` 切片，将 `addrs` 中的每个地址添加到 `mas` 中。接下来，它使用 `peer.AddrInfo` 的函数创建一个 `peer.AddrInfo` 对象，包含 `peerID` 和 `addrs`。最后，如果没有错误，它返回该 `peer.AddrInfo` 对象。


```go
func decodePrivKey(keyB64 string) (ic.PrivKey, error) {
	pk, err := base64.StdEncoding.DecodeString(keyB64)
	if err != nil {
		return nil, err
	}

	return ic.UnmarshalPrivateKey(pk)
}

func createAddrInfo(peerID string, addrs []string) (peer.AddrInfo, error) {
	pID, err := peer.Decode(peerID)
	if err != nil {
		return peer.AddrInfo{}, err
	}

	var mas []ma.Multiaddr
	for _, a := range addrs {
		m, err := ma.NewMultiaddr(a)
		if err != nil {
			return peer.AddrInfo{}, err
		}

		mas = append(mas, m)
	}

	return peer.AddrInfo{
		ID:    pID,
		Addrs: mas,
	}, nil
}

```

这段代码定义了一个名为ExtraDHTParams的结构体类型，它代表了DHT路由器所需的参数。

函数dhtRoutingFromConfig接收一个DHT路由器和DHT参数配置的参数。首先，它检查参数是否指向一个DHT路由器参数配置的实例，如果不是，就返回一个 nil 值并错误地布署。

如果参数是一个DHT路由器参数配置的实例，那么该函数将根据路由器设置的加速DHT客户端模式确定DHT模式。然后，它将创建一个适当的DHT路由器实例，并将其返回。

函数内部使用了以下类型的变量：

- ExtraDHTParams:DHT路由器所需的参数配置实例
- config.Router:DHT路由器参数配置的实例
- ExtraDHTParams：上面提到的DHT路由器参数配置实例
- dht.Mode:DHT模式的枚举类型
- dht.ModeOpt:DHT模式的枚举类型选项
- host.Host：用于保存主机名称的变量
- peer.AddrInfo：用于保存对等机端口号的参数实例


```go
type ExtraDHTParams struct {
	BootstrapPeers []peer.AddrInfo
	Host           host.Host
	Validator      record.Validator
	Datastore      datastore.Batching
	Context        context.Context
}

func dhtRoutingFromConfig(conf config.Router, extra *ExtraDHTParams) (routing.Routing, error) {
	params, ok := conf.Parameters.(*config.DHTRouterParams)
	if !ok {
		return nil, errors.New("incorrect params for DHT router")
	}

	if params.AcceleratedDHTClient {
		return createFullRT(extra)
	}

	var mode dht.ModeOpt
	switch params.Mode {
	case config.DHTModeAuto:
		mode = dht.ModeAuto
	case config.DHTModeClient:
		mode = dht.ModeClient
	case config.DHTModeServer:
		mode = dht.ModeServer
	default:
		return nil, fmt.Errorf("invalid DHT mode: %q", params.Mode)
	}

	return createDHT(extra, params.PublicIPNetwork, mode)
}

```

该函数创建了一个分布式哈希表(DHT)，其中使用了公开模式(public mode)，并接受了DHT参数和模式选项(ExtraDHTParams)，并返回一个Routing和一个error。

下面是函数的更详细的解释：

1. 函数参数：
  - ExtraDHTParams：一个DHT参数结构体，它包含了DHT的多个选项，包括查询过滤器(QueryFilter)、路由表过滤器(RoutingTableFilter)、RoutingTablePeerDiversityFilter等。
  - public：布尔类型，表示DHT的公共模式选项。

2. 函数实现：
  - 初始化opts数组为DHT的多个选项。
  - 如果public为真，则添加DHT.QueryFilter(dht.PublicQueryFilter)、DHT.RoutingTableFilter(dht.PublicRoutingTableFilter)和DHT.RoutingTablePeerDiversityFilter(dht.NewRTPeerDiversityFilter(params.Host, 2, 3))。
  - 如果public为假，则添加DHT.ProtocolExtension(dual.LanExtension)、DHT.QueryFilter(dht.PrivateQueryFilter)和DHT.RoutingTableFilter(dht.PrivateRoutingTableFilter)。
  - 添加DHT.Concurrency(10)以支持多线程操作。
  - 设置DHT.Mode(mode)以指定DHT的运行模式。
  - 添加DHT.Datastore(params.Datastore)以指定DHT的存储选项。
  - 添加DHT.Validator(params.Validator)以指定DHT的验证选项。
  - 添加DHT.BootstrapPeers(params.BootstrapPeers...)以指定DHT的引导节点。
  - 返回DHT.New(params.Context, params.Host, opts...)。

3. 函数输出：
  - Routing:DHT的公共模式的下一跳路由。
  - error：错误对象(如果创建DHT时出现错误)。


```go
func createDHT(params *ExtraDHTParams, public bool, mode dht.ModeOpt) (routing.Routing, error) {
	var opts []dht.Option

	if public {
		opts = append(opts, dht.QueryFilter(dht.PublicQueryFilter),
			dht.RoutingTableFilter(dht.PublicRoutingTableFilter),
			dht.RoutingTablePeerDiversityFilter(dht.NewRTPeerDiversityFilter(params.Host, 2, 3)))
	} else {
		opts = append(opts, dht.ProtocolExtension(dual.LanExtension),
			dht.QueryFilter(dht.PrivateQueryFilter),
			dht.RoutingTableFilter(dht.PrivateRoutingTableFilter))
	}

	opts = append(opts,
		dht.Concurrency(10),
		dht.Mode(mode),
		dht.Datastore(params.Datastore),
		dht.Validator(params.Validator),
		dht.BootstrapPeers(params.BootstrapPeers...))

	return dht.New(
		params.Context, params.Host, opts...,
	)
}

```

该函数创建一个完整的 RT 实例，并将其返回。它接受一个名为 params 的 *ExtraDHTParams 结构体，其中包含主机、DHT 设置和验证器等设置。

具体来说，函数首先创建一个名为 "fullrt" 的路由器实例，然后设置其 host 属性的值为传入的 host 参数。接下来，函数使用 dht.DefaultPrefix 设置默认的键前缀，该键前缀用于在路由器中查找数据存储。

函数的下一行使用 fullrt.DHTOption() 函数设置路由器的 DHT 选项。该函数接受一个结构体，其中包含以下设置：

 - dht.Validator(params.Validator)：用于验证键是否有效。
 - dht.Datastore(params.Datastore)：用于存储键值对，并将键值对存储在指定的数据存储器中。
 - dht.BootstrapPeers(params.BootstrapPeers...)：用于指定自动发现 bootstrap 节点的 IP 地址。
 - dht.BucketSize(params.BucketSize)：用于设置数据存储桶的大小。

最后，函数调用路由器的 NewFullRT() 函数来创建新的路由器实例，并将其返回。如果设置一切正常，函数将返回一个指向新路由器实例的指针，否则会输出一个错误。


```go
func createFullRT(params *ExtraDHTParams) (routing.Routing, error) {
	return fullrt.NewFullRT(params.Host,
		dht.DefaultPrefix,
		fullrt.DHTOption(
			dht.Validator(params.Validator),
			dht.Datastore(params.Datastore),
			dht.BootstrapPeers(params.BootstrapPeers...),
			dht.BucketSize(20),
		),
	)
}

```

# `routing/delegated_test.go`

This is a Go program that configures a network router. The router supports the HTTP and sequence router types.

The `Routers` configuration section defines the router parameters. For example, the `r1` router has an endpoint at `testEndpoint`.

The `Methods` configuration section defines the endpoints that can be invoked by the router. For example, the router can be queried for the IP address of a given index.

The router also supports an extra-Https (eHttps) endpoint, which is not defined in the configuration.


```go
package routing

import (
	"crypto/rand"
	"encoding/base64"
	"testing"

	"github.com/ipfs/kubo/config"
	"github.com/libp2p/go-libp2p/core/crypto"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/stretchr/testify/require"
)

func TestParser(t *testing.T) {
	require := require.New(t)

	pid, sk, err := generatePeerID()
	require.NoError(err)

	router, err := Parse(config.Routers{
		"r1": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeHTTP,
				Parameters: &config.HTTPRouterParams{
					Endpoint: "testEndpoint",
				},
			},
		},
		"r2": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeSequential,
				Parameters: &config.ComposableRouterParams{
					Routers: []config.ConfigRouter{
						{
							RouterName: "r1",
						},
					},
				},
			},
		},
	}, config.Methods{
		config.MethodNameFindPeers: config.Method{
			RouterName: "r1",
		},
		config.MethodNameFindProviders: config.Method{
			RouterName: "r1",
		},
		config.MethodNameGetIPNS: config.Method{
			RouterName: "r1",
		},
		config.MethodNamePutIPNS: config.Method{
			RouterName: "r2",
		},
		config.MethodNameProvide: config.Method{
			RouterName: "r2",
		},
	}, &ExtraDHTParams{}, &ExtraHTTPParams{
		PeerID:     string(pid),
		PrivKeyB64: sk,
	})

	require.NoError(err)

	comp, ok := router.(*Composer)
	require.True(ok)

	require.Equal(comp.FindPeersRouter, comp.FindProvidersRouter)
	require.Equal(comp.ProvideRouter, comp.PutValueRouter)
}

```

This code looks like it is configuring a router for a microservice architecture.

The code uses the Composable transport concept, which allows for the use of reusable code in a Composable service.

The router is configured using a `ConfigRouter` object, which defines the routes for the router.

The router is then configured using the `RouterParser` object, which defines the parse structure for the router configuration.

The `Router` property is defined as a `ComposableRouter` with two sub-routers, one for HTTP1 and one for HTTP2.

The `Routers` property is a slice of `ConfigRouter` objects, each of which defines a router for the corresponding HTTP version.

Finally, the router is initialized with a `Kty` of `http1` and an `ID` of `"http1:1"`.


```go
func TestParserRecursive(t *testing.T) {
	require := require.New(t)

	pid, sk, err := generatePeerID()
	require.NoError(err)

	router, err := Parse(config.Routers{
		"http1": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeHTTP,
				Parameters: &config.HTTPRouterParams{
					Endpoint: "testEndpoint1",
				},
			},
		},
		"http2": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeHTTP,
				Parameters: &config.HTTPRouterParams{
					Endpoint: "testEndpoint2",
				},
			},
		},
		"http3": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeHTTP,
				Parameters: &config.HTTPRouterParams{
					Endpoint: "testEndpoint3",
				},
			},
		},
		"composable1": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeSequential,
				Parameters: &config.ComposableRouterParams{
					Routers: []config.ConfigRouter{
						{
							RouterName: "http1",
						},
						{
							RouterName: "http2",
						},
					},
				},
			},
		},
		"composable2": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeParallel,
				Parameters: &config.ComposableRouterParams{
					Routers: []config.ConfigRouter{
						{
							RouterName: "composable1",
						},
						{
							RouterName: "http3",
						},
					},
				},
			},
		},
	}, config.Methods{
		config.MethodNameFindPeers: config.Method{
			RouterName: "composable2",
		},
		config.MethodNameFindProviders: config.Method{
			RouterName: "composable2",
		},
		config.MethodNameGetIPNS: config.Method{
			RouterName: "composable2",
		},
		config.MethodNamePutIPNS: config.Method{
			RouterName: "composable2",
		},
		config.MethodNameProvide: config.Method{
			RouterName: "composable2",
		},
	}, &ExtraDHTParams{}, &ExtraHTTPParams{
		PeerID:     string(pid),
		PrivKeyB64: sk,
	})

	require.NoError(err)

	_, ok := router.(*Composer)
	require.True(ok)
}

```

This is a Go function that initializes a new router with two entries in a routing configuration file. The router is expected to have two endpoints, "composable1" and "composable2", which are expected to return the "Hello, world!" message in response to a GET or POST request.

The function first creates a new `require.New` function and then parses the routing configuration file using the `Parse` function from the `config.Router` struct. The router is initialized with two entries for the "composable1" and "composable2" endpoints, each entry including the router name, the router type, and the parameters for the router.

Finally, the function returns an instance of the `ExtraDHTParams` struct, which includes some extra information for debugging and logging, and then exits the function with an error containing any information about any issues that were found during initialization.


```go
func TestParserRecursiveLoop(t *testing.T) {
	require := require.New(t)

	_, err := Parse(config.Routers{
		"composable1": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeSequential,
				Parameters: &config.ComposableRouterParams{
					Routers: []config.ConfigRouter{
						{
							RouterName: "composable2",
						},
					},
				},
			},
		},
		"composable2": config.RouterParser{
			Router: config.Router{
				Type: config.RouterTypeParallel,
				Parameters: &config.ComposableRouterParams{
					Routers: []config.ConfigRouter{
						{
							RouterName: "composable1",
						},
					},
				},
			},
		},
	}, config.Methods{
		config.MethodNameFindPeers: config.Method{
			RouterName: "composable2",
		},
		config.MethodNameFindProviders: config.Method{
			RouterName: "composable2",
		},
		config.MethodNameGetIPNS: config.Method{
			RouterName: "composable2",
		},
		config.MethodNamePutIPNS: config.Method{
			RouterName: "composable2",
		},
		config.MethodNameProvide: config.Method{
			RouterName: "composable2",
		},
	}, &ExtraDHTParams{}, nil)

	require.ErrorContains(err, "dependency loop creating router with name \"composable2\"")
}

```

这段代码定义了一个名为 `generatePeerID` 的函数，它接受两个参数 `pid` 和 `encodedPeerPublicKey`，并返回一个名为 `pid` 的字符串和一个编码后的 `pk`。

具体来说，这段代码执行以下操作：

1. 使用 `crypto.GenerateEd25519Key` 函数生成一个随机密钥 `sk`。
2. 使用 `crypto.MarshalPrivateKey` 函数将 `sk` 中的私钥编码为字节切片并返回。
3. 将编码后的 `pk` 和编码后的 `sk` 字节片组成一个 `encodedPeerPublicKey` 并返回。
4. 使用 `peer.IDFromPublicKey` 函数将 `encodedPeerPublicKey` 解析为 `pid`。
5. 如果任何步骤出现错误，函数返回 `nil`。

这段代码的主要目的是提供一个从本地随机生成到远程 `pk` 的方法，以便在客户端之间安全地交换消息。


```go
func generatePeerID() (string, string, error) {
	sk, pk, err := crypto.GenerateEd25519Key(rand.Reader)
	if err != nil {
		return "", "", err
	}

	bytes, err := crypto.MarshalPrivateKey(sk)
	if err != nil {
		return "", "", err
	}

	enc := base64.StdEncoding.EncodeToString(bytes)
	if err != nil {
		return "", "", err
	}

	pid, err := peer.IDFromPublicKey(pk)
	return pid.String(), enc, err
}

```