# `kubebench-aquasecurity\main.go`

```go
// 版权声明，声明代码版权归 Aqua Security Software Ltd. 所有
//
// 根据 Apache 许可证 2.0 版本授权
// 除非符合许可证的规定，否则不得使用此文件
// 您可以在以下网址获取许可证的副本
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// 除非适用法律要求或书面同意，否则按"原样"分发软件
// 没有任何明示或暗示的担保或条件
// 请参阅许可证以了解特定语言下的权限和限制

package main

// 导入 kube-bench/cmd 包
import (
    "github.com/aquasecurity/kube-bench/cmd"
)

// 主函数
func main() {
    // 执行 cmd 包中的 Execute 函数
    cmd.Execute()
}
```