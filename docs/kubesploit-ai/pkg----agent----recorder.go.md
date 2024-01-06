# `kubesploit\pkg\agent\recorder.go`

```
// 定义一个名为 agent 的包
package agent

// 导入 fmt、os、strings 和 time 包
import (
	"fmt"
	"os"
	"strings"
	"time"
)

// 定义 Recorder 结构体
type Recorder struct {
	FileName string // 文件名属性
}

// 创建一个新的 Recorder 对象
func NewRecorder(filename string) Recorder {
	// 初始化一个 Recorder 对象并设置文件名属性
	r := Recorder{
		FileName: filename,
	}

	// 返回创建的 Recorder 对象
	return r
}
# 记录命令和参数到文件
func (r *Recorder) recordCommand(command string, args []string){
    # 创建一个字符串构建器
    var sb strings.Builder
    # 将时间戳和提示信息写入字符串构建器
    sb.WriteString(fmt.Sprintf("%s [*] Payload:\n", time.Now().UTC().Format(time.RFC3339)))
    # 将命令写入字符串构建器
    sb.WriteString(command + "\n")
    # 遍历参数列表，将每个参数写入字符串构建器
    for _, arg := range(args) {
        sb.WriteString(arg + "\n")
    }
    # 写入换行符
    sb.WriteString("\n")
    # 将字符串构建器中的内容记录到文件
    recordToFile(sb.String())
}

# 记录输出到文件
func (r *Recorder) recordOutput(output string){
    # 创建一个字符串构建器
    var sb strings.Builder
    # 将时间戳和提示信息写入字符串构建器
    sb.WriteString(fmt.Sprintf("%s [*] Output:\n", time.Now().UTC().Format(time.RFC3339)))
    # 将输出内容写入字符串构建器
    sb.WriteString(output + "\n")
    # 将字符串构建器中的内容记录到文件
    recordToFile(sb.String())
}
# 将记录写入文件
func recordToFile(output string){
    # 定义文件名
    filename := "/tmp/agent_record"

    # 打开文件，如果文件不存在则创建，以追加模式写入，权限为0644
    f, err := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0644)
    if err != nil {
        panic(err)
    }

    # 延迟关闭文件
    defer f.Close()
    
    # 将记录写入文件
    if _, err = f.WriteString(output); err != nil {
        # 如果写入失败，输出警告信息
        message("warn", fmt.Sprintf("Failed to write to file: %s, error: %s", filename, output))
    }
}
```