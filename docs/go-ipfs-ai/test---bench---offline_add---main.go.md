# `kubo\test\bench\offline_add\main.go`

```
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "log"  // 导入 log 包，用于日志记录
    "os"   // 导入 os 包，提供对操作系统功能的访问
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "path"  // 导入 path 包，用于处理文件路径
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/thirdparty/unit"  // 导入第三方包 unit，用于处理单位信息

    config "github.com/ipfs/kubo/config"  // 导入自定义包 config，用于处理配置信息
    random "github.com/jbenet/go-random"  // 导入第三方包 random，用于生成随机数
)

func main() {
    if err := compareResults(); err != nil {  // 如果比较结果出错，则记录错误并退出程序
        log.Fatal(err)
    }
}

func compareResults() error {
    var amount unit.Information  // 声明一个变量 amount，类型为 unit.Information
    for amount = 10 * unit.MB; amount > 0; amount = amount * 2 {  // 循环，每次 amount 乘以 2
        if results, err := benchmarkAdd(int64(amount)); err != nil {  // 调用 benchmarkAdd 函数进行基准测试，并处理可能的错误
            return err  // 如果出错，则返回错误
        } else {
            log.Println(amount, "\t", results)  // 记录测试数据和结果
        }
    }
    return nil  // 返回空错误，表示比较结果正常
}

func benchmarkAdd(amount int64) (*testing.BenchmarkResult, error) {
    results := testing.Benchmark(func(b *testing.B) {  // 进行基准测试
        b.SetBytes(amount)  // 设置测试数据大小
        for i := 0; i < b.N; i++ {  // 循环执行测试
            b.StopTimer()  // 暂停计时器
            tmpDir := b.TempDir()  // 获取临时目录

            env := append(os.Environ(), fmt.Sprintf("%s=%s", config.EnvDir, path.Join(tmpDir, config.DefaultPathName)))  // 设置环境变量
            setupCmd := func(cmd *exec.Cmd) {  // 定义设置命令的函数
                cmd.Env = env  // 设置命令的环境变量
            }

            cmd := exec.Command("ipfs", "init", "-b=2048")  // 创建执行命令对象
            setupCmd(cmd)  // 设置命令
            if err := cmd.Run(); err != nil {  // 执行命令并处理可能的错误
                b.Fatal(err)  // 如果出错，则记录错误并退出测试
            }

            const seed = 1  // 定义种子值
            f, err := os.CreateTemp("", "")  // 创建临时文件
            if err != nil {  // 处理可能的错误
                b.Fatal(err)  // 如果出错，则记录错误并退出测试
            }
            defer os.Remove(f.Name())  // 在函数返回前删除临时文件

            err = random.WritePseudoRandomBytes(amount, f, seed)  // 写入伪随机字节
            if err != nil {  // 处理可能的错误
                b.Fatal(err)  // 如果出错，则记录错误并退出测试
            }
            if err := f.Close(); err != nil {  // 关闭文件
                b.Fatal(err)  // 如果出错，则记录错误并退出测试
            }

            b.StartTimer()  // 启动计时器
            cmd = exec.Command("ipfs", "add", f.Name())  // 创建执行命令对象
            setupCmd(cmd)  // 设置命令
            if err := cmd.Run(); err != nil {  // 执行命令并处理可能的错误
                b.Fatal(err)  // 如果出错，则记录错误并退出测试
            }
            b.StopTimer()  // 暂停计时器
        }
    })
    return &results, nil  // 返回基准测试结果和空错误
}
```