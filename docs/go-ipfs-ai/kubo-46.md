# go-ipfs 源码解析 46

# `plugin/daemon.go`

这段代码定义了一个名为`PluginDaemon`的接口，用于定义一个用于运行插件的`Plugin`实例。这个插件可以运行在命令行或系统启动时，它通过`CoreAPI`实现对底层系统的访问。

具体来说，这个插件插件 daemon 进程，一旦开始 运行，它将调用`Plugin`接口的`Start`方法， passing in a reference to the `coreiface` package的`CoreAPI`类型。这个`CoreAPI`类型提供了一些与底层系统交互的接口，通过这些接口，插件可以访问各种数据和功能。

然后，插件会通过`PluginDaemon`接口的`Plugin`方法向`CoreAPI`发送请求，并执行相应的操作，完成自己的任务。

最后，插件 daemon 进程可以通过`CoreAPI`提供的API来访问底层系统，如`/ethereum/v1/accounts`等。


```go
package plugin

import (
	coreiface "github.com/ipfs/boxo/coreiface"
)

// PluginDaemon is an interface for daemon plugins. These plugins will be run on
// the daemon and will be given access to an implementation of the CoreAPI.
type PluginDaemon interface {
	Plugin

	Start(coreiface.CoreAPI) error
}

```

# `plugin/daemoninternal.go`

这段代码定义了一个名为“plugin”的包。这个包包含一个名为“PluginDaemonInternal”的接口类型。这个接口类型定义了一个名为“PluginDaemonInternal”的类型，以及一个名为“Plugin”的接口类型。

“PluginDaemonInternal”接口类型表示一个daemon插件，这个插件将提供一个IpfsNode的直接访问。通过使用“PluginDaemonInternal”接口，可以创建一个自己的IpfsNode，并将它作为参数传递给“Start”方法。如果“Start”方法返回一个错误，将会返回一个错误对象。

通过实现“PluginDaemonInternal”接口，可以创建自己的daemon插件，并将它与IpfsNode进行交互。这个插件可以用来实现自己的Ipfs相关操作，例如创建或删除IpfsNode、获取IpfsClient等。


```go
package plugin

import "github.com/ipfs/kubo/core"

// PluginDaemonInternal is an interface for daemon plugins. These plugins will be run on
// the daemon and will be given a direct access to the IpfsNode.
//
// Note: PluginDaemonInternal is considered internal and no guarantee is made concerning
// the stability of its API. If you can, use PluginAPI instead.
type PluginDaemonInternal interface {
	Plugin

	Start(*core.IpfsNode) error
}

```

# `plugin/datastore.go`

这段代码定义了一个名为`PluginDatastore`的接口，用于实现添加对不同数据存储器的处理器。它实现了`PluginDatastore`接口，提供了`Plugin`和`DatastoreTypeName`以及`DatastoreConfigParser`两个方法。

具体来说，这段代码定义了一个名为`PluginDatastore`的接口，其中`Plugin`表示该接口的实现者需要实现一个名为`Plugin`的函数，用于定义自己的数据存储器和处理器。`DatastoreTypeName`表示数据存储器类型名称，用于指定数据存储器类型。`DatastoreConfigParser`表示用于解析配置文件的函数，它从映射中获取配置信息，并返回一个`fsrepo.ConfigFromMap`的实例，用于将配置信息应用到数据存储器上。

为了使用这个`PluginDatastore`接口，可以创建一个实现`PluginDatastore`的类，并使用它的`Plugin`和`DatastoreConfigParser`方法进行初始化。例如：



```go
package plugin

import (
	"github.com/ipfs/kubo/repo/fsrepo"
)

// PluginDatastore is an interface that can be implemented to add handlers for
// for different datastores.
type PluginDatastore interface {
	Plugin

	DatastoreTypeName() string
	DatastoreConfigParser() fsrepo.ConfigFromMap
}

```

# `plugin/fx.go`

这段代码定义了一个名为"plugin"的包，其中包含了一些关于如何自定义Go-IPFS应用中fx选项的接口定义。

这里使用了两个主要的库：ipfs-kubo和go-uber。ipfs-kubo是一个用于管理Kubernetes对象的库，而go-uber是一个用于函数式编程的工具库。

具体来说，这段代码实现了一个自定义的fx选项函数，允许用户通过调用go-ipfs.Init()函数来设置自定义选项。这些选项可以包括设置Kubernetes对象的选项，如应用程序的名称、配置文件、安全性等。

通过导入ipfs-kubo和go-uber库，以及定义一个名为"PluginFx"的接口，该接口允许函数式编程的方式来设置这些选项。在函数式编程风格中，这种方法是非常自然的，因为它允许用户通过组合可重用组件的方式来构建自己的应用程序。

总之，这段代码提供了一个简单而灵活的方法，让用户可以自定义Go-IPFS应用的选项。


```go
package plugin

import (
	"github.com/ipfs/kubo/core"
	"go.uber.org/fx"
)

// PluginFx can be used to customize the fx options passed to the go-ipfs app when it is initialized.
//
// This is invasive and depends on internal details such as the structure of the dependency graph,
// so breaking changes might occur between releases.
// So it's recommended to keep this as simple as possible, and stick to overriding interfaces
// with fx.Replace() or fx.Decorate().
//
// The returned options become the complete array of options passed to fx.
```

这段代码定义了一个名为PluginFx的接口，其实现了FXOptions的Option方法，用于返回一个可选的选项组合。

具体来说，这个接口 PluginFx 的实现包括以下几个方法：

- Plugin：这个接口中的第一个方法，使用Options的实现，接收一个核心的FXNodeInfo对象，并返回一个包含多个fx.Option选项的Native Furniturefx对象。
- Options：这个接口中的第二个方法，使用Options的实现，接收一个核心的FXNodeInfo对象，并返回一个包含多个fx.Option选项的Native Furniturefx对象，如果设置成功则返回None，否则返回一个非Native Furniturefx对象。

通过 PluginFx 的 Plugin 和 Options 方法，你可以在使用 Native Furniturefx 时定义自己的选项，并通过这个接口来返回这些选项。


```go
// Generally you'll want to append additional options to NodeInfo.FXOptions and return that.
type PluginFx interface {
	Plugin
	Options(core.FXNodeInfo) ([]fx.Option, error)
}

```

# `plugin/ipld.go`

这段代码定义了一个名为`PluginIPLD`的接口，表示一个可以注册IPLD(即IPLD编码器)处理程序的插件。

注册过程中，会创建一个`multicodec.Registry`实例，用于存储注册的编码器。如果注册成功，将返回一个`error`。

该接口的实现者将继承自`Plugin`接口，实现`Plugin`接口中的`Register`方法。如果实现者创建了一个`PluginIPLD`实例并注册，将可以使用`register`方法将IPLD编码器注册到注册表中。

因此，该代码定义了一个用于添加IPLD编码器处理程序的插件接口，并为实现者提供了注册IPLD编码器到注册表的方法。


```go
package plugin

import (
	multicodec "github.com/ipld/go-ipld-prime/multicodec"
)

// PluginIPLD is an interface that can be implemented to add handlers for
// for different IPLD codecs.
type PluginIPLD interface {
	Plugin

	Register(multicodec.Registry) error
}

```

# `plugin/plugin.go`

这段代码定义了一个名为"plugin"的环境类，用于在IPFS插件的初始化时传递给插件的配置环境。

该Environment结构体包含两个字段：Repo,IPFS仓库的仓库路径，Config，表示插件的配置，如果通过用户配置的"plugin-name"插件，则可以从该插件的配置文件中读取。

如果通过Go-IPFS的配置，则可以在该插件的配置文件中手动指定该Environment的Config字段，此时该字段将是一个JSON-like的对象，可能是由Go-IPFS的"encoding/json"库中的Unmarshal函数进行解析得到的。

最后，该插件的代码中定义了一个名为"Plugin"的接口，该接口定义了插件的API访问点，以及插件启动时执行的操作。


