# go-ipfs 源码解析 44

# `/opt/kubo/fuse/node/mount_darwin.go`

这段代码是一个 Go 语言中的声明，它将导出一个名为 "node" 的 package。通过执行以下命令，Go 编译器会将其构建为独立的 Go 包：


go build !nofuse


这个命令将编译所有以 "node" 为前缀的 Go 源文件。`!nofuse` 是选项参数，用于告诉编译器不要在输出中包含任何 unused 的输出。

生成的可执行文件将包含名为 "node_main.js" 的文件，它导出了一个名为 "node" 的包。同时，还会生成一个名为 "node_output.txt" 的文件，用于输出编译器的输出。

这个代码片段定义了一个名为 "node" 的包，它导出了一个名为 "main" 的函数，该函数执行以下操作：

1. 下载并打印 kubernetes.gen.repo 和 kubernetes.time. yaml 文件的日月量 (奇偶) 值。
2. 通过调用 ipfs.gz 函数，将 kubernetes.gen.repo 和 kubernetes.time. yaml 文件内容存储为 bytes 切片，并将其解压缩。
3. 打印文件的哈希值。

代码分析：

该代码片段使用 Go 的包系统来声明一个名为 "node" 的包，然后在 Go 编译器中执行以下操作：

1. 从主网域下载 kubernetes.gen.repo 和 kubernetes.time. yaml 文件。

2. 通过调用 ipfs.gz 函数，将下载的文件内容存储为 bytes 切片，并将其解压缩。ipfs.gz 是 IPFS(InterPlanetary File System) 库的 gz 函数，用于将文件内容打包为字节切片并压缩。

3. 打印文件的哈希值。fmt.Printf 是 Go 语言中的一个函数，用于打印格式化字符串。在这个代码片段中，fmt.Printf 将 "main.stats.json" 的哈希值打印出来。

4. 通过运行 "node_main.js" 函数，将 kubernetes.gen.repo 和 kubernetes.time. yaml 文件的日月量(奇偶)值打印出来。


```
//go:build !nofuse
// +build !nofuse

package node

import (
	"bytes"
	"fmt"
	"os/exec"
	"runtime"
	"strings"

	core "github.com/ipfs/kubo/core"

	"github.com/blang/semver/v4"
	unix "golang.org/x/sys/unix"
)

```

这段代码定义了一个名为 "init" 的函数，其作用是在程序启动时执行一些自定义操作。

在函数内部，首先定义了一个名为 "platformFuseChecks" 的常量，其值为 "darwinFuseCheckVersion" 的哈希值，似乎是一个用于操作系统测试的 FUSE 工具链版本。

接下来定义了一个名为 "dontCheckOSXFUSEConfigKey" 的常量，其值为 "DontCheckOSXFUSE"，似乎是一个让用户决定是否忽略 FUSE 检查的配置键。

然后定义了一个名为 "fuseVersionPkg" 的常量，其值为 "github.com/jbenet/go-fuse-version/fuse-version" 中的 Go FUSE 包的 URL。

最后定义了一个名为 "errStrFuseRequired" 的变量，其初始值为 `"OSXFUSE not found."`，似乎是一个错误消息，用于在用户没有安装 FUSE 工具链时抛出。


```
func init() {
	// this is a hack, but until we need to do it another way, this works.
	platformFuseChecks = darwinFuseCheckVersion
}

// dontCheckOSXFUSEConfigKey is a key used to let the user tell us to
// skip fuse checks.
const dontCheckOSXFUSEConfigKey = "DontCheckOSXFUSE"

// fuseVersionPkg is the go pkg url for fuse-version.
const fuseVersionPkg = "github.com/jbenet/go-fuse-version/fuse-version"

// errStrFuseRequired is returned when we're sure the user does not have fuse.
var errStrFuseRequired = `OSXFUSE not found.

```

这段代码是一个 FUSE (Filesystem in Userspace) 相关的错误信息输出，它询问你是否安装了名为 "osxfuse" 的软件包，并建议你从官方网站 (http://osxfuse.github.io/) 安装它。

如果没有安装 "osxfuse"，该段代码会输出一个错误消息，其中包含 "NO such file or directory: '/usr/local/lib/libosxfuse.dylib'"，这表明你无法在 /usr/local/lib 目录下找到名为 libosxfuse.dylib 的库文件。

此外，该段代码还建议你安装 "osxfuse" 之前，请尝试在最低版本 2.7.2（或更高）的操作系统上安装它，因为较低版本的操作系统不支持更高版本的软件包。


```
OSXFUSE is required to mount, please install it.
NOTE: Version 2.7.2 or higher required; prior versions are known to kernel panic!
It is recommended you install it from the OSXFUSE website:

	http://osxfuse.github.io/

For more help, see:

	https://github.com/ipfs/kubo/issues/177
`

// errStrNoFuseHeaders is included in the output of `go get <fuseVersionPkg>` if there
// are no fuse headers. this means they don't have OSXFUSE installed.
var errStrNoFuseHeaders = "no such file or directory: '/usr/local/lib/libosxfuse.dylib'"

```

这段代码会输出一个错误信息，指出当前操作系统不支持名为"OSXFUSE"的软件包，同时提到了OSXFUSE版本2.7.2可能会导致系统崩溃。它建议升级到最新的OSXFUSE版本。此外，它还提供了一个指向OSXFUSE网站的链接，以获取更高级别的软件包。


```
var errStrUpgradeFuse = `OSXFUSE version %s not supported.

OSXFUSE versions <2.7.2 are known to cause kernel panics!
Please upgrade to the latest OSXFUSE version.
It is recommended you install it from the OSXFUSE website:

	http://osxfuse.github.io/

For more help, see:

	https://github.com/ipfs/kubo/issues/177
`

type errNeedFuseVersion struct {
	cause string
}

```

这段代码是一个函数，名为"func"，接受两个参数，一个是"me"表示函数内部使用的外设，另一个是"errNeedFuseVersion"表示错误信息返回给调用者的信息。函数的作用是在不输出源代码的情况下输出一条错误信息。

函数内部首先定义了一个字符串变量"return"，然后使用"fmt"包中的"Sprintf"函数将其内容设置为`unable to check fuse version`。这里使用了%fusing“格式化字符串”的技巧，通过"unable to check fuse version"来表达无法检查FUSE版本的状况。接着，使用"Dear User"来段落开头，然后输出一段信息，指出调用者需要检查他们的操作系统版本，否则无法安装一个安全版本。接下来，使用"we tried to install it, but something went wrong"来描述尝试安装一个辅助工具时遇到的问题。最后，使用"Please install it yourself"来鼓励调用者安装他们自己的操作系统版本。


```
func (me errNeedFuseVersion) Error() string {
	return fmt.Sprintf(`unable to check fuse version.

Dear User,

Before mounting, we must check your version of OSXFUSE. We are protecting
you from a nasty kernel panic we found in OSXFUSE versions <2.7.2.[1]. To
make matters worse, it's harder than it should be to check whether you have
the right version installed...[2]. We've automated the process with the
help of a little tool. We tried to install it, but something went wrong[3].
Please install it yourself by running:

	go get %s

You can also stop ipfs from running these checks and use whatever OSXFUSE
```

这段代码是一个交互式的命令行工具，它通过 `ipfs config` 命令配置了 IPFS（InterPlanetary File System）的一些选项。

`--bool %s true` 选项用于设置或取消 `%s`（这里是 `/usr/share/ipfs/ipfs-executable`）的可执行性。如果设置为 `true`，则 IPFS 将作为一个真正的操作系统工具来运行。

以下是此代码的更详细解释：

1. `ipfs config` 是用来设置 IPFS 配置的命令行工具。该命令的第一个参数指定要设置的选项类型。
2. `--bool %s true` 选项用于设置 `%s`（这里是 `/usr/share/ipfs/ipfs-executable`）的可执行性。`%s` 是下一层命令的参数，用于指定要设置的选项名称。`true` 是布尔值，表示如果设置为 `true`，则该选项将被启用。
3. `fuseVersionPkg` 是另一个命令行工具，用于检查操作系统上是否有 `fuse`（通常是 `unix-fuse`）可用的 `version`（“版本号”）包。如果没有 `fuse` 包，则执行以下操作时会报错。
4. `undefined <string>` 错误消息是在尝试使用 `undefined <string>` 时出现的。这种错误通常是由于操作系统上没有安装所需的软件包或服务时发生的。
5. `<path/to/fuse-repo>/bin/fuse` 是 `fuse` 包的安装路径。如果该路径设置正确，则可以通过运行 `<path/to/fuse-repo>/bin/fuse` 来安装 `fuse` 包。
6. `/usr/share/ipfs/ipfs-executable` 是 `ipfs-executable`（通常是 `ipfs`）的可执行文件的安装目录。如果该目录设置不正确，则可能无法访问 `/usr/share/ipfs/ipfs-executable` 目录。

这段代码的作用是设置 IPFS 的可执行性，并检查 `fuse` 包是否已安装。如果没有安装 `fuse`，则会报错并拒绝安装 `ipfs-executable`。


```
version you have by running:

	ipfs config --bool %s true

[1]: https://github.com/ipfs/kubo/issues/177
[2]: https://github.com/ipfs/kubo/pull/533
[3]: %s
`, fuseVersionPkg, dontCheckOSXFUSEConfigKey, me.cause)
}

var errStrFailedToRunFuseVersion = `unable to check fuse version.

