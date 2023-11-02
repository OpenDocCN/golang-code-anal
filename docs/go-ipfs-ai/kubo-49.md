# go-ipfs 源码解析 49

# `repo/fsrepo/fsrepo.go`

这段代码定义了一个名为 `fsrepo` 的包，它提供了一个 HomeDir 并且使用了验证、错误处理和格式化等安全特性。

具体来说，它实现了以下功能：

1. 导入了一些必要的库，包括 `filestore`、`keystore`、`repo` 和 `dir`。这些库提供了文件存储和访问的功能。

2. 实现了 `util`、`ds`、`measure` 和 `lockfile` 这些库的功能，它们提供了常用的工具和数据结构。

3. 实现了文件存储和访问的功能，包括使用 `filestore` 和 `dir` 库实现文件存储，使用 `keystore` 和 `dir` 库实现数据结构，以及使用锁来确保数据一致性。

4. 实现了日志记录的功能，包括记录基本信息和错误信息，以便在出现问题时进行追踪和诊断。

5. 实现了一个名为 `migrations` 的包，它提供了 HomeDir 的迁移功能，包括迁移历史和迁移状态的记录。

6. 实现了 `homedir` 和 `ma` 库的功能，其中 `homedir` 提供了对 HomeDir 的管理，而 `ma` 库提供了对 MultiAddr 类型的支持。

7. 通过 `util.Context`、`fmt.Println`、`errors.Print`、`strings.Replace` 等实现了文件系统repo的一些常用功能。


```go
package fsrepo

import (
	"context"
	"errors"
	"fmt"
	"io"
	"net"
	"os"
	"path/filepath"
	"strings"
	"sync"

	filestore "github.com/ipfs/boxo/filestore"
	keystore "github.com/ipfs/boxo/keystore"
	repo "github.com/ipfs/kubo/repo"
	"github.com/ipfs/kubo/repo/common"
	dir "github.com/ipfs/kubo/thirdparty/dir"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"

	util "github.com/ipfs/boxo/util"
	ds "github.com/ipfs/go-datastore"
	measure "github.com/ipfs/go-ds-measure"
	lockfile "github.com/ipfs/go-fs-lock"
	logging "github.com/ipfs/go-log"
	config "github.com/ipfs/kubo/config"
	serialize "github.com/ipfs/kubo/config/serialize"
	"github.com/ipfs/kubo/repo/fsrepo/migrations"
	homedir "github.com/mitchellh/go-homedir"
	ma "github.com/multiformats/go-multiaddr"
)

```

这段代码定义了一个名为 "repo.lock" 的文件，用于存储 Git 仓库的锁。这个文件名的含义是相对于配置目录的根目录。

然后，代码定义了一个名为 "RepoVersion" 的变量，用于记录当前期望看到的 Git 仓库版本号。

接下来，代码定义了一个名为 "migrationInstructions" 的字符串，用于存储指向移


```go
// LockFile is the filename of the repo lock, relative to config dir
// TODO rename repo lock and hide name.
const LockFile = "repo.lock"

var log = logging.Logger("fsrepo")

// RepoVersion is the version number that we are currently expecting to see.
var RepoVersion = 15

var migrationInstructions = `See https://github.com/ipfs/fs-repo-migrations/blob/master/run.md
Sorry for the inconvenience. In the future, these will run automatically.`

var programTooLowMessage = `Your programs version (%d) is lower than your repos (%d).
Please update ipfs to a version that supports the existing repo, or run
a migration in reverse.

```

这段代码是一个错误处理程序，它包含了三种不同类型的错误，并给出了一些错误消息。

这段代码定义了一个名为 NoRepoError 的结构体，它包含了一个路径字段和一个错误消息。

接着，代码使用一个名为 err 的变量来存储不同类型的错误。

然后，代码使用一个名为 NoRepoError 的变量来存储错误实例。

在接下来的几行中，代码使用 errors.New() 函数来创建新的错误实例，并使用一些字符串模板来设置错误消息。这些错误消息都是类似于 "没有找到指定文件" 或 "的需要进行迁移" 这样的错误消息。

最后，代码使用一些断言来检查错误是否为 NoRepoError 实例，如果是，就执行一些操作并返回错误消息。


```go
See https://github.com/ipfs/fs-repo-migrations/blob/master/run.md for details.`

var (
	ErrNoVersion     = errors.New("no version file found, please run 0-to-1 migration tool.\n" + migrationInstructions)
	ErrOldRepo       = errors.New("ipfs repo found in old '~/.go-ipfs' location, please run migration tool.\n" + migrationInstructions)
	ErrNeedMigration = errors.New("ipfs repo needs migration, please run migration tool.\n" + migrationInstructions)
)

type NoRepoError struct {
	Path string
}

var _ error = NoRepoError{}

func (err NoRepoError) Error() string {
	return fmt.Sprintf("no IPFS repo found in %s.\nplease run: 'ipfs init'", err.Path)
}

```

这段代码定义了一个名为 `datastore_spec` 的数据存储规范。这个规范定义了如何使用飞瓜 (Gateway) 和三周丰 (Swarm) 存储库来存储数据。

它还定义了一个名为 `packageLock` 的变量，它是一个 `sync.Mutex`，用于确保在访问任何与FSRepo状态相关的操作时，代码可以保证只有一个实例同时执行。

最后，它定义了一个名为 `swarmKeyFile` 的变量，它指定了三周丰存储库的私钥文件的路径。


```go
const (
	apiFile      = "api"
	gatewayFile  = "gateway"
	swarmKeyFile = "swarm.key"
)

const specFn = "datastore_spec"

var (

	// packageLock must be held to while performing any operation that modifies an
	// FSRepo's state field. This includes Init, Open, Close, and Remove.
	packageLock sync.Mutex

	// onlyOne keeps track of open FSRepo instances.
	//
	// TODO: once command Context / Repo integration is cleaned up,
	// this can be removed. Right now, this makes ConfigCmd.Run
	// function try to open the repo twice:
	//
	//     $ ipfs daemon &
	//     $ ipfs config foo
	//
	// The reason for the above is that in standalone mode without the
	// daemon, `ipfs config` tries to save work by not building the
	// full IpfsNode, but accessing the Repo directly.
	onlyOne repo.OnlyOne
)

```

该代码定义了一个名为FSRepo的结构体，表示一个IPFS文件系统repo。它安全地用于多个调用者。

该结构体包含以下字段：

- closed: 一个布尔值，表示FSRepo是否已经被关闭。默认为false。
- path: 一个字符串，表示文件系统的路径。
- Path to the configuration file that may or may not be inside the FSRepo: 一个字符串，表示配置文件的路径。该字段允许调用者在调用close()方法之前设置FSRepo的配置文件。配置文件可以放置在FSRepo的子目录中。
- lockfile: 一个 io.Closer 类型字段，表示防止多个调用者同时访问FSRepo的锁。
- config: 一个config.Config类型字段，表示FSRepo的配置。可以设置config的值，以告诉FSRepo如何初始化。
- userResourceOverrides: rcmgr.PartialLimitConfig类型字段，表示允许用户覆盖FSRepo的配置限制。
- ds: 一个repo.Datastore类型字段，表示FSRepo的datastore。
- keystore: 一个keystore.Keystore类型字段，表示FSRepo的密钥存储。
- filemgr: 一个filestore.FileManager类型字段，表示FSRepo的文件管理器。可以调用filemgr的open()和close()方法来打开和关闭文件。

该FSRepo结构体提供了一个统一的接口，让调用者可以使用FSRepo的多种属性和方法。可以安全地用于多个调用者，而不会互相干扰。


```go
// FSRepo represents an IPFS FileSystem Repo. It is safe for use by multiple
// callers.
type FSRepo struct {
	// has Close been called already
	closed bool
	// path is the file-system path
	path string
	// Path to the configuration file that may or may not be inside the FSRepo
	// path (see config.Filename for more details).
	configFilePath string
	// lockfile is the file system lock to prevent others from opening
	// the same fsrepo path concurrently
	lockfile              io.Closer
	config                *config.Config
	userResourceOverrides rcmgr.PartialLimitConfig
	ds                    repo.Datastore
	keystore              keystore.Keystore
	filemgr               *filestore.FileManager
}

```

这段代码定义了两个函数，Open 和 OpenWithUserConfig，它们都用于打开一个名为 FSRepo 的文件系统仓库。

这两个函数的行为相同，但 Open 函数有一个额外的功能，即它可以使用一个配置文件来指定仓库的初始化参数，而 OpenWithUserConfig 函数则只能使用一个文件来指定配置文件路径。

函数 Open 的实现较为复杂，它首先调用一个名为 open 的函数，然后使用 onlyOne.Open 函数来打开仓库。函数 OpenWithUserConfig 实现较为简单，它直接调用 Open 函数并传递一个用户指定的配置文件路径。

注意：在实际应用中，如果这段代码放在一个名为 "github.com/user/repo/master" 的 GitHub 仓库中，那么它将作为贡献者贡献到这个仓库中。


```go
var _ repo.Repo = (*FSRepo)(nil)

