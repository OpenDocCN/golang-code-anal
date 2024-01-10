# `kubo\cmd\ipfs\util\ulimit_test.go`

```
//go:build !windows && !plan9
// +build !windows,!plan9

package util

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，用于操作操作系统功能
    "strings"  // 导入 strings 包，用于处理字符串
    "syscall"  // 导入 syscall 包，用于访问操作系统底层接口
    "testing"  // 导入 testing 包，用于编写测试函数
)

func TestManageFdLimit(t *testing.T) {
    t.Log("Testing file descriptor count")  // 输出测试信息
    if _, _, err := ManageFdLimit(); err != nil {  // 调用 ManageFdLimit 函数，检查是否出错
        t.Errorf("Cannot manage file descriptors")  // 输出错误信息
    }

    if maxFds != uint64(8192) {  // 检查最大文件描述符数是否为默认值
        t.Errorf("Maximum file descriptors default value changed")  // 输出错误信息
    }
}

func TestManageInvalidNFds(t *testing.T) {
    t.Logf("Testing file descriptor invalidity")  // 输出测试信息
    var err error  // 声明错误变量
    if err = os.Unsetenv("IPFS_FD_MAX"); err != nil {  // 尝试清除环境变量 IPFS_FD_MAX
        t.Fatal("Cannot unset the IPFS_FD_MAX env variable")  // 输出致命错误信息
    }

    rlimit := syscall.Rlimit{}  // 声明 syscall.Rlimit 结构体变量
    if err = syscall.Getrlimit(syscall.RLIMIT_NOFILE, &rlimit); err != nil {  // 获取系统文件描述符限制
        t.Fatal("Cannot get the file descriptor count")  // 输出致命错误信息
    }

    value := rlimit.Max + rlimit.Cur  // 计算文件描述符限制的总和
    if err = os.Setenv("IPFS_FD_MAX", fmt.Sprintf("%d", value)); err != nil {  // 设置环境变量 IPFS_FD_MAX
        t.Fatal("Cannot set the IPFS_FD_MAX env variable")  // 输出致命错误信息
    }

    t.Logf("setting ulimit to %d, max %d, cur %d", value, rlimit.Max, rlimit.Cur)  // 输出设置 ulimit 的信息

    if changed, new, err := ManageFdLimit(); err == nil {  // 调用 ManageFdLimit 函数，检查是否出错
        t.Errorf("ManageFdLimit should return an error: changed %t, new: %d", changed, new)  // 输出错误信息
    } else {
        flag := strings.Contains(err.Error(), "failed to raise ulimit to IPFS_FD_MAX")  // 检查错误信息中是否包含特定字符串
        if !flag {
            t.Error("ManageFdLimit returned unexpected error", err)  // 输出错误信息
        }
    }

    // unset all previous operations
    if err = os.Unsetenv("IPFS_FD_MAX"); err != nil {  // 清除环境变量 IPFS_FD_MAX
        t.Fatal("Cannot unset the IPFS_FD_MAX env variable")  // 输出致命错误信息
    }
}

func TestManageFdLimitWithEnvSet(t *testing.T) {
    t.Logf("Testing file descriptor manager with IPFS_FD_MAX set")  // 输出测试信息
    var err error  // 声明错误变量
    if err = os.Unsetenv("IPFS_FD_MAX"); err != nil {  // 尝试清除环境变量 IPFS_FD_MAX
        t.Fatal("Cannot unset the IPFS_FD_MAX env variable")  // 输出致命错误信息
    }

    rlimit := syscall.Rlimit{}  // 声明 syscall.Rlimit 结构体变量
    // 获取系统文件描述符限制
    if err = syscall.Getrlimit(syscall.RLIMIT_NOFILE, &rlimit); err != nil {
        t.Fatal("Cannot get the file descriptor count")
    }

    // 计算可用文件描述符数量
    value := rlimit.Max - rlimit.Cur + 1
    // 设置环境变量 IPFS_FD_MAX 为可用文件描述符数量
    if err = os.Setenv("IPFS_FD_MAX", fmt.Sprintf("%d", value)); err != nil {
        t.Fatal("Cannot set the IPFS_FD_MAX env variable")
    }

    // 管理文件描述符限制
    if _, _, err = ManageFdLimit(); err != nil {
        t.Errorf("Cannot manage file descriptor count")
    }

    // 取消之前设置的环境变量
    if err = os.Unsetenv("IPFS_FD_MAX"); err != nil {
        t.Fatal("Cannot unset the IPFS_FD_MAX env variable")
    }
# 闭合前面的函数定义
```