```go
package plugin

// Environment is the environment passed into the plugin on init.
type Environment struct {
	// Path to the IPFS repo.
	Repo string

	// The plugin's config, if specified in the
	// Plugins.Plugins["plugin-name"].Config field of the user's go-ipfs
	// config. See docs/plugins.md for details.
	//
	// This is an arbitrary JSON-like object unmarshaled into an interface{}
	// according to https://golang.org/pkg/encoding/json/#Unmarshal.
	Config interface{}
}

```

这段代码定义了一个名为 "Plugin" 的接口，用于所有类型的 go-ipfs 插件。

任何实现了该接口的插件都应该具有以下名称和版本：

- 名称应该返回插件的唯一名称。
- 版本应该返回插件的当前版本。

插件还可以实现 io.Closer，以在卸载时具有终止步骤，但这是可选的。

该接口的实现者必须遵循以下规则：

- 在初始化时，插件应该将其环境包含的 IPFS 仓库的路径和插件配置传递给一个名为 "env" 的 *Environment 类型的参数。

初始化插件时，如果插件使用了 io.Closer，它应该在插件卸载时执行终止步骤。


```go
// Plugin is the base interface for all kinds of go-ipfs plugins
// It will be included in interfaces of different Plugins
//
// Optionally, Plugins can implement io.Closer if they want to
// have a termination step when unloading.
type Plugin interface {
	// Name should return unique name of the plugin
	Name() string

	// Version returns current version of the plugin
	Version() string

	// Init is called once when the Plugin is being loaded
	// The plugin is passed an environment containing the path to the
	// (possibly uninitialized) IPFS repo and the plugin's config.
	Init(env *Environment) error
}

```

# `plugin/tracer.go`

这段代码定义了一个名为`PluginTracer`的接口，用于实现一个插件，可以在父程序中添加跟踪器。

具体来说，该代码实现了一个插件，这个插件可以被用来在父程序中添加跟踪器。插件的实现需要使用`opentracing`库，因此导入了`opentracing-go`包。

另外，该插件还定义了一个名为`PluginTracer`的类型，这个类型包含一个名为`Plugin`的类型，以及一个名为`InitTracer`的函数。

`Plugin`类型表示插件的接口，这个接口中定义了一个`PluginTracer`类型，用于将插件的实现与抽象接口匹配。

`InitTracer`函数是插件的一个必须的函数，用于初始化跟踪器。该函数的实现将在创建跟踪器实例时执行，初始化跟踪器并将之添加到分配的上下文中。如果函数在执行时出现错误，将会返回一个`opentracing.IgnoreCriteria`类型的错误，而不是一个具体的错误信息。


```go
package plugin

import (
	"github.com/opentracing/opentracing-go"
)

// PluginTracer is an interface that can be implemented to add a tracer.
type PluginTracer interface {
	Plugin

	InitTracer() (opentracing.Tracer, error)
}

```

# `plugin/loader/loader.go`

这段代码是一个 Go 语言编写的库，名为 "loader"，旨在为在 IPFS(InterPlanetary File System) 系统中使用 Kubernetes 集群时提供一种简单的方法。它实现了以下主要功能：

1. 配置文件解析：该库支持 JSON,YAML 和 path 格式的配置文件。通过读取这些文件来设置默认的加载器选项和存储库根目录。

2. 文件系统挂载：当读取到包含子目录的文件系统时，该库将其挂载到 IPFS 客户端上，并将其子目录和文件记录为 IPFS 资源。

3. 资源提取：该库提供了一些函数来提取 IPFS 资源，例如提取一个文件的所有内容并输出为 JSON 格式。

4. 日志记录：该库使用 Go 标准库中的 logging 包来记录操作系统的操作日志。同时，它还使用 Go 标准库中的 opentracing 包来记录操作系统的跟踪信息。

5. 插件支持：该库支持通过插件扩展它的功能。它提供了一个插件机制，允许用户通过编写自定义插件来定制 loader 的行为。

6. FsRepository：该库的 "fsrepo" 包通过文件系统提供对 IPFS 资源文件夹的访问。它支持多线程并发访问，并使用一个 Redis 数据库来保存已下载的文件信息，以避免在高负载情况下对文件的频繁访问。

7. Kubernetes 支持：该库假设用户正在使用一个支持 Kubernetes 的 IPFS 客户端。它提供了一些 Kubernetes 资源类型，例如 Deployment,Service 和 ConfigMap，可以通过 Loader 进行拉取和推送。


```go
package loader

import (
	"encoding/json"
	"fmt"
	"io"
	"os"
	"path/filepath"
	"runtime"
	"strings"

	config "github.com/ipfs/kubo/config"
	"github.com/ipld/go-ipld-prime/multicodec"

	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/coreapi"
	plugin "github.com/ipfs/kubo/plugin"
	fsrepo "github.com/ipfs/kubo/repo/fsrepo"

	logging "github.com/ipfs/go-log"
	opentracing "github.com/opentracing/opentracing-go"
)

```

该代码定义了一个名为 `Preload` 的函数，用于在初始化完成时将预加载的插件添加到 `preloadPlugins` 数组中。

具体来说，该函数接收一个或多个 `plugin.Plugin` 类型的参数，并将它们添加到 `preloadPlugins` 数组中。这个函数只在初始化完成时调用一次，以确保只添加一次插件。

此外，代码中还定义了一个名为 `log` 的变量，用于记录加载器中的日志信息。

另外，定义了一个名为 `loadPluginFunc` 的函数，用于加载指定的插件。该函数接收一个字符串参数，返回一个包含多个 `plugin.Plugin` 类型的结果，或者是错误信息。如果所支持的操作系统不支持指定的插件，函数将返回错误信息。

最后，定义了一个名为 `loaderState` 的枚举类型，用于记录加载器的状态。


```go
var preloadPlugins []plugin.Plugin

// Preload adds one or more plugins to the preload list. This should _only_ be called during init.
func Preload(plugins ...plugin.Plugin) {
	preloadPlugins = append(preloadPlugins, plugins...)
}

var log = logging.Logger("plugin/loader")

var loadPluginFunc = func(string) ([]plugin.Plugin, error) {
	return nil, fmt.Errorf("unsupported platform %s", runtime.GOOS)
}

type loaderState int

```

这段代码定义了一个名为 `loaderState` 的结构体，它包含了七个不同的状态，用数字 `iota` 作为键，每个状态都有一个默认的名称。这些状态代表了加载器的不同阶段，例如 `loaderLoading` 表示加载器正在加载数据，`loaderInitializing` 表示加载器正在准备加载数据，`loaderInitialized` 表示加载器已经准备好加载数据，`loaderInjecting` 表示数据正在被注入到加载器中，`loaderInjected` 表示数据已经完全注入到加载器中，`loaderStarting` 表示加载器正在启动，`loaderStarted` 表示加载器已经启动并正在加载数据，`loaderClosing` 表示加载器正在关闭，`loaderClosed` 表示加载器已经关闭并释放了所有的资源，`loaderFailed` 表示加载器在加载数据时遇到了错误。

每个 `loaderState` 值都有一个对应的 `String()` 函数返回，例如 `loaderLoading` 对应的函数返回字符串 `"Loading"`，`loaderFailed` 对应的函数返回字符串 `"Failed"`。这些函数可以通过 `fmt.Printf` 函数来输出，例如：

fmt.Printf("%v\n", loaderState)

这样就可以在控制台输出加载器的不同状态了。


```go
const (
	loaderLoading loaderState = iota
	loaderInitializing
	loaderInitialized
	loaderInjecting
	loaderInjected
	loaderStarting
	loaderStarted
	loaderClosing
	loaderClosed
	loaderFailed
)

func (ls loaderState) String() string {
	switch ls {
	case loaderLoading:
		return "Loading"
	case loaderInitializing:
		return "Initializing"
	case loaderInitialized:
		return "Initialized"
	case loaderInjecting:
		return "Injecting"
	case loaderInjected:
		return "Injected"
	case loaderStarting:
		return "Starting"
	case loaderStarted:
		return "Started"
	case loaderClosing:
		return "Closing"
	case loaderClosed:
		return "Closed"
	case loaderFailed:
		return "Failed"
	default:
		return "Unknown"
	}
}

```

