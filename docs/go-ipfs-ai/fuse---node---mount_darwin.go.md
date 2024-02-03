# `kubo\fuse\node\mount_darwin.go`

```go
// 如果不是 nofuse 构建标签，则编译此包
// +build !nofuse

package node

import (
    "bytes"  // 导入 bytes 包
    "fmt"  // 导入 fmt 包
    "os/exec"  // 导入 exec 包
    "runtime"  // 导入 runtime 包
    "strings"  // 导入 strings 包

    core "github.com/ipfs/kubo/core"  // 导入 core 包

    "github.com/blang/semver/v4"  // 导入 semver 包
    unix "golang.org/x/sys/unix"  // 导入 unix 包
)

func init() {
    // 这是一个 hack，但在需要另一种方式之前，这样可以工作。
    platformFuseChecks = darwinFuseCheckVersion  // 初始化 platformFuseChecks 变量
}

// dontCheckOSXFUSEConfigKey 是一个键，用于告诉用户跳过 fuse 检查。
const dontCheckOSXFUSEConfigKey = "DontCheckOSXFUSE"

// fuseVersionPkg 是 fuse-version 的 go 包 URL。
const fuseVersionPkg = "github.com/jbenet/go-fuse-version/fuse-version"

// errStrFuseRequired 在确定用户没有安装 fuse 时返回。
var errStrFuseRequired = `OSXFUSE not found.

OSXFUSE is required to mount, please install it.
NOTE: Version 2.7.2 or higher required; prior versions are known to kernel panic!
It is recommended you install it from the OSXFUSE website:

    http://osxfuse.github.io/

For more help, see:

    https://github.com/ipfs/kubo/issues/177
`

// errStrNoFuseHeaders 包含在 `go get <fuseVersionPkg>` 的输出中，如果没有 fuse 头文件，则表示他们没有安装 OSXFUSE。
var errStrNoFuseHeaders = "no such file or directory: '/usr/local/lib/libosxfuse.dylib'"

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

func (me errNeedFuseVersion) Error() string {
    return fmt.Sprintf(`unable to check fuse version.

Dear User,

Before mounting, we must check your version of OSXFUSE. We are protecting
you from a nasty kernel panic we found in OSXFUSE versions <2.7.2.[1]. To
// 错误信息模板，用于提示用户安装或检查 OSXFUSE 版本
var errStrFailedToRunFuseVersion = `unable to check fuse version.

Dear User,

Before mounting, we must check your version of OSXFUSE. We are protecting
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

Just make sure the number is 2.7.2 or higher. You can then stop ipfs from
trying to run these checks with:

    ipfs config --bool %s true

[1]: https://github.com/ipfs/kubo/issues/177
[2]: https://github.com/ipfs/kubo/pull/533
[3]: %s
`

// 错误信息模板，用于提示用户配置键无效
var errStrFixConfig = `config key invalid: %s %v
You may be able to get this error to go away by setting it again:

    ipfs config --bool %s true

Either way, please tell us at: http://github.com/ipfs/kubo/issues
`

