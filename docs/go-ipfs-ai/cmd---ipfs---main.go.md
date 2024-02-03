# `kubo\cmd\ipfs\main.go`

```go
# 导入 main 包
package main

# 导入 os 包
import (
    "os"

    # 导入 ipfs/kubo 包
    "github.com/ipfs/kubo/cmd/ipfs/kubo"
)

# 主函数
func main() {
    # 退出程序并返回给定状态码
    os.Exit(kubo.Start(kubo.BuildDefaultEnv))
}
```