Dear User,

Before mounting, we must check your version of OSXFUSE. We are protecting
```

这段代码是一个 FUSE（Filesystem in Userspace）工具的命令行工具，用于检查用户空间 FUSE 代理的版本是否符合特定的要求。

首先，它从 OSXFUSE 版本 2.7.2 或其他版本中自动下载了一个严重的内核慌张，并显示出来。

然后，它显示一个警告，说这个版本似乎不符合要求，同时提供了一个自动化工具来解决问题。

最后，它通过运行 "go get fuse-version" 来自动安装 FUSE 工具，并输出 "fuse-version -only agent" 命令的输出结果，显示用户空间 FUSE 代理的版本为 "OSXFUSE.AgentVersion: 2.7.3"。


```
you from a nasty kernel panic we found in OSXFUSE versions <2.7.2.[1]. To
make matters worse, it's harder than it should be to check whether you have
the right version installed...[2]. We've automated the process with the
help of a little tool. We tried to run it, but something went wrong[3].
Please, try to run it yourself with:

	go get %s
	fuse-version

You should see something like this:

	> fuse-version
	fuse-version -only agent
	OSXFUSE.AgentVersion: 2.7.3

```

这段代码是一个 Bash 脚本，用于配置 Kubernetes Cluster（KCL）中的 Ipfs 类根服务器。它主要做了以下几件事情：

1. 检查配置文件中 Ipfs 服务的版本是否高于或等于 2.7.2。如果是，则执行以下命令：

	
  ipfs config --bool IpfsPinYin true
  

	这个命令会将 `IpfsPinYin` 设置为 `true`，从而允许使用 Ipfs 服务。

2. 如果上述命令执行失败，或者配置文件中 Ipfs 服务的版本低于 2.7.2，则输出一条错误消息并停止 Ipfs 的运行。

3. 输出一条友好消息，告诉用户在修复了这个问题后可以重新尝试运行 Ipfs 服务。


```
Just make sure the number is 2.7.2 or higher. You can then stop ipfs from
trying to run these checks with:

	ipfs config --bool %s true

[1]: https://github.com/ipfs/kubo/issues/177
[2]: https://github.com/ipfs/kubo/pull/533
[3]: %s
`

var errStrFixConfig = `config key invalid: %s %v
You may be able to get this error to go away by setting it again:

	ipfs config --bool %s true

```

该函数的作用是检查 FUSE 文件的版本是否符合要求。它主要分两个步骤：

1. 在操作系统为 Darwin（苹果用户）的情况下，检查 FUSE 版本。如果您的操作系统不是 Darwin，函数会返回一个错误。
2. 如果操作系统为 Darwin，并且 FUSE 版本符合要求（通过调用 `tryGFV()` 函数得到），则返回空错误。否则，函数会尝试使用 `userAskedToSkipFuseCheck()` 函数询问用户是否要检查 FUSE 版本。如果用户选择不检查，或者函数在尝试调用 `userAskedToSkipFuseCheck()` 函数时遇到错误，那么函数会返回 `errGFV()` 错误。如果用户不选择不检查，并且 FUSE 版本符合要求，那么函数不会返回任何错误。

该函数的输出将是一个字符串，其中包含有关 FUSE 版本的详细信息。


```
Either way, please tell us at: http://github.com/ipfs/kubo/issues
`

func darwinFuseCheckVersion(node *core.IpfsNode) error {
	// on OSX, check FUSE version.
	if runtime.GOOS != "darwin" {
		return nil
	}

	ov, errGFV := tryGFV()
	if errGFV != nil {
		// if we failed AND the user has told us to ignore the check we
		// continue. this is in case fuse-version breaks or the user cannot
		// install it, but is sure their fuse version will work.
		if skip, err := userAskedToSkipFuseCheck(node); err != nil {
			return err
		} else if skip {
			return nil // user told us not to check version... ok....
		}
		return errGFV
	}

	log.Debug("mount: osxfuse version:", ov)

	min := semver.MustParse("2.7.2")
	curr, err := semver.Make(ov)
	if err != nil {
		return err
	}

	if curr.LT(min) {
		return fmt.Errorf(errStrUpgradeFuse, ov)
	}
	return nil
}

```



该代码定义了一个名为 `tryGFV` 的函数，其作用是尝试使用 `Sysctl` 命令来获取操作系统中的 `x86_64` 签名文件元数据，如果 `Sysctl` 命令成功执行，则返回其版本号和 `None` 错误，否则返回 `"x86_64"` 和相应的错误信息。

具体来说，代码中先调用 `trySysctl` 函数获取 `x86_64` 签名文件元数据并检查是否出错，如果出错则记录错误信息并返回。如果 `trySysctl` 命令成功执行，则调用 `tryGFVFromFuseVersion` 函数获取操作系统中的签名文件元数据，并返回其版本号和 `None` 错误。

`trySysctl` 函数使用 `unix.Sysctl` 包发送 `Sysctl` 命令并获取其版本号。具体来说，该函数调用 `unix.Sysctl.<profile>` 字段来获取 `Sysctl` 命令的输出，然后从输出中提取版本号部分并返回。

`tryGFVFromFuseVersion` 函数使用 `GFV` 包中的 `<fuse>` 函数获取操作系统中的 `x86_64` 签名文件元数据，并返回其版本号和 `None` 错误。具体来说，该函数调用 `<fuse>.<profile>` 字段来获取 `GFV` 包中的 `<fuse>` 函数，然后从该函数的输出中提取版本号部分并返回，如果调用失败则返回 `"x86_64"` 和相应的错误信息。


```
func tryGFV() (string, error) {
	// first try sysctl. it may work!
	ov, err := trySysctl()
	if err == nil {
		return ov, nil
	}
	log.Debug(err)

	return tryGFVFromFuseVersion()
}

func trySysctl() (string, error) {
	v, err := unix.Sysctl("osxfuse.version.number")
	if err != nil {
		log.Debug("mount: sysctl osxfuse.version.number:", "failed")
		return "", err
	}
	log.Debug("mount: sysctl osxfuse.version.number:", v)
	return v, nil
}

```

这段代码的作用是尝试从FUSE versions 1.0_2版中获取文件系统功能验证（agent）的命令行工具，用于在Linux系统上检测OSXFUSE配置是否正确。

该代码首先检查FUSE版本是否已安装。如果检查失败，会输出错误信息并返回。然后，该代码会执行以下命令并将其输出赋值给变量out：
bash
fuse-version -q -only agent -s OSXFUSE

该命令用于从OSXFUSE版本1.0_2中下载并运行"fuse-version"命令，用于获取文件系统功能验证 agent 的命令行工具。

如果下载或运行命令成功，该代码会返回out缓冲区的字符串，表示文件系统功能验证 agent 的版本，同时返回一个 nil error。如果下载或运行命令失败，该代码会输出错误信息并捕获该错误，以便进行更详细的错误处理。


```
func tryGFVFromFuseVersion() (string, error) {
	if err := ensureFuseVersionIsInstalled(); err != nil {
		return "", err
	}

	cmd := exec.Command("fuse-version", "-q", "-only", "agent", "-s", "OSXFUSE")
	out := new(bytes.Buffer)
	cmd.Stdout = out
	if err := cmd.Run(); err != nil {
		return "", fmt.Errorf(errStrFailedToRunFuseVersion, fuseVersionPkg, dontCheckOSXFUSEConfigKey, err)
	}

	return out.String(), nil
}

```

该函数的作用是检查 FUSE 版本是否已经安装，如果检查成功则返回，否则执行一系列操作来安装 FUSE 版本。以下是具体的解释：

1. 首先检查是否已经安装了 FUSE 版本，如果没有安装，则返回。
2. 如果 FUSE 版本未安装，尝试使用以下命令来安装它：

go installgithub.com/jbenet/go-fuse-version/fuse-version

3. 如果安装 FUSE 版本时出现错误，如果错误消息包含 "errStrNoFuseHeaders"，则返回错误信息，否则返回错误。
4. 如果尝试安装 FUSE 版本后仍然无法成功，返回错误。
5. 如果仍然无法安装 FUSE 版本，尝试再次安装，如果仍然失败，则返回错误。
6. 如果安装成功，则返回。


```
func ensureFuseVersionIsInstalled() error {
	// see if fuse-version is there
	if _, err := exec.LookPath("fuse-version"); err == nil {
		return nil // got it!
	}

	// try installing it...
	log.Debug("fuse-version: no fuse-version. attempting to install.")
	cmd := exec.Command("go", "install", "github.com/jbenet/go-fuse-version/fuse-version")
	cmdout := new(bytes.Buffer)
	cmd.Stdout = cmdout
	cmd.Stderr = cmdout
	if err := cmd.Run(); err != nil {
		// Ok, install fuse-version failed. is it they don't have fuse?
		cmdoutstr := cmdout.String()
		if strings.Contains(cmdoutstr, errStrNoFuseHeaders) {
			// yes! it is! they don't have fuse!
			return fmt.Errorf(errStrFuseRequired)
		}

		log.Debug("fuse-version: failed to install.")
		s := err.Error() + "\n" + cmdoutstr
		return errNeedFuseVersion{s}
	}

	// ok, try again...
	if _, err := exec.LookPath("fuse-version"); err != nil {
		log.Debug("fuse-version: failed to install?")
		return errNeedFuseVersion{err.Error()}
	}

	log.Debug("fuse-version: install success")
	return nil
}

```