// Open the FSRepo at path. Returns an error if the repo is not
// initialized.
func Open(repoPath string) (repo.Repo, error) {
	fn := func() (repo.Repo, error) {
		return open(repoPath, "")
	}
	return onlyOne.Open(repoPath, fn)
}

// OpenWithUserConfig is the equivalent to the Open function above but with the
// option to set the configuration file path instead of using the default.
func OpenWithUserConfig(repoPath string, userConfigFilePath string) (repo.Repo, error) {
	fn := func() (repo.Repo, error) {
		return open(repoPath, userConfigFilePath)
	}
	return onlyOne.Open(repoPath, fn)
}

```

This is a Go function that creates a new `FileMgr` instance for a given directory, given a path to a config object, and options for a file manager. It checks for errors and returns either the newly created `FileMgr` instance or an error if any.


```go
func open(repoPath string, userConfigFilePath string) (repo.Repo, error) {
	packageLock.Lock()
	defer packageLock.Unlock()

	r, err := newFSRepo(repoPath, userConfigFilePath)
	if err != nil {
		return nil, err
	}

	// Check if its initialized
	if err := checkInitialized(r.path); err != nil {
		return nil, err
	}

	r.lockfile, err = lockfile.Lock(r.path, LockFile)
	if err != nil {
		return nil, err
	}
	keepLocked := false
	defer func() {
		// unlock on error, leave it locked on success
		if !keepLocked {
			r.lockfile.Close()
		}
	}()

	// Check version, and error out if not matching
	ver, err := migrations.RepoVersion(r.path)
	if err != nil {
		if os.IsNotExist(err) {
			return nil, ErrNoVersion
		}
		return nil, err
	}

	if RepoVersion > ver {
		return nil, ErrNeedMigration
	} else if ver > RepoVersion {
		// program version too low for existing repo
		return nil, fmt.Errorf(programTooLowMessage, RepoVersion, ver)
	}

	// check repo path, then check all constituent parts.
	if err := dir.Writable(r.path); err != nil {
		return nil, err
	}

	if err := r.openConfig(); err != nil {
		return nil, err
	}

	if err := r.openUserResourceOverrides(); err != nil {
		return nil, err
	}

	if err := r.openDatastore(); err != nil {
		return nil, err
	}

	if err := r.openKeystore(); err != nil {
		return nil, err
	}

	if r.config.Experimental.FilestoreEnabled || r.config.Experimental.UrlstoreEnabled {
		r.filemgr = filestore.NewFileManager(r.ds, filepath.Dir(r.path))
		r.filemgr.AllowFiles = r.config.Experimental.FilestoreEnabled
		r.filemgr.AllowUrls = r.config.Experimental.UrlstoreEnabled
	}

	keepLocked = true
	return r, nil
}

```

这段代码定义了一个名为 `newFSRepo` 的函数，它接收两个参数：`rpath` 和 `userConfigFilePath`。

函数首先尝试使用 `homedir.Expand` 函数扩展 `rpath` 的根目录。如果扩展过程出现错误，函数返回 `nil` 和错误信息。

然后，函数调用 `config.Filename` 函数来查找并返回 `rpath` 和 `userConfigFilePath` 指定的配置文件路径。如果这个函数也出现错误，函数将返回 `nil` 和错误信息。

最后，函数创建一个名为 `FSRepo` 的结构体，它包含一个 `path` 字段和一个 `configFilePath` 字段。函数返回这个结构体，如果所有参数都被正确传递，不会输出任何错误信息。


```go
func newFSRepo(rpath string, userConfigFilePath string) (*FSRepo, error) {
	expPath, err := homedir.Expand(filepath.Clean(rpath))
	if err != nil {
		return nil, err
	}

	configFilePath, err := config.Filename(rpath, userConfigFilePath)
	if err != nil {
		// FIXME: Personalize this when the user config path is "".
		return nil, fmt.Errorf("finding config filepath from repo %s and user config %s: %w",
			rpath, userConfigFilePath, err)
	}
	return &FSRepo{path: expPath, configFilePath: configFilePath}, nil
}

```

这两函数是用来检查IPFS和Go-IPFS是否已经初始化好的。

函数`checkInitialized`的作用是检查给定的路径是否已经初始化好。如果还没有初始化好，则执行以下操作：

1. 检查给定的路径是否已经存在Go-IPFS资源。如果不存在，将路径中的".ipfs"替换为".go-ipfs"，并将路径作为参数传递给`strings.Replace`函数。

2. 如果给定的路径已经存在Go-IPFS资源，但是当前的路径与之前的路径不同，则返回`ErrOldRepo`错误。

3. 如果上述步骤都成功，则返回`NoRepoError`并设置为`nil`。

函数`configIsInitialized`的作用是检查给定的路径是否已经被初始化。如果路径没有被初始化，则返回`false`。如果路径已经初始化，则返回`true`。

初始化这些函数时，需要保证在调用之前，IPFS和Go-IPFS都已经初始化好了。


```go
func checkInitialized(path string) error {
	if !isInitializedUnsynced(path) {
		alt := strings.Replace(path, ".ipfs", ".go-ipfs", 1)
		if isInitializedUnsynced(alt) {
			return ErrOldRepo
		}
		return NoRepoError{Path: path}
	}
	return nil
}

// configIsInitialized returns true if the repo is initialized at
// provided |path|.
func configIsInitialized(path string) bool {
	configFilename, err := config.Filename(path, "")
	if err != nil {
		return false
	}
	if !util.FileExists(configFilename) {
		return false
	}
	return true
}

```

这段代码定义了一个名为`initConfig`的函数，它接受一个路径参数`path`和一个指向`config.Config`类型的参数`conf`。函数的作用是在初始化配置文件时执行的一系列操作。

首先，函数检查`configIsInitialized`函数是否已经成功初始化配置文件。如果已经初始化成功，函数直接返回`nil`，否则会进一步处理错误。

然后，函数尝试从配置文件中读取一个配置文件名，并返回给调用者一个非`nil`的错误。如果配置文件读取失败，函数会记录错误并返回。

接下来，函数调用`serialize.WriteConfigFile`函数，将当前的`config.Config`类型内容写入配置文件。如果写入失败，函数会记录错误并返回。

最后，函数会检查调用者是否已经初始化过配置文件。如果是，函数直接返回`nil`，否则会处理错误并返回。


```go
func initConfig(path string, conf *config.Config) error {
	if configIsInitialized(path) {
		return nil
	}
	configFilename, err := config.Filename(path, "")
	if err != nil {
		return err
	}
	// initialization is the one time when it's okay to write to the config
	// without reading the config from disk and merging any user-provided keys
	// that may exist.
	if err := serialize.WriteConfigFile(configFilename, conf); err != nil {
		return err
	}

	return nil
}

```

此代码定义了一个名为`initSpec`的函数，接受两个参数：`path`字符串和`conf` map对象。函数的作用是初始化指定的存储配置文件。

具体来说，函数首先使用`config.Path`函数将指定的路径作为参数传递给`specFn`函数，然后检查返回值是否为`nil`。如果是，说明初始化成功，否则继续执行下面的操作。

接着，函数判断文件是否存在，如果文件存在，则返回`nil`，否则继续执行下面的操作。

然后，函数使用`AnyDatastoreConfig`函数的`conf`参数，根据指定的存储配置文件创建一个`dsc` map对象。

接着，函数使用`dsc.DiskSpec().Bytes()`方法返回指定的DSC（磁盘规格）的磁盘规格字节数组。

最后，函数使用`os.WriteFile`函数将指定的文件写入指定的路径，并设置文件权限为0600。如果指定的路径不存在，则返回错误。


```go
func initSpec(path string, conf map[string]interface{}) error {
	fn, err := config.Path(path, specFn)
	if err != nil {
		return err
	}

	if util.FileExists(fn) {
		return nil
	}

	dsc, err := AnyDatastoreConfig(conf)
	if err != nil {
		return err
	}
	bytes := dsc.DiskSpec().Bytes()

	return os.WriteFile(fn, bytes, 0o600)
}

```

此代码的作用是初始化一个名为"myfsrepo"的新FSRepo，该Repo使用提供的配置文件初始化。主要步骤如下：

1. 确保Repo没有在运行时初始化多次，需要使用packageLock来确保Repo只被初始化一次。
2. 检查Repo是否已初始化过，如果已初始化过，则返回，否则继续执行下一步。
3. 初始化配置文件。
4. 初始化数据存储规范。
5. 将Repo版本写入migrations.WriteRepoVersion函数。
6. 返回是否有错误。


```go
// Init initializes a new FSRepo at the given path with the provided config.
// TODO add support for custom datastores.
func Init(repoPath string, conf *config.Config) error {
	// packageLock must be held to ensure that the repo is not initialized more
	// than once.
	packageLock.Lock()
	defer packageLock.Unlock()

	if isInitializedUnsynced(repoPath) {
		return nil
	}

	if err := initConfig(repoPath, conf); err != nil {
		return err
	}

	if err := initSpec(repoPath, conf.Datastore.Spec); err != nil {
		return err
	}

	if err := migrations.WriteRepoVersion(repoPath, RepoVersion); err != nil {
		return err
	}

	return nil
}

