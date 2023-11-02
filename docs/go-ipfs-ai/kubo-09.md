# go-ipfs 源码解析 9

# `config/serialize/serialize.go`

这段代码是一个 Go 语言 package，名为 "fsrepo"，旨在实现文件系统 (File System) 存储资源 (File System Storage Resource) 的封装和访问。它实现了两个主要功能：创建一个新的文件系统存储资源 (File System Storage Resource) 对象，以及在文件系统存储资源上执行原子操作 (Atomic Operations)。

具体来说，这段代码可以执行以下操作：

1. 创建一个新的文件系统存储资源对象。当调用 `create` 函数并传递有效的键 (文件系统存储资源的关键字) 时，该函数会创建一个新的文件系统存储资源并返回其引用。
2. 在文件系统存储资源上执行原子操作。当调用 `原子` 函数并传递一个操作对象时，该函数会在文件系统存储资源上执行指定的操作。这里的操作对象是 `原子file` 包中的一个类型，它实现了原子操作的功能。


```go
package fsrepo

import (
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"os"
	"path/filepath"

	"github.com/ipfs/kubo/config"

	"github.com/facebookgo/atomicfile"
)

```

这段代码定义了一个名为 `ErrNotInitialized` 的错误类型，用于在尝试初始化 IPFS(Interoperable纪念日面板)时出现的情况。如果尝试读取不存在的设计模式配置文件，该错误类型将会被 `errors.New` 函数返回，其值为 "ipfs not initialized, please run 'ipfs init'"。

该代码中，还定义了一个名为 `ReadConfigFile` 的函数，它从指定文件名读取配置文件并返回。如果读取配置文件时出现任何错误，该函数将返回一个 `ErrNotInitialized` 错误。

`ReadConfigFile` 函数的作用是读取一个名为 `filename` 的文件，该文件包含 IPFS 的配置信息。函数首先尝试打开该文件以读取配置信息，如果打开失败，则检查文件是否存在，如果存在则代表一个错误，否则继续尝试读取。如果文件能够成功打开，则使用 `json.NewDecoder` 函数将文件内容解析为 JSON 对象，并返回。如果解析过程中出现错误，该函数将返回一个错误信息。




```go
// ErrNotInitialized is returned when we fail to read the config because the
// repo doesn't exist.
var ErrNotInitialized = errors.New("ipfs not initialized, please run 'ipfs init'")

// ReadConfigFile reads the config from `filename` into `cfg`.
func ReadConfigFile(filename string, cfg interface{}) error {
	f, err := os.Open(filename)
	if err != nil {
		if os.IsNotExist(err) {
			err = ErrNotInitialized
		}
		return err
	}
	defer f.Close()
	if err := json.NewDecoder(f).Decode(cfg); err != nil {
		return fmt.Errorf("failure to decode config: %w", err)
	}
	return nil
}

```

这段代码的作用是创建一个名为`filename`的文件，并将来自`cfg`的配置信息写入该文件。

具体来说，代码首先创建一个名为`filename`的目录，如果目录已存在，则创建失败，此时返回错误。然后，代码创建一个名为`filename`的文件，并使用`atomicfile`类型的新建文件系统，指定文件权限为0o755(即只读写)。

接着，代码创建一个名为`cfg`的接口，并尝试将其写入到`filename`中。如果写入失败，代码返回错误。如果写入成功，代码关闭文件并返回0。


```go
// WriteConfigFile writes the config from `cfg` into `filename`.
func WriteConfigFile(filename string, cfg interface{}) error {
	err := os.MkdirAll(filepath.Dir(filename), 0o755)
	if err != nil {
		return err
	}

	f, err := atomicfile.New(filename, 0o600)
	if err != nil {
		return err
	}
	defer f.Close()

	return encode(f, cfg)
}

```

这段代码定义了两个函数，`encode`函数的作用是将一个`value`接口类型的数据进行JSON编码并输出到指定的写入器（例如`io.Writer`）中，同时还提供了对`value`进行`MarshalIndent`的功能。而`Load`函数的作用是从一个指定的文件中读取配置文件并返回，如果文件读取失败，则返回一个错误。

在`encode`函数中，首先定义了一个名为`buf`的缓冲区变量，用于存储要编码的`value`数据，接着调用一个名为`config.Marshal`的函数将`value`类型转换为json编码的字节数组，然后将这个字节数组写入到`buf`中。最后，调用`w.Write`函数将`buf`中的字节数组写入到指定的写入器中，并返回任何错误。

在`Load`函数中，首先定义了一个名为`cfg`的`config.Config`类型变量，表示要读取的配置文件。然后调用一个名为`ReadConfigFile`的函数从指定的文件中读取配置文件，并将读取到的配置文件存储到`cfg`中。如果文件读取失败，则返回一个错误，否则返回`cfg`。

这两个函数都在`config.conf`文件中定义，文件中包含了`encode`函数的定义。


```go
// encode configuration with JSON.
func encode(w io.Writer, value interface{}) error {
	// need to prettyprint, hence MarshalIndent, instead of Encoder
	buf, err := config.Marshal(value)
	if err != nil {
		return err
	}
	_, err = w.Write(buf)
	return err
}

// Load reads given file and returns the read config, or error.
func Load(filename string) (*config.Config, error) {
	var cfg config.Config
	err := ReadConfigFile(filename, &cfg)
	if err != nil {
		return nil, err
	}

	return &cfg, err
}

```

# `config/serialize/serialize_test.go`

这段代码的作用是测试一个名为"fsrepo"的包中的一部分，主要功能是向指定文件夹下的".ipfsconfig"文件中写入配置文件。具体来说，代码实现了以下步骤：

1. 定义了一个名为"TestConfig"的测试函数，用于对配置文件进行测试。

2. 创建了一个名为"cfgWritten"的变量，该变量是一个指向名为"config.Config"的结构体的指针。

3. 构造了一个包含"faketest"作为对等方ID的配置对象，并将其写入到文件中。

4. 调用"WriteConfigFile"函数将配置文件写入到指定的文件中。

5. 调用"Load"函数从文件中读取配置文件。

6. 检查两个读取的配置文件是否包含相同的对等方ID。

7. 检查文件是否可读写。

8. 使用"math/rand"包生成一个随机整数，并将其与"ipfsconfig"文件中当前的对等方ID比较，以验证文件是否可读写。

9. 因为生成的随机整数在测试中失败，所以程序可能会崩溃或产生不可预测的行为。


```go
package fsrepo

import (
	"os"
	"runtime"
	"testing"

	config "github.com/ipfs/kubo/config"
)

func TestConfig(t *testing.T) {
	const filename = ".ipfsconfig"
	cfgWritten := new(config.Config)
	cfgWritten.Identity.PeerID = "faketest"

	err := WriteConfigFile(filename, cfgWritten)
	if err != nil {
		t.Fatal(err)
	}
	cfgRead, err := Load(filename)
	if err != nil {
		t.Fatal(err)
	}
	if cfgWritten.Identity.PeerID != cfgRead.Identity.PeerID {
		t.Fatal()
	}
	st, err := os.Stat(filename)
	if err != nil {
		t.Fatalf("cannot stat config file: %v", err)
	}

	if runtime.GOOS != "windows" { // see https://golang.org/src/os/types_windows.go
		if g := st.Mode().Perm(); g&0o117 != 0 {
			t.Fatalf("config file should not be executable or accessible to world: %v", g)
		}
	}
}

```

# `core/builder.go`

这段代码是一个名为 "core" 的包，它定义了一系列函数和结构体，以及一些常量。

它从 "github.com/ipfs/boxo/bootstrap" 和 "github.com/ipfs/kubo/core/node" 导入了一些来自 "boxo" 和 "kubo" 包的函数和结构体，这些函数和结构体用于管理 IPFS(InterPlanetary File System) 对象。

该包定义了一些函数用于创建和操作 IPFS 对象，包括：

- "createNode"函数，它接收一个 IPFS 句柄(可能是一个 URL)，并返回一个代表 IPFS 节点的对象。
- "create目錄"函数，它接收一个 IPFS 句柄和一个目录名，并返回一个代表 IPFS 目录的对象。
- "writeFile"函数，它接收一个 IPFS 句柄和一个文件名，并写入文件内容到 IPFS。

该包还定义了一些常量，包括：

- "boxo.NodeType"，它定义了 IPFS 节点类型。
- "boxo.NodeID"，它定义了每个 IPFS 节点的唯一 ID。
- "boxo.ObserverSetting"，它定义了 IPFS 节点的观察者设置。

此外，该包还导入了 "github.com/ipfs/go-metrics-interface" 和 "go.uber.org/dig" 和 "go.ipfs.org/v6/kepters" 等 packages，这些 packages 有助于收集 IPFS 对象的统计信息。


