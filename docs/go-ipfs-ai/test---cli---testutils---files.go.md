# `kubo\test\cli\testutils\files.go`

```
package testutils

import (
    "log"  // 导入日志包
    "os"   // 导入操作系统包
    "path/filepath"  // 导入路径包
)

func MustOpen(name string) *os.File {
    f, err := os.Open(name)  // 打开指定文件
    if err != nil {
        log.Panicf("opening %s: %s", name, err)  // 如果出错，记录错误信息并终止程序
    }
    return f  // 返回打开的文件
}

// 在指定目录及其父目录中搜索文件
// 如果找不到文件，则返回空字符串
func FindUp(name, dir string) string {
    curDir := dir  // 设置当前目录为指定目录
    for {
        entries, err := os.ReadDir(curDir)  // 读取当前目录的文件列表
        if err != nil {
            panic(err)  // 如果出错，终止程序并输出错误信息
        }
        for _, e := range entries {  // 遍历当前目录的文件列表
            if name == e.Name() {  // 如果找到指定文件
                return filepath.Join(curDir, name)  // 返回文件的完整路径
            }
        }
        newDir := filepath.Dir(curDir)  // 获取当前目录的父目录
        if newDir == curDir {  // 如果父目录和当前目录相同
            return ""  // 返回空字符串，表示未找到文件
        }
        curDir = newDir  // 将当前目录更新为父目录
    }
}
```