```

这段代码定义了一个名为`LockedByOtherProcess`的函数，它用于检查一个文件系统挂载点（FSRepo）是否被另一个进程锁定。如果锁定状态为true，那么这个FSRepo将无法被当前进程打开。函数有两个参数：`repoPath`表示需要检查的FSRepo的路径，` locked`和` err`是整型变量，用于存储FSRepo的状态和错误信息。

函数首先将`repoPath`使用`filepath.Clean`函数处理，然后使用`lockfile.Locked`函数检查FSRepo是否被锁定。如果锁定状态为true，函数将打印一条日志消息，并返回`locked`和`err`的值。

函数的第二个参数`APIAddr`返回了注册的API地址，根据该地址可以访问FSRepo中存储的API数据。由于这是一项并发操作，因此任何能够读取或写入该文件系统的进程都应该使用`mv`命令来覆盖整个文件，而不是对文件进行修改，以避免读写冲突。


```go
// LockedByOtherProcess returns true if the FSRepo is locked by another
// process. If true, then the repo cannot be opened by this process.
func LockedByOtherProcess(repoPath string) (bool, error) {
	repoPath = filepath.Clean(repoPath)
	locked, err := lockfile.Locked(repoPath, LockFile)
	if locked {
		log.Debugf("(%t)<->Lock is held at %s", locked, repoPath)
	}
	return locked, err
}

// APIAddr returns the registered API addr, according to the api file
// in the fsrepo. This is a concurrent operation, meaning that any
// process may read this file. modifying this file, therefore, should
// use "mv" to replace the whole file and avoid interleaved read/writes.
```

这段代码是一个名为APIAddr的函数，它接受一个名为repoPath的参数，并返回一个名为ma.Multiaddr的元组，或者一个名为error的错误。函数的作用是通过读取API文件并检查其大小来创建一个API地址。

函数首先将repoPath通过filepath.Clean函数进行清理，然后使用filepath.Join函数将API文件路径与repo路径合并。接着，函数尝试打开API文件并读取其内容。如果打开API文件时出现错误，则函数可能会返回错误，并指出原因。

如果API文件读取成功，函数会从文件中读取内容并将其转换为字符串，然后使用string的TrimSpace函数将其截去多余的空格，最后创建一个名为ma.Multiaddr的多地址类型。函数的实现遵循了安全编程实践的建议，避免了潜在的错误和攻击。


```go
func APIAddr(repoPath string) (ma.Multiaddr, error) {
	repoPath = filepath.Clean(repoPath)
	apiFilePath := filepath.Join(repoPath, apiFile)

	// if there is no file, assume there is no api addr.
	f, err := os.Open(apiFilePath)
	if err != nil {
		if os.IsNotExist(err) {
			return nil, repo.ErrApiNotRunning
		}
		return nil, err
	}
	defer f.Close()

	// read up to 2048 bytes. io.ReadAll is a vulnerability, as
	// someone could hose the process by putting a massive file there.
	//
	// NOTE(@stebalien): @jbenet probably wasn't thinking straight when he
	// wrote that comment but I'm leaving the limit here in case there was
	// some hidden wisdom. However, I'm fixing it such that:
	// 1. We don't read too little.
	// 2. We don't truncate and succeed.
	buf, err := io.ReadAll(io.LimitReader(f, 2048))
	if err != nil {
		return nil, err
	}
	if len(buf) == 2048 {
		return nil, fmt.Errorf("API file too large, must be <2048 bytes long: %s", apiFilePath)
	}

	s := string(buf)
	s = strings.TrimSpace(s)
	return ma.NewMultiaddr(s)
}

```

该代码定义了两个函数：一个是 `Keystore()`，另一个是 `Path()`。

`Keystore()` 函数接收一个名为 `FSRepo` 的 `*FSRepo` 类型的参数，并返回该对象的 `keystore` 引用。

`Path()` 函数接收一个名为 `FSRepo` 的 `*FSRepo` 类型的参数，并返回该对象的 `path` 字段。

此外，该代码中还有一条注释，指出 `SetAPIAddr()` 函数的作用是：将一个 `ma.Multiaddr` 类型的参数写入到 `/api` 文件中。


```go
func (r *FSRepo) Keystore() keystore.Keystore {
	return r.keystore
}

func (r *FSRepo) Path() string {
	return r.path
}

// SetAPIAddr writes the API Addr to the /api file.
func (r *FSRepo) SetAPIAddr(addr ma.Multiaddr) error {
	// Create a temp file to write the address, so that we don't leave empty file when the
	// program crashes after creating the file.
	f, err := os.Create(filepath.Join(r.path, "."+apiFile+".tmp"))
	if err != nil {
		return err
	}

	if _, err = f.WriteString(addr.String()); err != nil {
		return err
	}
	if err = f.Close(); err != nil {
		return err
	}

	// Atomically rename the temp file to the correct file name.
	if err = os.Rename(filepath.Join(r.path, "."+apiFile+".tmp"), filepath.Join(r.path,
		apiFile)); err == nil {
		return nil
	}
	// Remove the temp file when rename return error
	if err1 := os.Remove(filepath.Join(r.path, "."+apiFile+".tmp")); err1 != nil {
		return fmt.Errorf("file Rename error: %s, file remove error: %s", err.Error(),
			err1.Error())
	}
	return err
}

```

这段代码的作用是向名为 `/gateway` 的文件中写入一个网络地址的 Gateway 地址。

代码中，首先创建一个临时文件 `gatewayFile` 和一个名为 `tmpPath` 的文件，用于存储 Gateway 地址。然后使用 `os.Create` 函数创建一个临时文件，并使用 `fmt.Fprintf` 函数将 Gateway 地址写入到文件中。如果创建和写入文件的过程中出现错误，代码会返回相应的错误。

接着，代码会使用 `defer` 关键字来延迟一些操作，例如在文件写入完成后延迟关闭文件，以及在文件删除失败时延迟删除文件。

最后，如果所有的写入和读取操作都成功完成，代码会返回 `nil` 表示没有错误。否则，代码会返回一些错误信息，例如文件重命名错误、文件无法删除时的错误。


```go
// SetGatewayAddr writes the Gateway Addr to the /gateway file.
func (r *FSRepo) SetGatewayAddr(addr net.Addr) error {
	// Create a temp file to write the address, so that we don't leave empty file when the
	// program crashes after creating the file.
	tmpPath := filepath.Join(r.path, "."+gatewayFile+".tmp")
	f, err := os.Create(tmpPath)
	if err != nil {
		return err
	}
	var good bool
	// Silently remove as worst last case with defers.
	defer func() {
		if !good {
			os.Remove(tmpPath)
		}
	}()
	defer f.Close()

	if _, err := fmt.Fprintf(f, "http://%s", addr.String()); err != nil {
		return err
	}
	if err := f.Close(); err != nil {
		return err
	}

	// Atomically rename the temp file to the correct file name.
	err = os.Rename(tmpPath, filepath.Join(r.path, gatewayFile))
	good = err == nil
	if good {
		return nil
	}
	// Remove the temp file when rename return error
	if err1 := os.Remove(tmpPath); err1 != nil {
		return fmt.Errorf("file Rename error: %w, file remove error: %s", err, err1.Error())
	}
	return err
}

```

这段代码的作用是FSRepo类的两个方法，分别是openConfig和openUserResourceOverrides。

1. openConfig()函数，如果配置文件不存在，则会抛出error。该函数首先从给定的configFilePath读取配置文件，如果读取失败，则返回error。然后将读取到的配置文件存储在r.config变量中，并返回0。

2. openUserResourceOverrides()函数，如果文件不存在，则会抛出error。该函数读取名为"libp2p-resource-limit-overrides.json"的文件，并从该文件中读取所有的overrides，如果读取失败，则返回error。如果文件存在，则将文件读取的overrides存储在r.userResourceOverrides变量中，并返回0。

注意，以上解释中提到的文件名libp2p-resource-limit-overrides.json在给定的代码中是固定的，如果文件名发生改变，需要相应地进行修改。


```go
// openConfig returns an error if the config file is not present.
func (r *FSRepo) openConfig() error {
	conf, err := serialize.Load(r.configFilePath)
	if err != nil {
		return err
	}
	r.config = conf
	return nil
}