这段代码定义了一个名为 `userAskedToSkipFuseCheck` 的函数，接收一个名为 `core.IpfsNode` 的 `IpfsNode` 类型的参数，并返回一个名为 `skip` 的布尔值和一个名为 `err` 的错误对象。

函数的作用是检查操作系统（OS）和 FUSE（Filesystem in Userspace）文件系统上是否存在 FUSE 配置文件，并允许用户手动指定是否绕过 FUSE 检查。

具体来说，函数首先从传入的 `IpfsNode` 对象的 `Repo` 方法中获取一个名为 `dontCheckOSXFUSEConfigKey` 的配置键，然后根据这个配置键的值，执行以下操作：

1. 如果 `dontCheckOSXFUSEConfigKey` 的值为空，那么函数返回 `false` 和 `nil`，即不执行任何人身指定，也不进行 FUSE 检查。
2. 如果 `dontCheckOSXFUSEConfigKey` 的值不为空，那么函数首先尝试从传入的 `val` 变量中获取一个字符串类型的值，即 FUSE 配置文件的值。如果 `val` 的值为 `"true"`，那么函数返回 `true` 和 `nil`，即已经存在 FUSE 配置文件，不需要再检查。如果 `val` 的值不为 `"true"`，那么函数获取到的 `val` 可能是一个布尔类型或者一个字符串类型（可能是用户输入的错误值），此时函数将尝试执行用户指定。
3. 如果 `val` 的类型不符合预期，那么函数将返回 `false` 和一个错误对象，错误对象包含 `err` 字段，其中包含错误信息。


```
func userAskedToSkipFuseCheck(node *core.IpfsNode) (skip bool, err error) {
	val, err := node.Repo.GetConfigKey(dontCheckOSXFUSEConfigKey)
	if err != nil {
		return false, nil // failed to get config value. don't skip check.
	}

	switch val := val.(type) {
	case string:
		return val == "true", nil
	case bool:
		return val, nil
	default:
		// got config value, but it's invalid... don't skip check, ask the user to fix it...
		return false, fmt.Errorf(errStrFixConfig, dontCheckOSXFUSEConfigKey, val,
			dontCheckOSXFUSEConfigKey)
	}
}

```

# `/opt/kubo/fuse/node/mount_nofuse.go`

这段代码是一个 Go 语言编写的文本，它通过结合两个条件 `!windows` 和 `nofuse` 来禁止在 Windows 平台上使用某些功能。它调用了 Node.js 中一个名为 `core` 的包，并定义了一个名为 `Mount` 的函数，用于将文件系统挂载点挂载到本地根目录。

具体来说，这段代码的作用是：

1. 如果当前目录不是 Windows 系统，则编译并执行以下命令，禁止在 Windows 系统上执行 `Mount` 函数的代码部分。
2. 如果当前目录是 Windows 系统，并且已经禁止了在 Windows 系统上执行 `Mount` 函数的代码部分，则编译并执行以下命令，允许在 Windows 系统上执行 `Mount` 函数的代码部分。

这段代码的具体实现可能还需要结合其他部分代码才能正常工作，比如定义 `core` 包时需要引入哪些依赖项，以及如何定义 `Mount` 函数实现具体的挂载操作等。


```
//go:build !windows && nofuse
// +build !windows,nofuse

package node

import (
	"errors"

	core "github.com/ipfs/kubo/core"
)

func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
	return errors.New("not compiled in")
}

```

# `/opt/kubo/fuse/node/mount_notsupp.go`

这段代码是一个 FUSE 文件系统挂载命令，用于在启用了 FUSE 的 Linux 系统中安装依赖库以支持在 FUSE 文件系统中使用。

具体来说，代码会尝试安装 netbsd、openbsd 和 plan9 三个库，如果系统中启用了 FUSE，则会自动安装它们，如果系统中未启用 FUSE，则会尝试安装这 3 个库。如果在尝试安装时出现任何错误，则会返回一个错误。

在代码中，首先包含了 `build` 指令，表示在生产模式下编译。接下来，定义了一个名为 `Mount` 的函数，用于将 FUSE 文件系统挂载到指定的目录。函数的实现中，使用了 `openbsd`、`netbsd` 和 `plan9` 三个变量来表示要安装的库。

通过调用 `Mount` 函数，可以实现将 FUSE 文件系统挂载到指定的目录的功能。


```
//go:build (!nofuse && openbsd) || (!nofuse && netbsd) || (!nofuse && plan9)
// +build !nofuse,openbsd !nofuse,netbsd !nofuse,plan9

package node

import (
	"errors"

	core "github.com/ipfs/kubo/core"
)

func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
	return errors.New("FUSE not supported on OpenBSD or NetBSD. See #5334 (https://github.com/ipfs/kubo/issues/5334).")
}

```

# `/opt/kubo/fuse/node/mount_test.go`

这段代码是一个 Go 语言编写的测试用例，用于在不同的操作系统上测试是否支持 OpenBSD、nofuse、netbsd 和 Plan9 这些古董级的 FUSE 分支。

首先，该代码使用了 `build` 指令，这是 Go 语言自带的构建工具，用于创建一个只读的构建文件。这个构建文件中定义了一系列条件判断，用来保证 FUSE 分支只有在所有支持这些分支的操作系统上才会被编译。

具体来说，该代码中定义了以下四个条件：

1. `!openbsd`：表示不希望编译 OpenBSD 分支，因为该分支不够新，也不够稳定。
2. `!nofuse`：表示不希望编译 nofuse 分支，因为该分支同样不够新，也不够稳定。
3. `!netbsd`：表示不希望编译 netbsd 分支，因为该分支同样不够新，也不够稳定。
4. `!plan9`：表示不希望编译 Plan9 分支，因为该分支同样不够新，也不够稳定。

然后，该代码中又定义了一个 `+build` 指令，用于编译这些条件判断，即只有在所有支持这些分支的操作系统上才会编译。

最后，该代码还定义了一个 `test` 包，包含了一些测试用例，用于测试 FUSE 是否支持这些条件。


```
//go:build !openbsd && !nofuse && !netbsd && !plan9
// +build !openbsd,!nofuse,!netbsd,!plan9

package node

import (
	"context"
	"os"
	"strings"
	"testing"
	"time"

	"bazil.org/fuse"

	core "github.com/ipfs/kubo/core"
	ipns "github.com/ipfs/kubo/fuse/ipns"
	mount "github.com/ipfs/kubo/fuse/mount"

	ci "github.com/libp2p/go-libp2p-testing/ci"
)

```

This appears to be a Go program that performs tests for the Go FUSE implementation.

It does the following:

* Skips tests that use the FUSE built-in testing framework if the operating system is Linux.
* Migrates the IPFS data structure to the node by running the `ipns InitializeKeyspace` function and the `Mount` function.
* mounts the IPFS directory and starts the `fs.Serve` goroutine that runs the tests.
* Runs the shell command `mount -it /tmp/TestExternalUnmount` to externally unmount the directory.
* Uses a `time.Sleep` to give the goroutine enough time to complete its cleanup.
* Attempts to unmount IPFS and reports an error if it fails.

The program includes a function `Short()` that does nothing.

It appears that the program is using the `core.NewNode` function to create a new FUSE node, and the `ipns.InitializeKeyspace` and `ipns.Mount` functions to initialize and mount the IPFS data structure.


```
func maybeSkipFuseTests(t *testing.T) {
	if ci.NoFuse() {
		t.Skip("Skipping FUSE tests")
	}
}

func mkdir(t *testing.T, path string) {
	err := os.Mkdir(path, os.ModeDir|os.ModePerm)
	if err != nil {
		t.Fatal(err)
	}
}

// Test externally unmounting, then trying to unmount in code.
func TestExternalUnmount(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}

	// TODO: needed?
	maybeSkipFuseTests(t)

	node, err := core.NewNode(context.Background(), &core.BuildCfg{})
	if err != nil {
		t.Fatal(err)
	}

	err = ipns.InitializeKeyspace(node, node.PrivateKey)
	if err != nil {
		t.Fatal(err)
	}

	// get the test dir paths (/tmp/TestExternalUnmount)
	dir := t.TempDir()

	ipfsDir := dir + "/ipfs"
	ipnsDir := dir + "/ipns"
	mkdir(t, ipfsDir)
	mkdir(t, ipnsDir)

	err = Mount(node, ipfsDir, ipnsDir)
	if err != nil {
		if strings.Contains(err.Error(), "unable to check fuse version") || err == fuse.ErrOSXFUSENotFound {
			t.Skip(err)
		}
	}

	if err != nil {
		t.Fatalf("error mounting: %v", err)
	}

	// Run shell command to externally unmount the directory
	cmd, err := mount.UnmountCmd(ipfsDir)
	if err != nil {
		t.Fatal(err)
	}

	if err := cmd.Run(); err != nil {
		t.Fatal(err)
	}

	// TODO(noffle): it takes a moment for the goroutine that's running fs.Serve to be notified and do its cleanup.
	time.Sleep(time.Millisecond * 100)

	// Attempt to unmount IPFS; it should unmount successfully.
	err = node.Mounts.Ipfs.Unmount()
	if err != mount.ErrNotMounted {
		t.Fatal("Unmount should have failed")
	}
}

```