这是一个用Go编写的插件加载器（Plugin Loader）的实例。插件加载器的作用是管理和加载已加载的插件。以下是插件加载器的一些主要函数和成员变量：

1. `loadPlugins`：加载指定的插件。这个函数接受一个Load和LoadDirectory选项，并在函数内部使用这些选项加载插件。

2. `init`：运行插件加载器的初始化逻辑。这个函数在创建插件加载器实例时调用。

3. `inject`：注册插件。这个函数接受一个Plugin选项，并在插件加载器启动时注册插件。

4. `start`：启动插件。这个函数可以用来启动所有的插件，并按照插件加载器配置文件中的顺序加载插件。

5. `close`：关闭插件。这个函数会在插件加载器关闭时调用，无论当前正在加载的插件数量。

6. `plugins`：定义插件加载器的插件列表。

7. `started`：定义已经启动的插件列表。

8. `config`：定义插件加载器的配置文件。

9. `repo`：定义插件加载器要下载的repo地址。

此插件加载器使用了一个结构体（Plugin Loader struct）来保存所有的插件信息。`state`变量保存了加载器的当前状态，包括加载的插件列表、注册的插件列表和配置文件。`plugins`和`started`变量分别保存了当前已加载的插件和已启动的插件列表。而`config`变量则包含了插件加载器的配置文件内容。

通过使用`LoadAndLoadDirectory`和`Plugin`类型的函数，可以实现插件加载器的主要功能。在初始化函数中，加载器会读取任何需要的插件，并将它们注册到实例中。然后，可以调用`start`函数来启动插件加载器，并可以选择性地调用`start`函数来加载插件。调用`close`函数可以关闭插件加载器。


```go
// PluginLoader keeps track of loaded plugins.
//
// To use:
//  1. Load any desired plugins with Load and LoadDirectory. Preloaded plugins
//     will automatically be loaded.
//  2. Call Initialize to run all initialization logic.
//  3. Call Inject to register the plugins.
//  4. Optionally call Start to start plugins.
//  5. Call Close to close all plugins.
type PluginLoader struct {
	state   loaderState
	plugins []plugin.Plugin
	started []plugin.Plugin
	config  config.Plugins
	repo    string
}

```

这段代码定义了一个名为NewPluginLoader的函数，用于创建一个新的插件加载器。

函数接收一个字符串参数repo，表示要加载的插件仓库的路径。函数创建一个名为PluginLoader的实例，该实例包含一个plugins数组和一个repo参数。

函数首先检查repo参数是否为空，如果是，则加载插件的过程将在函数内部执行。否则，函数将加载插件配置文件，如果加载成功，将加载插件到插件加载器中。

接下来，函数遍历preloadPlugins数组，加载每个插件。如果加载任何插件时出现错误，函数将返回 nil，并输出错误信息。

最后，函数加载整个repo目录中的插件，如果加载成功，将返回PluginLoader实例，否则返回 nil。


```go
// NewPluginLoader creates new plugin loader.
func NewPluginLoader(repo string) (*PluginLoader, error) {
	loader := &PluginLoader{plugins: make([]plugin.Plugin, 0, len(preloadPlugins)), repo: repo}
	if repo != "" {
		switch plugins, err := readPluginsConfig(repo, config.DefaultConfigFile); {
		case err == nil:
			loader.config = plugins
		case os.IsNotExist(err):
		default:
			return nil, err
		}
	}

	for _, v := range preloadPlugins {
		if err := loader.Load(v); err != nil {
			return nil, err
		}
	}

	if err := loader.LoadDirectory(filepath.Join(repo, "plugins")); err != nil {
		return nil, err
	}
	return loader, nil
}

```

这段代码的主要作用是读取IPFS配置文件中的插件部分，并只读取插件部分，以避免读取到其他部分。这样可以让我们对插件进行修改，而不会影响其他部分。

具体来说，代码首先定义了一个名为`cfg`的结构体，该结构体包含一个名为`Plugins`的`config.Plugins`字段。然后，定义了一个`readPluginsConfig`函数，该函数接收一个`repoRoot`和一个`userConfigFile`参数。函数首先定义了一个`cfg`结构体，然后使用`config.Filename`函数来获取配置文件路径。接着，使用`os.Open`函数打开配置文件，并使用`json.NewDecoder`函数将JSON编码的配置文件内容读取到`cfg`结构体中。最后，函数返回`cfg.Plugins`和nil，分别表示读取到的插件和没有错误。


```go
// readPluginsConfig reads the Plugins section of the IPFS config, avoiding
// reading anything other than the Plugin section. That way, we're free to
// make arbitrary changes to all _other_ sections in migrations.
func readPluginsConfig(repoRoot string, userConfigFile string) (config.Plugins, error) {
	var cfg struct {
		Plugins config.Plugins
	}

	cfgPath, err := config.Filename(repoRoot, userConfigFile)
	if err != nil {
		return config.Plugins{}, err
	}

	cfgFile, err := os.Open(cfgPath)
	if err != nil {
		return config.Plugins{}, err
	}
	defer cfgFile.Close()

	err = json.NewDecoder(cfgFile).Decode(&cfg)
	if err != nil {
		return config.Plugins{}, err
	}

	return cfg.Plugins, nil
}

```

这两函数是JavaScript中的函数指针，定义了一个PluginLoader的实现。它们的作用是确保PluginLoader在执行特定 transition 状态时，始终满足预期的加载器状态。

函数声明中，第一个函数名为func，有2个参数，第一个参数是一个指向PluginLoader的引用，第二个参数是一个整型，表示加载器当前的状态。第二个函数名为assertState，有2个参数，第一个参数是一个整型，表示期望的加载器状态，第二个参数是加载器当前的状态。函数返回一个整型，表示不产生任何错误，或者产生一个与第一个参数中状态不同的错误。

assertState函数的作用是检查加载器当前的状态是否与期望的状态相同。如果两个状态不同，函数将返回一个错误。如果两个状态相同，函数将返回一个 nil 值。

transition函数的作用是在从当前状态加载到目标状态时，确保加载器始终满足预期的加载器状态。函数有两个参数，第一个参数是从当前状态加载器需要的负载器状态，第二个参数是目标状态加载器需要的加载器状态。函数返回一个错误，如果发生错误则返回，否则不会返回任何错误。

这两个函数一起使用，确保在执行transition函数从当前状态加载器到目标状态时，始终使用正确的加载器状态，从而避免可能出现的错误。


```go
func (loader *PluginLoader) assertState(state loaderState) error {
	if loader.state != state {
		return fmt.Errorf("loader state must be %s, was %s", state, loader.state)
	}
	return nil
}

func (loader *PluginLoader) transition(from, to loaderState) error {
	if err := loader.assertState(from); err != nil {
		return err
	}
	loader.state = to
	return nil
}

```

这段代码定义了一个名为 PluginLoader 的 Plugin加载器，它的作用是将一个插件加载到插件加载器中。

具体来说，代码首先检查加载器是否正在加载中，如果是，就检查加载器的状态是否正确，如果不是，就返回一个错误。然后，代码遍历加载器中的所有插件，检查插件是否与要加载的插件名称相同，如果是，就返回一个错误信息。如果不是，就继续遍历。

接下来，代码检查配置中是否启用了要加载的插件，如果启用了，但是插件已被禁用，就输出一条日志信息并返回一个 nil 值。最后，代码将加载器中的插件列表添加到加载器中，并返回一个 nil 值。

因此，这段代码的作用是将一个插件加载到插件加载器中，并允许在插件加载器中加载已禁用但仍然存在于配置中的插件。


```go
// Load loads a plugin into the plugin loader.
func (loader *PluginLoader) Load(pl plugin.Plugin) error {
	if err := loader.assertState(loaderLoading); err != nil {
		return err
	}

	name := pl.Name()

	for _, p := range loader.plugins {
		if p.Name() == name {
			// plugin is already loaded
			return fmt.Errorf(
				"plugin: %s, is duplicated in version: %s, "+
					"while trying to load dynamically: %s",
				name, p.Version(), pl.Version())
		}
	}

	if loader.config.Plugins[name].Disabled {
		log.Infof("not loading disabled plugin %s", name)
		return nil
	}
	loader.plugins = append(loader.plugins, pl)
	return nil
}

```