```go
package core

import (
	"context"
	"fmt"
	"reflect"
	"sync"
	"time"

	"github.com/ipfs/boxo/bootstrap"
	"github.com/ipfs/kubo/core/node"

	"github.com/ipfs/go-metrics-interface"
	"go.uber.org/dig"
	"go.uber.org/fx"
)

```

这段代码定义了一个名为FXNodeInfo的结构体，它包含了一些与fx选项有关的有用信息。这个结构体作为扩展点，用于向fx插件提供更多的信息以便于决定包含哪些选项。

接下来定义了一个名为fxOptFunc的函数，它接收一个FXNodeInfo类型的参数，并返回一个包含所有fx选项的切片和错误类型的数组。然后定义了一个名为fxOptionFuncs的数组，用于存储所有的fx选项函数。

最后，注册了一个名为注册FXOptionFunc的函数，它将在fx应用程序启动之前运行。这个函数会按注册顺序 invoke fxOptionFuncs 数组中的每个函数，并将返回的选项和错误类型的切片传递给下一个函数的 FXNodeInfo。


```go
// FXNodeInfo contains information useful for adding fx options.
// This is the extension point for providing more info/context to fx plugins
// to make decisions about what options to include.
type FXNodeInfo struct {
	FXOptions []fx.Option
}

// fxOptFunc takes in some info about the IPFS node and returns the full set of fx opts to use.
type fxOptFunc func(FXNodeInfo) ([]fx.Option, error)

var fxOptionFuncs []fxOptFunc

// RegisterFXOptionFunc registers a function that is run before the fx app is initialized.
// Functions are invoked in the order they are registered,
// and the resulting options are passed into the next function's FXNodeInfo.
```

这段代码定义了一个名为 `RegisterFXOptionFunc` 的函数类型，它接受一个名为 `fxOptFunc` 的参数，并且将该函数类型存储在一个名为 `fxOptionFuncs` 的数组中。

这种函数类型的作用是，让 `fxOptionFuncs` 数组中的每个函数都可以应用全局的、在所有 `NewNode` 的 invocations（调用新节点）中使用。它可能适用于多种情况，例如在初始化仓库、启动守护进程或运行迁移时使用。

这段代码还提醒说，如果您的 `fx` 选项中包含了一些复杂的行为，您应该考虑到这种情况下可能会发生的问题。例如，如果在使用一个禁止使用非允许 CID 的 blockservice，可能会导致一些迁移失败。


```go
//
// Note that these are applied globally, by all invocations of NewNode.
// There are multiple places in Kubo that construct nodes, such as:
//   - Repo initialization
//   - Daemon initialization
//   - When running migrations
//   - etc.
//
// If your fx options are doing anything sophisticated, you should keep this in mind.
//
// For example, if you plug in a blockservice that disallows non-allowlisted CIDs,
// this may break migrations that fetch migration code over IPFS.
func RegisterFXOptionFunc(optFunc fxOptFunc) {
	fxOptionFuncs = append(fxOptionFuncs, optFunc)
}

```

This is a Go function that creates a Kubernetes (K8s) service. The function is called `n.CreateService`.

Here is the code for the function:
go
func n.CreateService(cfg config.Service) (fixture.ServiceBootstrapper, error) {
	// Check if the service already exists.
	 _, err := n.Service(cfg.Service.Name)
	if err != nil {
		log.Warningf("Service %s already exists", err)
		return nil, err
	}

	// Check if the service should be online.
	opts := append(opts, fx.Extract(cfg.Service.Online))
	if !cfg.Service.Online {
		return nil, logAndUnwrapFxError(err)
	}

	app := fx.New(opts...)

	var once sync.Once
	var stopErr error
	n.stop = func() error {
		once.Do(func() {
			stopErr = app.Stop(context.Background())
			if stopErr != nil {
				log.Error("failure on stop: ", stopErr)
			}
			// Cancel the context _after_ the app has stopped.
			cancel()
		})
		return stopErr
	}
	n.IsOnline = cfg.Online

	go func() {
		// Shutdown the application if the lifetime context is canceled.
		// NOTE: we _should_ stop the application by calling `Close()`
		// on the process. But we currently manage everything with contexts.
		select {
		case <-lctx.Done():
			err := n.stop()
			if err != nil {
				log.Error("failure on stop: ", err)
			}
		case <-ctx.Done():
		}
	}()

	if app.Err() != nil {
		return nil, logAndUnwrapFxError(app.Err())
	}

	if err := app.Start(ctx); err != nil {
		return nil, logAndUnwrapFxError(err)
	}

	// TODO: How soon will bootstrap move to libp2p?
	return n, nil
}

This function is used to create a K8s service. It takes a configuration object as input, which specifies the service's name, online status, and other options.

Here are the configuration options that are extracted from the input configuration:
yaml
fx opts:
	- name:                                                - The name of the service
		+ fx.stringopt:is-hit-after-all:       Is the service hit after all?
		+ fx.stringopt:net-allow-query:            Is query allowed for the service?
		+ fx.stringopt:tcp-80:                      Is there a port 80 for the service?
		+ fx.stringopt:tcp-443:                      Is there a port 443 for the service?
		+ fx.stringopt:trusted-auth:              Is authentication trusted?
		+ fx.stringopt:remote-debug:          Is debugging remote dependencies?
		+ fx.stringopt:trusted-一大论：           Is appending to the "trusted-apps" list allowed?
			 merkle-生成的认证：
				检验在验证时是否使用完整的配置列表。

If the input configuration is missing any of these options, the function will log and return an error.

If the function creates a service successfully, it will return a `fixture.ServiceBootstrapper` object. If an error occurs, it will return an error.


```go
// from https://stackoverflow.com/a/59348871
type valueContext struct {
	context.Context
}

func (valueContext) Deadline() (deadline time.Time, ok bool) { return }
func (valueContext) Done() <-chan struct{}                   { return nil }
func (valueContext) Err() error                              { return nil }

type BuildCfg = node.BuildCfg // Alias for compatibility until we properly refactor the constructor interface

// NewNode constructs and returns an IpfsNode using the given cfg.
func NewNode(ctx context.Context, cfg *BuildCfg) (*IpfsNode, error) {
	// save this context as the "lifetime" ctx.
	lctx := ctx

	// derive a new context that ignores cancellations from the lifetime ctx.
	ctx, cancel := context.WithCancel(valueContext{ctx})

	// add a metrics scope.
	ctx = metrics.CtxScope(ctx, "ipfs")

	n := &IpfsNode{
		ctx: ctx,
	}

	opts := []fx.Option{
		node.IPFS(ctx, cfg),
		fx.NopLogger,
	}
	for _, optFunc := range fxOptionFuncs {
		var err error
		opts, err = optFunc(FXNodeInfo{FXOptions: opts})
		if err != nil {
			cancel()
			return nil, fmt.Errorf("building fx opts: %w", err)
		}
	}
	//nolint:staticcheck // https://github.com/ipfs/kubo/pull/9423#issuecomment-1341038770
	opts = append(opts, fx.Extract(n))

	app := fx.New(opts...)

	var once sync.Once
	var stopErr error
	n.stop = func() error {
		once.Do(func() {
			stopErr = app.Stop(context.Background())
			if stopErr != nil {
				log.Error("failure on stop: ", stopErr)
			}
			// Cancel the context _after_ the app has stopped.
			cancel()
		})
		return stopErr
	}
	n.IsOnline = cfg.Online

	go func() {
		// Shut down the application if the lifetime context is canceled.
		// NOTE: we _should_ stop the application by calling `Close()`
		// on the process. But we currently manage everything with contexts.
		select {
		case <-lctx.Done():
			err := n.stop()
			if err != nil {
				log.Error("failure on stop: ", err)
			}
		case <-ctx.Done():
		}
	}()

	if app.Err() != nil {
		return nil, logAndUnwrapFxError(app.Err())
	}

	if err := app.Start(ctx); err != nil {
		return nil, logAndUnwrapFxError(err)
	}

	// TODO: How soon will bootstrap move to libp2p?
	if !cfg.Online {
		return n, nil
	}

	return n, n.Bootstrap(bootstrap.DefaultBootstrapConfig)
}

```

这段代码的作用是输出应用程序错误信息中的仅 Inner 层错误信息，以便用户更好地理解应用程序的构建过程。现有的错误信息通常包含应用程序依赖项（如 `dig` 包）中的未公开错误。这些错误信息可能会包含应用程序错误日志的多个层次结构，因此在应用程序中解决错误通常需要查看错误链的详细信息。但是，对于开发人员来说，了解错误发生的位置是有用的。因此，这段代码允许用户更好地了解应用程序的构建过程，同时输出仅 Inner 层错误信息，以便用户更好地理解应用程序的构建过程。