// openUserResourceOverrides will remove all overrides if the file is not present.
// It will error if the decoding fails.
func (r *FSRepo) openUserResourceOverrides() error {
	// This filepath is documented in docs/libp2p-resource-management.md and be kept in sync.
	err := serialize.ReadConfigFile(filepath.Join(r.path, "libp2p-resource-limit-overrides.json"), &r.userResourceOverrides)
	if errors.Is(err, serialize.ErrNotInitialized) {
		err = nil
	}
	return err
}

```

This is a Go file that defines a FSRepo struct which represents a Kubernetes DatastoreFS repository.

The struct has the following fields:

* r: a reference to the keystore file.
* ks: a Keystore FS element.
* config: a DatastoreSpec object containing the configuration for the Datastore.
* datastore: a DatastoreFS element representing the Datastore.

The struct has the following methods:

* openDatastore: returns an error if the Datastore configuration file is not present.
* readSpec: returns the DatastoreSpec object for reading the Datastore's configuration.

The openDatastore method reads the DatastoreSpec file and attempts to read the configuration from it. If the file is not found, an error is returned. If the file is found, the method returns no error.

The readSpec method reads the DatastoreSpec file and returns a DatastoreSpec object.

The main function of the struct initializes the Datastore and sets the keystore to it. It also wraps the Datastore with metrics gathering.


```go
func (r *FSRepo) openKeystore() error {
	ksp := filepath.Join(r.path, "keystore")
	ks, err := keystore.NewFSKeystore(ksp)
	if err != nil {
		return err
	}

	r.keystore = ks

	return nil
}

// openDatastore returns an error if the config file is not present.
func (r *FSRepo) openDatastore() error {
	if r.config.Datastore.Type != "" || r.config.Datastore.Path != "" {
		return fmt.Errorf("old style datatstore config detected")
	} else if r.config.Datastore.Spec == nil {
		return fmt.Errorf("required Datastore.Spec entry missing from config file")
	}
	if r.config.Datastore.NoSync {
		log.Warn("NoSync is now deprecated in favor of datastore specific settings. If you want to disable fsync on flatfs set 'sync' to false. See https://github.com/ipfs/kubo/blob/master/docs/datastores.md#flatfs.")
	}

	dsc, err := AnyDatastoreConfig(r.config.Datastore.Spec)
	if err != nil {
		return err
	}
	spec := dsc.DiskSpec()

	oldSpec, err := r.readSpec()
	if err != nil {
		return err
	}
	if oldSpec != spec.String() {
		return fmt.Errorf("datastore configuration of '%s' does not match what is on disk '%s'",
			oldSpec, spec.String())
	}

	d, err := dsc.Create(r.path)
	if err != nil {
		return err
	}
	r.ds = d

	// Wrap it with metrics gathering
	prefix := "ipfs.fsrepo.datastore"
	r.ds = measure.New(prefix, r.ds)

	return nil
}

```

这两函数的作用是读取文件并返回其内容类型及错误。

第一个函数 `readSpec` 接收一个 `FSRepo` 类型的参数 `r`，并使用它路径中的 `config.Path` 函数配置 `specFn`。如果配置成功，它将返回一个字符串表示读取文件的内容，或者是错误。如果读取文件时出现错误，函数将返回一个错误对象。

第二个函数 `Close` 同样接收一个 `FSRepo` 类型的参数 `r`，并使用它路径中的 `Close` 函数关闭 `FSRepo`，释放所有持有的资源。如果 `FSRepo` 已经是关闭状态，函数将返回一个错误。函数将在调用时确保所有持有的资源都已释放，并将其状态设置为 `closed`。


```go
func (r *FSRepo) readSpec() (string, error) {
	fn, err := config.Path(r.path, specFn)
	if err != nil {
		return "", err
	}
	b, err := os.ReadFile(fn)
	if err != nil {
		return "", err
	}
	return strings.TrimSpace(string(b)), nil
}

// Close closes the FSRepo, releasing held resources.
func (r *FSRepo) Close() error {
	packageLock.Lock()
	defer packageLock.Unlock()

	if r.closed {
		return errors.New("repo is closed")
	}

	err := os.Remove(filepath.Join(r.path, apiFile))
	if err != nil && !os.IsNotExist(err) {
		log.Warn("error removing api file: ", err)
	}

	err = os.Remove(filepath.Join(r.path, gatewayFile))
	if err != nil && !os.IsNotExist(err) {
		log.Warn("error removing gateway file: ", err)
	}

	if err := r.ds.Close(); err != nil {
		return err
	}

	// This code existed in the previous versions, but
	// EventlogComponent.Close was never called. Preserving here
	// pending further discussion.
	//
	// TODO It isn't part of the current contract, but callers may like for us
	// to disable logging once the component is closed.
	// logging.Configure(logging.Output(os.Stderr))

	r.closed = true
	return r.lockfile.Close()
}

```

这段代码是用来配置当前的配置文件。函数内部配置文件不会被复制，因此调用者不能在函数内部修改配置。同时，这个函数不会抛出错误，但是如果配置文件不能访问，会抛出一个不可预测的错误。


```go
// Config the current config. This function DOES NOT copy the config. The caller
// MUST NOT modify it without first calling `Clone`.
//
// Result when not Open is undefined. The method may panic if it pleases.
func (r *FSRepo) Config() (*config.Config, error) {
	// It is not necessary to hold the package lock since the repo is in an
	// opened state. The package lock is _not_ meant to ensure that the repo is
	// thread-safe. The package lock is only meant to guard against removal and
	// coordinate the lockfile. However, we provide thread-safety to keep
	// things simple.
	packageLock.Lock()
	defer packageLock.Unlock()

	if r.closed {
		return nil, errors.New("cannot access config, repo not open")
	}
	return r.config, nil
}

```

此函数名为`UserResourceOverrides`，定义在`func`内部。

该函数接收一个名为`r`的`FSRepo`结构体参数。

函数首先尝试获取`packageLock`，但由于`repo`处于打开状态，因此不需要获取`packageLock`。

接下来，函数检查`r`是否关闭。如果是，则返回`rcmgr.PartialLimitConfig`和错误`e`，否则返回`r.userResourceOverrides`和`nil`。

函数的作用是确保`r`处于关闭状态，并且允许函数安全地访问`rcmgr.PartialLimitConfig`类型。


```go
func (r *FSRepo) UserResourceOverrides() (rcmgr.PartialLimitConfig, error) {
	// It is not necessary to hold the package lock since the repo is in an
	// opened state. The package lock is _not_ meant to ensure that the repo is
	// thread-safe. The package lock is only meant to guard against removal and
	// coordinate the lockfile. However, we provide thread-safety to keep
	// things simple.
	packageLock.Lock()
	defer packageLock.Unlock()

	if r.closed {
		return rcmgr.PartialLimitConfig{}, errors.New("cannot access config, repo not open")
	}
	return r.userResourceOverrides, nil
}

```

这段代码定义了两个函数，分别作用于一个名为FSRepo的FSM�类型变量r上。

第一个函数是FileManager，它返回一个指向filestore.FileManager类型的指针。函数内部并没有做任何修改，它只是返回了r.filemgr，其中r是FSRepo类型的引用，filemgr是FSRepo中一个名为filemgr的成员变量。

第二个函数是BackupConfig，它接收一个前缀字符串prefix，然后返回一个配置文件名和一个错误。函数内部创建一个临时文件temp，然后尝试打开一个名为r.configFilePath的文件，并尝试读取和写入该文件。如果打开文件或创建文件的过程出现错误，函数将返回一个错误对象。然后，函数将原始文件的内容复制到temp文件中。最后，函数返回原始文件的名称，以及一个 nil的错误对象。


```go
func (r *FSRepo) FileManager() *filestore.FileManager {
	return r.filemgr
}

func (r *FSRepo) BackupConfig(prefix string) (string, error) {
	temp, err := os.CreateTemp(r.path, "config-"+prefix)
	if err != nil {
		return "", err
	}
	defer temp.Close()

	orig, err := os.OpenFile(r.configFilePath, os.O_RDONLY, 0o600)
	if err != nil {
		return "", err
	}
	defer orig.Close()

	_, err = io.Copy(temp, orig)
	if err != nil {
		return "", err
	}

	return orig.Name(), nil
}