这段代码定义了一个名为`LoadDirectory`的函数，属于一个名为`PluginLoader`的接口的实现。这个函数的作用是加载一个目录中的插件到插件加载器中。

函数的参数包括一个名为`pluginDir`的字符串，表示插件目录，以及一个名为`loader`的指针，用于表示插件加载器。函数返回一个错误对象，如果在加载目录过程中出现错误，该错误对象将被抛出。

函数首先检查`loader`是否处于加载状态，如果当前不处于加载状态，则会执行`assertState`函数来确保`loader`处于加载状态。如果当前已经处于加载状态，则会执行以下操作：

1. 从`pluginDir`中加载动态插件。
2. 对于加载到的每个插件，调用`loader.Load`函数加载它。
3. 在加载过程中，遇到错误时返回一个错误对象。

函数的具体实现可以看作是在插件加载器初始化过程中，加载所有可用的插件并将其注册到加载器中。这样，当加载器需要加载插件时，它可以从插件目录中的所有插件中选择一个或多个来加载，而不会因为某些插件不存在而崩溃。


```go
// LoadDirectory loads a directory of plugins into the plugin loader.
func (loader *PluginLoader) LoadDirectory(pluginDir string) error {
	if err := loader.assertState(loaderLoading); err != nil {
		return err
	}
	newPls, err := loadDynamicPlugins(pluginDir)
	if err != nil {
		return err
	}

	for _, pl := range newPls {
		if err := loader.Load(pl); err != nil {
			return err
		}
	}
	return nil
}

```

这段代码定义了一个名为`loadDynamicPlugins`的函数，它接受一个名为`pluginDir`的静态参数。函数的作用是加载插件，并在插件目录中查找非目录、非可执行文件，尝试加载它们。

该函数首先检查插件目录是否存在，如果不存在，则返回两个空数组。如果目录存在，则继续下一步。

函数使用`filepath.Walk`函数递归遍历插件目录，并处理每一个文件。如果遍历过程中遇到非目录文件，函数会打印警告信息。如果遍历过程中遇到不可执行文件，函数会打印错误信息并返回。

对于每一个可执行文件，函数调用一个名为`loadPluginFunc`的内部函数。如果这个函数返回错误，函数将返回该错误。如果`loadPluginFunc`返回一个新`plugin.Plugin`对象，函数将添加到`plugins`数组中。

最后，函数返回包含所有插件的数组，或者是错误。


```go
func loadDynamicPlugins(pluginDir string) ([]plugin.Plugin, error) {
	_, err := os.Stat(pluginDir)
	if os.IsNotExist(err) {
		return nil, nil
	}
	if err != nil {
		return nil, err
	}

	var plugins []plugin.Plugin

	err = filepath.Walk(pluginDir, func(fi string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}
		if info.IsDir() {
			if fi != pluginDir {
				log.Warnf("found directory inside plugins directory: %s", fi)
			}
			return nil
		}

		if info.Mode().Perm()&0o111 == 0 {
			// file is not executable let's not load it
			// this is to prevent loading plugins from for example non-executable
			// mounts, some /tmp mounts are marked as such for security
			log.Errorf("non-executable file in plugins directory: %s", fi)
			return nil
		}

		if newPlugins, err := loadPluginFunc(fi); err == nil {
			plugins = append(plugins, newPlugins...)
		} else {
			return fmt.Errorf("loading plugin %s: %s", fi, err)
		}
		return nil
	})

	return plugins, err
}

```

这段代码是一个 plugin 加载器的初始化函数。其作用是初始化所有的加载器，并确保加载器已加载了所有的插件。

具体来说，代码首先检查加载器是否已加载完成，如果已加载完但期间发生错误，则返回错误。然后，代码遍历加载器加载到的插件，对于每个插件，代码调用插件的初始化函数，以确保插件已正确初始化。

如果初始化过程中发生错误，代码将状态设置为加载器已失败，并返回错误。否则，代码将状态设置为加载器已成功初始化，可以继续加载插件。

整个函数的实现重点是确保所有的加载器都已正确加载和初始化，以便 plugins 可以正常使用。


```go
// Initialize initializes all loaded plugins.
func (loader *PluginLoader) Initialize() error {
	if err := loader.transition(loaderLoading, loaderInitializing); err != nil {
		return err
	}
	for _, p := range loader.plugins {
		err := p.Init(&plugin.Environment{
			Repo:   loader.repo,
			Config: loader.config.Plugins[p.Name()].Config,
		})
		if err != nil {
			loader.state = loaderFailed
			return err
		}
	}

	return loader.transition(loaderInitializing, loaderInitialized)
}

```

这段代码是一个二进制插件加载器，其作用是将所有插件注入到适当的子系统中。

具体来说，这段代码首先检查加载器是否已初始化，如果是，则执行加载器初始化。如果不是，则加载器将尝试将插件注入到适当的子系统中。

接下来，代码遍历加载器中的所有插件。对于每个插件，代码会尝试将其注入到适当的子系统中。如果注入过程中出现错误，则会将加载器的状态设置为加载器失败，并返回错误信息。

最终，这段代码将所有插件注入到适当的子系统中，并返回一个表示成功或失败的结果。


```go
// Inject hooks all the plugins into the appropriate subsystems.
func (loader *PluginLoader) Inject() error {
	if err := loader.transition(loaderInitialized, loaderInjecting); err != nil {
		return err
	}

	for _, pl := range loader.plugins {
		if pl, ok := pl.(plugin.PluginIPLD); ok {
			err := injectIPLDPlugin(pl)
			if err != nil {
				loader.state = loaderFailed
				return err
			}
		}
		if pl, ok := pl.(plugin.PluginTracer); ok {
			err := injectTracerPlugin(pl)
			if err != nil {
				loader.state = loaderFailed
				return err
			}
		}
		if pl, ok := pl.(plugin.PluginDatastore); ok {
			err := injectDatastorePlugin(pl)
			if err != nil {
				loader.state = loaderFailed
				return err
			}
		}
		if pl, ok := pl.(plugin.PluginFx); ok {
			err := injectFxPlugin(pl)
			if err != nil {
				loader.state = loaderFailed
				return err
			}
		}
	}

	return loader.transition(loaderInjecting, loaderInjected)
}

```

这段代码是一个名为`PluginLoader`的外部插件加载器，它的作用是启动所有长期运行的插件。具体来说，它做了以下几件事情：

1. 加载插件加载器
2. 启动插件加载器
3. 启动所有插件
4. 关闭加载器

插件加载器的作用是将插件加载器启动后，将所有插件启动顺序安排好，并且关闭加载器。

具体实现中，首先定义了一个`PluginLoader`类型的变量`loader`，然后定义了一个`Start`函数，该函数接收一个`core.IpfsNode`类型的参数`node`，接着如果错误，返回错误。在`Start`函数中，首先检查是否有错误，然后如果有错误，则关闭加载器并返回。接着，使用`transition`函数将加载器从`loaderInjected`状态转移到`loaderStarting`状态，然后再使用`coreapi.NewCoreAPI`函数获取一个`CoreAPI`实例，如果错误，返回错误。然后，开始遍历所有插件，对于每个插件，使用`plugins` slice获取插件的`PluginDaemon`类型，如果是，则使用`Start`函数启动插件。在循环过程中，将启动的插件添加到`loader.started`数组中，然后使用`transition`函数将加载器从`loaderStarting`状态转移到`loaderStarted`状态，最后使用`transition`函数将加载器从`loaderStarted`状态转移到`loaderStarted`状态，这样所有的插件都启动成功并添加到`loader.started`数中。