# `/opt/kubo/fuse/node/mount_unix.go`

这段代码是一个 Go 语言编写的函数，它通过结合多个条件来判断是否支持使用某些软件包。这里列举了几个常用的软件包，如 `windows`、`openbsd`、`netbsd`、`plan9` 和 `nofuse`，表示需要这些软件包才能正常工作。同时，该函数还判断了是否启用了 FUSE 子系统，如果启用了，则不执行任何操作。

具体来说，该函数首先判断了多个条件，如果其中任意一个条件为 `true`，则函数将输出一条信息，说明当前环境不支持该软件包的使用。如果所有条件都为 `false`，则函数将不做任何操作，不会影响 `node` 函数的执行。


```
//go:build !windows && !openbsd && !netbsd && !plan9 && !nofuse
// +build !windows,!openbsd,!netbsd,!plan9,!nofuse

package node

import (
	"errors"
	"fmt"
	"strings"
	"sync"

	core "github.com/ipfs/kubo/core"
	ipns "github.com/ipfs/kubo/fuse/ipns"
	mount "github.com/ipfs/kubo/fuse/mount"
	rofs "github.com/ipfs/kubo/fuse/readonly"

	logging "github.com/ipfs/go-log"
)

```

这段代码定义了一个名为 "log" 的日志输出变量，该变量通过名为 "node" 的 FUSE 应用程序日志配置为输出 FUSE 错误和退出状态。

接着，定义了一个名为 "fuseNoDirectory" 的常量，用于检查 FUSE 是否无法访问挂载点。

然后，定义了一个名为 "fuseExitStatus1" 的常量，用于检查 FUSE 是否返回 exit 状态为 1。

接下来，定义了一个名为 "platformFuseChecks" 的函数，该函数接收一个名为 "core.IpfsNode" 的 IpfsNode 类型的输入参数，并返回一个 FUSE 错误。该函数使用 fuseNoDirectory 和 fuseExitStatus1 作为输入检查 FUSE 错误，并根据 platform-specific 文件覆盖 fuseChecks 函数的输出。

最后，定义了一个名为 "Mount" 的函数，该函数接收一个名为 "core.IpfsNode" 的 IpfsNode 类型的输入参数，以及一个 FUSE 挂载点和一个 FUSE 子目录。函数首先检查是否 already have live mounts，如果已经存在，则尝试关闭它们并重新尝试。如果 fuseNoDirectory 和 fuseExitStatus1 检查失败，函数将调用 platformFuseChecks 函数来执行 FUSE 错误检查。如果所有检查都成功，函数将正常挂载文件系统。


```
var log = logging.Logger("node")

// fuseNoDirectory used to check the returning fuse error.
const fuseNoDirectory = "fusermount: failed to access mountpoint"

// fuseExitStatus1 used to check the returning fuse error.
const fuseExitStatus1 = "fusermount: exit status 1"

// platformFuseChecks can get overridden by arch-specific files
// to run fuse checks (like checking the OSXFUSE version).
var platformFuseChecks = func(*core.IpfsNode) error {
	return nil
}

func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
	// check if we already have live mounts.
	// if the user said "Mount", then there must be something wrong.
	// so, close them and try again.
	if node.Mounts.Ipfs != nil && node.Mounts.Ipfs.IsActive() {
		// best effort
		_ = node.Mounts.Ipfs.Unmount()
	}
	if node.Mounts.Ipns != nil && node.Mounts.Ipns.IsActive() {
		// best effort
		_ = node.Mounts.Ipns.Unmount()
	}

	if err := platformFuseChecks(node); err != nil {
		return err
	}

	return doMount(node, fsdir, nsdir)
}

```

This is a function that syncs a file system mount point across both FUSE and IPCNFS (a filesystem that uses IPC to communicate between client and server). The function takes two arguments: `node` and `fsdir`.

The `node` argument is the node object that provides the IPC services, and `fsdir` is the directory to sync the file system mount point.

The function returns either an error or nil if there was an error in mounting the file systems. If an error in mounted file system occurs, the function will return a custom error.


```
func doMount(node *core.IpfsNode, fsdir, nsdir string) error {
	fmtFuseErr := func(err error, mountpoint string) error {
		s := err.Error()
		if strings.Contains(s, fuseNoDirectory) {
			s = strings.Replace(s, `fusermount: "fusermount:`, "", -1)
			s = strings.Replace(s, `\n", exit status 1`, "", -1)
			return errors.New(s)
		}
		if s == fuseExitStatus1 {
			s = fmt.Sprintf("fuse failed to access mountpoint %s", mountpoint)
			return errors.New(s)
		}
		return err
	}

	// this sync stuff is so that both can be mounted simultaneously.
	var fsmount, nsmount mount.Mount
	var err1, err2 error

	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		fsmount, err1 = rofs.Mount(node, fsdir)
	}()

	if node.IsOnline {
		wg.Add(1)
		go func() {
			defer wg.Done()
			nsmount, err2 = ipns.Mount(node, nsdir, fsdir)
		}()
	}

	wg.Wait()

	if err1 != nil {
		log.Errorf("error mounting: %s", err1)
	}

	if err2 != nil {
		log.Errorf("error mounting: %s", err2)
	}

	if err1 != nil || err2 != nil {
		if fsmount != nil {
			_ = fsmount.Unmount()
		}
		if nsmount != nil {
			_ = nsmount.Unmount()
		}

		if err1 != nil {
			return fmtFuseErr(err1, fsdir)
		}
		return fmtFuseErr(err2, nsdir)
	}

	// setup node state, so that it can be cancelled
	node.Mounts.Ipfs = fsmount
	node.Mounts.Ipns = nsmount
	return nil
}

```

# `/opt/kubo/fuse/node/mount_windows.go`

这段代码定义了一个名为 "Mount" 的函数，它接受两个参数：一个表示 IPFS 节点的指针（<core.IpfsNode> 类型）和一个表示目标文件系统的路径（字符串）。函数的作用是挂载文件系统到 IPFS 节点上。

具体来说，这段代码执行以下操作：

1. 导入 necessary 包。
2. 导入 "github.com/ipfs/kubo/core" 包，该包实现了 IPFS 和 kubo 的交互。
3. 定义一个名为 "Mount" 的函数，它接收两个参数：一个 IPFS 节点指针和一个目标文件系统路径。
4. 在函数体内部，没有执行任何具体的逻辑。
5. 函数返回一个名为 "nil" 的值，表示成功挂载。

总之，这段代码定义了一个简单的函数，用于将 IPFS 节点挂载到指定的文件系统上。


```
package node

import (
	"github.com/ipfs/kubo/core"
)

func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
	// TODO
	// currently a no-op, but we don't want to return an error
	return nil
}

```

# `/opt/kubo/fuse/readonly/doc.go`

这段代码定义了一个名为"fuse/readonly"的FUSE（Filesystem in Userspace）文件系统实现，旨在通过IPFS（InterPlanetary File System，星际文件系统）访问文件。它主要存储在ipfs.目录中，提供了一个可以访问IPFS资源的方式，允许用户在不受文件系统限制的情况下进行文件操作。

具体来说，这段代码实现了以下功能：

1. 导入ipfs.h头文件，这是FUSE库中定义IPFS相关函数的助手函数。
2. 定义了一个名为"fuse/readonly"的内部类，实现了FUSE库中关于文件系统的所有功能。
3. 通过使用"stored inside of ipfs."，表示这个内部类存储在ipfs.文件系统目录下。
4. 实现了"get正好"函数，用于获取文件系统中的文件。
5. 实现了"-println"函数，用于打印文件系统中的文件名。
6. 实现了"-printlnF"函数，类似于"-println"，但可以格式化输出文件名。
7. 实现了"-readlink"函数，用于获取文件的链接。
8. 实现了"-write"函数，用于创建文件或更新文件。
9. 实现了"-Append"函数，用于在文件末尾添加内容。
10. 实现了"-Overwrite"函数，用于覆盖文件。
11. 实现了"-Noatime"函数，用于设置文件访问时间。
12. 实现了"-Permissions"函数，用于设置文件或目录的权限。
13. 实现了"-目錄"函数，用于创建一个新的目录。
14. 实现了"-叶子"函数，用于删除一个目录。
15. 实现了"-递归"函数，用于递归地执行一个文件系统操作。

总之，这段代码定义了一个FUSE文件系统实现，可以让你通过IPFS访问文件，并提供了一系列文件操作功能。


```
// package fuse/readonly implements a fuse filesystem to access files
// stored inside of ipfs.
package readonly

