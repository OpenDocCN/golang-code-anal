# `grype\cmd\grype\cli\cli_test.go`

```
package cli

import (
	"testing"  // 导入测试包

	"github.com/stretchr/testify/require"  // 导入断言库

	"github.com/anchore/clio"  // 导入自定义的包
)

func Test_Command(t *testing.T) {
	// 创建一个命令行对象，并指定名称和版本
	root := Command(clio.Identification{
		Name:    "test-name",
		Version: "test-version",
	})

	// 断言命令行对象的名称是否为"test-name"
	require.Equal(t, root.Name(), "test-name")
	// 断言命令行对象的命令列表不为空
	require.NotEmpty(t, root.Commands())
}
```