```go
// Start starts all long-running plugins.
func (loader *PluginLoader) Start(node *core.IpfsNode) error {
	if err := loader.transition(loaderInjected, loaderStarting); err != nil {
		return err
	}
	iface, err := coreapi.NewCoreAPI(node)
	if err != nil {
		return err
	}
	for _, pl := range loader.plugins {
		if pl, ok := pl.(plugin.PluginDaemon); ok {
			err := pl.Start(iface)
			if err != nil {
				_ = loader.Close()
				return err
			}
			loader.started = append(loader.started, pl)
		}
		if pl, ok := pl.(plugin.PluginDaemonInternal); ok {
			err := pl.Start(node)
			if err != nil {
				_ = loader.Close()
				return err
			}
			loader.started = append(loader.started, pl)
		}
	}

	return loader.transition(loaderStarting, loaderStarted)
}

```

这段代码是一个名为 `PluginLoader` 的函数，它的作用是关闭所有长运行期的插件。函数有两个状态，`loaderClosing` 表示插件正在关闭中，`loaderFailed` 表示在关闭插件时发生了错误，`loaderClosed` 表示插件已经关闭完成。

当 `loaderClosing` 状态成立时，函数不做任何操作，直接返回 `nil`。

当 `loaderFailed` 状态成立时，函数会遍历所有已启动的插件，对于每个正在运行的插件，会尝试使用该插件的 `Close` 方法关闭插件，并记录错误信息。如果关闭插件失败，函数会将错误信息添加到 `errs` 数组中，并返回一个错误信息。

当 `loaderClosed` 状态成立时，函数会将 `loaderFailed` 和 `loaderClosing` 状态中的一个设置为 `loaderClosed`，并返回 `nil`。


```go
// Close stops all long-running plugins.
func (loader *PluginLoader) Close() error {
	switch loader.state {
	case loaderClosing, loaderFailed, loaderClosed:
		// nothing to do.
		return nil
	}
	loader.state = loaderClosing

	var errs []string
	started := loader.started
	loader.started = nil
	for _, pl := range started {
		if closer, ok := pl.(io.Closer); ok {
			err := closer.Close()
			if err != nil {
				errs = append(errs, fmt.Sprintf(
					"error closing plugin %s: %s",
					pl.Name(),
					err.Error(),
				))
			}
		}
	}
	if errs != nil {
		loader.state = loaderFailed
		return fmt.Errorf(strings.Join(errs, "\n"))
	}
	loader.state = loaderClosed
	return nil
}

```

这段代码定义了三个函数分别用于在 Go应用程序中注入不同的数据存储、命令行界面（IPLD）和日志记录器（Tracer）插件。

1. injectDatastorePlugin:
该函数接收一个 PluginDatastore 类型的参数，然后将其传递给 fsrepo.AddDatastoreConfigHandler 函数。函数的作用是将 PluginDatastore 中的数据存储配置加载到应用程序中。

2. injectIPLDPlugin:
该函数接收一个 PluginIPLD 类型的参数，然后将其传递给 pl.Register 函数。函数的作用是在应用程序中注册 IPLD 插件。

3. injectTracerPlugin:
该函数接收一个 PluginTracer 类型的参数，但没有对其进行任何操作。函数的作用是提醒在将来应该使用 OpenTelemetry 收集器而不是当前的 Tracer 插件。然后在函数内部创建一个新的 Tracer 实例，并将其注册到应用程序中。


```go
func injectDatastorePlugin(pl plugin.PluginDatastore) error {
	return fsrepo.AddDatastoreConfigHandler(pl.DatastoreTypeName(), pl.DatastoreConfigParser())
}

func injectIPLDPlugin(pl plugin.PluginIPLD) error {
	return pl.Register(multicodec.DefaultRegistry)
}

func injectTracerPlugin(pl plugin.PluginTracer) error {
	log.Warn("Tracer plugins are deprecated, it's recommended to configure an OpenTelemetry collector instead.")
	tracer, err := pl.InitTracer()
	if err != nil {
		return err
	}
	opentracing.SetGlobalTracer(tracer)
	return nil
}

```

这段代码定义了一个名为 `injectFxPlugin` 的函数，接受一个名为 `pl` 的参数 `plugin.PluginFx`。

这个函数的作用是注册一个名为 `fx` 的插件，该插件使用了 `plugin.PluginFx` 选项。具体来说，函数首先调用 `core.RegisterFXOptionFunc` 来注册 `fx` 插件的选项功能。然后，它返回一个 `nil` 表示成功或错误，失败时返回 `core.NewError`。


```go
func injectFxPlugin(pl plugin.PluginFx) error {
	core.RegisterFXOptionFunc(pl.Options)
	return nil
}

```

# `plugin/loader/load_nocgo.go`

这段代码是一个Go语言编写的Linux、Darwin和FreeBSD系统的构建加载器。它包含三个构建函数，分别用于在没有CGO、noplugin和免费BSD等插件的情况下构建系统。

首先，在`init`函数中，加载器将使用`nocgoLoadPlugin`函数加载Google开发的nocgo构建器。

然后，通过使用`build`函数，分别构建没有CGO、noplugin和免费BSD的系统。这里，`!cgo`表示不使用CGO插件，`!noplugin`表示不使用noplugin插件，`linux`、`darwin`和`freebsd`则分别表示构建针对的操作系统。`freebsd`是一个示例，这里表示为免费BSD系统构建。

最后，通过运行这些构建函数，加载器将构建系统。


```go
//go:build !cgo && !noplugin && (linux || darwin || freebsd)
// +build !cgo
// +build !noplugin
// +build linux darwin freebsd

package loader

import (
	"errors"

	iplugin "github.com/ipfs/kubo/plugin"
)

func init() {
	loadPluginFunc = nocgoLoadPlugin
}

```

此函数的作用是加载一个名为"nocgoLoadPlugin"的插件，并返回插件列表和错误。它接受一个字符串参数"fi"，表示插件文件的接口。函数返回 nil 和一个错误对象 errors.New，当插件无法使用cgo支持时，错误对象将包含错误信息。


```go
func nocgoLoadPlugin(fi string) ([]iplugin.Plugin, error) {
	return nil, errors.New("not built with cgo support")
}

```

# `plugin/loader/load_noplugin.go`

这段代码定义了一个名为“loader”的包，其中包含一个名为“init”的函数。

在函数内部，我们首先导入了“iplugin”包，它是“github.com/ipfs/kubo/plugin”的依赖项。然后，我们创建了一个名为“loadPluginFunc”的函数，它使用了“oplugin”和“noplugin”的组合。

接下来，我们没有做任何具体的实现，直接返回了一个无参函数。根据组合函数的特性，我们可以猜测这个函数会在需要时被调用，然后在函数内部执行加载IHF（InterPlanetary File System）插件的操作。不过，由于没有提供具体的输入和输出，因此无法进一步验证我们的猜测。


```go
//go:build noplugin
// +build noplugin

package loader

import (
	"errors"

	iplugin "github.com/ipfs/kubo/plugin"
)

func init() {
	loadPluginFunc = nopluginLoadPlugin
}

```

这是一段用于在IPlugin接口中加载插件的函数，其作用是返回一个名为[]iplugin.Plugin的切片，若错误则返回一个名为error的错误对象。

函数接收一个字符串参数，表示插件的名称。函数内部首先尝试使用该插件，若尝试失败，则返回一个名为error的错误对象，该对象的值为"not built with plugin support"。

在实际应用中，该函数用于在运行时检查是否支持所要加载的插件，若插件支持，则可以正常返回所需的Plugin；若插件不支持，则返回一个错误信息，用户可以根据此进行相应调整。


```go
func nopluginLoadPlugin(string) ([]iplugin.Plugin, error) {
	return nil, errors.New("not built with plugin support")
}

```

# `plugin/loader/load_unix.go`

这段代码是一个Go语言编写的Makefile，用于编译并运行一个名为"kubo"的KubernetesIPFS分布式存储库的子模块。

具体来说，这段代码的作用如下：

1. 首先，通过`go build`命令编译KubernetesIPFS相关代码为可执行文件；
2. 然后，通过`cgo`命令编译KubernetesIPFS主代码为名为"kubernetes"的C文件；
3. 如果编译过程中存在名为"noplugin"的可执行文件，则执行`./noplugin`可执行文件，否则继续执行后续编译步骤；
4. 对于每个编译选项，先编译Linux平台，然后编译Darwin和FreeBSD平台；
5. 最后，加载KubernetesIPFS插件。


