# `kubebench-aquasecurity\main.go`

```
// 版权声明，声明代码版权归Aqua Security Software Ltd.所有
// 根据Apache License, Version 2.0许可证授权
// 除非符合许可证的规定，否则不得使用此文件
// 可以在以下网址获取许可证的副本：http://www.apache.org/licenses/LICENSE-2.0
// 除非适用法律要求或书面同意，否则根据许可证分发的软件是基于"AS IS"的基础分发的，没有任何明示或暗示的担保或条件
// 请查看许可证以了解特定语言的权限和限制

// 主程序包
package main

// 导入kube-bench/cmd包
import (
	"github.com/aquasecurity/kube-bench/cmd"
)
# 主函数入口，调用cmd包中的Execute函数
func main() {
    cmd.Execute()
}
```