```

# `/opt/kubo/fuse/readonly/ipfs_test.go`

这段代码是一个 Go 语言中的匿名函数，它通过结合多个条件，来判断是否使用某些第三方库。具体来说，该函数的作用是：

1. 如果当前目录下没有 `nofuse`、`openbsd`、`netbsd` 或 `plan9` 依赖库，就执行下面的构建命令，即 `go build`。这意味着您需要确保您的项目已经安装了这些库，或者在 `build` 目录下手动安装了这些库。
2. 构建完上面的依赖库之后，不使用 `nofuse`、`openbsd`、`netbsd` 或 `plan9` 任何一个依赖库。

代码中涉及到的第三方库和构建命令如下：

* `netbsd` 和 `plan9`：需要从 GitHub 上 pull，使用 `git clone` 命令来获取代码。
* `nofuse`：用于在 Go 语言 1.10 中提供 Linux 文件系统的支持。
* `openbsd`：用于提供 Go 语言 1.10 和 1.11 对 Linux 文件系统的支持。
* `build`：在 Go 语言中，构建工具链的过程。
* `npm`：用于安装和管理 Node.js 应用程序中的依赖项。
* `g开行武夷山双腿导游图 flask小游戏 一起去二连湖作文 左右做人流体描图摸摸集 英语中动词和副词的辨析 董卿的父亲是谁 野生大乱斗下载 对不起我不认识你 拍照那个app下载 我和儿子重名了怎么办呢 幼儿园老师 美国国旗和约 91红心福利视频大全 小黄人2在线观看 百度云盘下载软件下载网站 迪迦奥特曼模型 标致4008海外版本 91红心福利视频大全 超级马里奥256背景 路由器上不了网怎么处理 小黄人2下载 奥特曼是日本买的吗 迪士尼公主英文名字 2022考研初试时间 小米redmi k4pro参数 2022考研初试成绩查询 2022考研初试成绩查询网站 董卿的儿子是谁


```
//go:build !nofuse && !openbsd && !netbsd && !plan9
// +build !nofuse,!openbsd,!netbsd,!plan9

package readonly

import (
	"bytes"
	"context"
	"errors"
	"fmt"
	"io"
	"math/rand"
	"os"
	gopath "path"
	"strings"
	"sync"
	"testing"

	"bazil.org/fuse"

	core "github.com/ipfs/kubo/core"
	coreapi "github.com/ipfs/kubo/core/coreapi"
	coremock "github.com/ipfs/kubo/core/mock"

	fstest "bazil.org/fuse/fs/fstestutil"
	chunker "github.com/ipfs/boxo/chunker"
	"github.com/ipfs/boxo/files"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	importer "github.com/ipfs/boxo/ipld/unixfs/importer"
	uio "github.com/ipfs/boxo/ipld/unixfs/io"
	"github.com/ipfs/boxo/path"
	u "github.com/ipfs/boxo/util"
	ipld "github.com/ipfs/go-ipld-format"
	ci "github.com/libp2p/go-libp2p-testing/ci"
)

```

这段代码定义了一个名为func maybeSkipFuseTests的函数，它接受一个名为t的测试标头和一个名为nd的IPFS节点对象，以及一个名为size的整数参数。函数的作用是判断是否在当前环境中有FUSE测试，如果没有，就跳过这些测试，否则执行函数内部的业务逻辑。

func randObj(t *testing.T, nd *core.IpfsNode, size int64) (ipld.Node, []byte) {
	buf := make([]byte, size)
	_, err := io.ReadFull(u.NewTimeSeededRand(), buf)
	if err != nil {
		t.Fatal(err)
	}
	read := bytes.NewReader(buf)
	obj, err := importer.BuildTrickleDagFromReader(nd.DAG, chunker.DefaultSplitter(read))
	if err != nil {
		t.Fatal(err)
	}

	return obj, buf
}


```
func maybeSkipFuseTests(t *testing.T) {
	if ci.NoFuse() {
		t.Skip("Skipping FUSE tests")
	}
}

func randObj(t *testing.T, nd *core.IpfsNode, size int64) (ipld.Node, []byte) {
	buf := make([]byte, size)
	_, err := io.ReadFull(u.NewTimeSeededRand(), buf)
	if err != nil {
		t.Fatal(err)
	}
	read := bytes.NewReader(buf)
	obj, err := importer.BuildTrickleDagFromReader(nd.DAG, chunker.DefaultSplitter(read))
	if err != nil {
		t.Fatal(err)
	}

	return obj, buf
}

```

这段代码定义了一个名为setupIpfsTest的函数，该函数接受两个参数： testing.T 类型的测试客户端和一个 CoreIpfsNode 类型的实现了 IpfsNode 接口的节点。该函数的作用是在创建一个 IpfsNode 实例并将其返回时执行一系列测试。

具体来说，该函数在以下步骤中执行操作：

1. 执行可能需要跳过 FUSE 测试的检查，以防止在测试中影响结果的问题。
2. 创建一个 IpfsNode 实例，如果遇到错误，将记录错误并跳过测试。
3. 创建一个 FileSystem 实例，并将其与 IpfsNode 一起使用，以便在测试中执行其他操作。
4. 使用 fstest.MountedT 函数将 IpfsNode 挂载到一个测试文件系统上，并获取一个已完成的操作：MountedT 函数返回的 *MountedT 类型。
5. 如果操作成功，将打印成功消息。如果出现错误，将捕获并记录错误，以便在测试中进行更详细的错误处理。


```
func setupIpfsTest(t *testing.T, node *core.IpfsNode) (*core.IpfsNode, *fstest.Mount) {
	t.Helper()
	maybeSkipFuseTests(t)

	var err error
	if node == nil {
		node, err = coremock.NewMockNode()
		if err != nil {
			t.Fatal(err)
		}
	}

	fs := NewFileSystem(node)
	mnt, err := fstest.MountedT(t, fs, nil)
	if err == fuse.ErrOSXFUSENotFound {
		t.Skip(err)
	}
	if err != nil {
		t.Fatalf("error mounting temporary directory: %v", err)
	}

	return node, mnt
}

```

这段代码的主要作用是测试在Ipfs中读取文件的正确性。该代码首先创建一个随机对象，然后使用Ipfs的`random_object`函数将该对象随机存储到Ipfs的某个目录中。接着，代码使用`path.Join`函数将Ipfs目录的路径与随机对象的键（即随机对象的序列号）组合成一个新的文件名。最后，代码使用`os.ReadFile`函数读取文件并比较读取到的内容与之前存储的随机对象的内容是否相等。如果它们不相等，则代码会输出错误信息并跳过当前的测试。


```
// Test writing an object and reading it back through fuse.
func TestIpfsBasicRead(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	nd, mnt := setupIpfsTest(t, nil)
	defer mnt.Close()

	fi, data := randObj(t, nd, 10000)
	k := fi.Cid()
	fname := gopath.Join(mnt.Dir, k.String())
	rbuf, err := os.ReadFile(fname)
	if err != nil {
		t.Fatal(err)
	}

	if !bytes.Equal(rbuf, data) {
		t.Fatal("Incorrect Read!")
	}
}

```

该函数`getPaths`接受三个参数：

* `t`：测试结构体的指针，类型为 `testing.T`。
* `ipfs`：IpfsNode 类型的指针，表示一个 Ipfs 节点的实例。
* `name`：字符串参数，表示一个命名空间。
* `n`：Protobuf 节点参数，类型为 `dag.ProtoNode`。

函数的作用是获取给定命名空间中所有子节点的路径，并将它们添加到结果数组中。

函数首先检查 `n` 节点中的链接是否为空，如果是，则返回该命名空间的名称。

然后，函数遍历 `n` 节点中的所有链接，并获取每个链接的 child 节点。如果某个链接的 child 节点返回时出现错误，函数将打印错误并退出。

接下来，函数使用 `getPaths` 函数递归地获取子节点的路径，并将它们添加到结果数组中。这里使用了 `gopath.Join` 函数将当前路径和链接名称连接起来。

最后，函数返回结果数组。


```
func getPaths(t *testing.T, ipfs *core.IpfsNode, name string, n *dag.ProtoNode) []string {
	if len(n.Links()) == 0 {
		return []string{name}
	}
	var out []string
	for _, lnk := range n.Links() {
		child, err := lnk.GetNode(ipfs.Context(), ipfs.DAG)
		if err != nil {
			t.Fatal(err)
		}

		childpb, ok := child.(*dag.ProtoNode)
		if !ok {
			t.Fatal(dag.ErrNotProtobuf)
		}

		sub := getPaths(t, ipfs, gopath.Join(name, lnk.Name), childpb)
		out = append(out, sub...)
	}
	return out
}

```

This is a Go program that performs a file operation. It reads the contents of a file and checks if they match the contents of the same file that was modified by a different program. The modified file is considered "read incorrectly" if it is considered corrupted.

The program takes a file path and a buffer of data to read from the file. It reads the contents of the file and checks if it matches the contents of the same file that was modified by a different program. It uses the Unix file system to read the file.

The program uses a goroutine to read the contents of the file. It reads the file in chunks, rather than all at once, to avoid hitting a limitation of 8128 goroutines in Go test mode.

The program also uses a context with a cancel function to cancel the operation when it is done.