```

This is a Go class that manages a file system (FS) repository's configuration. The class has a `SetConfig` method that takes a `config.Config` object and writes the updated configuration to a file. The file is written with the `writeConfigFile` function.

TheFSRepo class maintains a `configFilePath` field to store the path to the file containing the user's configuration. The class also holds a `map[string]interface{}` field for storing the configurations.

The `SetConfig` method first acquires a lock to ensure thread safety and reads the config file from disk as a map. It then converts the config file to a map and merges it with the new config object.

Finally, the method writes the updated config to the file using the `writeConfigFile` function.

The FSRepo class warns users not to modify the config object after calling the `SetConfig` method. It also emphasizes that storing non-user-generated Go config structures as user-generated JSON nested maps is not allowed.


```go
// SetConfig updates the FSRepo's config. The user must not modify the config
// object after calling this method.
// FIXME: There is an inherent contradiction with storing non-user-generated
// Go config.Config structures as user-generated JSON nested maps. This is
// evidenced by the issue of `omitempty` property of fields that aren't defined
// by the user and Go still needs to initialize them to its default (which
// is not reflected in the repo's config file, see
// https://github.com/ipfs/kubo/issues/8088 for more details).
// In general we should call this API with a JSON nested maps as argument
// (`map[string]interface{}`). Many calls to this function are forced to
// synthesize the config.Config struct from their available JSON map just to
// satisfy this (causing incompatibilities like the `omitempty` one above).
// We need to comb SetConfig calls and replace them when possible with a
// JSON map variant.
func (r *FSRepo) SetConfig(updated *config.Config) error {
	// packageLock is held to provide thread-safety.
	packageLock.Lock()
	defer packageLock.Unlock()

	// to avoid clobbering user-provided keys, must read the config from disk
	// as a map, write the updated struct values to the map and write the map
	// to disk.
	var mapconf map[string]interface{}
	if err := serialize.ReadConfigFile(r.configFilePath, &mapconf); err != nil {
		return err
	}
	m, err := config.ToMap(updated)
	if err != nil {
		return err
	}
	mergedMap := common.MapMergeDeep(mapconf, m)
	if err := serialize.WriteConfigFile(r.configFilePath, mergedMap); err != nil {
		return err
	}
	// Do not use `*r.config = ...`. This will modify the *shared* config
	// returned by `r.Config`.
	r.config = updated
	return nil
}

```

此代码的作用是实现一个名为`GetConfigKey`的函数，用于获取指定键的配置文件中的值。函数内部首先获取一个名为`packageLock`的锁，以确保在函数中可以对`packageLock`进行读取和设置。然后，它打开一个名为`configFilePath`的配置文件路径，并使用`serialize.ReadConfigFile`函数将配置文件的内容读取到一个名为`cfg`的 map对象中。如果函数在读取配置文件时遇到错误，它将返回一个`nil`值，并使用`errors.New`函数返回一个错误。

接下来，函数使用`common.MapGetKV`函数从`cfg` map对象中获取指定键的值。如果配置文件存在且配置文件中的键是有效的配置键，函数将返回该配置文件的值和`nil`。如果配置文件不存在或键不是有效的配置键，函数将返回一个`nil`错误。


```go
// GetConfigKey retrieves only the value of a particular key.
func (r *FSRepo) GetConfigKey(key string) (interface{}, error) {
	packageLock.Lock()
	defer packageLock.Unlock()

	if r.closed {
		return nil, errors.New("repo is closed")
	}

	var cfg map[string]interface{}
	if err := serialize.ReadConfigFile(r.configFilePath, &cfg); err != nil {
		return nil, err
	}
	return common.MapGetKV(cfg, key)
}

```

这段代码的作用是设置一个名为“myConfig”的配置键的值。具体的实现过程如下：

1. 首先，代码会尝试读取并返回一个名为“myConfig”的配置文件，如果遇到错误，则会返回。
2. 接着，如果当前的Repo实例是关闭的，那么会抛出异常并输出错误信息。
3. 读取并加载了包含“myConfig”键的配置文件中的数据。
4. 尝试从配置文件中加载一个名为“privateKeySelector”的私有密钥，如果加载失败，则会抛出错误并输出错误信息。
5. 如果配置文件中包含名为“privateKeySelector”的键，那么将其设置为传入的值。
6. 如果配置文件中包含名为“myConfig”的键，那么将其设置为从配置文件中读取到的数据。
7. 最后，如果配置文件中的数据无法成功设置，则会抛出错误并输出错误信息。
8. 如果所有的步骤都成功完成，则返回 nil。


```go
// SetConfigKey writes the value of a particular key.
func (r *FSRepo) SetConfigKey(key string, value interface{}) error {
	packageLock.Lock()
	defer packageLock.Unlock()

	if r.closed {
		return errors.New("repo is closed")
	}

	// Load into a map so we don't end up writing any additional defaults to the config file.
	var mapconf map[string]interface{}
	if err := serialize.ReadConfigFile(r.configFilePath, &mapconf); err != nil {
		return err
	}

	// Load private key to guard against it being overwritten.
	// NOTE: this is a temporary measure to secure this field until we move
	// keys out of the config file.
	pkval, err := common.MapGetKV(mapconf, config.PrivKeySelector)
	if err != nil {
		return err
	}

	// Set the key in the map.
	if err := common.MapSetKV(mapconf, key, value); err != nil {
		return err
	}

	// replace private key, in case it was overwritten.
	if err := common.MapSetKV(mapconf, config.PrivKeySelector, pkval); err != nil {
		return err
	}

	// This step doubles as to validate the map against the struct
	// before serialization
	conf, err := config.FromMap(mapconf)
	if err != nil {
		return err
	}
	r.config = conf

	if err := serialize.WriteConfigFile(r.configFilePath, mapconf); err != nil {
		return err
	}

	return nil
}

```

这段代码定义了一个名为FSRepo的 struct，它是一个Datastore类型的变量。它的作用是返回一个repo-owned的datastore。

如果FSRepo所代表的FSRepo已经关闭，则函数Datastore()将返回undefined。

此外，还定义了一个名为GetStorageUsage的函数，它计算了FSRepo在bytes中的存储空间占用量。

最后一个函数是名为SwarmKey的函数，它返回一个由字节组成的切片和一个error。它使用Fsrepo.path计算一个swarm key文件，并使用计算存储空间占用的函数计算所需的存储空间，然后返回它们。如果计算存储空间占用的函数返回错误，则SwarmKey函数将返回nil。


```go
// Datastore returns a repo-owned datastore. If FSRepo is Closed, return value
// is undefined.
func (r *FSRepo) Datastore() repo.Datastore {
	packageLock.Lock()
	d := r.ds
	packageLock.Unlock()
	return d
}

// GetStorageUsage computes the storage space taken by the repo in bytes.
func (r *FSRepo) GetStorageUsage(ctx context.Context) (uint64, error) {
	return ds.DiskUsage(ctx, r.Datastore())
}

func (r *FSRepo) SwarmKey() ([]byte, error) {
	repoPath := filepath.Clean(r.path)
	spath := filepath.Join(repoPath, swarmKeyFile)

	f, err := os.Open(spath)
	if err != nil {
		if os.IsNotExist(err) {
			err = nil
		}
		return nil, err
	}
	defer f.Close()

	return io.ReadAll(f)
}

```

这段代码定义了一个名为FSRepo的结构体，并创建了两个指向FSRepo实例的引用。然后，它定义了一个名为IsInitialized的函数，该函数使用闭包来确保在函数调用期间只有一个FSRepo实例被初始化。最后，它提供了一个判断FSRepo实例是否已初始化的函数，如果初始化成功则返回true，否则返回false。


```go
var (
	_ io.Closer = &FSRepo{}
	_ repo.Repo = &FSRepo{}
)

// IsInitialized returns true if the repo is initialized at provided |path|.
func IsInitialized(path string) bool {
	// packageLock is held to ensure that another caller doesn't attempt to
	// Init or Remove the repo while this call is in progress.
	packageLock.Lock()
	defer packageLock.Unlock()

	return isInitializedUnsynced(path)
}

```

这段代码定义了一个名为`isInitializedUnsynced`的私有方法，该方法接收一个`repoPath`参数，用于判断是否已初始化并获取锁。该方法的作用是报告给调用者，初始化状态是否同步。

在该方法中，首先调用一个名为`configIsInitialized`的私有方法，获取repo是否已初始化。如果初始化成功，则返回`true`，否则返回`false`。最后，该方法使用`reportError`方法将初始化状态报告给调用者，如果初始化失败，则使用`makeDirect`方法输出错误信息。

需要注意的是，该方法有一个保护锁`packageLock`，并通过调用者的` hold`方法来确保该锁在调用者退出时被释放。


```go
// private methods below this point. NB: packageLock must held by caller.

// isInitializedUnsynced reports whether the repo is initialized. Caller must
// hold the packageLock.
func isInitializedUnsynced(repoPath string) bool {
	return configIsInitialized(repoPath)
}

