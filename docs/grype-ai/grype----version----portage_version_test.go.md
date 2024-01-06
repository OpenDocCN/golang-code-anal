# `grype\grype\version\portage_version_test.go`

```
package version

import (
	"testing"
)

func TestVersionPortage(t *testing.T) {
	tests := []struct {  // 定义测试用例结构体
		v1     string      // 版本1
		v2     string      // 版本2
		result int         // 预期结果
	}{
		{"1", "1", 0},      // 版本1和版本2相同，预期结果为0
		{"12.2.5", "12.2b", 1},  // 版本1大于版本2，预期结果为1
		{"12.2a", "12.2b", -1}, // 版本1小于版本2，预期结果为-1
		{"12.2", "12.2.0", -1}, // 版本1小于版本2，预期结果为-1
		{"1.01", "1.1", -1},    // 版本1小于版本2，预期结果为-1
		{"1_p1", "1_p0", 1},    // 版本1大于版本2，预期结果为1
		{"1_p0", "1", 1},       // 版本1大于版本2，预期结果为1
		{"1-r1", "1", 1},       // 版本1大于版本2，预期结果为1
# 定义测试用例，每个测试用例包括两个版本号和预期比较结果
tests := []struct {
	v1     string
	v2     string
	result int
}{
	{"1.2.3-r2", "1.2.3-r1", 1},  # 版本号1比版本号2大，预期比较结果为1
	{"1.2.3-r1", "1.2.3-r2", -1}, # 版本号1比版本号2小，预期比较结果为-1
}

# 遍历测试用例
for _, test := range tests {
	# 拼接测试名称
	name := test.v1 + "_vs_" + test.v2
	# 运行测试
	t.Run(name, func(t *testing.T) {
		# 创建版本号对象
		v1 := newPortageVersion(test.v1)
		v2 := newPortageVersion(test.v2)

		# 执行版本号比较
		actual := v1.compare(v2)

		# 检查实际比较结果是否与预期一致
		if actual != test.result {
			t.Errorf("bad result: %+v (expected: %+v)", actual, test.result)
		}
	})
}
```