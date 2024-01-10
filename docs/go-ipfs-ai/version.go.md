# `kubo\version.go`

```
// 导入必要的包
package ipfs

import (
    "fmt"
    "runtime"

    "github.com/ipfs/kubo/repo/fsrepo"
)

// CurrentCommit 是当前的 git 提交，这是在 Makefile 中设置的 ldflag。
var CurrentCommit string

// CurrentVersionNumber 是当前应用程序的版本文字。
const CurrentVersionNumber = "0.26.0-dev"

const ApiVersion = "/kubo/" + CurrentVersionNumber + "/" //nolint

// GetUserAgentVersion 是 go-ipfs 使用的 libp2p 用户代理。
//
// 注意：当没有提交可用时，这将以 `/` 结尾。这是预期的。
func GetUserAgentVersion() string {
    userAgent := "kubo/" + CurrentVersionNumber + "/" + CurrentCommit
    if userAgentSuffix != "" {
        if CurrentCommit != "" {
            userAgent += "/"
        }
        userAgent += userAgentSuffix
    }
    return userAgent
}

var userAgentSuffix string

// SetUserAgentSuffix 设置用户代理后缀
func SetUserAgentSuffix(suffix string) {
    userAgentSuffix = suffix
}

// VersionInfo 是版本信息结构体
type VersionInfo struct {
    Version string
    Commit  string
    Repo    string
    System  string
    Golang  string
}

// GetVersionInfo 获取版本信息
func GetVersionInfo() *VersionInfo {
    return &VersionInfo{
        Version: CurrentVersionNumber,
        Commit:  CurrentCommit,
        Repo:    fmt.Sprint(fsrepo.RepoVersion),
        System:  runtime.GOARCH + "/" + runtime.GOOS, // TODO: Precise version here
        Golang:  runtime.Version(),
    }
}
```