```go
// Log the entire `app.Err()` but return only the innermost one to the user
// given the full error can be very long (as it can expose the entire build
// graph in a single string).
//
// The fx.App error exposed through `app.Err()` normally contains un-exported
// errors from its low-level `dig` package:
// * https://github.com/uber-go/dig/blob/5e5a20d/error.go#L82
// These usually wrap themselves in many layers to expose where in the build
// chain did the error happen. Although useful for a developer that needs to
// debug it, it can be very confusing for a user that just wants the IPFS error
// that he can probably fix without being aware of the entire chain.
// Unwrapping everything is not the best solution as there can be useful
// information in the intermediate errors, mainly in the next to last error
// that locates which component is the build error coming from, but it's the
// best we can do at the moment given all errors in dig are private and we
```

这段代码定义了一个名为 `logAndUnwrapFxError` 的函数，它接收一个名为 `fxAppErr` 的错误参数。函数的作用是检查 `fxAppErr` 是否为空，如果是，则返回 `nil`，否则会输出一条错误消息并返回 `fxAppErr`。

函数内部首先检查 `fxAppErr` 是否为空，如果是，则直接返回 `nil`。否则，会调用一个名为 `dig.RootCause` 的函数，并将 `err` 作为参数传递给该函数。

接着，函数会循环调用 `dig.RootCause` 函数，每次将 `err` 作为参数传递给函数，并记录 extraction 出的错误。如果发现提取出的错误与 `err` 相同，则认为该错误已经找到了，可以停止循环。否则，继续循环，继续提取新的错误。

最后，函数会返回一个格式化错误消息，其中包含 `err` 的详细信息，并输出该错误消息。


```go
// just have the generic `RootCause` API.
func logAndUnwrapFxError(fxAppErr error) error {
	if fxAppErr == nil {
		return nil
	}

	log.Error("constructing the node: ", fxAppErr)

	err := fxAppErr
	for {
		extractedErr := dig.RootCause(err)
		// Note that the `RootCause` name is misleading as it just unwraps only
		// *one* error layer at a time, so we need to continuously call it.
		if !reflect.TypeOf(extractedErr).Comparable() {
			// Some internal errors are not comparable (e.g., `dig.errMissingTypes`
			// which is a slice) and we can't go further.
			break
		}
		if extractedErr == err {
			// We didn't unwrap any new error in the last call, reached the innermost one.
			break
		}
		err = extractedErr
	}

	return fmt.Errorf("constructing the node (see log for full detail): %w", err)
}

```

# `core/core.go`

This is a list of packages in the libp2p project that are related to the peers-to-peers (P2P) network. The P2P network is designed to allow for decentralized, peer-to-peer networking.

The packages in this list include:

* `boxo`: This package provides a simplified Boxo-based IPFS node implementation with local storage.
* `ipnsrp`: This package implements the IPFS namesys router.
* `ipns`: This package provides support for IPFS namesys.
* `ipfsadm`: This package provides support for the IPFSAdm protocol.
* `ipfslocal`: This package provides a Boxo-based IPFS node implementation with local storage.
* `ipfsmount`: This package provides support for Boxo's `ipfsmount` function.
* `ipfsimountio`: This package provides support for the IPFSIO Mount interface.
* `ipfsimountrw`: This package provides support for the IPFSIO Mount read/write functionality.
* `ipfsim personally`: This package provides support for the IPFSIO Mount personal functionality.
* `boxo`: This package provides a simplified Boxo-based IPFS node implementation with local storage.
* `boxopsdns`: This package provides support for IPFS DNS.
* `boxofsyscall`: This package provides a Boxo-based system call out interface.
* `h恒河`: This package provides a double Free framework for Chinese characters.
* `p2p/go-libp2p/core/connmgr"	ic: This package provides the`


```go
/*
Package core implements the IpfsNode object and related methods.

Packages underneath core/ provide a (relatively) stable, low-level API
to carry out most IPFS-related tasks.  For more details on the other
interfaces and how core/... fits into the bigger IPFS picture, see:

	$ godoc github.com/ipfs/go-ipfs
*/
package core

import (
	"context"
	"encoding/json"
	"io"
	"time"

	"github.com/ipfs/boxo/filestore"
	pin "github.com/ipfs/boxo/pinning/pinner"
	"github.com/ipfs/go-datastore"

	bserv "github.com/ipfs/boxo/blockservice"
	bstore "github.com/ipfs/boxo/blockstore"
	exchange "github.com/ipfs/boxo/exchange"
	"github.com/ipfs/boxo/fetcher"
	mfs "github.com/ipfs/boxo/mfs"
	pathresolver "github.com/ipfs/boxo/path/resolver"
	provider "github.com/ipfs/boxo/provider"
	"github.com/ipfs/go-graphsync"
	ipld "github.com/ipfs/go-ipld-format"
	logging "github.com/ipfs/go-log"
	goprocess "github.com/jbenet/goprocess"
	ddht "github.com/libp2p/go-libp2p-kad-dht/dual"
	pubsub "github.com/libp2p/go-libp2p-pubsub"
	psrouter "github.com/libp2p/go-libp2p-pubsub-router"
	record "github.com/libp2p/go-libp2p-record"
	connmgr "github.com/libp2p/go-libp2p/core/connmgr"
	ic "github.com/libp2p/go-libp2p/core/crypto"
	p2phost "github.com/libp2p/go-libp2p/core/host"
	metrics "github.com/libp2p/go-libp2p/core/metrics"
	"github.com/libp2p/go-libp2p/core/network"
	peer "github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
	routing "github.com/libp2p/go-libp2p/core/routing"
	"github.com/libp2p/go-libp2p/p2p/discovery/mdns"
	p2pbhost "github.com/libp2p/go-libp2p/p2p/host/basic"
	ma "github.com/multiformats/go-multiaddr"
	madns "github.com/multiformats/go-multiaddr-dns"

	"github.com/ipfs/boxo/bootstrap"
	"github.com/ipfs/boxo/namesys"
	ipnsrp "github.com/ipfs/boxo/namesys/republisher"
	"github.com/ipfs/boxo/peering"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core/node"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/fuse/mount"
	"github.com/ipfs/kubo/p2p"
	"github.com/ipfs/kubo/repo"
	irouting "github.com/ipfs/kubo/routing"
)

```

This is a Go-style struct that defines the settings of a Go service that uses the IPFS (Interior Permanent Foster) system.

The struct includes several fields related to routing, DNS resolver, and IPLD (Interior Permanent Foster) path resolver. The `Routing` field is set to `"ipfs-dht"` and specifies the routing system.

The `DNSResolver` field is set to `madns.Resolver` and is responsible for resolving domain names to IP addresses.

The `IPLDPathResolver` field is set to `pathresolver.Resolver` and is responsible for resolving IPLD paths to file paths.

The `UnixFSPathResolver` field is set to `pathresolver.Resolver` and is responsible for resolving UnixFS paths to file paths.

The `OfflineIPLDPathResolver` field is set to `pathresolver.Resolver` and is responsible for resolving IPLD paths to paths that are not available locally.

The `OfflineUnixFSPathResolver` field is set to `pathresolver.Resolver` and is responsible for resolving UnixFS paths to paths that are not available locally.

The `Exchange` field is set to `exchange.Interface` and is responsible for the block exchange and strategy (bitswap).

The `Namesys` field is set to `namesys.NameSystem` and is responsible for resolving paths to hashes using the name system.

The `Provider` field is set to `provider.System` and is responsible for the value provider system.

The `IpnsRepub` field is set to `ipnsrp.Republisher` and is responsible for publishing and subscribing to IPNS (Interior Permanent Foster) blocks.

The `GraphExchange` field is set to `graphsync.GraphExchange` and is responsible for synchronizing graph data.

The `ResourceManager` field is set to `network.ResourceManager` and is responsible for managing resources for the service.

The `PubSub` field is set to `pubsub.PubSub` and is responsible for handling messages from and publishing to出版/订阅管道.

The `PSRouter` field is set to `psrouter.PubsubValueStore` and is responsible for storing and retrieving values from the `Pubsub` field.

The `DHT` field is set to `ddht.DHT` and is responsible for managing Distributed Hash Table (DHT) data.

The `DHTClient` field is set to `routing.Routing` and is responsible for interacting with the DHT.

The `P2P` field is set to `p2p.P2P` and is responsible for implementing the peer-to-peer (P2P) functionality.

The `Stop` field is set to `goprocess.Process` and is responsible for shutting down the service when it is stopped.

The `IsOnline` field is set to `bool` and is responsible for determining whether the service is running online or offline.

The `IsDaemon` field is set to `bool` and is responsible for determining if the service is running as a daemon or not.


