# `v2ray-core\infra\vprotogen\main.go`

```
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，用于操作系统功能
    "os/exec"  // 导入 exec 包，用于执行外部命令
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "runtime"  // 导入 runtime 包，用于获取运行时信息
    "strings"  // 导入 strings 包，用于处理字符串

    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
)

func main() {
    pwd, wdErr := os.Getwd()  // 获取当前工作目录
    if wdErr != nil {  // 如果获取失败
        fmt.Println("Can not get current working directory.")  // 输出错误信息
        os.Exit(1)  // 退出程序
    }

    GOBIN := common.GetGOBIN()  // 获取 GOBIN 路径
    protoc := core.ProtocMap[runtime.GOOS]  // 获取当前操作系统对应的 protoc 路径

    protoFilesMap := make(map[string][]string)  // 创建存放 proto 文件路径的 map
    walkErr := filepath.Walk("./", func(path string, info os.FileInfo, err error) error {  // 遍历当前目录下的文件
        if err != nil {  // 如果遍历出错
            fmt.Println(err)  // 输出错误信息
            return err  // 返回错误
        }

        if info.IsDir() {  // 如果是目录
            return nil  // 返回空
        }

        dir := filepath.Dir(path)  // 获取文件所在目录
        filename := filepath.Base(path)  // 获取文件名
        if strings.HasSuffix(filename, ".proto") {  // 如果文件是以 .proto 结尾
            protoFilesMap[dir] = append(protoFilesMap[dir], path)  // 将文件路径添加到 map 中
        }

        return nil  // 返回空
    })
    if walkErr != nil {  // 如果遍历出错
        fmt.Println(walkErr)  // 输出错误信息
        os.Exit(1)  // 退出程序
    }

    for _, files := range protoFilesMap {  // 遍历 proto 文件路径的 map
        for _, relProtoFile := range files {  // 遍历每个目录下的 proto 文件
            var args []string  // 创建参数列表
            if core.ProtoFilesUsingProtocGenGoFast[relProtoFile] {  // 如果使用 protoc-gen-go-fast
                args = []string{"--gofast_out", pwd, "--plugin", "protoc-gen-gofast=" + GOBIN + "/protoc-gen-gofast"}  // 设置参数
            } else {  // 否则
                args = []string{"--go_out", pwd, "--go-grpc_out", pwd, "--plugin", "protoc-gen-go=" + GOBIN + "/protoc-gen-go", "--plugin", "protoc-gen-go-grpc=" + GOBIN + "/protoc-gen-go-grpc"}  // 设置参数
            }
            args = append(args, relProtoFile)  // 添加 proto 文件路径到参数列表
            cmd := exec.Command(protoc, args...)  // 创建执行命令
            cmd.Env = append(cmd.Env, os.Environ()...)  // 设置命令环境变量
            cmd.Env = append(cmd.Env, "GOBIN="+GOBIN)  // 设置 GOBIN 环境变量
            output, cmdErr := cmd.CombinedOutput()  // 执行命令并获取输出
            if len(output) > 0 {  // 如果有输出
                fmt.Println(string(output))  // 输出输出内容
            }
            if cmdErr != nil {  // 如果执行命令出错
                fmt.Println(cmdErr)  // 输出错误信息
                os.Exit(1)  // 退出程序
            }
        }
    }
}
    // 调用 common 包中的 GetModuleName 函数，获取模块名称和可能的错误
    moduleName, gmnErr := common.GetModuleName(pwd)
    // 如果获取模块名称时发生错误，打印错误信息并退出程序
    if gmnErr != nil {
        fmt.Println(gmnErr)
        os.Exit(1)
    }
    // 将模块名称转换为路径，并赋给 modulePath
    modulePath := filepath.Join(strings.Split(moduleName, "/")...)

    // 创建一个映射，用于存储目录和对应的 .pb.go 文件路径
    pbGoFilesMap := make(map[string][]string)
    // 遍历模块路径下的所有文件和目录
    walkErr2 := filepath.Walk(modulePath, func(path string, info os.FileInfo, err error) error {
        // 如果遍历过程中出现错误，打印错误信息并返回错误
        if err != nil {
            fmt.Println(err)
            return err
        }

        // 如果当前路径是目录，则跳过
        if info.IsDir() {
            return nil
        }

        // 获取当前路径的父目录和文件名
        dir := filepath.Dir(path)
        filename := filepath.Base(path)
        // 如果文件名以 .pb.go 结尾，则将其路径添加到映射中
        if strings.HasSuffix(filename, ".pb.go") {
            pbGoFilesMap[dir] = append(pbGoFilesMap[dir], path)
        }

        return nil
    })
    // 如果遍历过程中出现错误，打印错误信息并退出程序
    if walkErr2 != nil {
        fmt.Println(walkErr2)
        os.Exit(1)
    }

    var err error
    # 遍历 pbGoFilesMap 中的值（切片类型）
    for _, srcPbGoFiles := range pbGoFilesMap {
        # 遍历 srcPbGoFiles 中的值（字符串类型）
        for _, srcPbGoFile := range srcPbGoFiles {
            # 声明变量 dstPbGoFile 为字符串类型
            var dstPbGoFile string
            # 获取 srcPbGoFile 相对于 modulePath 的相对路径
            dstPbGoFile, err = filepath.Rel(modulePath, srcPbGoFile)
            # 如果出现错误，打印错误信息并继续下一次循环
            if err != nil {
                fmt.Println(err)
                continue
            }
            # 创建硬链接，将 srcPbGoFile 链接到 dstPbGoFile
            err = os.Link(srcPbGoFile, dstPbGoFile)
            # 如果出现错误
            if err != nil {
                # 如果文件不存在，打印错误信息并继续下一次循环
                if os.IsNotExist(err) {
                    fmt.Printf("'%s' does not exist\n", srcPbGoFile)
                    continue
                }
                # 如果没有权限，打印错误信息并继续下一次循环
                if os.IsPermission(err) {
                    fmt.Println(err)
                    continue
                }
                # 如果文件已存在
                if os.IsExist(err) {
                    # 删除目标文件
                    err = os.Remove(dstPbGoFile)
                    # 如果删除失败，打印错误信息并继续下一次循环
                    if err != nil {
                        fmt.Printf("Failed to delete file '%s'\n", dstPbGoFile)
                        continue
                    }
                    # 移动 srcPbGoFile 到 dstPbGoFile
                    err = os.Rename(srcPbGoFile, dstPbGoFile)
                    # 如果移动失败，打印错误信息
                    if err != nil {
                        fmt.Printf("Can not move '%s' to '%s'\n", srcPbGoFile, dstPbGoFile)
                    }
                    continue
                }
            }
            # 移动 srcPbGoFile 到 dstPbGoFile
            err = os.Rename(srcPbGoFile, dstPbGoFile)
            # 如果移动失败，打印错误信息
            if err != nil {
                fmt.Printf("Can not move '%s' to '%s'\n", srcPbGoFile, dstPbGoFile)
            }
            continue
        }
    }

    # 如果没有错误
    if err == nil {
        # 移除 modulePath 中第一个斜杠之前的所有内容
        err = os.RemoveAll(strings.Split(modulePath, "/")[0])
        # 如果移除失败，打印错误信息
        if err != nil {
            fmt.Println(err)
        }
    }
# 闭合前面的函数定义
```