```
// Perform a large number of concurrent reads to stress the system.
func TestIpfsStressRead(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	nd, mnt := setupIpfsTest(t, nil)
	defer mnt.Close()

	api, err := coreapi.NewCoreAPI(nd)
	if err != nil {
		t.Fatal(err)
	}

	var nodes []ipld.Node
	var paths []string

	nobj := 50
	ndiriter := 50

	// Make a bunch of objects
	for i := 0; i < nobj; i++ {
		fi, _ := randObj(t, nd, rand.Int63n(50000))
		nodes = append(nodes, fi)
		paths = append(paths, fi.Cid().String())
	}

	// Now make a bunch of dirs
	for i := 0; i < ndiriter; i++ {
		db := uio.NewDirectory(nd.DAG)
		for j := 0; j < 1+rand.Intn(10); j++ {
			name := fmt.Sprintf("child%d", j)

			err := db.AddChild(nd.Context(), name, nodes[rand.Intn(len(nodes))])
			if err != nil {
				t.Fatal(err)
			}
		}
		newdir, err := db.GetNode()
		if err != nil {
			t.Fatal(err)
		}

		err = nd.DAG.Add(nd.Context(), newdir)
		if err != nil {
			t.Fatal(err)
		}

		nodes = append(nodes, newdir)
		npaths := getPaths(t, nd, newdir.Cid().String(), newdir.(*dag.ProtoNode))
		paths = append(paths, npaths...)
	}

	// Now read a bunch, concurrently
	wg := sync.WaitGroup{}
	errs := make(chan error)

	for s := 0; s < 4; s++ {
		wg.Add(1)
		go func() {
			defer wg.Done()

			for i := 0; i < 2000; i++ {
				item, err := path.NewPath(paths[rand.Intn(len(paths))])
				if err != nil {
					errs <- err
					continue
				}

				relpath := strings.Replace(item.String(), item.Namespace(), "", 1)
				fname := gopath.Join(mnt.Dir, relpath)

				rbuf, err := os.ReadFile(fname)
				if err != nil {
					errs <- err
				}

				// nd.Context() is never closed which leads to
				// hitting 8128 goroutine limit in go test -race mode
				ctx, cancelFunc := context.WithCancel(context.Background())

				read, err := api.Unixfs().Get(ctx, item)
				if err != nil {
					errs <- err
				}

				data, err := io.ReadAll(read.(files.File))
				if err != nil {
					errs <- err
				}

				cancelFunc()

				if !bytes.Equal(rbuf, data) {
					errs <- errors.New("incorrect read")
				}
			}
		}()
	}

	go func() {
		wg.Wait()
		close(errs)
	}()

	for err := range errs {
		if err != nil {
			t.Fatal(err)
		}
	}
}

```

该测试的作用是验证Ipfs的基本目录读取功能。具体步骤如下：

1. 创建一个名为"actual"的目录，并将其添加到nd.DAG上。
2. 创建一个大小为10K的随机文件，并将其添加到刚刚创建的目录中。
3. 遍历目录中的所有节点，并输出当前节点名称。
4. 读取目录中"actual"目录节点的内容，并将其与随机文件中的内容进行比较。
5. 如果二者内容不匹配，或者目录节点不存在，则抛出错误并打印错误信息。


```
// Test writing a file and reading it back.
func TestIpfsBasicDirRead(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	nd, mnt := setupIpfsTest(t, nil)
	defer mnt.Close()

	// Make a 'file'
	fi, data := randObj(t, nd, 10000)

	// Make a directory and put that file in it
	db := uio.NewDirectory(nd.DAG)
	err := db.AddChild(nd.Context(), "actual", fi)
	if err != nil {
		t.Fatal(err)
	}

	d1nd, err := db.GetNode()
	if err != nil {
		t.Fatal(err)
	}

	err = nd.DAG.Add(nd.Context(), d1nd)
	if err != nil {
		t.Fatal(err)
	}

	dirname := gopath.Join(mnt.Dir, d1nd.Cid().String())
	fname := gopath.Join(dirname, "actual")
	rbuf, err := os.ReadFile(fname)
	if err != nil {
		t.Fatal(err)
	}

	dirents, err := os.ReadDir(dirname)
	if err != nil {
		t.Fatal(err)
	}
	if len(dirents) != 1 {
		t.Fatal("Bad directory entry count")
	}
	if dirents[0].Name() != "actual" {
		t.Fatal("Bad directory entry")
	}

	if !bytes.Equal(rbuf, data) {
		t.Fatal("Incorrect Read!")
	}
}

```

这段代码的作用是测试文件系统是否正确地报告文件大小。它使用 Go 标准库中的 `testing` 和 `path/filepath` 包来执行测试。

具体来说，它实现了以下步骤：

1. 设置 I/O 设备，以便在测试过程中挂载文件系统。
2. 使用 `randObject` 生成一个随机的文件系统对象。
3. 使用 `gopath` 将文件系统对象路径 join 到 `mnt` 设备上。
4. 使用 `os.Stat` 获取文件的元数据，并检查它是否等于生成的文件大小。
5. 如果元数据等于生成的文件大小，就跳过一些错误测试。

如果测试过程中出现错误，程序会打印错误消息并停止执行。


```
// Test to make sure the filesystem reports file sizes correctly.
func TestFileSizeReporting(t *testing.T) {
	if testing.Short() {
		t.SkipNow()
	}
	nd, mnt := setupIpfsTest(t, nil)
	defer mnt.Close()

	fi, data := randObj(t, nd, 10000)
	k := fi.Cid()

	fname := gopath.Join(mnt.Dir, k.String())

	finfo, err := os.Stat(fname)
	if err != nil {
		t.Fatal(err)
	}

	if finfo.Size() != int64(len(data)) {
		t.Fatal("Read incorrect size from stat!")
	}
}

```

# `/opt/kubo/fuse/readonly/mount_unix.go`

这段代码是一个 Go 语言编写的工具函数，用于构建 Go 语言程序的依赖文件。

首先，它使用 `go build` 命令在构建时添加特定的依赖，分别为 Linux、Darwin 和 FreeBSD。如果系统中不存在这些依赖，则输出 `!nofuse`，即不使用任何依赖。这里 `!nofuse` 是抑制输出 `nofuse` 信号以避免输出不必要的错误信息。

然后，它定义了一个名为 `Mount` 的函数，用于将 IPFS 挂载到指定路径。函数接收两个参数：一个 IPFS 节点和一个挂载点。函数首先尝试从配置文件中查找有关允许其他操作系统访问的设置，如果没有找到，则输出一个 `N/A` 错误。然后，它创建一个新的文件系统并将其传递给 `Mount` 函数，以便在 IPFS 节点上执行挂载操作。最后，函数将返回一个 `mount.Mount` 类型，表示已完成的挂载操作，以及可能的错误。


```
//go:build (linux || darwin || freebsd) && !nofuse
// +build linux darwin freebsd
// +build !nofuse

package readonly

import (
	core "github.com/ipfs/kubo/core"
	mount "github.com/ipfs/kubo/fuse/mount"
)

// Mount mounts IPFS at a given location, and returns a mount.Mount instance.
func Mount(ipfs *core.IpfsNode, mountpoint string) (mount.Mount, error) {
	cfg, err := ipfs.Repo.Config()
	if err != nil {
		return nil, err
	}
	allowOther := cfg.Mounts.FuseAllowOther
	fsys := NewFileSystem(ipfs)
	return mount.NewMount(ipfs.Process, fsys, mountpoint, allowOther)
}

```

# `/opt/kubo/fuse/readonly/readonly_unix.go`

这段代码是一个 Go 语言编写的 build 函数，用于构建 Go 语言程序的依赖文件。它使用了 build- as-BUILD- 模式，这种模式将依赖文件提取到 build 中而不是在运行时同时构建。

具体来说，这段代码的作用是以下几个步骤：

1. 首先使用 !nofuse 构建 Linux 和 Darwin 操作系统下的依赖文件。
2. 如果已经构建了支持 FUSE 的操作系统，则使用 !nofuse 构建 Darwin 操作系统下的依赖文件。
3. 构建支持 FUSE 的操作系统，但不使用 !nofuse。

这里 !nofuse 是用来避免在构建时同时构建 Linux 和 Darwin 下的文件系统，因为 build-as-BUILD- 模式会同时构建这两个操作系统下的依赖文件，这可能导致一些问题。


```
//go:build (linux || darwin || freebsd) && !nofuse
// +build linux darwin freebsd
// +build !nofuse

package readonly

import (
	"context"
	"fmt"
	"io"
	"os"
	"syscall"

	fuse "bazil.org/fuse"
	fs "bazil.org/fuse/fs"
	mdag "github.com/ipfs/boxo/ipld/merkledag"
	ft "github.com/ipfs/boxo/ipld/unixfs"
	uio "github.com/ipfs/boxo/ipld/unixfs/io"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	logging "github.com/ipfs/go-log"
	core "github.com/ipfs/kubo/core"
	ipldprime "github.com/ipld/go-ipld-prime"
	cidlink "github.com/ipld/go-ipld-prime/linking/cid"
)

```

这段代码定义了一个名为 "fuse/ipfs" 的 Logger 实例，用于输出关于 IPFS(FUSE 文件系统)的日志信息。

具体来说，它创建了一个名为 "FileSystem" 的结构体，该结构体包含一个名为 "Ipfs" 的成员，该成员是一个 Core.IpfsNode 类型的实例，这个节点表示一个 IPFS 文件系统节点。

此外，它还定义了一个名为 "NewFileSystem" 的函数，该函数接收一个 Core.IpfsNode 类型的参数，并返回一个 FileSystem 类型的实例，这个实例包含了在 Core.IpfsNode 类型的实例中指定的 IPFS 节点。

最后，它还定义了一个名为 "Root" 的结构体，该结构体包含一个名为 "Ipfs" 的成员和一个名为 "fs.Node" 的成员，后者用于将 IPFS 节点转换为 fs.Node 类型。该 "Root" 结构体还定义了一个名为 "Error" 的成员和一个名为 "nil" 的成员，后者的作用是避免在输出错误信息时产生不必要的混乱。