```go
var log = logging.Logger("core")

// IpfsNode is IPFS Core module. It represents an IPFS instance.
type IpfsNode struct {
	// Self
	Identity peer.ID // the local node's identity

	Repo repo.Repo

	// Local node
	Pinning         pin.Pinner             // the pinning manager
	Mounts          Mounts                 `optional:"true"` // current mount state, if any.
	PrivateKey      ic.PrivKey             `optional:"true"` // the local node's private Key
	PNetFingerprint libp2p.PNetFingerprint `optional:"true"` // fingerprint of private network

	// Services
	Peerstore                   pstore.Peerstore          `optional:"true"` // storage for other Peer instances
	Blockstore                  bstore.GCBlockstore       // the block store (lower level)
	Filestore                   *filestore.Filestore      `optional:"true"` // the filestore blockstore
	BaseBlocks                  node.BaseBlocks           // the raw blockstore, no filestore wrapping
	GCLocker                    bstore.GCLocker           // the locker used to protect the blockstore during gc
	Blocks                      bserv.BlockService        // the block service, get/add blocks.
	DAG                         ipld.DAGService           // the merkle dag service, get/add objects.
	IPLDFetcherFactory          fetcher.Factory           `name:"ipldFetcher"`          // fetcher that paths over the IPLD data model
	UnixFSFetcherFactory        fetcher.Factory           `name:"unixfsFetcher"`        // fetcher that interprets UnixFS data
	OfflineIPLDFetcherFactory   fetcher.Factory           `name:"offlineIpldFetcher"`   // fetcher that paths over the IPLD data model without fetching new blocks
	OfflineUnixFSFetcherFactory fetcher.Factory           `name:"offlineUnixfsFetcher"` // fetcher that interprets UnixFS data without fetching new blocks
	Reporter                    *metrics.BandwidthCounter `optional:"true"`
	Discovery                   mdns.Service              `optional:"true"`
	FilesRoot                   *mfs.Root
	RecordValidator             record.Validator

	// Online
	PeerHost                  p2phost.Host               `optional:"true"` // the network host (server+client)
	Peering                   *peering.PeeringService    `optional:"true"`
	Filters                   *ma.Filters                `optional:"true"`
	Bootstrapper              io.Closer                  `optional:"true"` // the periodic bootstrapper
	Routing                   irouting.ProvideManyRouter `optional:"true"` // the routing system. recommend ipfs-dht
	DNSResolver               *madns.Resolver            // the DNS resolver
	IPLDPathResolver          pathresolver.Resolver      `name:"ipldPathResolver"`          // The IPLD path resolver
	UnixFSPathResolver        pathresolver.Resolver      `name:"unixFSPathResolver"`        // The UnixFS path resolver
	OfflineIPLDPathResolver   pathresolver.Resolver      `name:"offlineIpldPathResolver"`   // The IPLD path resolver that uses only locally available blocks
	OfflineUnixFSPathResolver pathresolver.Resolver      `name:"offlineUnixFSPathResolver"` // The UnixFS path resolver that uses only locally available blocks
	Exchange                  exchange.Interface         // the block exchange + strategy (bitswap)
	Namesys                   namesys.NameSystem         // the name system, resolves paths to hashes
	Provider                  provider.System            // the value provider system
	IpnsRepub                 *ipnsrp.Republisher        `optional:"true"`
	GraphExchange             graphsync.GraphExchange    `optional:"true"`
	ResourceManager           network.ResourceManager    `optional:"true"`

	PubSub   *pubsub.PubSub             `optional:"true"`
	PSRouter *psrouter.PubsubValueStore `optional:"true"`

	DHT       *ddht.DHT       `optional:"true"`
	DHTClient routing.Routing `name:"dhtc" optional:"true"`

	P2P *p2p.P2P `optional:"true"`

	Process goprocess.Process
	ctx     context.Context

	stop func() error

	// Flags
	IsOnline bool `optional:"true"` // Online is set when networking is enabled.
	IsDaemon bool `optional:"true"` // Daemon is set when running on a long-running daemon.
}

```

这段代码定义了一个名为 "Mounts" 的结构体，它表示节点挂载的状态。这个结构体包含两个成员变量：ipfs 和 ipns，它们都是代表节点挂载的文件系统类型。

接下来，定义了一个名为 "IpfsNode" 的结构体，它代表一个 ipfs 节点。在这个结构体中，有一个名为 "Close" 的方法，它关闭了当前节点，同时也会关闭 ipfs 服务。

另外，还有一个名为 "Context" 的方法，它返回了一个与当前节点上下文相关的 context，如果当前节点上下文还没有被创建，则会创建一个新的上下文并返回它。

最后，在 "IpfsNode" 的 "Close" 方法中，调用了 "stop" 方法来关闭节点，这个方法可能需要在 ipfs 服务自身进行实现。


```go
// Mounts defines what the node's mount state is. This should
// perhaps be moved to the daemon or mount. It's here because
// it needs to be accessible across daemon requests.
type Mounts struct {
	Ipfs mount.Mount
	Ipns mount.Mount
}

// Close calls Close() on the App object
func (n *IpfsNode) Close() error {
	return n.stop()
}

// Context returns the IpfsNode context
func (n *IpfsNode) Context() context.Context {
	if n.ctx == nil {
		n.ctx = context.TODO()
	}
	return n.ctx
}

```

This is a Go function that initializes the data store and retrieves the first bootstrap peer from the configuration, or in case the configuration is not specified, it will get the freshest bootstrap peers from the code.

It first checks if the bootstrap peers configuration is set, if not, it will attempt to load the freshest bootstrap peers from the code. Then, it checks if the backup peers configuration is set, if not, it will attempt to load the freshest backup peers from the code.

Finally, it sets the BackupBootstrapInterval configuration, which controls how often the internal backup bootstrap peers are updated.


```go
// Bootstrap will set and call the IpfsNodes bootstrap function.
func (n *IpfsNode) Bootstrap(cfg bootstrap.BootstrapConfig) error {
	// TODO what should return value be when in offlineMode?
	if n.Routing == nil {
		return nil
	}

	if n.Bootstrapper != nil {
		n.Bootstrapper.Close() // stop previous bootstrap process.
	}

	// if the caller did not specify a bootstrap peer function, get the
	// freshest bootstrap peers from config. this responds to live changes.
	if cfg.BootstrapPeers == nil {
		cfg.BootstrapPeers = func() []peer.AddrInfo {
			ps, err := n.loadBootstrapPeers()
			if err != nil {
				log.Warn("failed to parse bootstrap peers from config")
				return nil
			}
			return ps
		}
	}
	if load, _ := cfg.BackupPeers(); load == nil {
		save := func(ctx context.Context, peerList []peer.AddrInfo) {
			err := n.saveTempBootstrapPeers(ctx, peerList)
			if err != nil {
				log.Warnf("saveTempBootstrapPeers failed: %s", err)
				return
			}
		}
		load = func(ctx context.Context) []peer.AddrInfo {
			peerList, err := n.loadTempBootstrapPeers(ctx)
			if err != nil {
				log.Warnf("loadTempBootstrapPeers failed: %s", err)
				return nil
			}
			return peerList
		}
		cfg.SetBackupPeers(load, save)
	}

	repoConf, err := n.Repo.Config()
	if err != nil {
		return err
	}
	if repoConf.Internal.BackupBootstrapInterval != nil {
		cfg.BackupBootstrapInterval = repoConf.Internal.BackupBootstrapInterval.WithDefault(time.Hour)
	}

	n.Bootstrapper, err = bootstrap.Bootstrap(n.Identity, n.PeerHost, n.Routing, cfg)
	return err
}

```

这段代码的作用是实现了一个 IpfsNode 类型的对象，用于管理本地temp bootstrap peer的注册和注销。

具体来说，它完成了以下两件事情：

1. 加载本地temp bootstrap peer的注册信息，返回一个包含多个 peer.AddrInfo 对象的切片。
2. 将本地temp bootstrap peer的注册信息保存到本地datastore中，并在一定条件下同步该操作。

代码中定义了两个函数，分别用于加载和保存本地temp bootstrap peer的注册信息。这两个函数都在函数式接口中定义，使用了 gRPC 协程的单例模式来管理状态。同时，代码中还定义了一个名为 IpfsNode 的 struct，用于封装有关 IpfsNode 类型的信息。


```go
var TempBootstrapPeersKey = datastore.NewKey("/local/temp_bootstrap_peers")

func (n *IpfsNode) loadBootstrapPeers() ([]peer.AddrInfo, error) {
	cfg, err := n.Repo.Config()
	if err != nil {
		return nil, err
	}

	return cfg.BootstrapPeers()
}

func (n *IpfsNode) saveTempBootstrapPeers(ctx context.Context, peerList []peer.AddrInfo) error {
	ds := n.Repo.Datastore()
	bytes, err := json.Marshal(config.BootstrapPeerStrings(peerList))
	if err != nil {
		return err
	}

	if err := ds.Put(ctx, TempBootstrapPeersKey, bytes); err != nil {
		return err
	}
	return ds.Sync(ctx, TempBootstrapPeersKey)
}

```

这段代码定义了一个名为func的函数，接收一个名为n的IpfsNode参数，并返回一个包含 peers.AddrInfo 的切片。