```go
//go:build cgo && !noplugin && (linux || darwin || freebsd)
// +build cgo
// +build !noplugin
// +build linux darwin freebsd

package loader

import (
	"errors"
	"plugin"

	iplugin "github.com/ipfs/kubo/plugin"
)

func init() {
	loadPluginFunc = unixLoadPlugin
}

```

这段代码定义了一个名为"unixLoadPlugin"的函数，接受一个字符串参数"fi"。函数返回一个由名为"iplugin.Plugin"的接口数组和可能存在的错误组成的元组，具体实现如下：

1. 函数首先调用名为"plugin.Open"的函数，接收一个文件路径参数"fi"，这个函数的作用是加载并打开指定文件，如果失败则返回 nil 和错误。

2. 函数接着使用 "pl.Lookup" 方法，接收一个文件路径参数 "fi"，返回名为 "Plugins" 的结点，如果失败则返回 nil 和错误。

3. 然后函数定义了一个名为 "typePls" 的类型变量，该类型变量被赋值为 "*[]iplugin.Plugin"，表示类型 Plugin 的数组长度为 "ipnl插件" 类型。

4. 最后，函数调用 "pls.容纳" 方法，接收一个 "*[]iplugin.Plugin" 的类型变量，并检查该数组是否包含名为 "Plugins" 的元素，如果包含则返回它，否则输出错误并返回一个 "error" 类型的变量。

5. 函数返回的结果是，由名为 "ipnl插件" 的接口数组和可能的错误组成的元组，如果调用失败则输出错误。


```go
func unixLoadPlugin(fi string) ([]iplugin.Plugin, error) {
	pl, err := plugin.Open(fi)
	if err != nil {
		return nil, err
	}
	pls, err := pl.Lookup("Plugins")
	if err != nil {
		return nil, err
	}

	typePls, ok := pls.(*[]iplugin.Plugin)
	if !ok {
		return nil, errors.New("filed 'Plugins' didn't contain correct type")
	}

	return *typePls, nil
}

```

# `plugin/loader/preload.go`

这段代码定义了一个名为“loader”的包。这个包定义了一系列导入的第三方插件，这些插件都是用来提供额外的功能和增强表现，包括：

- nopfs：提供本地文件系统对IPFS的支持
- ipldagjose：提供基于DAG和JSON的APIJava客户端
- flatfs：提供基于FlatFS的APIJava客户端
- badgerds：提供基于Badger的APIJava客户端
- dagjose：提供基于DAG和JSON的APIJava客户端
- git：提供基于Git的APIJava客户端
- levelds：提供基于LevelDS的APIJava客户端
- fxtest：提供基于Fxtest的APIJava客户端

通过导入这些插件，“loader”可以提供更多的功能和方便，使得开发人员可以更轻松地开发基于IPFS的Kubernetes应用。


```go
package loader

import (
	pluginbadgerds "github.com/ipfs/kubo/plugin/plugins/badgerds"
	pluginiplddagjose "github.com/ipfs/kubo/plugin/plugins/dagjose"
	pluginflatfs "github.com/ipfs/kubo/plugin/plugins/flatfs"
	pluginfxtest "github.com/ipfs/kubo/plugin/plugins/fxtest"
	pluginipldgit "github.com/ipfs/kubo/plugin/plugins/git"
	pluginlevelds "github.com/ipfs/kubo/plugin/plugins/levelds"
	pluginnopfs "github.com/ipfs/kubo/plugin/plugins/nopfs"
	pluginpeerlog "github.com/ipfs/kubo/plugin/plugins/peerlog"
)

// DO NOT EDIT THIS FILE
// This file is being generated as part of plugin build process
```

这段代码是一个插件加载器（plugin loader）脚本，它的作用是加载和管理多个插件（plugins）的依赖关系。这个脚本是使用Go语言编写的，并使用了Go插件规范（Go Plugin Specification）定义了插件的接口。

具体来说，这个脚本的主要作用是执行以下操作：

1. 初始化插件加载器。
2. 加载由pluginipldgit.Plugins...命名的所有插件。
3. 加载由pluginiplddagjose.Plugins...命名的所有插件。
4. 加载由pluginbadgerds.Plugins...命名的所有插件。
5. 加载由pluginflatfs.Plugins...命名的所有插件。
6. 加载由pluginlevelds.Plugins...命名的所有插件。
7. 加载由pluginpeerlog.Plugins...命名的所有插件。
8. 加载由pluginfxtest.Plugins...命名的所有插件。
9. 加载由pluginnopfs.Plugins...命名的所有插件。

这个插件加载器脚本仅作为插件的一级加载器，它并不会对插件进行任何修改或执行其他操作。要使用插件，需要将插件的代码段复制到插件的原始缓冲区（source code），并使用编译器将插件编译为可执行文件。


```go
// To change it, modify the plugin/loader/preload.sh

func init() {
	Preload(pluginipldgit.Plugins...)
	Preload(pluginiplddagjose.Plugins...)
	Preload(pluginbadgerds.Plugins...)
	Preload(pluginflatfs.Plugins...)
	Preload(pluginlevelds.Plugins...)
	Preload(pluginpeerlog.Plugins...)
	Preload(pluginfxtest.Plugins...)
	Preload(pluginnopfs.Plugins...)
}

```

# `plugin/plugins/badgerds/badgerds.go`

这段代码是一个 Go 语言编写的库，名为 "badgerds"，用于在 Kubernetes 中使用 IPFS(InterPlanetary File System)数据源。它实现了以下功能：

1. 导入了必要的库：fmt、os、path/filepath、github.com/ipfs/kubo/plugin、github.com/ipfs/kubo/repo、github.com/ipfs/kubo/repo/fsrepo、humanize 和 badgerds。

2. 定义了一个名为 "badgerds" 的接口，该接口定义了库中所有功能的一致性。

3. 实现了文件系统的读写操作：读取和写入文件系统中的文件和目录。使用 Repo 和 FileSuite(代表了 IPFS 的文件系统)提供的功能来实现。

4. 通过调用 GitHub 仓库中的 BadgerDS(一个 IPFS 数据源)实现了数据源的拉取和入度控制。

5. 通过调用 humanize.NewFormatter() 实现了文件名的格式化。

总结起来，这段代码是一个库，它实现了在 Kubernetes 中使用 IPFS 数据源的基本功能。


```go
package badgerds

import (
	"fmt"
	"os"
	"path/filepath"

	"github.com/ipfs/kubo/plugin"
	"github.com/ipfs/kubo/repo"
	"github.com/ipfs/kubo/repo/fsrepo"

	humanize "github.com/dustin/go-humanize"
	badgerds "github.com/ipfs/go-ds-badger"
)

```

这段代码定义了一个名为"Plugins"的不可变切片，它包含一个具体的插件对象，该插件对象属于一个名为"badgerdsPlugin"的插件类。

badgerdsPlugin是一个匿名类型，它代表了一个没有名称和版本的插件。通过将badgerdsPlugin赋值给Plugins切片中的第一个元素，可以获取到该插件的实例，从而使用该插件的各种属性和方法。

在badgerdsPlugin的定义中，使用了三个方法：Name、Version以及一个没有实现的0*切片访问模式(&badgerdsPlugin)[]interface{}。其中，Name和Version方法简单的实现了name和version属性的设置，而0*切片访问模式则被用来访问badgerdsPlugin的实现接口。

由于badgerdsPlugin是一个匿名类型，因此无法直接访问其实例的Name和Version属性，但是可以像访问普通切片一样使用它们的默认实现。


```go
// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{
	&badgerdsPlugin{},
}

type badgerdsPlugin struct{}

var _ plugin.PluginDatastore = (*badgerdsPlugin)(nil)

func (*badgerdsPlugin) Name() string {
	return "ds-badgerds"
}

func (*badgerdsPlugin) Version() string {
	return "0.1.0"
}

```