```

# `repo/fsrepo/fsrepo_test.go`

这段代码定义了一个名为 "fsrepo" 的包，它导入了以下外部库：

- "bytes": 字节字节流传输缓冲区
- "context": 上下文
- "os": 操作系统
- "path/filepath": 文件路径
- "testing": testing 包
- "github.com/ipfs/kubo/thirdparty/assert": 假设 ipfs/kubo库
- "github.com/ipfs/go-datastore": 假设 ipfs/go-datastore库
- "github.com/ipfs/kubo/config": 假设 ipfs/kubo库的配置

然后，在 "fsrepo" 包内部，定义了一些变量和函数，包括：

- "datastore.Query" 类型定义了查询语句，用于从存储器中获取数据
- "datastore.Values" 类型定义了查询结果的数据值
- "fsrepo.Node" 类型定义了表示文件系统的节点
- "fsrepo.File" 类型定义了表示文件的节点
- "fsrepo.Branch" 类型定义了表示目录分支的节点
- "fsrepo.目录" 类型定义了表示目录的节点
- "fsrepo.文件系统配置" 函数，用于设置 fsrepo 配置
- "fsrepo.的目录配置" 函数，用于设置 fsrepo 目录配置
- "fsrepo.子目录配置" 函数，用于设置 fsrepo 子目录配置
- "fsrepo.的子目录配置" 函数，用于设置 fsrepo 子目录配置
- "fsrepo.子节点配置" 函数，用于设置 fsrepo 子节点配置
- "fsrepo.访问权限" 函数，用于设置文件的访问权限
- "fsrepo.写权限" 函数，用于设置文件的写权限
- "fsrepo.读权限" 函数，用于设置文件的读权限
- "fsrepo.大小限制" 函数，用于设置文件的大小限制
- "fsrepo.超时" 函数，用于设置文件超时时间
- "fsrepo.的目录统计" 函数，用于设置 fsrepo 目录统计
- "fsrepo.子目录统计" 函数，用于设置 fsrepo 子目录统计
- "fsrepo.子节点统计" 函数，用于设置 fsrepo 子节点统计
- "fsrepo.文件的统计" 函数，用于设置 fsrepo 文件的统计
- "fsrepo.扫描" 函数，用于扫描目录树
- "fsrepo.的目录扫描" 函数，用于设置 fsrepo 目录扫描
- "fsrepo.的子目录扫描" 函数，用于设置 fsrepo 子目录扫描
- "fsrepo.的子节点扫描" 函数，用于设置 fsrepo 子节点扫描
- "fsrepo.的文件统计" 函数，用于设置 fsrepo 文件的统计
- "fsrepo.的目录遍历" 函数，用于设置 fsrepo 目录遍历
- "fsrepo.的子目录遍历" 函数，用于设置 fsrepo 子目录遍历
- "fsrepo.的文件遍历" 函数，用于设置 fsrepo 文件遍历
- "fsrepo.的目录并集" 函数，用于设置 fsrepo 目录并集
- "fsrepo.的目录交集" 函数，用于设置 fsrepo 目录交集
- "fsrepo.的目录差集" 函数，用于设置 fsrepo 目录差集
- "fsrepo.的目录聚类" 函数，用于设置 fsrepo 目录聚类
- "fsrepo.的子目录并集" 函数，用于设置 fsrepo 子目录并集
- "fsrepo.的子目录交集" 函数，用于设置 fsrepo 子目录交集
- "fsrepo.的子目录聚类" 函数，用于设置 fsrepo 子目录聚类
- "fsrepo.文件系统的配置" 函数，用于设置 fsrepo 配置
- "fsrepo.的目录配置" 函数，用于设置 fsrepo 目录配置
- "fsrepo.的子目录配置" 函数，用于设置 fsrepo 子目录配置
- "fsrepo.的子节点配置" 函数，用于设置 fsrepo 子节点配置
- "fsrepo.访问权限" 函数，用于设置文件的访问权限
- "fsrepo.写权限" 函数，用于设置文件的写权限
- "fsrepo.读权限" 函数，用于设置文件的读权限
- "fsrepo.大小限制" 函数，用于设置文件的大小限制
- "fsrepo.超时" 函数，用于设置文件的超时时间
- "fsrepo.扫描目录" 函数，用于扫描目录树
- "fsrepo.目录统计" 函数，用于设置 fsrepo 目录统计
- "fsrepo.子目录统计" 函数，用于设置 fsrepo 子目录统计
- "fsrepo.子节点统计" 函数，用于设置 fsrepo 子节点统计
- "fsrepo.文件统计" 函数，用于设置 fsrepo 文件的统计
- "fsrepo.目录遍历" 函数，用于设置 fsrepo 目录遍历
- "fsrepo.子目录遍历" 函数，用于设置 fsrepo 子目录遍历
- "fsrepo.文件遍历" 函数，用于设置 fsrepo 文件遍历
- "fsrepo.目录并集" 函数，用于设置 fsrepo 目录并集
- "fsrepo.目录交集" 函数，用于设置 fsrepo 目录交集
- "fsrepo.目录差集" 函数，用于设置 fsrepo 目录差集
- "fsrepo.目录聚类" 函数，用于设置 fsrepo 目录聚类
- "fsrepo.子目录并集" 函数，用于设置 fsrepo 子目录并集
- "fsrepo.子目录交集" 函数，用于设置 fsrepo 子目录交集
- "fsrepo.子目录聚类" 函数，用于设置 fsrepo 子目录聚类
- "文件系统配置" 函数，用于设置 fsrepo 配置
- "目录配置" 函数，用于设置 fsrepo 目录配置
- "子目录配置" 函数，用于设置 fsrepo 子目录配置
- "子节点配置" 函数，用于设置 fsrepo 子节点配置
- "访问权限" 函数，用于设置文件的访问权限
- "写权限" 函数，用于设置文件的写权限
- "读权限" 函数，用于设置文件的读权限
- "大小限制" 函数，用于设置文件的大小限制
- "超时" 函数，用于设置文件的超时时间
- "扫描目录" 函数，用于扫描目录树
- "目录统计" 函数，用于设置 fsrepo 目录统计
- "子目录统计" 函数，用于设置 fsrepo 子目录统计
- "子节点统计" 函数，用于设置 fsrepo 子节点统计
- "文件统计" 函数，用于设置 fsrepo 文件的统计
- "目录遍历" 函数，用于设置 fsrepo 目录遍历
- "子目录遍历" 函数，用于设置 fsrepo 子目录遍历
- "文件遍历" 函数，用于设置 fsrepo 文件遍历
- "目录并集" 函数，用于设置 fsrepo 目录并集
- "目录交集" 函数，用于设置 fsrepo 目录交集
- "目录差集" 函数，用于设置 fsrepo 目录差集
- "目录聚类" 函数，用于设置 fsrepo 目录聚类
- "子目录并集" 函数，用于设置 fsrepo 子目录并集
- "子目录交集" 函数，用于设置 fsrepo 子目录交集
- "子目录聚类" 函数，用于设置 fsrep


```go
package fsrepo

import (
	"bytes"
	"context"
	"os"
	"path/filepath"
	"testing"

	"github.com/ipfs/kubo/thirdparty/assert"

	datastore "github.com/ipfs/go-datastore"
	config "github.com/ipfs/kubo/config"
)

```


func TestInitIdempotence(t *testing.T) {
	t.Parallel()
	path := t.TempDir()
	for i := 0; i < 10; i++ {
		assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "multiple calls to init should succeed")
	}
}

func Remove(repoPath string) error {
	repoPath = filepath.Clean(repoPath)
	return os.RemoveAll(repoPath)
}

func TestCanManageReposIndependently(t *testing.T) {
	t.Parallel()
	pathA := t.TempDir()
	pathB := t.TempDir()

	t.Log("initialize two repos")
	assert.Nil(Init(pathA, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "a", "should initialize successfully")
	assert.Nil(Init(pathB, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "b", "should initialize successfully")

	t.Log("ensure repos initialized")
	assert.True(IsInitialized(pathA), t, "a should be initialized")
	assert.True(IsInitialized(pathB), t, "b should be initialized")

	t.Log("open the two repos")
	repoA, err := Open(pathA)
	assert.Nil(err, t, "a")
	repoB, err := Open(pathB)
	assert.Nil(err, t, "b")

	t.Log("close and remove b while a is open")
	assert.Nil(repoB.Close(), t, "close b")
	assert.Nil(Remove(pathB), t, "remove b")

	t.Log("close and remove a")
	assert.Nil(repoA.Close(), t)
	assert.Nil(Remove(pathA), t, "close a")
}

This is the output of the `func TestInitIdempotence(t *testing.T)` function which is used to test the InitIdempotence of the `func Init(path string, config *config.Config)` function. This function creates a temporary directory, initializes a new SQL database, and returns whether the initialization process was successful. It also includes a test to remove the database in case of failure.


```go
func TestInitIdempotence(t *testing.T) {
	t.Parallel()
	path := t.TempDir()
	for i := 0; i < 10; i++ {
		assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "multiple calls to init should succeed")
	}
}

func Remove(repoPath string) error {
	repoPath = filepath.Clean(repoPath)
	return os.RemoveAll(repoPath)
}

