# `grype\cmd\grype\cli\cli_test.go`

```go
# 导入 cli 包
package cli

# 导入 testing 包
import (
    "testing"

    # 导入 testify 包中的 require 模块
    "github.com/stretchr/testify/require"

    # 导入 anchore 包中的 clio 模块
    "github.com/anchore/clio"
)

# 定义测试函数 Test_Command
func Test_Command(t *testing.T) {
    # 创建一个命令行对象 root，使用 clio.Identification 结构体初始化
    root := Command(clio.Identification{
        Name:    "test-name",
        Version: "test-version",
    })

    # 断言 root 的名称为 "test-name"
    require.Equal(t, root.Name(), "test-name")
    # 断言 root 的命令列表不为空
    require.NotEmpty(t, root.Commands())
}
```