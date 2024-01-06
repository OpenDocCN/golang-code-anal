# `grype\grype\db\v5\pkg\resolver\python\resolver_test.go`

```
package python  // 导入 python 包

import (  // 导入测试和断言包
	"testing"
	"github.com/stretchr/testify/assert"
)

func TestResolver_Normalize(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例
		packageName string  // 包名
		normalized  string  // 标准化后的包名
	}{
		{
			packageName: "PyYAML",  // 测试用例1：包名为 PyYAML
			normalized:  "pyyaml",  // 标准化后的包名为 pyyaml
		},
		{
			packageName: "oslo.concurrency",  // 测试用例2：包名为 oslo.concurrency
			normalized:  "oslo-concurrency",  // 标准化后的包名为 oslo-concurrency
		},
		{
			packageName: "",  # 设置包名为空字符串
			normalized:  "",  # 设置规范化后的包名为空字符串
		},
		{
			packageName: "test---1",  # 设置包名为"test---1"
			normalized:  "test-1",  # 设置规范化后的包名为"test-1"
		},
		{
			packageName: "AbCd.-__.--.-___.__.--1234____----....XyZZZ",  # 设置包名为"AbCd.-__.--.-___.__.--1234____----....XyZZZ"
			normalized:  "abcd-1234-xyzzz",  # 设置规范化后的包名为"abcd-1234-xyzzz"
		},
	}

	resolver := Resolver{}  # 创建 Resolver 对象

	for _, test := range tests:  # 遍历测试用例
		resolvedNames := resolver.Normalize(test.packageName)  # 调用 Resolver 对象的 Normalize 方法，对包名进行规范化
		assert.Equal(t, resolvedNames, test.normalized)  # 使用断言检查规范化后的包名是否符合预期
这部分代码缺少上下文，无法添加注释。
```