func TestCanManageReposIndependently(t *testing.T) {
	t.Parallel()
	pathA := t.TempDir()
	pathB := t.TempDir()

	t.Log("initialize two repos")
	assert.Nil(Init(pathA, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "a", "should initialize successfully")
	assert.Nil(Init(pathB, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "b", "should initialize successfully")

	t.Log("ensure repos initialized")
	assert.True(IsInitialized(pathA), t, "a should be initialized")
	assert.True(IsInitialized(pathB), t, "b should be initialized")

	t.Log("open the two repos")
	repoA, err := Open(pathA)
	assert.Nil(err, t, "a")
	repoB, err := Open(pathB)
	assert.Nil(err, t, "b")

	t.Log("close and remove b while a is open")
	assert.Nil(repoB.Close(), t, "close b")
	assert.Nil(Remove(pathB), t, "remove b")

	t.Log("close and remove a")
	assert.Nil(repoA.Close(), t)
	assert.Nil(Remove(pathA), t)
}

```

这段代码是一个名为 `TestDatastoreGetNotAllowedAfterClose` 的函数测试。函数的作用是测试一个名为 `GetNotAllowedAfterClose` 的函数在关闭数据存储器后是否能够正常返回数据。以下是这段代码的解释：

1. 首先，函数创建了一个临时目录 `path`，并检查它是否已初始化。如果不初始化，函数会输出一个警告信息。
2. 然后，函数尝试初始化数据存储器。如果初始化成功，函数会输出一个警告信息。
3. 接下来，函数尝试打开数据存储器。如果这步成功，函数会继续下一步。
4. 在打开数据存储器后，函数尝试将一个键（`key`）和一些字节数据（`data`）存储到数据存储器中。如果存储成功，函数会继续下一步。
5. 然后，函数尝试关闭数据存储器。如果关闭成功，函数会继续下一步。
6. 在关闭数据存储器后，函数尝试再次获取存储的键。如果获取成功，函数会输出一个错误信息。如果之前的错误信息提示函数在关闭数据存储器后无法获取数据，函数不会输出任何错误信息。


```go
func TestDatastoreGetNotAllowedAfterClose(t *testing.T) {
	t.Parallel()
	path := t.TempDir()

	assert.True(!IsInitialized(path), t, "should NOT be initialized")
	assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t, "should initialize successfully")
	r, err := Open(path)
	assert.Nil(err, t, "should open successfully")

	k := "key"
	data := []byte(k)
	assert.Nil(r.Datastore().Put(context.Background(), datastore.NewKey(k), data), t, "Put should be successful")

	assert.Nil(r.Close(), t)
	_, err = r.Datastore().Get(context.Background(), datastore.NewKey(k))
	assert.Err(err, t, "after closer, Get should be fail")
}

```

这段代码的作用是测试数据存储库从一个存储库复制到另一个存储库是否正确。具体来说，它实现了以下操作：

1. 创建一个临时目录，用于存储数据。
2. 打开第一个存储库。
3. 将键 "key" 存储到第一个存储库中。
4. 使用第一个存储库读取数据，并将其存储到 "expected" 数组中。
5. 使用第一个存储库关闭。
6. 打开第二个存储库。
7. 使用第二个存储库读取数据，并将其存储到 "actual" 数组中。
8. 使用第二个存储库关闭。
9. 比较 "expected" 和 "actual" 数组是否相等。如果相等，说明数据已成功复制。


```go
func TestDatastorePersistsFromRepoToRepo(t *testing.T) {
	t.Parallel()
	path := t.TempDir()

	assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t)
	r1, err := Open(path)
	assert.Nil(err, t)

	k := "key"
	expected := []byte(k)
	assert.Nil(r1.Datastore().Put(context.Background(), datastore.NewKey(k), expected), t, "using first repo, Put should be successful")
	assert.Nil(r1.Close(), t)

	r2, err := Open(path)
	assert.Nil(err, t)
	actual, err := r2.Datastore().Get(context.Background(), datastore.NewKey(k))
	assert.Nil(err, t, "using second repo, Get should be successful")
	assert.Nil(r2.Close(), t)
	assert.True(bytes.Equal(expected, actual), t, "data should match")
}

```

这段代码是一个 Go 语言中的测试函数，名为 TestOpenMoreThanOnceInSameProcess。它的作用是测试两个在同一个进程中的 Go 语言资源库（repo）是否可以通过不同的路径打开并成功读取多个文件。

具体来说，这段代码实现了以下功能：

1. 创建一个临时目录（tempdir），并将其赋值给一个名为 path 的变量。
2. 调用一个名为 Init 的函数，该函数的作用是设置路径配置参数，包括设置 datastore 为默认值。
3. 分别使用 Open 函数以两个不同的路径尝试打开资源库。
4. 如果两个调用中的第一个 Open 函数成功，则确保第二个 Open 函数不会成功，从而触发测试失败。
5. 如果两个 Open 函数中的任意一个失败，则记录错误并继续测试下一个。
6. 如果两个 Open 函数都成功，则记录它们返回的值是否相同。
7. 关闭两个 Open 函数分别对资源库进行操作。


```go
func TestOpenMoreThanOnceInSameProcess(t *testing.T) {
	t.Parallel()
	path := t.TempDir()
	assert.Nil(Init(path, &config.Config{Datastore: config.DefaultDatastoreConfig()}), t)

	r1, err := Open(path)
	assert.Nil(err, t, "first repo should open successfully")
	r2, err := Open(path)
	assert.Nil(err, t, "second repo should open successfully")
	assert.True(r1 == r2, t, "second open returns same value")

	assert.Nil(r1.Close(), t)
	assert.Nil(r2.Close(), t)
}

```

# `repo/fsrepo/misc.go`

这段代码是一个名为 "fsrepo" 的包，它定义了一个名为 "BestKnownPath" 的函数，用于查找 fsrepo 存储库的最佳已知路径。

该函数首先从环境中读取 fsrepo 的默认路径根，如果环境中存在一个以 fsrepo 为前缀的变量，则该变量将覆盖默认路径根。否则，函数将使用 homedir 包中的 "Expand" 函数将 fsrepo 路径中的 ./ 目录及其子目录和子目录扩展为 fsrepo 允许的路径。

最后，函数返回最佳已知路径，如果不存在或出现错误，则返回 nil。


```go
package fsrepo

import (
	"os"

	config "github.com/ipfs/kubo/config"
	homedir "github.com/mitchellh/go-homedir"
)

// BestKnownPath returns the best known fsrepo path. If the ENV override is
// present, this function returns that value. Otherwise, it returns the default
// repo path.
func BestKnownPath() (string, error) {
	ipfsPath := config.DefaultPathRoot
	if os.Getenv(config.EnvDir) != "" {
		ipfsPath = os.Getenv(config.EnvDir)
	}
	ipfsPath, err := homedir.Expand(ipfsPath)
	if err != nil {
		return "", err
	}
	return ipfsPath, nil
}

```

# `repo/fsrepo/migrations/fetch.go`

这段代码定义了一个名为"migrations"的包，它使用了以下的一些库：

- `bufio`：一个用于字符数据的库，提供了文件输入/输出操作的功能。
- `bytes`：一个用于字节数据的库，提供了字节数据的操作功能。
- `fmt`：一个用于格式化字符数据的库，用于将结构化数据转换为字符串格式的功能。
- `io`：一个通用的输入/输出库，提供了用于操作系统交互的接口。
- `os`：一个操作系统接口的库，提供了与操作系统交互的功能。
- `path/filepath`：一个用于文件路径操作的库，提供了对路径解析等功能。
- `runtime`：一个用于运行时操作的库，提供了与计算机虚拟机交互的功能。
- `strings`：一个用于字符串操作的库，提供了字符串的 operations 功能。

这段代码的主要作用是提供一个包含了多个工具函数的包，用于进行数据迁移操作。这些工具函数可以用来将数据从一个地方迁移到另一个地方，例如将一个文件的内容从一个地方复制到另一个地方，或者在两个地方之间拆分/合并数据。具体来说，这些函数可以被用来执行以下操作：

- `create_input_channel()`：创建一个输入 channel，可以用来从文件或其他地方读取数据。
- `create_output_channel()`：创建一个输出 channel，可以用来将数据写入到文件或其他地方。
- `read_input_channel()`：读取一个输入 channel 中的数据。
- `write_output_channel()`：向一个输出 channel 写入数据。
- `split_data_by_line()`：将一个字符串按照行拆分成多个数据。
- `merge_data_by_line()`：将多个字符串合并成一个字符串。
- `count_data_by_line()`：统计一个字符串中每个数据出现的次数。


```go
package migrations

import (
	"bufio"
	"bytes"
	"context"
	"fmt"
	"io"
	"os"
	"os/exec"
	"path/filepath"
	"runtime"
	"strings"
)

```

这段代码定义了一个名为"DownloadDirectory"的变量，用于设置下载目录。如果未指定下载目录，则下载的归档文件会保存在一个临时目录中，该目录会在归档文件被解压后自动删除。

接下来，定义了一个名为"FetchBinary"的函数，该函数使用一个名为"ctx"的上下文对象和一个名为"fetcher"的函数，下载一个从分布网站下载的归档文件。函数的第一实参是一个字符串参数，表示二进制文件的完整名称，第二个实参是一个函数指针，用于指定二进制文件的临时文件名，该函数指针的参数包括一个字符串参数和一个表示tmpDir的整数参数。

通过调用FetchBinary函数，可以下载一个二进制文件并将其保存为归档文件。如果"DownloadDirectory"变量被设置为指定下载目录，则下载的归档文件将保存在该目录中。


```go
// DownloadDirectory can be set as the location for FetchBinary to save the
// downloaded archive file in.  If not set, then FetchBinary saves the archive
// in a temporary directory that is removed after the contents of the archive
// is extracted.
var DownloadDirectory string