函数首先从本地存储介质（如本地文件系统）中获取一个名为TempBootstrapPeersKey的数据存储块，然后解码JSON字节为切片中的peer.AddrInfo。

然后函数使用 config.ParseBootstrapPeers 函数将解码后的切片传递给 config.ParsePeerAddress 函数，这样就可以将IpfsNode的peer.AddrInfo设置为从Config中获得的peer.AddrInfo。

此外，这段代码定义了一个名为ConstructPeerHostOpts的结构体，其中包含用于配置对等网络主机选项的属性和方法。这些选项包括添加自定义对等网络主机以设置默认端口映射、禁用NAT端口映射、禁用中继、启用中继等。


```go
func (n *IpfsNode) loadTempBootstrapPeers(ctx context.Context) ([]peer.AddrInfo, error) {
	ds := n.Repo.Datastore()
	bytes, err := ds.Get(ctx, TempBootstrapPeersKey)
	if err != nil {
		return nil, err
	}

	var addrs []string
	if err := json.Unmarshal(bytes, &addrs); err != nil {
		return nil, err
	}
	return config.ParseBootstrapPeers(addrs)
}

type ConstructPeerHostOpts struct {
	AddrsFactory      p2pbhost.AddrsFactory
	DisableNatPortMap bool
	DisableRelay      bool
	EnableRelayHop    bool
	ConnectionManager connmgr.ConnManager
}

```

# `core/core_test.go`

This is a testing function that checks if the `NewNode` function correctly constructs a Quasar service instance with the given configuration. The function uses the `repo.Mock` struct to simulate the interactions with the `datastore` package, and the `syncds.MutexWrap` struct to create a mock implementation of a `MapDatastore`.

The function checks whether the `bad` slice is populated with `config.Config` objects, which should represent configurations that have syntax errors. It also checks whether the `bad` slice is populated with `Node` instances that were successfully created, or `NewNode` returns an error. If any issues are found, it logs them using the `t.Error` function.

If the `bad` slice is not populated with `Node` instances, the function logs the error and then tries to create a `Node` instance for each bad configuration using the `NewNode` function again. If the `bad` slice still contains `Node` instances for all configurations, the function logs the error and then aborts.


```go
package core

import (
	"testing"

	context "context"

	"github.com/ipfs/kubo/repo"

	datastore "github.com/ipfs/go-datastore"
	syncds "github.com/ipfs/go-datastore/sync"
	config "github.com/ipfs/kubo/config"
)

func TestInitialization(t *testing.T) {
	ctx := context.Background()
	id := testIdentity

	good := []*config.Config{
		{
			Identity: id,
			Addresses: config.Addresses{
				Swarm: []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
				API:   []string{"/ip4/127.0.0.1/tcp/8000"},
			},
		},

		{
			Identity: id,
			Addresses: config.Addresses{
				Swarm: []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
				API:   []string{"/ip4/127.0.0.1/tcp/8000"},
			},
		},
	}

	bad := []*config.Config{
		{},
	}

	for i, c := range good {
		r := &repo.Mock{
			C: *c,
			D: syncds.MutexWrap(datastore.NewMapDatastore()),
		}
		n, err := NewNode(ctx, &BuildCfg{Repo: r})
		if n == nil || err != nil {
			t.Error("Should have constructed.", i, err)
		}
	}

	for i, c := range bad {
		r := &repo.Mock{
			C: *c,
			D: syncds.MutexWrap(datastore.NewMapDatastore()),
		}
		n, err := NewNode(ctx, &BuildCfg{Repo: r})
		if n != nil || err == nil {
			t.Error("Should have failed to construct.", i)
		}
	}
}

```

Yes, the text you provided includes a valid Base64 encoded PNG image. When decoded, the image is a PNG file that can be opened and viewed in a variety of image editors and devices, such as Google Chrome, Microsoft Paint, and the like.


```go
var testIdentity = config.Identity{
	PeerID:  "QmNgdzLieYi8tgfo2WfTUzNVH5hQK9oAYGVf6dxN12NrHt",
	PrivKey: "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",
}

```

# `core/commands/active.go`

这段代码定义了一个名为 "commands" 的包，其中定义了一些常用的命令，包括打印命令、打印输出、打印所有命令、将命令输出到控制台、并行执行命令等。

具体来说，该代码实现了一系列命令行工具的常用功能。首先，通过导入 "fmt"、"io"、"sort" 和 "text/tabwriter" 等标准库，提供了格式化输出、输入/输出流操作、排序和表格输出等功能。

接着，通过导入 "github.com/ipfs/kubo/commands" 和 "github.com/ipfs/go-ipfs-cmds" 两个库，实现了打印命令行工具和 ipfs-cmds 库的结合使用。其中，第一个库 "github.com/ipfs/kubo/commands" 提供了更丰富的命令行工具，而第二个库 "github.com/ipfs/go-ipfs-cmds" 则提供了对 ipfs-cmds 库的封装和兼容。

最后，定义了一个名为 "verbose" 的选项，设置了是否在命令行工具的输出中显示更多的信息。


```go
package commands

import (
	"fmt"
	"io"
	"sort"
	"text/tabwriter"
	"time"

	oldcmds "github.com/ipfs/kubo/commands"

	cmds "github.com/ipfs/go-ipfs-cmds"
)

const (
	verboseOptionName = "verbose"
)

```

This appears to be a Go program that parses command line options and usage strings for a command-line tool. The program takes input from the standard input and writes the parsed options and usage information to the output.

The program has a verbose flag, which causes the program to write more detailed information for each command line option and usage string. The verbose flag is not explicitly documented in the program.

The program has a dependency on the `strings` package, which provides the `Split` function used in the program to split the input by whitespace.

The program also has a dependency on the `fmt` package, which provides the `Fprint` function used in the program to format the output.

The program has a function named `cmdLoop` that iterates over the input lines and parses each command line option and usage string. The parsed information is then written to the output using the `fmt` package.

The program has a function named `main` that initializes the program and runs it.

I hope this helps! Let me know if you have any questions.


```go
var ActiveReqsCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List commands run on this IPFS node.",
		ShortDescription: `
Lists running and recently run commands.
`,
	},
	NoLocal: true,
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		ctx := env.(*oldcmds.Context)
		return cmds.EmitOnce(res, ctx.ReqLog.Report())
	},
	Options: []cmds.Option{
		cmds.BoolOption(verboseOptionName, "v", "Print extra information."),
	},
	Subcommands: map[string]*cmds.Command{
		"clear":    clearInactiveCmd,
		"set-time": setRequestClearCmd,
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *[]*cmds.ReqLogEntry) error {
			verbose, _ := req.Options[verboseOptionName].(bool)

			tw := tabwriter.NewWriter(w, 4, 4, 2, ' ', 0)
			if verbose {
				fmt.Fprint(tw, "ID\t")
			}
			fmt.Fprint(tw, "Command\t")
			if verbose {
				fmt.Fprint(tw, "Arguments\tOptions\t")
			}
			fmt.Fprintln(tw, "Active\tStartTime\tRunTime")

			for _, req := range *out {
				if verbose {
					fmt.Fprintf(tw, "%d\t", req.ID)
				}
				fmt.Fprintf(tw, "%s\t", req.Command)
				if verbose {
					fmt.Fprintf(tw, "%v\t[", req.Args)
					var keys []string
					for k := range req.Options {
						keys = append(keys, k)
					}
					sort.Strings(keys)

					for _, k := range keys {
						fmt.Fprintf(tw, "%s=%v,", k, req.Options[k])
					}
					fmt.Fprintf(tw, "]\t")
				}

				var live time.Duration
				if req.Active {
					live = time.Since(req.StartTime)
				} else {
					live = req.EndTime.Sub(req.StartTime)
				}
				t := req.StartTime.Format(time.Stamp)
				fmt.Fprintf(tw, "%t\t%s\t%s\n", req.Active, t, live)
			}
			tw.Flush()

			return nil
		}),
	},
	Type: []*cmds.ReqLogEntry{},
}

```

这两段代码定义了两个命令，分别是"clearInactiveCmd"和"setRequestClearCmd"。

"clearInactiveCmd"命令的作用是清除日志中未活动的请求。具体来说，它创建了一个新的"oldcmds.Context"类型的变量"ctx"，然后调用了一个名为"ReqLog.ClearInactive"的函数来清除日志中未活动的请求。最后，它返回一个 nil 表示成功。

"setRequestClearCmd"命令的作用是设置在日志中保留多少未活动的请求。具体来说，它创建了一个新的"oldcmds.Context"类型的变量"ctx"，然后设置了一个名为"ReqLog.SetKeepTime"的函数来设置请求保留时间。该函数接受一个名为"time"的参数，它是一个时间戳类型。最后，它返回一个 nil 表示成功。


