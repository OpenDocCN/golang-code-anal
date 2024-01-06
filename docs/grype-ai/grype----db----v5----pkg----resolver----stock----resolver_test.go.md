# `grype\grype\db\v5\pkg\resolver\stock\resolver_test.go`

```
package stock

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
)

func TestResolver_Normalize(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例
		packageName string  // 包名
		normalized  string  // 标准化后的包名
	}{
		{
			packageName: "PyYAML",  // 测试用例1：包名为PyYAML
			normalized:  "pyyaml",  // 期望的标准化后的包名为pyyaml
		},
		{
			packageName: "oslo.concurrency",  // 测试用例2：包名为oslo.concurrency
			normalized:  "oslo.concurrency",  // 期望的标准化后的包名为oslo.concurrency
# 创建一个空的解析器对象
resolver := Resolver{}

# 遍历测试数据列表
for _, test := range tests {
    # 对测试数据中的包名进行规范化处理
    resolvedNames := resolver.Normalize(test.packageName)
    # 断言规范化后的包名与预期的规范化结果相等
    assert.Equal(t, resolvedNames, test.normalized)
}
这部分代码是一个函数的结束标志，表示函数的定义结束。
```