// FetchBinary downloads an archive from the distribution site and unpacks it.
//
// The base name of the binary inside the archive may differ from the base
// archive name.  If it does, then specify binName.  For example, the following
// is needed because the archive "go-ipfs_v0.7.0_linux-amd64.tar.gz" contains a
// binary named "ipfs"
//
//	FetchBinary(ctx, fetcher, "go-ipfs", "v0.7.0", "ipfs", tmpDir)
//
```

This is a function that downloads an archive file from the IPFS (InterPlanetary File System) using the IPFS protocol, and extracts it to a local temporary directory. The function takes two arguments:

- `dist`: The download destination for the archive. This can be a path, a file name, or a URL.
- `arcName`: The name of the archive to be downloaded.
- `atype`: The format of the archive to be downloaded. It can be "tar.gz" or "zip".
- `out`: The file path to write the extracted archive data in.

The function first checks if the destination directory already exists. If it does, it returns an error. If it doesn't, it creates the directory using the `os.MkdirTemp` function, and then returns an error if it fails to create the directory.

If the directory exists and the archive format is correct, the function downloads the archive using the `fetcher.Fetch` function and extracts it to the temporary directory using the `os.Makedirs` and `os.RemoveAll` functions.

The function then creates a file to write the archive data to and opens a connection to download the archive from the IPFS path using the `fetcher.Fetch` function. The downloaded archive data is then copied to the file and the archive is extracted using the `unpackArchive` function. Finally, the archive is written to binary mode using the `os.Chmod` function, and the file is given the file mode 'executable'.

It is important to note that the function assumes that the IPFS URL passed by the user is valid and the archive is in the correct format.


```go
// If out is a directory, then the binary is written to that directory with the
// same name it has inside the archive.  Otherwise, the binary file is written
// to the file named by out.
func FetchBinary(ctx context.Context, fetcher Fetcher, dist, ver, binName, out string) (string, error) {
	// The archive file name is the base of dist. This is to support a possible subdir in
	// dist, for example: "ipfs-repo-migrations/fs-repo-11-to-12"
	arcName := filepath.Base(dist)
	// If binary base name is not specified, then it is same as archive base name.
	if binName == "" {
		binName = arcName
	}

	// Name of binary that exists inside archive
	binName = ExeName(binName)

	// Return error if file exists or stat fails for reason other than not
	// exists.  If out is a directory, then write extracted binary to that dir.
	fi, err := os.Stat(out)
	if !os.IsNotExist(err) {
		if err != nil {
			return "", err
		}
		if !fi.IsDir() {
			return "", &os.PathError{
				Op:   "FetchBinary",
				Path: out,
				Err:  os.ErrExist,
			}
		}
		// out exists and is a directory, so compose final name
		out = filepath.Join(out, binName)
		// Check if the binary already exists in the directory
		_, err = os.Stat(out)
		if !os.IsNotExist(err) {
			if err != nil {
				return "", err
			}
			return "", &os.PathError{
				Op:   "FetchBinary",
				Path: out,
				Err:  os.ErrExist,
			}
		}
	}

	tmpDir := DownloadDirectory
	if tmpDir != "" {
		fi, err = os.Stat(tmpDir)
		if err != nil {
			return "", err
		}
		if !fi.IsDir() {
			return "", &os.PathError{
				Op:   "FetchBinary",
				Path: tmpDir,
				Err:  os.ErrExist,
			}
		}
	} else {
		// Create temp directory to store download
		tmpDir, err = os.MkdirTemp("", arcName)
		if err != nil {
			return "", err
		}
		defer os.RemoveAll(tmpDir)
	}

	atype := "tar.gz"
	if runtime.GOOS == "windows" {
		atype = "zip"
	}

	arcDistPath, arcFullName := makeArchivePath(dist, arcName, ver, atype)

	// Create a file to write the archive data to
	arcPath := filepath.Join(tmpDir, arcFullName)
	arcFile, err := os.Create(arcPath)
	if err != nil {
		return "", err
	}
	defer arcFile.Close()

	// Open connection to download archive from ipfs path and write to file
	arcBytes, err := fetcher.Fetch(ctx, arcDistPath)
	if err != nil {
		return "", err
	}

	// Write download data
	_, err = io.Copy(arcFile, bytes.NewReader(arcBytes))
	if err != nil {
		return "", err
	}
	arcFile.Close()

	// Unpack the archive and write binary to out
	err = unpackArchive(arcPath, atype, dist, binName, out)
	if err != nil {
		return "", err
	}

	// Set mode of binary to executable
	err = os.Chmod(out, 0o755)
	if err != nil {
		return "", err
	}

	return out, nil
}

```

这段代码定义了一个名为 `osWithVariant` 的函数，它返回操作系统名称，可能带有一个可选的变体。函数首先检查运行时OS是否为 "linux"，如果是，则函数返回 "linux"，否则返回 "linux-musl"。

接下来，函数使用 `exec.Command` 函数运行 `"sh"` 命令，使用 `"-c"` 参数运行 `"ldd --version || true"` 命令，获得操作系统输出。函数将输出通过 `bufio.NewScanner` 读取并存储到一个字符缓冲区 `out` 中。

接着，函数使用 `bufio.NewScanner` 循环遍历 `out` 并查找是否有 "musl" 这个关键字。如果找到了 "musl"，则返回带有 "musl" 的操作系统名称，否则返回 "linux"。

最后，函数返回 "linux-musl" 或 "linux"，具体取决于是否找到了 "musl"。函数在返回前添加了一个空括号，这意味着它返回的是一个名为 "linux-musl" 或 "linux" 的字符串。


```go
// osWithVariant returns the OS name with optional variant.
// Currently returns either runtime.GOOS, or "linux-musl".
func osWithVariant() (string, error) {
	if runtime.GOOS != "linux" {
		return runtime.GOOS, nil
	}

	// ldd outputs the system's kind of libc.
	// - on standard ubuntu: ldd (Ubuntu GLIBC 2.23-0ubuntu5) 2.23
	// - on alpine: musl libc (x86_64)
	//
	// we use the combined stdout+stderr,
	// because ldd --version prints differently on different OSes.
	// - on standard ubuntu: stdout
	// - on alpine: stderr (it probably doesn't know the --version flag)
	//
	// we suppress non-zero exit codes (see last point about alpine).
	out, err := exec.Command("sh", "-c", "ldd --version || true").CombinedOutput()
	if err != nil {
		return "", err
	}

	// now just see if we can find "musl" somewhere in the output
	scan := bufio.NewScanner(bytes.NewBuffer(out))
	for scan.Scan() {
		if strings.Contains(scan.Text(), "musl") {
			return "linux-musl", nil
		}
	}

	return "linux", nil
}

```

这段代码定义了一个名为 `makeArchivePath` 的函数，它接受四个参数：`dist`，`name`，`ver` 和 `atype`。函数的作用是将传入的四个参数使用指定的格式合并成一个新的字符串，该字符串作为下载二进制的目的路径。

具体来说，函数的实现首先从 `dist` 参数中提取一个字符串，然后从该字符串中提取版本号、架构和操作系统，并将它们拼接到 `name` 和 `ver` 参数上，再将它们和 `atype` 合并为一个字符串。最后，函数使用 `fmt.Sprintf` 函数将合并后的字符串格式化为一个新的字符串，该字符串作为下载二进制的目的路径。函数的返回值是目的路径和文件名。


```go
// makeArchivePath composes the path, relative to the distribution site, from which to
// download a binary.  The path returned does not contain the distribution site path,
// e.g. "/ipns/dist.ipfs.tech/", since that is know to the fetcher.
//
// Returns the archive path and the base name.
//
// The ipfs path format is: distribution/version/archiveName
//   - distribution is the name of a distribution, such as "go-ipfs"
//   - version is the version to fetch, such as "v0.8.0-rc2"
//   - archiveName is formatted as name_version_osv-GOARCH.atype, such as
//     "go-ipfs_v0.8.0-rc2_linux-amd64.tar.gz"
//
// This would form the path:
// go-ipfs/v0.8.0/go-ipfs_v0.8.0_linux-amd64.tar.gz
func makeArchivePath(dist, name, ver, atype string) (string, string) {
	arcName := fmt.Sprintf("%s_%s_%s-%s.%s", name, ver, runtime.GOOS, runtime.GOARCH, atype)
	return fmt.Sprintf("%s/%s/%s", dist, ver, arcName), arcName
}

```