// 检查 OSXFUSE 版本
func darwinFuseCheckVersion(node *core.IpfsNode) error {
    // 在 OSX 系统上检查 FUSE 版本
    if runtime.GOOS != "darwin" {
        return nil
    }

    // 尝试获取 FUSE 版本
    ov, errGFV := tryGFV()
    // 如果 errGFV 不为空，表示出现了错误
    if errGFV != nil {
        // 如果失败了并且用户告诉我们忽略检查，我们继续执行。
        // 这是为了防止 fuse 版本出现问题或用户无法安装它，但确信他们的 fuse 版本可以工作。
        if skip, err := userAskedToSkipFuseCheck(node); err != nil {
            return err
        } else if skip {
            return nil // 用户告诉我们不要检查版本...好的....
        }
        return errGFV
    }

    // 打印调试信息，显示 osxfuse 的版本
    log.Debug("mount: osxfuse version:", ov)

    // 定义最低支持的 osxfuse 版本
    min := semver.MustParse("2.7.2")
    // 解析当前的 osxfuse 版本
    curr, err := semver.Make(ov)
    if err != nil {
        return err
    }

    // 如果当前版本低于最低版本，返回错误
    if curr.LT(min) {
        return fmt.Errorf(errStrUpgradeFuse, ov)
    }
    // 没有错误，返回空
    return nil
func tryGFV() (string, error) {
    // 尝试使用 sysctl 获取版本号，可能成功！
    ov, err := trySysctl()
    if err == nil {
        return ov, nil
    }
    log.Debug(err)

    // 如果 sysctl 失败，则尝试从 Fuse 版本获取
    return tryGFVFromFuseVersion()
}

func trySysctl() (string, error) {
    // 使用 sysctl 获取 osxfuse.version.number
    v, err := unix.Sysctl("osxfuse.version.number")
    if err != nil {
        log.Debug("mount: sysctl osxfuse.version.number:", "failed")
        return "", err
    }
    log.Debug("mount: sysctl osxfuse.version.number:", v)
    return v, nil
}

func tryGFVFromFuseVersion() (string, error) {
    // 确保 Fuse 版本已安装
    if err := ensureFuseVersionIsInstalled(); err != nil {
        return "", err
    }

    // 执行 fuse-version 命令获取版本信息
    cmd := exec.Command("fuse-version", "-q", "-only", "agent", "-s", "OSXFUSE")
    out := new(bytes.Buffer)
    cmd.Stdout = out
    if err := cmd.Run(); err != nil {
        return "", fmt.Errorf(errStrFailedToRunFuseVersion, fuseVersionPkg, dontCheckOSXFUSEConfigKey, err)
    }

    return out.String(), nil
}

func ensureFuseVersionIsInstalled() error {
    // 检查是否存在 fuse-version
    if _, err := exec.LookPath("fuse-version"); err == nil {
        return nil // 存在！
    }

    // 尝试安装 fuse-version
    log.Debug("fuse-version: no fuse-version. attempting to install.")
    cmd := exec.Command("go", "install", "github.com/jbenet/go-fuse-version/fuse-version")
    cmdout := new(bytes.Buffer)
    cmd.Stdout = cmdout
    cmd.Stderr = cmdout
    if err := cmd.Run(); err != nil {
        // 安装 fuse-version 失败，可能是缺少 fuse 头文件
        cmdoutstr := cmdout.String()
        if strings.Contains(cmdoutstr, errStrNoFuseHeaders) {
            // 是的！缺少 fuse 头文件！
            return fmt.Errorf(errStrFuseRequired)
        }

        log.Debug("fuse-version: failed to install.")
        s := err.Error() + "\n" + cmdoutstr
        return errNeedFuseVersion{s}
    }

    // 再次尝试...
    # 检查是否存在名为 "fuse-version" 的可执行文件
    if _, err := exec.LookPath("fuse-version"); err != nil:
        # 如果不存在，则记录错误信息并返回需要 fuse 版本的错误
        log.Debug("fuse-version: failed to install?")
        return errNeedFuseVersion{err.Error()}
    
    # 如果存在，则记录安装成功的信息
    log.Debug("fuse-version: install success")
    # 返回空值表示安装成功
    return nil
func userAskedToSkipFuseCheck(node *core.IpfsNode) (skip bool, err error) {
    // 从节点的存储库中获取配置键值
    val, err := node.Repo.GetConfigKey(dontCheckOSXFUSEConfigKey)
    // 如果获取配置值出错，则返回 false，不跳过检查
    if err != nil {
        return false, nil
    }

    // 根据配置值的类型进行不同的处理
    switch val := val.(type) {
    // 如果配置值是字符串类型，则返回是否等于 "true"
    case string:
        return val == "true", nil
    // 如果配置值是布尔类型，则直接返回该值
    case bool:
        return val, nil
    // 如果配置值类型不是字符串或布尔类型，则返回错误信息
    default:
        return false, fmt.Errorf(errStrFixConfig, dontCheckOSXFUSEConfigKey, val,
            dontCheckOSXFUSEConfigKey)
    }
}
```