```go
var clearInactiveCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Clear inactive requests from the log.",
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		ctx := env.(*oldcmds.Context)
		ctx.ReqLog.ClearInactive()
		return nil
	},
}

var setRequestClearCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Set how long to keep inactive requests in the log.",
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("time", true, false, "Time to keep inactive requests in log."),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		tval, err := time.ParseDuration(req.Arguments[0])
		if err != nil {
			return err
		}
		ctx := env.(*oldcmds.Context)
		ctx.ReqLog.SetKeepTime(tval)

		return nil
	},
}

```

# `core/commands/add.go`

这段代码定义了一个名为 "commands" 的包，其中定义了一些函数和变量，包括：

- 一个名为 "errors" 的错误类型，其构造函数为 "errors.SyntaxError"。
- 一个名为 "fmt" 的字符串格式化函数，用于将 "格式化字符串" 的参数 "strings" 转换成格式化字符串的格式。
- 一个名为 "io" 的 "io" 包中的 "ioutil.ReadCloser" 函数，该函数从通道或读物中读取数据并关闭读取者。
- 一个名为 "os" 的 "os" 包中的 "exec" 函数，该函数用于运行 "command" 并返回标准输出和标准错误。
- 一个名为 "gopath" 的 "path" 包中的 "路径" 函数，该函数用于将 "path.Combine" 构建的路径连接到根目录。
- 一个名为 "strings" 的 "strings" 包中的函数，定义了 "capitalize" 和 "lowercase" 函数。
- 一个名为 "cmdenv" 的 "cmdenv" 包中的函数，定义了 "环境变量" 函数。
- 一个名为 "ipfs" 的 "ipfs" 包中的函数，定义了 "文件系统" 函数。
- 一个名为 "mfs" 的 "mfs" 包中的函数，定义了 "文件系统" 函数。
- 一个名为 "path" 的 "path" 包中的函数，定义了 "路径.Relative" 函数。
- 一个名为 "cmds" 的 "ipfs.cmds" 包中的函数，定义了 "ipfs.Command" 函数。
- 一个名为 "ipld" 的 "ipld" 包中的函数，定义了 "ipld.Color" 函数。
- 一个名为 "mh" 的 "multiformats.go" 包中的函数，定义了 "multihash.C环" 函数。

这些函数和变量的作用不明确，具体取决于使用场景。


```go
package commands

import (
	"errors"
	"fmt"
	"io"
	"os"
	gopath "path"
	"strings"

	"github.com/ipfs/kubo/core/commands/cmdenv"

	"github.com/cheggaaa/pb"
	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/files"
	mfs "github.com/ipfs/boxo/mfs"
	"github.com/ipfs/boxo/path"
	cmds "github.com/ipfs/go-ipfs-cmds"
	ipld "github.com/ipfs/go-ipld-format"
	mh "github.com/multiformats/go-multihash"
)

```

这段代码定义了一个名为 `AddEvent` 的结构体类型，它包含了一些 JSON 字段，包括 `Name`、`Hash`、`Bytes` 和 `Size`。

它还定义了一个名为 `ErrDepthLimitExceeded` 的错误类型，它的含义是最大深度已达到极限。

接着，定义了一些选项，如 `quietOptionName`、`quieterOptionName`、`silentOptionName`、`progressOptionName`、`trickleOptionName`、`wrapOptionName`、`onlyHashOptionName`、`chunkerOptionName`、`pinOptionName`、`rawLeavesOptionName`、`noCopyOptionName`、`fstoreCacheOptionName` 和 `cidVersionOptionName`，它们表示了什么含义，可以用来控制一些 JSON 字段的输出。

最后，定义了一个名为 `ToFileOptionName` 的选项，它的含义是将 JSON 字段输出到文件中。


```go
// ErrDepthLimitExceeded indicates that the max depth has been exceeded.
var ErrDepthLimitExceeded = fmt.Errorf("depth limit exceeded")

type AddEvent struct {
	Name  string
	Hash  string `json:",omitempty"`
	Bytes int64  `json:",omitempty"`
	Size  string `json:",omitempty"`
}

const (
	quietOptionName       = "quiet"
	quieterOptionName     = "quieter"
	silentOptionName      = "silent"
	progressOptionName    = "progress"
	trickleOptionName     = "trickle"
	wrapOptionName        = "wrap-with-directory"
	onlyHashOptionName    = "only-hash"
	chunkerOptionName     = "chunker"
	pinOptionName         = "pin"
	rawLeavesOptionName   = "raw-leaves"
	noCopyOptionName      = "nocopy"
	fstoreCacheOptionName = "fscache"
	cidVersionOptionName  = "cid-version"
	hashOptionName        = "hash"
	inlineOptionName      = "inline"
	inlineLimitOptionName = "inline-limit"
	toFilesOptionName     = "to-files"
)

```

这段代码定义了一个名为"adderOutChanSize"的变量，其值为8。这个变量可能是在某种情况下需要的一个常量，但并没有进一步的定义或说明，所以它的作用是什么并不清楚。


```go
const adderOutChanSize = 8

var AddCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Add a file or directory to IPFS.",
		ShortDescription: `
Adds the content of <path> to IPFS. Use -r to add directories (recursively).
`,
		LongDescription: `
Adds the content of <path> to IPFS. Use -r to add directories.
Note that directories are added recursively, to form the IPFS
MerkleDAG.

If the daemon is not running, it will just add locally.
If the daemon is started later, it will be advertised after a few
```

这段代码是一个用于运行IPFS（InterPlanetary File System，互联网 Planetary File System）的命令行工具的代码。IPFS是一种点对点分布式文件系统，可以有效地对IPFS网络上的文件进行高速且安全地传输。

首先，这段代码引入了一个名为seconds的参数。如果使用的是'-w'选项，则会创建一个只包含已添加文件的目录。这个目录中的文件名将保留其原始文件名。

在实际应用中，用户可以通过执行以下命令行操作来将文件添加到IPFS：
ruby
ipfs add <file_name> [-w]

其中，`<file_name>`是文件名，`-w`选项用于将文件包裹到一个目录中。如果使用了'-w'选项，则会自动创建一个名为目录前缀加上文件名的目录，并将文件名保留为原始文件名。

通过执行以下命令，可以启动IPFS的repovider：
ruby
ipfs repo info <graph_url>

其中，`<graph_url>`是IPFS网络上的一个节点，用于告诉我们IPFS的repovider如何连接到这些节点。

最后，`seconds`的作用是让IPFS的repovider在运行时知道要等待多长时间，以确保在repoinfo命令执行完成后可以得到最新的repo信息。


```go
seconds when the reprovider runs.

The wrap option, '-w', wraps the file (or files, if using the
recursive option) in a directory. This directory contains only
the files which have been added, and means that the file retains
its filename. For example:

  > ipfs add example.jpg
  added QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH example.jpg
  > ipfs add example.jpg -w
  added QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH example.jpg
  added QmaG4FuMqEBnQNn3C8XJ5bpW8kLs7zq2ZXgHptJHbKDDVx

You can now refer to the added file in a gateway, like so:

  /ipfs/QmaG4FuMqEBnQNn3C8XJ5bpW8kLs7zq2ZXgHptJHbKDDVx/example.jpg

```

这段代码使用了 IPFS (InterPlanetary File System) API，用于在本地或通过网络快速存取文件。它主要展示了两个主要选项：将 IPFS 中的文件导入到本地并指定 "--pin=true"（受保护的 "pin" 模式），或者使用 "--to-files"（将文件输出到指定目录）。

具体来说，第一个选项会在 IPFS 文件系统中创建一个目录 "example.jpg"，然后添加一个文件 "example.jpg" 到该目录中。导入的文件将自动使用 "pin=true" 选项进行保护，因此可以在 IPFS 文件系统中方便地访问它们，即使离开计算机或在网络中。

第二个选项会将 "example.jpg" 文件从 IPFS 文件系统中获取到指定目录中，并在文件列表中查看文件的详细信息。通过运行 "ipfs files --help"，您可以查看有关使用 MFS (Mono败) API 进行文件跟踪更多信息。

此外，代码中提到了 "chunker" 选项，它是 IPFS 文件系统 chunking（切分）策略的指定。通过指定 "chunker" 选项，您可以指定 IPFS 文件系统将文件分成多少个独立的 chunk进行传输。这可以提高文件系统的性能和效率，特别是在需要大量文件或需要频繁地传输文件时。


```go
Files imported with 'ipfs add' are protected from GC (implicit '--pin=true'),
but it is up to you to remember the returned CID to get the data back later.

Passing '--to-files' creates a reference in Files API (MFS), making it easier
to find it in the future:

  > ipfs files mkdir -p /myfs/dir
  > ipfs add example.jpg --to-files /myfs/dir/
  > ipfs files ls /myfs/dir/
  example.jpg

See 'ipfs files --help' to learn more about using MFS
for keeping track of added files and directories.