```
var log = logging.Logger("fuse/ipfs")

// FileSystem is the readonly IPFS Fuse Filesystem.
type FileSystem struct {
	Ipfs *core.IpfsNode
}

// NewFileSystem constructs new fs using given core.IpfsNode instance.
func NewFileSystem(ipfs *core.IpfsNode) *FileSystem {
	return &FileSystem{Ipfs: ipfs}
}

// Root constructs the Root of the filesystem, a Root object.
func (f FileSystem) Root() (fs.Node, error) {
	return &Root{Ipfs: f.Ipfs}, nil
}

```

The `ResolvePath` function takes a path string and returns either a file descriptor or an error. If the path does not resolve to a valid file descriptor, it returns `syscall.ENOENT`.

The function first checks if the input `nd` is a valid cidlink node. If it is not, it logs an error and returns `syscall.ENOENT`.

Next, it attempts to convert the ipld-prime node to a universal node. If this conversion fails, it logs an error and returns `syscall.ENOTSUP`.

If the conversion succeeds, it creates a `Node` struct that represents the file descriptor, using the `Ipfs` property of the `Ipfs` object to store the file descriptor.

It is important to note that the `Node` struct in this implementation only has a single property, `Ipfs`, which is used to store the `Ipfs` object. This is not a full file descriptor struct and any additional functionality, such as support for file attributes or creation of channels, would need to be implemented.


```
// Root is the root object of the filesystem tree.
type Root struct {
	Ipfs *core.IpfsNode
}

// Attr returns file attributes.
func (*Root) Attr(ctx context.Context, a *fuse.Attr) error {
	a.Mode = os.ModeDir | 0o111 // -rw+x
	return nil
}

// Lookup performs a lookup under this node.
func (s *Root) Lookup(ctx context.Context, name string) (fs.Node, error) {
	log.Debugf("Root Lookup: '%s'", name)
	switch name {
	case "mach_kernel", ".hidden", "._.":
		// Just quiet some log noise on OS X.
		return nil, syscall.Errno(syscall.ENOENT)
	}

	p, err := path.NewPath(name)
	if err != nil {
		log.Debugf("fuse failed to parse path: %q: %s", name, err)
		return nil, syscall.Errno(syscall.ENOENT)
	}

	imPath, err := path.NewImmutablePath(p)
	if err != nil {
		log.Debugf("fuse failed to convert path: %q: %s", name, err)
		return nil, syscall.Errno(syscall.ENOENT)
	}

	nd, ndLnk, err := s.Ipfs.UnixFSPathResolver.ResolvePath(ctx, imPath)
	if err != nil {
		// todo: make this error more versatile.
		return nil, syscall.Errno(syscall.ENOENT)
	}

	cidLnk, ok := ndLnk.(cidlink.Link)
	if !ok {
		log.Debugf("non-cidlink returned from ResolvePath: %v", ndLnk)
		return nil, syscall.Errno(syscall.ENOENT)
	}

	// convert ipld-prime node to universal node
	blk, err := s.Ipfs.Blockstore.Get(ctx, cidLnk.Cid)
	if err != nil {
		log.Debugf("fuse failed to retrieve block: %v: %s", cidLnk, err)
		return nil, syscall.Errno(syscall.ENOENT)
	}

	var fnd ipld.Node
	switch cidLnk.Cid.Prefix().Codec {
	case cid.DagProtobuf:
		adl, ok := nd.(ipldprime.ADL)
		if ok {
			substrate := adl.Substrate()
			fnd, err = mdag.ProtoNodeConverter(blk, substrate)
		} else {
			fnd, err = mdag.ProtoNodeConverter(blk, nd)
		}
	case cid.Raw:
		fnd, err = mdag.RawNodeConverter(blk, nd)
	default:
		log.Error("fuse node was not a supported type")
		return nil, syscall.Errno(syscall.ENOTSUP)
	}
	if err != nil {
		log.Errorf("could not convert protobuf or raw node: %s", err)
		return nil, syscall.Errno(syscall.ENOENT)
	}

	return &Node{Ipfs: s.Ipfs, Nd: fnd}, nil
}

```

这段代码定义了一个名为 `ReadDirAll` 的函数，它读取一个特定的目录。然而，该函数在根目录下被禁止使用。

函数的实现包括以下步骤：

1. 首先，记录一些日志信息，以便在必要时进行调试。
2. 调用 `syscall.Errno()` 函数，将其作为函数的返回值，以便在日志中记录错误信息。
3. 如果返回值不为零，那么说明在读取目录时遇到了问题，将返回错误。
4. 如果返回值等于零，那么说明成功读取了目录并返回，将返回一个包含所有目录列表的切片。

函数的实现使用了以下数据结构：

* `Node` 类型：包含 `IpfsNode` 和 `ndipld.Node` 类型的节点结构体，用于表示文件系统树节点。
* `*Root` 类型：包含 `Node` 类型的指针，用于表示整个文件系统树。
* `*mdag.ProtoNode` 类型：包含 `mdag.Node` 类型的指针，用于表示文件系统数据结构中的 `mdag` 节点。
* `ft.FSNodeFromBytes` 函数：用于从字节数组中解析 `ft.FSNode` 类型的数据。

通过使用这些数据结构和函数，函数可以实现从指定目录中读取目录列表并返回的功能。需要注意的是，由于该函数在根目录下被禁止使用，因此需要进行特判以避免发生错误。


```
// ReadDirAll reads a particular directory. Disallowed for root.
func (*Root) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
	log.Debug("read Root")
	return nil, syscall.Errno(syscall.EPERM)
}

// Node is the core object representing a filesystem tree node.
type Node struct {
	Ipfs   *core.IpfsNode
	Nd     ipld.Node
	cached *ft.FSNode
}

func (s *Node) loadData() error {
	if pbnd, ok := s.Nd.(*mdag.ProtoNode); ok {
		fsn, err := ft.FSNodeFromBytes(pbnd.Data())
		if err != nil {
			return err
		}
		s.cached = fsn
	}
	return nil
}

```

这段代码定义了一个名为 `Attr` 的函数，接收一个名为 `a` 的 `fuse.Attr` 结构体参数，功能是获取指定节点的属性。

函数首先判断给定的 `s` 节点是否为原始节点类型，如果是，则执行以下操作：

1. 将 `a` 的 `Mode` 字段设置为 0o444，表示目录模式；
2. 将 `a` 的 `Size` 字段设置为 `len(rawnd.RawData())`，表示该目录下所有文件的大小之和；
3. 将 `a` 的 `Blocks` 字段设置为 1，表示该目录中文件个数的块数。

然后，如果 `s` 节点已经缓存了数据，函数将直接返回，不会执行加载数据的过程。

如果 `s` 节点没有缓存数据，函数会执行 `loadData` 函数来加载数据，并返回错误信息。如果加载数据成功，则根据 `s.cached.Type()` 获取到的数据类型，执行相应的写入操作。


```
// Attr returns the attributes of a given node.
func (s *Node) Attr(ctx context.Context, a *fuse.Attr) error {
	log.Debug("Node attr")
	if rawnd, ok := s.Nd.(*mdag.RawNode); ok {
		a.Mode = 0o444
		a.Size = uint64(len(rawnd.RawData()))
		a.Blocks = 1
		return nil
	}

	if s.cached == nil {
		if err := s.loadData(); err != nil {
			return fmt.Errorf("readonly: loadData() failed: %s", err)
		}
	}
	switch s.cached.Type() {
	case ft.TDirectory, ft.THAMTShard:
		a.Mode = os.ModeDir | 0o555
	case ft.TFile:
		size := s.cached.FileSize()
		a.Mode = 0o444
		a.Size = uint64(size)
		a.Blocks = uint64(len(s.Nd.Links()))
	case ft.TRaw:
		a.Mode = 0o444
		a.Size = uint64(len(s.cached.Data()))
		a.Blocks = uint64(len(s.Nd.Links()))
	case ft.TSymlink:
		a.Mode = 0o777 | os.ModeSymlink
		a.Size = uint64(len(s.cached.Data()))
	default:
		return fmt.Errorf("invalid data type - %s", s.cached.Type())
	}
	return nil
}

```

该代码定义了一个名为 `Lookup` 的函数，属于一个名为 `Node` 的 FUSE 树节点类中。

该函数接收一个名为 `name` 的字符串参数，并返回一个 `FSNode` 类型的结果，或者一个 `error` 类型的错误信息。

函数内部首先通过调用 `uio.ResolveUnixfsOnce` 函数，解决 FUSE 树节点中的数据链路代理（DAG）问题。该函数会尝试使用给定的 `name` 查找节点，并返回它的 Unix 文件系统路径。

如果查找失败，函数将返回一个 `nil` 值，并输出错误信息。如果查找成功，函数将返回包含 `Ipfs` 和 `Nd` 字段的 `Node` 实例，表示找到了包含给定 `name` 的节点。

函数的作用是帮助用户查找 FUSE 树中指定的节点，并在查找失败时给出错误信息。