这是一段使用C Defined Application Programming Interface (C-DAAPI) 的代码。具体解释如下：

1. `func (*badgerdsPlugin) Init(_ *plugin.Environment) error {
	return nil
}` 是函数，名称为`Init`，接受一个`plugin.Environment`类型的参数，并返回一个`error`类型的参数。初始化函数在加载插件时执行，它的作用是将坏gerds插件加载到环境中。如果没有错误，函数返回` nil`，否则返回一个非` nil`值。

2. `func (*badgerdsPlugin) DatastoreTypeName() string {
	return "badgerds"
}` 是函数，名称为`DatastoreTypeName`，它返回一个字符串，表示`datastore`类型的名称。

3. `type datastoreConfig struct {
	path       string                   `json:"path"`         // 设置存储路径
	syncWrites bool                `json:"syncWrites"`  // 设置是否同步写入
	truncate   bool                `json:"truncate"`    // 设置是否归零
	vlogFileSize int64            `json:"vlogFileSize"` // 设置日志文件大小
}` 定义了一个`datastoreConfig`结构体，它包含存储数据的数据存储器配置。

4. `*badgerdsPlugin` 是指针，指向一个名为`badgerdsPlugin`的类型，该类型实现了`badgerdsPlugin`接口。

5. `Init`函数接受一个`plugin.Environment`类型的参数，并返回一个`error`类型的参数。在初始化函数中，如果初始化成功，`badgerdsPlugin`将`Init`返回的` nil`作为结果。

6. `DatastoreTypeName`函数返回一个字符串，表示`datastore`类型的名称。

7. `badgerdsPlugin`是一个指向`badgerds`实现的`badgerdsPlugin`类型。它实现了`Init`和`DatastoreTypeName`函数，因此，它可以在初始化函数中使用自身来执行这些函数。


```go
func (*badgerdsPlugin) Init(_ *plugin.Environment) error {
	return nil
}

func (*badgerdsPlugin) DatastoreTypeName() string {
	return "badgerds"
}

type datastoreConfig struct {
	path       string
	syncWrites bool
	truncate   bool

	vlogFileSize int64
}

```

This appears to be a function that creates a DatastoreConfig object and returns it, passing in a map of parameters. The function takes a single parameter of type `params`, which maps a string to an interface that can be either a `fsrepo.DatastoreConfig` object or an error. The function has the following signature:

params map[string]interface{}) (fsrepo.DatastoreConfig, error) {

This means that the function expects the value to be passed in as a key, and the key must be of the specified type. The return type of the function is `fsrepo.DatastoreConfig, error`. If the function evaluates successfully, it returns a value of type `fsrepo.DatastoreConfig, error`. If it fails, it returns an error of type `error`.


```go
// BadgerdsDatastoreConfig returns a configuration stub for a badger datastore
// from the given parameters.
func (*badgerdsPlugin) DatastoreConfigParser() fsrepo.ConfigFromMap {
	return func(params map[string]interface{}) (fsrepo.DatastoreConfig, error) {
		var c datastoreConfig
		var ok bool

		c.path, ok = params["path"].(string)
		if !ok {
			return nil, fmt.Errorf("'path' field is missing or not string")
		}

		sw, ok := params["syncWrites"]
		if !ok {
			c.syncWrites = false
		} else {
			if swb, ok := sw.(bool); ok {
				c.syncWrites = swb
			} else {
				return nil, fmt.Errorf("'syncWrites' field was not a boolean")
			}
		}

		truncate, ok := params["truncate"]
		if !ok {
			c.truncate = true
		} else {
			if truncate, ok := truncate.(bool); ok {
				c.truncate = truncate
			} else {
				return nil, fmt.Errorf("'truncate' field was not a boolean")
			}
		}

		vls, ok := params["vlogFileSize"]
		if !ok {
			// default to 1GiB
			c.vlogFileSize = badgerds.DefaultOptions.ValueLogFileSize
		} else {
			if vlogSize, ok := vls.(string); ok {
				s, err := humanize.ParseBytes(vlogSize)
				if err != nil {
					return nil, err
				}
				c.vlogFileSize = int64(s)
			} else {
				return nil, fmt.Errorf("'vlogFileSize' field was not a string")
			}
		}

		return &c, nil
	}
}

```

这段代码定义了两个函数：

1. `func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec`函数：该函数返回一个代表datastore的DiskSpec结构体。其中，DiskSpec结构体包含以下字段：`type`（类型），`path`（路径）。函数的作用是将datastore的配置文件`c`中的`path`字段和`type`字段解析为DiskSpec结构体，并返回该结构体。

2. `func (c *datastoreConfig) Create(path string) (repo.Datastore, error)`函数：该函数创建一个新的datastore实例，并返回其datastore和可能的错误。函数的作用是：

a. 检查给定的路径是否为绝对路径。如果不是，则将路径转换为绝对路径。

b. 创建一个新目录，如果失败则返回错误。

c. 根据datastore的配置选项设置坏账容错选项。

d. 调用`badgerds.NewDatastore`函数，使用新的datastore配置创建新的datastore实例。

e. 返回新datastore实例和调用成功。


```go
func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec {
	return map[string]interface{}{
		"type": "badgerds",
		"path": c.path,
	}
}

func (c *datastoreConfig) Create(path string) (repo.Datastore, error) {
	p := c.path
	if !filepath.IsAbs(p) {
		p = filepath.Join(path, p)
	}

	err := os.MkdirAll(p, 0o755)
	if err != nil {
		return nil, err
	}

	defopts := badgerds.DefaultOptions
	defopts.SyncWrites = c.syncWrites
	defopts.Truncate = c.truncate
	defopts.ValueLogFileSize = c.vlogFileSize

	return badgerds.NewDatastore(p, &defopts)
}

```

# `plugin/plugins/dagjose/dagjose.go`

这段代码定义了一个名为 "dagjose" 的 package，它包含了一些导入、定义和导入声明，以及一个名为 "Plugins" 的字段，其值为一个包含一个名为 "dagjose" 的列表类型的 "plugin.Plugin" 结构体。

具体来说，这段代码以下几个步骤实现了上述功能：

1. 导入 "github.com/ipfs/kubo/plugin"、"github.com/ceramicnetwork/go-dag-jose/dagjose"、"github.com/ipld/go-ipld-prime/multicodec" 和 "github.com/multiformats/go-multicodec" 这四个 "plugin.Plugin" 类型。

2. 定义了一个名为 "dagjose" 的 "plugin.Plugin" 类型，它实现了 "dagjose.plugin.DagjosePlugin" 接口，该接口定义了该插件的一些基本属性和方法。

3. 在 "dagjose" 插件的实现中，使用了 "github.com/ipfs/kubo/plugin" 中的 "kubelet-plugins" 功能，通过实现 "plugin.Kubelet" 接口，将 "dagjose" 插件注册到 Kubernetes cluster 中，以便在集群中使用 DAG-Jose。

4. 通过导入 "multicodec" 和 "go-multicodec" 包，实现了多个数据编码的混合格式，以便在 DAG-Jose 使用不同的数据编码形式。


```go
package dagjose

import (
	"github.com/ipfs/kubo/plugin"

	"github.com/ceramicnetwork/go-dag-jose/dagjose"
	"github.com/ipld/go-ipld-prime/multicodec"
	mc "github.com/multiformats/go-multicodec"
)

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{
	&dagjosePlugin{},
}

```

这段代码定义了一个名为 `dagjosePlugin` 的类型，它是一个 `plugin.PluginIPLD` 类型的左闭合接口。

然后，它通过 `var _ plugin.PluginIPLD = (*dagjosePlugin)(nil)` 变量，将 `dagjosePlugin` 的 pointer 类型赋值为 `nil`，即 `0`。

接着，它通过 `fmt.Println` 函数输出了 "ipld-codec-dagjose" 这个名称，这是 `dagjosePlugin` 的名称。

然后，它通过 `fmt.Println` 函数输出了 "0.0.1" 这个版本，这是 `dagjosePlugin` 的版本。

接下来，它通过 `func (*dagjosePlugin) Name() string` 函数返回了一个字符串 "ipld-codec-dagjose"。

然后，它通过 `func (*dagjosePlugin) Version() string` 函数返回了一个字符串 "0.0.1"。

最后，它通过 `func (*dagjosePlugin) Init(_ *plugin.Environment) error` 函数，表示 `dagjosePlugin` 的初始化，它接受一个 `*plugin.Environment` 类型的参数，但没有实现具体的初始化操作。


```go
type dagjosePlugin struct{}

var _ plugin.PluginIPLD = (*dagjosePlugin)(nil)

func (*dagjosePlugin) Name() string {
	return "ipld-codec-dagjose"
}

func (*dagjosePlugin) Version() string {
	return "0.0.1"
}

func (*dagjosePlugin) Init(_ *plugin.Environment) error {
	return nil
}

```

这是一段使用Go语言编写的函数指针类型（函数指针是一个指向函数的指针）的代码。函数指针名为"func (*dagjosePlugin) Register(reg multicodec.Registry)"，它接收一个名为"reg"的参数，代表一个名为"Registry"的接口类型，该接口类型包含一个名为"Register"的函数。函数指针返回一个名为"error"的类型，代表一个发生错误的结果。

该函数的作用是注册一个名为"DagJose"的加密/解密器到名为"Registry"的注册表中。注册器函数需要传递两个参数，第一个参数是一个代表"DagJose"的接口类型，第二个参数是一个代表"Registry"的接口类型。函数的作用是执行"Registry"注册"DagJose"加密/解密器操作。


```go
func (*dagjosePlugin) Register(reg multicodec.Registry) error {
	reg.RegisterEncoder(uint64(mc.DagJose), dagjose.Encode)
	reg.RegisterDecoder(uint64(mc.DagJose), dagjose.Decode)
	return nil
}

```

# `plugin/plugins/flatfs/flatfs.go`

这段代码定义了一个名为"flatfs"的包，其中包含了一个名为"flatfsPlugin"的静态错误类型。

"flatfs"包提供了一个"/path/to/filesystem/on-demand"接口，用于实现离线读写文件系统。

具体来说，该代码导入了自"github.com/ipfs/kubo"包中的"filepath"和"github.com/ipfs/kubo/plugin"类型，用于获取文件系统的路径和加载flatfs插件。同时，该代码还定义了一个名为"Plugins"的静态变量，用于存储所有加载的插件，其中包括"flatfsPlugin"。

"flatfsPlugin"是一个实现了"plugin.Plugin"接口的类，其中包含了一些与flatfs相关的函数和标记，例如"create/Open/Close"函数，以及"file/子类"。

最后，该代码还定义了一些常量和变量，用于定义flatfs的一些参数和选项。


```go
package flatfs

import (
	"fmt"
	"path/filepath"

	"github.com/ipfs/kubo/plugin"
	"github.com/ipfs/kubo/repo"
	"github.com/ipfs/kubo/repo/fsrepo"

	flatfs "github.com/ipfs/go-ds-flatfs"
)

// Plugins is exported list of plugins that will be loaded.
var Plugins = []plugin.Plugin{
	&flatfsPlugin{},
}

```

这段代码定义了一个名为 flatfsPlugin 的 struct 类型，表示一个 flat file system plugin。这个 plugin被声明为 plugin.PluginDatastore 的类型，这意味着它是一个 datastore plugin，用于提供对数据存储的访问。

接下来，该 struct 类型包含两个方法，用于设置和获取 flat file system 的一些设置和版本信息。

第一个方法是名为 "Name" 的方法，它返回 flat file system 的名称。

第二个方法是名为 "Version" 的方法，它返回 flat file system 的版本号。

第三个方法是名为 "Init" 的方法，它接受一个 plugin.Environment 类型的参数，用于初始化 flat file system 插件。在初始化时，它返回一个 nil 值，表示没有错误。

最后，该 struct 类型还包含一个名为 "ds-flatfs" 的类型，用于将 flat file system 映射到 plugin.PluginDatastore。


```go
type flatfsPlugin struct{}

var _ plugin.PluginDatastore = (*flatfsPlugin)(nil)

func (*flatfsPlugin) Name() string {
	return "ds-flatfs"
}

func (*flatfsPlugin) Version() string {
	return "0.1.0"
}

func (*flatfsPlugin) Init(_ *plugin.Environment) error {
	return nil
}

```

这段代码定义了一个名为`DatastoreConfigParser`的函数，该函数接收一个参数`params`（键值对），然后返回一个`fsrepo.DatastoreConfig`结构体，或者错误。

具体来说，这段代码实现了一个`BadgerdsDatastoreConfig`的解析函数，该函数接受一个包含`path`、`shardFunc`和`sync`字段的参数`params`。函数首先验证`path`和`shardFunc`是否都已存在，如果都存在，则尝试解析`shardFunc`，并将解析得到的`shardFunc`和`sync`字段值存储到`c`结构体中。最后，函数返回`c`结构体，或者错误。

这段代码的作用是实现了一个`BadgerdsDatastoreConfig`的解析函数，该函数可以根据传入的参数解析出一个`BadgerdsDatastoreConfig`结构体，然后返回该结构体。


```go
func (*flatfsPlugin) DatastoreTypeName() string {
	return "flatfs"
}

type datastoreConfig struct {
	path      string
	shardFun  *flatfs.ShardIdV1
	syncField bool
}

// BadgerdsDatastoreConfig returns a configuration stub for a badger datastore
// from the given parameters.
func (*flatfsPlugin) DatastoreConfigParser() fsrepo.ConfigFromMap {
	return func(params map[string]interface{}) (fsrepo.DatastoreConfig, error) {
		var c datastoreConfig
		var ok bool
		var err error

		c.path, ok = params["path"].(string)
		if !ok {
			return nil, fmt.Errorf("'path' field is missing or not boolean")
		}

		sshardFun, ok := params["shardFunc"].(string)
		if !ok {
			return nil, fmt.Errorf("'shardFunc' field is missing or not a string")
		}
		c.shardFun, err = flatfs.ParseShardFunc(sshardFun)
		if err != nil {
			return nil, err
		}

		c.syncField, ok = params["sync"].(bool)
		if !ok {
			return nil, fmt.Errorf("'sync' field is missing or not boolean")
		}
		return &c, nil
	}
}

```

这段代码定义了两个函数，分别接收一个名为 `datastoreConfig` 的整数类型的参数，并返回一个名为 `DiskSpec` 的整数类型类型的结果。

第一个函数名为 `func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec`，函数接收一个整数类型的参数 `c`，并返回一个整数类型类型的结果，该结果包含以下字段：

* "type": 32字段，类型为 "flatfs"，表示数据存储类型为 flatfs 存储器。
* "path": 64字段，类型为 string，表示数据存储的路径。
* "shardFunc": 32字段，类型为 "shardFunc"，表示用于对路径进行分片的函数。

第二个函数名为 `func (c *datastoreConfig) Create(path string) (repo.Datastore, error)`，函数接收一个字符串类型的参数 `path`，并返回一个名为 `repo.Datastore` 的整数类型类型的结果和一个名为 `error` 的错误类型的结果。

函数首先将 `c` 指向的路径的绝对路径。然后使用 `filepath.Join` 函数将路径和当前工作目录连接起来，以便创建数据存储文件。接着使用 `flatfs.CreateOrOpen` 函数创建或打开数据存储文件，并设置一个字符类型的参数 `c.shardFun` 和一个布尔类型的参数 `c.syncField`，表示是否自动创建数据存储文件并同步数据文件。最后，函数返回创建的数据存储文件的结果。


```go
func (c *datastoreConfig) DiskSpec() fsrepo.DiskSpec {
	return map[string]interface{}{
		"type":      "flatfs",
		"path":      c.path,
		"shardFunc": c.shardFun.String(),
	}
}

func (c *datastoreConfig) Create(path string) (repo.Datastore, error) {
	p := c.path
	if !filepath.IsAbs(p) {
		p = filepath.Join(path, p)
	}

	return flatfs.CreateOrOpen(p, c.shardFun, c.syncField)
}

```