The chunker option, '-s', specifies the chunking strategy that dictates
```

这段代码是一个命令行工具，它可以将一个文件分割成不同的块（block），并且可以对同一块中的内容进行去重。它允许用户指定不同的块大小，"size-262144"是一个预设的固定块大小，也可以使用Buzhash或Rabin指纹 chunker来对文件进行内容定义的块大小的指定。

该命令行的作用是：将一个文件分割成指定大小的块，并且允许用户指定块的大小。通过指定不同的块大小，用户可以获得不同的块，对同一块内容进行去重，然后将文件保存到块中。


```go
how to break files into blocks. Blocks with same content can
be deduplicated. Different chunking strategies will produce different
hashes for the same file. The default is a fixed block size of
256 * 1024 bytes, 'size-262144'. Alternatively, you can use the
Buzhash or Rabin fingerprint chunker for content defined chunking by
specifying buzhash or rabin-[min]-[avg]-[max] (where min/avg/max refer
to the desired chunk sizes in bytes), e.g. 'rabin-262144-524288-1048576'.

The following examples use very small byte sizes to demonstrate the
properties of the different chunkers on a small file. You'll likely
want to use a 1024 times larger chunk sizes for most files.

  > ipfs add --chunker=size-2048 ipfs-logo.svg
  added QmafrLBfzRLV4XSH1XcaMMeaXEUhDJjmtDfsYU95TrWG87 ipfs-logo.svg
  > ipfs add --chunker=rabin-512-1024-2048 ipfs-logo.svg
  added Qmf1hDN65tR55Ubh2RN1FPxr69xq3giVBz1KApsresY8Gn ipfs-logo.svg

```

这段代码是一个 IPFS（InterPlanetary File System）命令行工具，用于创建和管理 IPFS 对象链接。IPFS 是一个分布式文件系统，可以在全球范围内快速、可靠地存储和共享文件。

代码的作用是检查已经创建的块（block），并输出这些块的哈希值（CID）。CID 是每个 IPFS 对象的唯一标识符，用于标识和访问对象。

IPFS 对象链接是 IPFS 中的一个概念，允许您将 IPFS 对象与哈希值关联。这样，您可以通过哈希值来快速查找和引用 IPFS 对象。

这段代码还输出了一个关于 CID determinism 注点和 'ipfs add' 命令的说明。CID determinism 是指 IPFS 对象链接的哈希值是否发生变化。而 'ipfs add' 命令是一个用于添加 IPFS 对象的常用命令。


```go
You can now check what blocks have been created by:

  > ipfs object links QmafrLBfzRLV4XSH1XcaMMeaXEUhDJjmtDfsYU95TrWG87
  QmY6yj1GsermExDXoosVE3aSPxdMNYr6aKuw3nA8LoWPRS 2059
  Qmf7ZQeSxq2fJVJbCmgTrLLVN9tDR9Wy5k75DxQKuz5Gyt 1195
  > ipfs object links Qmf1hDN65tR55Ubh2RN1FPxr69xq3giVBz1KApsresY8Gn
  QmY6yj1GsermExDXoosVE3aSPxdMNYr6aKuw3nA8LoWPRS 2059
  QmerURi9k4XzKCaaPbsK6BL5pMEjF7PGphjDvkkjDtsVf3 868
  QmQB28iwSriSUSMqG2nXDTLtdPHgWb4rebBrU7Q1j4vxPv 338

Finally, a note on hash (CID) determinism and 'ipfs add' command.

Almost all the flags provided by this command will change the final CID, and
new flags may be added in the future. It is not guaranteed for the implicit
defaults of 'ipfs add' to remain the same in future Kubo releases, or for other
```

This is a Go program that implements a `bar` progress bar that updates its value and display based on the incoming requests. It uses the `res` channel to receive the results of the `bar` operation and the `outChan` to send the results to the outside systems.

The program has several parameters:

* `res`: a channel to receive the results of the `bar` operation
* `req`: an instance of a `Request` struct that implements the `io.Reader` and `io.Writer` interfaces. This struct is used to handle the results and errors that may occur during the `bar` operation.
* `sizeChan`: a channel to receive the size of the `bar`. This is used to update the `bar.Total` field when a new value is received.
* `outChan`: a channel to send the results of the `bar` operation to the outside systems. This is used to send the `res.Result` field when the operation is complete.
* `req.Context`: an instance of the `context.Context` struct that is used to store the error of the `bar` operation.
* `e`: an instance of an `error` struct that is used to store any errors that may occur during the `res.Result` field.
* `errChan`: a channel to receive the error of the `bar` operation. This is used to send the error to the outside systems.
* `progChan`: a channel to receive the progress updates of the `bar`. This is used to update the `bar.Percent` field when the operation is complete.
* `reqDoneChan`: a channel to receive the signal indicating that the `req.Context.Done()` has occurred. This is used to clear the `req.Context.Done()` field.
* `cancelChan`: a channel to receive the signal indicating that the operation should be cancelled. This is used to clear the `cancel.Once` field.
* `resChan`: a channel to receive the result of the `res.Result` field. This is used to send the result of the `bar` operation to the outside systems.
* `reqChan`: a channel to receive the request from the `res.Context`. This is used to send the request to the outside systems.
* `resChan`: a channel to receive the result of the `res.Result` field. This is used to send the result of the `bar` operation to the outside systems.
* `resChan`: a channel to receive the error of the `res.Result` field. This is used to send the error to the outside systems.
* `resChan`: a channel to receive the result of the `res.Result` field. This is used to send the result of the `bar` operation to the outside systems.


```go
IPFS software to use the same import parameters as Kubo.

