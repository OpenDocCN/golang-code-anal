# `kubo\cmd\ipfs\runmain_test.go`

```go
//go:build testrunmain
// +build testrunmain
// 标记此文件只在 testrunmain 构建标记下构建

package main_test
// 导入所需的包
import (
    "flag" // 导入 flag 包，用于命令行参数解析
    "fmt" // 导入 fmt 包，用于格式化输出
    "os" // 导入 os 包，提供对操作系统功能的访问
    "testing" // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/cmd/ipfs/kubo" // 导入自定义包
)

// this abuses go so much that I felt dirty writing this code
// but it is the only way to do it without writing custom compiler that would
// be a clone of go-build with go-test.
// 定义测试函数 TestRunMain
func TestRunMain(t *testing.T) {
    // 获取命令行参数
    args := flag.Args()
    // 重置 os.Args，将命令行参数添加到其中
    os.Args = append([]string{os.Args[0]}, args...)

    // 调用 kubo.Start 函数，传入默认环境参数，获取返回值
    ret := kubo.Start(kubo.BuildDefaultEnv)

    // 获取环境变量 IPFS_COVER_RET_FILE 的值
    p := os.Getenv("IPFS_COVER_RET_FILE")
    // 如果值不为空
    if len(p) != 0 {
        // 将返回值转换为字节数组，并写入指定文件
        os.WriteFile(p, []byte(fmt.Sprintf("%d\n", ret)), 0o777)
    }

    // 关闭输出，以便 go testing 不打印任何内容
    null, _ := os.OpenFile(os.DevNull, os.O_RDWR, 0755)
    os.Stderr = null
    os.Stdout = null
}
```