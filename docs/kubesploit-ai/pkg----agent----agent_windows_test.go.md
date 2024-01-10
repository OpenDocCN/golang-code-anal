# `kubesploit\pkg\agent\agent_windows_test.go`

```
// +build windows

package agent

import (
    "bytes"
    "testing"
)

func TestGetProcess(t *testing.T) {
    // 确保绝对存在的进程返回一个值
    lsassPid, _, err := getProccess("lsass.exe", 0)
    if lsassPid == 0 || err != nil {
        t.Error("找不到 lsass.exe")
    }
    // 确保绝对不存在的进程返回一个零值
    _, garbagePid, err := getProccess("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.exe")
    if garbagePid != 0 || err == nil {
        t.Error("对于垃圾进程返回了一个非零值")
    }
}

func TestMinidump(t *testing.T) {

    // 检查一个良好的 minidump 是否有效
    md, err := miniDump("", "go.exe", 0)
    byts := md.FileContent
    if err != nil {
        t.Error("在已知进程上失败的 minidump（如果在非 Windows 环境中运行可能是假阳性）:", err)
    }
    if bytes.Compare(byts[:4], []byte("MDMP")) != 0 {
        t.Error("生成的 minidump 文件无效（基于文件头）")
    }

    // 检查一个未知进程的 minidump 是否有效
    _, err = miniDump("", "notarealprocess.exe", 0)
    if err == nil {
        t.Error("在不应该找到进程时找到了...")
    }

    // 检查使用空字符串提供 pid 的 minidump 是否有效
    pid, _, err := getProccess("go.exe", 0)
    md, err = miniDump("", "", pid)
    byts = md.FileContent
    if err != nil || len(byts) == 0 {
        t.Error("使用 pid 失败的 minidump")
    }

    // 验证进程名称是否匹配
    if md.ProcName != "go.exe" {
        t.Error("Minidump 进程名称不匹配: ", "go.exe", md.ProcName)
    }

    // 检查使用有效 pid 但无效字符串的 minidump 是否有效（pid 应优先）
    md, err = miniDump("", "notarealprocess.exe", pid)
    byts = md.FileContent
    if err != nil || len(byts) == 0 {
        t.Error("使用有效 pid 和无效进程名称失败的 minidump")
    }
}
    // 如果进程名称不是"go.exe"，则输出错误信息
    if md.ProcName != "go.exe" {
        t.Error("Minidump proc name does not match: ", "go.exe", md.ProcName)
    }

    // 检查具有有效进程名称但无效进程ID的minidump是否失败
    md, err = miniDump("", "go.exe", 123456789)
    byts = md.FileContent
    if err == nil {
        t.Error("Minidump dumped a process even though provided pid was invalid")
    }

    // 检查不存在的路径（目录）
    md, err = miniDump("C:\\thispathbetternot\\exist\\", "go.exe", 0)
    if err == nil {
        t.Error("Didn't get an error on non-existing path (check to make sure the path doesn't actually exist)")
    }

    // 检查存在的路径（目录）
    md, err = miniDump("C:\\Windows\\temp\\", "go.exe", 0)
    if err != nil {
        t.Error("Got an error on existing path (check to make sure the path actually exists)")
        t.Error(err)
    }

    // 检查存在的文件
    md, err = miniDump("C:\\Windows\\System32\\calc.exe", "go.exe", 0)
    if err == nil {
        t.Error("Didn't get an error on existing file (check to make sure the path & file actually exist)")
    }
# 闭合前面的函数定义
```