If you need to back up or transport content-addressed data using a non-IPFS
medium, CID can be preserved with CAR files.
See 'dag export' and 'dag import' for more information.
`,
	},

	Arguments: []cmds.Argument{
		cmds.FileArg("path", true, true, "The path to a file to be added to IPFS.").EnableRecursive().EnableStdin(),
	},
	Options: []cmds.Option{
		cmds.OptionRecursivePath, // a builtin option that allows recursive paths (-r, --recursive)
		cmds.OptionDerefArgs,     // a builtin option that resolves passed in filesystem links (--dereference-args)
		cmds.OptionStdinName,     // a builtin option that optionally allows wrapping stdin into a named file
		cmds.OptionHidden,
		cmds.OptionIgnore,
		cmds.OptionIgnoreRules,
		cmds.BoolOption(quietOptionName, "q", "Write minimal output."),
		cmds.BoolOption(quieterOptionName, "Q", "Write only final hash."),
		cmds.BoolOption(silentOptionName, "Write no output."),
		cmds.BoolOption(progressOptionName, "p", "Stream progress data."),
		cmds.BoolOption(trickleOptionName, "t", "Use trickle-dag format for dag generation."),
		cmds.BoolOption(onlyHashOptionName, "n", "Only chunk and hash - do not write to disk."),
		cmds.BoolOption(wrapOptionName, "w", "Wrap files with a directory object."),
		cmds.StringOption(chunkerOptionName, "s", "Chunking algorithm, size-[bytes], rabin-[min]-[avg]-[max] or buzhash").WithDefault("size-262144"),
		cmds.BoolOption(rawLeavesOptionName, "Use raw blocks for leaf nodes."),
		cmds.BoolOption(noCopyOptionName, "Add the file using filestore. Implies raw-leaves. (experimental)"),
		cmds.BoolOption(fstoreCacheOptionName, "Check the filestore for pre-existing blocks. (experimental)"),
		cmds.IntOption(cidVersionOptionName, "CID version. Defaults to 0 unless an option that depends on CIDv1 is passed. Passing version 1 will cause the raw-leaves option to default to true."),
		cmds.StringOption(hashOptionName, "Hash function to use. Implies CIDv1 if not sha2-256. (experimental)").WithDefault("sha2-256"),
		cmds.BoolOption(inlineOptionName, "Inline small blocks into CIDs. (experimental)"),
		cmds.IntOption(inlineLimitOptionName, "Maximum block size to inline. (experimental)").WithDefault(32),
		cmds.BoolOption(pinOptionName, "Pin locally to protect added files from garbage collection.").WithDefault(true),
		cmds.StringOption(toFilesOptionName, "Add reference to Files API (MFS) at the provided path."),
	},
	PreRun: func(req *cmds.Request, env cmds.Environment) error {
		quiet, _ := req.Options[quietOptionName].(bool)
		quieter, _ := req.Options[quieterOptionName].(bool)
		quiet = quiet || quieter

		silent, _ := req.Options[silentOptionName].(bool)

		if quiet || silent {
			return nil
		}

		// ipfs cli progress bar defaults to true unless quiet or silent is used
		_, found := req.Options[progressOptionName].(bool)
		if !found {
			req.Options[progressOptionName] = true
		}

		return nil
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		progress, _ := req.Options[progressOptionName].(bool)
		trickle, _ := req.Options[trickleOptionName].(bool)
		wrap, _ := req.Options[wrapOptionName].(bool)
		hash, _ := req.Options[onlyHashOptionName].(bool)
		silent, _ := req.Options[silentOptionName].(bool)
		chunker, _ := req.Options[chunkerOptionName].(string)
		dopin, _ := req.Options[pinOptionName].(bool)
		rawblks, rbset := req.Options[rawLeavesOptionName].(bool)
		nocopy, _ := req.Options[noCopyOptionName].(bool)
		fscache, _ := req.Options[fstoreCacheOptionName].(bool)
		cidVer, cidVerSet := req.Options[cidVersionOptionName].(int)
		hashFunStr, _ := req.Options[hashOptionName].(string)
		inline, _ := req.Options[inlineOptionName].(bool)
		inlineLimit, _ := req.Options[inlineLimitOptionName].(int)
		toFilesStr, toFilesSet := req.Options[toFilesOptionName].(string)

		hashFunCode, ok := mh.Names[strings.ToLower(hashFunStr)]
		if !ok {
			return fmt.Errorf("unrecognized hash function: %q", strings.ToLower(hashFunStr))
		}

		enc, err := cmdenv.GetCidEncoder(req)
		if err != nil {
			return err
		}

		toadd := req.Files
		if wrap {
			toadd = files.NewSliceDirectory([]files.DirEntry{
				files.FileEntry("", req.Files),
			})
		}

		opts := []options.UnixfsAddOption{
			options.Unixfs.Hash(hashFunCode),

			options.Unixfs.Inline(inline),
			options.Unixfs.InlineLimit(inlineLimit),

			options.Unixfs.Chunker(chunker),

			options.Unixfs.Pin(dopin),
			options.Unixfs.HashOnly(hash),
			options.Unixfs.FsCache(fscache),
			options.Unixfs.Nocopy(nocopy),

			options.Unixfs.Progress(progress),
			options.Unixfs.Silent(silent),
		}

		if cidVerSet {
			opts = append(opts, options.Unixfs.CidVersion(cidVer))
		}

		if rbset {
			opts = append(opts, options.Unixfs.RawLeaves(rawblks))
		}

		if trickle {
			opts = append(opts, options.Unixfs.Layout(options.TrickleLayout))
		}

		opts = append(opts, nil) // events option placeholder

		ipfsNode, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}
		var added int
		var fileAddedToMFS bool
		addit := toadd.Entries()
		for addit.Next() {
			_, dir := addit.Node().(files.Directory)
			errCh := make(chan error, 1)
			events := make(chan interface{}, adderOutChanSize)
			opts[len(opts)-1] = options.Unixfs.Events(events)

			go func() {
				var err error
				defer close(events)
				pathAdded, err := api.Unixfs().Add(req.Context, addit.Node(), opts...)
				if err != nil {
					errCh <- err
					return
				}

				// creating MFS pointers when optional --to-files is set
				if toFilesSet {
					if toFilesStr == "" {
						toFilesStr = "/"
					}
					toFilesDst, err := checkPath(toFilesStr)
					if err != nil {
						errCh <- fmt.Errorf("%s: %w", toFilesOptionName, err)
						return
					}
					dstAsDir := toFilesDst[len(toFilesDst)-1] == '/'

					if dstAsDir {
						mfsNode, err := mfs.Lookup(ipfsNode.FilesRoot, toFilesDst)
						// confirm dst exists
						if err != nil {
							errCh <- fmt.Errorf("%s: MFS destination directory %q does not exist: %w", toFilesOptionName, toFilesDst, err)
							return
						}
						// confirm dst is a dir
						if mfsNode.Type() != mfs.TDir {
							errCh <- fmt.Errorf("%s: MFS destination %q is not a directory", toFilesOptionName, toFilesDst)
							return
						}
						// if MFS destination is a dir, append filename to the dir path
						toFilesDst += gopath.Base(addit.Name())
					}

					// error if we try to overwrite a preexisting file destination
					if fileAddedToMFS && !dstAsDir {
						errCh <- fmt.Errorf("%s: MFS destination is a file: only one entry can be copied to %q", toFilesOptionName, toFilesDst)
						return
					}

					_, err = mfs.Lookup(ipfsNode.FilesRoot, gopath.Dir(toFilesDst))
					if err != nil {
						errCh <- fmt.Errorf("%s: MFS destination parent %q %q does not exist: %w", toFilesOptionName, toFilesDst, gopath.Dir(toFilesDst), err)
						return
					}

					var nodeAdded ipld.Node
					nodeAdded, err = api.Dag().Get(req.Context, pathAdded.RootCid())
					if err != nil {
						errCh <- err
						return
					}
					err = mfs.PutNode(ipfsNode.FilesRoot, toFilesDst, nodeAdded)
					if err != nil {
						errCh <- fmt.Errorf("%s: cannot put node in path %q: %w", toFilesOptionName, toFilesDst, err)
						return
					}
					fileAddedToMFS = true
				}
				errCh <- err
			}()

			for event := range events {
				output, ok := event.(*coreiface.AddEvent)
				if !ok {
					return errors.New("unknown event type")
				}

				h := ""
				if (output.Path != path.ImmutablePath{}) {
					h = enc.Encode(output.Path.RootCid())
				}

				if !dir && addit.Name() != "" {
					output.Name = addit.Name()
				} else {
					output.Name = gopath.Join(addit.Name(), output.Name)
				}

				if err := res.Emit(&AddEvent{
					Name:  output.Name,
					Hash:  h,
					Bytes: output.Bytes,
					Size:  output.Size,
				}); err != nil {
					return err
				}
			}

			if err := <-errCh; err != nil {
				return err
			}
			added++
		}

		if addit.Err() != nil {
			return addit.Err()
		}

		if added == 0 {
			return fmt.Errorf("expected a file argument")
		}

		return nil
	},
	PostRun: cmds.PostRunMap{
		cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
			sizeChan := make(chan int64, 1)
			outChan := make(chan interface{})
			req := res.Request()

			// Could be slow.
			go func() {
				size, err := req.Files.Size()
				if err != nil {
					log.Warnf("error getting files size: %s", err)
					// see comment above
					return
				}

				sizeChan <- size
			}()

			progressBar := func(wait chan struct{}) {
				defer close(wait)

				quiet, _ := req.Options[quietOptionName].(bool)
				quieter, _ := req.Options[quieterOptionName].(bool)
				quiet = quiet || quieter

				progress, _ := req.Options[progressOptionName].(bool)

				var bar *pb.ProgressBar
				if progress {
					bar = pb.New64(0).SetUnits(pb.U_BYTES)
					bar.ManualUpdate = true
					bar.ShowTimeLeft = false
					bar.ShowPercent = false
					bar.Output = os.Stderr
					bar.Start()
				}

				lastFile := ""
				lastHash := ""
				var totalProgress, prevFiles, lastBytes int64

			LOOP:
				for {
					select {
					case out, ok := <-outChan:
						if !ok {
							if quieter {
								fmt.Fprintln(os.Stdout, lastHash)
							}

							break LOOP
						}
						output := out.(*AddEvent)
						if len(output.Hash) > 0 {
							lastHash = output.Hash
							if quieter {
								continue
							}

							if progress {
								// clear progress bar line before we print "added x" output
								fmt.Fprintf(os.Stderr, "\033[2K\r")
							}
							if quiet {
								fmt.Fprintf(os.Stdout, "%s\n", output.Hash)
							} else {
								fmt.Fprintf(os.Stdout, "added %s %s\n", output.Hash, cmdenv.EscNonPrint(output.Name))
							}

						} else {
							if !progress {
								continue
							}

							if len(lastFile) == 0 {
								lastFile = output.Name
							}
							if output.Name != lastFile || output.Bytes < lastBytes {
								prevFiles += lastBytes
								lastFile = output.Name
							}
							lastBytes = output.Bytes
							delta := prevFiles + lastBytes - totalProgress
							totalProgress = bar.Add64(delta)
						}

						if progress {
							bar.Update()
						}
					case size := <-sizeChan:
						if progress {
							bar.Total = size
							bar.ShowPercent = true
							bar.ShowBar = true
							bar.ShowTimeLeft = true
						}
					case <-req.Context.Done():
						// don't set or print error here, that happens in the goroutine below
						return
					}
				}

				if progress && bar.Total == 0 && bar.Get() != 0 {
					bar.Total = bar.Get()
					bar.ShowPercent = true
					bar.ShowBar = true
					bar.ShowTimeLeft = true
					bar.Update()
				}
			}

			if e := res.Error(); e != nil {
				close(outChan)
				return e
			}

			wait := make(chan struct{})
			go progressBar(wait)

			defer func() { <-wait }()
			defer close(outChan)

			for {
				v, err := res.Next()
				if err != nil {
					if err == io.EOF {
						return nil
					}

					return err
				}

				select {
				case outChan <- v:
				case <-req.Context.Done():
					return req.Context.Err()
				}
			}
		},
	},
	Type: AddEvent{},
}

```