```
// Lookup performs a lookup under this node.
func (s *Node) Lookup(ctx context.Context, name string) (fs.Node, error) {
	log.Debugf("Lookup '%s'", name)
	link, _, err := uio.ResolveUnixfsOnce(ctx, s.Ipfs.DAG, s.Nd, []string{name})
	switch err {
	case os.ErrNotExist, mdag.ErrLinkNotFound:
		// todo: make this error more versatile.
		return nil, syscall.Errno(syscall.ENOENT)
	case nil:
		// noop
	default:
		log.Errorf("fuse lookup %q: %s", name, err)
		return nil, syscall.Errno(syscall.EIO)
	}

	nd, err := s.Ipfs.DAG.Get(ctx, link.Cid)
	if err != nil && !ipld.IsNotFound(err) {
		log.Errorf("fuse lookup %q: %s", name, err)
		return nil, err
	}

	return &Node{Ipfs: s.Ipfs, Nd: nd}, nil
}

```

This is a function that retrieves a directory child node from an IPFS (InterPlanetary File System) cluster. It takes a `lnk` object as an input, which is a link to an IPFS directory node, and returns either a success or failure error.

The function first parses the name of the input `lnk` object to determine its directory parent. If the name is empty, it sets the name to the directory node's caterpillar ID (cid) converted to a string. The function then creates a `mdag.RawNode` object to represent the directory child node and retrieves its data from the IPFS cluster using the `Get` method of the `Ipfs.DAG` instance.

The function then checks the type of the data and converts it to the appropriate `fuse.DT_Type` based on its type. If the data is a file, it creates a `fuse.DT_File` entry. If it is a directory node, it creates a `fuse.DT_Dir` entry. If it is a symlink, it creates a `fuse.DT_Link` entry. If it is a metadata object, the function logs an error and returns immediately.

The function then appends the `fuse.Dirent` entries for the retrieved directory child node to a slice of entries and returns either the slice or an error. If any errors occur, the function returns immediately. If the function successfully retrieves and formats the entries, it returns them.


```
// ReadDirAll reads the link structure as directory entries.
func (s *Node) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
	log.Debug("Node ReadDir")
	dir, err := uio.NewDirectoryFromNode(s.Ipfs.DAG, s.Nd)
	if err != nil {
		return nil, err
	}

	var entries []fuse.Dirent
	err = dir.ForEachLink(ctx, func(lnk *ipld.Link) error {
		n := lnk.Name
		if len(n) == 0 {
			n = lnk.Cid.String()
		}
		nd, err := s.Ipfs.DAG.Get(ctx, lnk.Cid)
		if err != nil {
			log.Warn("error fetching directory child node: ", err)
		}

		t := fuse.DT_Unknown
		switch nd := nd.(type) {
		case *mdag.RawNode:
			t = fuse.DT_File
		case *mdag.ProtoNode:
			if fsn, err := ft.FSNodeFromBytes(nd.Data()); err != nil {
				log.Warn("failed to unmarshal protonode data field:", err)
			} else {
				switch fsn.Type() {
				case ft.TDirectory, ft.THAMTShard:
					t = fuse.DT_Dir
				case ft.TFile, ft.TRaw:
					t = fuse.DT_File
				case ft.TSymlink:
					t = fuse.DT_Link
				case ft.TMetadata:
					log.Error("metadata object in fuse should contain its wrapped type")
				default:
					log.Error("unrecognized protonode data type: ", fsn.Type())
				}
			}
		}
		entries = append(entries, fuse.Dirent{Name: n, Type: t})
		return nil
	})
	if err != nil {
		return nil, err
	}

	if len(entries) > 0 {
		return entries, nil
	}
	return nil, syscall.Errno(syscall.ENOENT)
}

```

这段代码是一个名为“func”的函数，它接收三个参数：

1. 是一个指向名为“Node”的上下文上下文（ctx）的指针（s *Node）。
2. 是一个名为“fuse.GetxattrRequest”的接口类型的参数，以及一个名为“fuse.GetxattrResponse”的接口类型的参数。
3. 是一个名为“ctx context.Context”的上下文上下文（ctx）的参数。

该函数的主要目的是在“Node”对象上执行与文件系统相关的操作，并返回相应的结果。

下面是函数的实现细节：

1. 函数首先检查“Node”对象是否为空，如果是，则返回一个名为“nil”的值，作为回应。
2. 如果“Node”对象不为空，那么它包含一个名为“cached”的子对象，需要先检查它是否为空，如果是，则返回一个名为“”的错误。否则，尝试从“cached”对象中读取数据，并返回它。
3. 如果从“cached”对象中成功读取数据，则返回该数据的字符串表示形式，并将其作为响应。
4. 如果从“cached”对象中无法读取数据，则返回一个名为“”的错误。
5. 接下来，函数尝试使用一个名为“nd”的输入文件系统直径（迪治）和一个名为“ipfs.DAG”的输入文件系统数据目录（迪加）创建一个输入读取器（r）。
6. 最后，函数使用“r.Seek”和“r.Read”方法从输入文件系统中读取数据，并将其存储在“req.Offset”和“resp.Data”中。如果从“cached”对象中读取数据时，遇到任何错误（如“io.EOF”、“io.ErrUnexpectedEOF”或“io.Err”），则返回一个错误。
7. 最后，函数使用“close”方法关闭输入读取器（r），并返回一个名为“nil”的错误。


```
func (s *Node) Getxattr(ctx context.Context, req *fuse.GetxattrRequest, resp *fuse.GetxattrResponse) error {
	// TODO: is nil the right response for 'bug off, we ain't got none' ?
	resp.Xattr = nil
	return nil
}

func (s *Node) Readlink(ctx context.Context, req *fuse.ReadlinkRequest) (string, error) {
	if s.cached == nil || s.cached.Type() != ft.TSymlink {
		return "", fuse.Errno(syscall.EINVAL)
	}
	return string(s.cached.Data()), nil
}

func (s *Node) Read(ctx context.Context, req *fuse.ReadRequest, resp *fuse.ReadResponse) error {
	r, err := uio.NewDagReader(ctx, s.Nd, s.Ipfs.DAG)
	if err != nil {
		return err
	}
	_, err = r.Seek(req.Offset, io.SeekStart)
	if err != nil {
		return err
	}
	// Data has a capacity of Size
	buf := resp.Data[:int(req.Size)]
	n, err := io.ReadFull(r, buf)
	resp.Data = buf[:n]
	switch err {
	case nil, io.EOF, io.ErrUnexpectedEOF:
	default:
		return err
	}
	resp.Data = resp.Data[:n]
	return nil // may be non-nil / not succeeded
}

```

这段代码定义了一个名为 `roRoot` 的接口类型，它包含以下字段：

* `fs.Node`：表示一个 `Node` 类型，该接口支持 `fs.HandleReadDirAller` 和 `fs.NodeStringLookuper` 接口。
* `null`：表示一个不存在的 `Root` 类型。

然后，代码创建了一个名为 `_` 的名为 `roRoot` 的 pointer 变量，并将其初始化为 `nil`。

接下来，定义了一个名为 `roNode` 的接口类型，它包含以下字段：

* `fs.HandleReadDirAller`：表示一个 `NodeReadDirAller` 类型，该接口支持 `fs.Node` 和 `fs.NodeStringLookuper` 接口。
* `fs.HandleReader`：表示一个 `NodeReader` 类型，该接口支持 `fs.Node` 和 `fs.NodeStringLookuper` 接口。
* `fs.Node`：表示一个 `Node` 类型，该接口支持 `fs.HandleReadDirAller` 和 `fs.NodeStringLookuper` 接口。
* `fs.NodeStringLookupper`：表示一个 `NodeStringLookupper` 类型，该接口支持 `fs.NodeStringLookuper` 接口。
* `fs.NodeReadlinker`：表示一个 `NodeReadlinker` 类型，该接口支持 `fs.Node` 和 `fs.NodeStringLookuper` 接口。
* `fs.NodeGetxattrer`：表示一个 `NodeGetxattrer` 类型，该接口支持 `fs.Node` 和 `fs.NodeStringLookuper` 接口。
* `fs.NodeStringLookupper`：表示一个 `NodeStringLookupper` 类型，该接口支持 `fs.NodeStringLookupper` 接口。

最后，代码创建了一个名为 `roNodeImpl` 的名为 `NodeImpl` 的实参类型，该实参类型包含了一个名为 `fs.NodeImpl` 的类型字段，该类型字段包含了一个 `Node` 类型字段和一个 `NodeStringLookupper` 类型字段。


```
// to check that our Node implements all the interfaces we want.
type roRoot interface {
	fs.Node
	fs.HandleReadDirAller
	fs.NodeStringLookuper
}

var _ roRoot = (*Root)(nil)

type roNode interface {
	fs.HandleReadDirAller
	fs.HandleReader
	fs.Node
	fs.NodeStringLookuper
	fs.NodeReadlinker
	fs.NodeGetxattrer
}

```

这段代码创建了一个名为`_roNode`的变量，并将其赋值为`(*Node)(nil)`。

在C++中，`*`符号表示取址，`(nil)`是一个表达式，表示一个空的智能指针。智能指针可以用来管理内存，比如在向量中分配内存时，就可以使用智能指针来确保只分配了足够的内存。

因此，`(*Node)(nil)`表示创建一个指向Node类型对象的智能指针，并将该智能指针初始化为 nil(指向一个空智能指针)。由于Node类型没有被定义，因此无法创建一个具体的Node对象，但可以定义一个名为`Node`的类型，以便在需要时创建Node对象。


```
var _ roNode = (*Node)(nil)

```