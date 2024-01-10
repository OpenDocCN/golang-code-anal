# `kubo\test\bench\bench_cli_ipfs_add\main.go`

```
package main

import (
    "flag"  // 导入 flag 包，用于命令行参数解析
    "fmt"   // 导入 fmt 包，用于格式化输出
    "log"   // 导入 log 包，用于日志记录
    "os"    // 导入 os 包，提供对操作系统功能的访问
    "os/exec"   // 导入 os/exec 包，用于执行外部命令
    "path"  // 导入 path 包，用于处理文件路径
    "testing"   // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/thirdparty/unit"   // 导入第三方包 unit

    config "github.com/ipfs/kubo/config"  // 导入自定义包 config
    random "github.com/jbenet/go-random"  // 导入自定义包 random
)

var (
    debug  = flag.Bool("debug", false, "direct ipfs output to console")  // 定义命令行参数 debug，用于控制是否将 ipfs 输出直接输出到控制台
    online = flag.Bool("online", false, "run the benchmarks with a running daemon")  // 定义命令行参数 online，用于控制是否在运行守护进程时运行基准测试
)

func main() {
    flag.Parse()  // 解析命令行参数
    if err := compareResults(); err != nil {  // 调用 compareResults 函数，如果返回错误则记录日志并退出程序
        log.Fatal(err)
    }
}

func compareResults() error {
    var amount unit.Information  // 声明变量 amount，类型为 unit.Information
    for amount = 10 * unit.MB; amount > 0; amount = amount * 2 {  // 循环，每次 amount 乘以 2
        if results, err := benchmarkAdd(int64(amount)); err != nil { // 调用 benchmarkAdd 函数进行基准测试，如果返回错误则返回该错误
            return err
        } else {
            log.Println(amount, "\t", results)  // 记录基准测试结果
        }
    }
    return nil  // 返回空错误
}

func benchmarkAdd(amount int64) (*testing.BenchmarkResult, error) {
    var benchmarkError error  // 声明变量 benchmarkError，用于记录基准测试错误
    })  // 执行基准测试
    if benchmarkError != nil {  // 如果基准测试出现错误
        return nil, benchmarkError  // 返回空结果和错误
    }
    return &results, nil  // 返回基准测试结果和空错误
}
```