# `kubesploit\pkg\agent\recorder.go`

```go
package agent

import (
    "fmt"  // 导入格式化输出包
    "os"  // 导入操作系统包
    "strings"  // 导入字符串处理包
    "time"  // 导入时间包
)

type Recorder struct {
    FileName string  // 定义 Recorder 结构体，包含文件名属性
}

func NewRecorder(filename string) Recorder {
    r := Recorder{  // 创建 Recorder 结构体实例
        FileName: filename,  // 设置文件名属性
    }

    return r  // 返回 Recorder 实例
}

func (r *Recorder) recordCommand(command string, args []string){
    var sb strings.Builder  // 创建字符串构建器
    sb.WriteString(fmt.Sprintf("%s [*] Payload:\n", time.Now().UTC().Format(time.RFC3339)))  // 格式化时间并写入字符串构建器
    sb.WriteString(command + "\n")  // 将命令写入字符串构建器
    for _, arg := range(args) {  // 遍历参数列表
        sb.WriteString(arg + "\n")  // 将参数写入字符串构建器
    }
    sb.WriteString("\n")  // 写入空行
    recordToFile(sb.String())  // 调用 recordToFile 函数记录到文件
}

func (r *Recorder) recordOutput(output string){
    var sb strings.Builder  // 创建字符串构建器
    sb.WriteString(fmt.Sprintf("%s [*] Output:\n", time.Now().UTC().Format(time.RFC3339)))  // 格式化时间并写入字符串构建器
    sb.WriteString(output + "\n")  // 将输出写入字符串构建器
    recordToFile(sb.String())  // 调用 recordToFile 函数记录到文件
}


func recordToFile(output string){
    filename := "/tmp/agent_record"  // 设置文件名

    f, err := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0644)  // 打开文件，如果不存在则创建
    if err != nil {  // 如果出现错误
        panic(err)  // 抛出异常
    }

    defer f.Close()  // 延迟关闭文件
    if _, err = f.WriteString(output); err != nil {  // 如果写入文件出现错误
        message("warn", fmt.Sprintf("Failed to write to file: %s, error: %s", filename, output))  // 输出警告信息
    }
}
```