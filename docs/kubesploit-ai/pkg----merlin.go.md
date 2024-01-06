# `kubesploit\pkg\merlin.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参阅<http://www.gnu.org/licenses/>。

// 包名为kubesploitVersion
// Version 是一个包含 Kubesploit 版本号的常量变量
const Version = "0.1.3"

// MerlinVersion 是一个包含 Merlin 包版本号的常量变量
const MerlinVersion = "0.9.1"

// Build 是根据编译时的 git 提交生成的唯一编号
var Build = "nonRelease"
```