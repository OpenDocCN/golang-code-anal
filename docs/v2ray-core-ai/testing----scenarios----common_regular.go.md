# `v2ray-core\testing\scenarios\common_regular.go`

```go
// +build !coverage

// 定义 scenarios 包，用于构建和运行 V2Ray 的测试场景
package scenarios

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "fmt"    // 导入 fmt 包，用于格式化输出
    "os"     // 导入 os 包，用于操作操作系统功能
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
)

// 构建 V2Ray 可执行文件
func BuildV2Ray() error {
    genTestBinaryPath()  // 生成测试二进制文件路径
    if _, err := os.Stat(testBinaryPath); err == nil {  // 检查测试二进制文件是否存在
        return nil  // 如果存在则返回空
    }

    fmt.Printf("Building V2Ray into path (%s)\n", testBinaryPath)  // 打印构建 V2Ray 的路径
    cmd := exec.Command("go", "build", "-o="+testBinaryPath, GetSourcePath())  // 执行构建命令
    return cmd.Run()  // 运行构建命令
}

// 运行 V2Ray 使用 Protobuf 格式的配置
func RunV2RayProtobuf(config []byte) *exec.Cmd {
    genTestBinaryPath()  // 生成测试二进制文件路径
    proc := exec.Command(testBinaryPath, "-config=stdin:", "-format=pb")  // 执行 V2Ray 命令，使用 Protobuf 格式的配置
    proc.Stdin = bytes.NewBuffer(config)  // 将配置数据写入标准输入流
    proc.Stderr = os.Stderr  // 将标准错误输出重定向到系统标准错误输出
    proc.Stdout = os.Stdout  // 将标准输出重定向到系统标准输出

    return proc  // 返回 V2Ray 进程
}
```