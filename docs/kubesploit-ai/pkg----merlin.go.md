# `kubesploit\pkg\merlin.go`

```go
// Kubesploit 是建立在 Russel Van Tuyl 的 Merlin 之上的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit 是自由软件：您可以根据 GNU 通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第 3 版还是任何以后的版本。

// Kubesploit 的分发希望能够有助于增强组织的安全性。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，没有任何保证；包括适销性或特定用途适用性的暗示保证。请参阅
// GNU 通用公共许可证以获取更多详细信息。

// 您应该已经收到了 GNU 通用公共许可证的副本。
// 如果没有，请参阅 <http://www.gnu.org/licenses/>。

package kubesploitVersion

// Version 是一个包含 Kubesploit 版本号的常量变量
const Version = "0.1.3"

// MerlinVersion 是一个包含 Merlin 包版本号的常量变量
const MerlinVersion = "0.9.1"

// Build 是基于编译时的 git 提交生成的唯一编号
var Build = "nonRelease"
```