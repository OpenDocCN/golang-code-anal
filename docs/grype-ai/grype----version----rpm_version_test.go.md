# `grype\grype\version\rpm_version_test.go`

```
package version

import (
	"testing"
)

func TestVersionRpm(t *testing.T) {
	tests := []struct {
		v1     string   // 第一个版本号
		v2     string   // 第二个版本号
		result int      // 比较结果
	}{
		// 以下是测试用例
		// from https://github.com/anchore/anchore-engine/blob/a447ee951c2d4e17c2672553d7280cfdb5e5f193/tests/unit/anchore_engine/util/test_rpm.py
		{"1", "1", 0},  // 版本号相同
		{"4.19.0a-1.el7_5", "4.19.0c-1.el7", -1},  // 版本号比较
		{"4.19.0-1.el7_5", "4.21.0-1.el7", -1},  // 版本号比较
		{"4.19.01-1.el7_5", "4.19.10-1.el7_5", -1},  // 版本号比较
		{"4.19.0-1.el7_5", "4.19.0-1.el7", 1},  // 版本号比较
		{"4.19.0-1.el7_5", "4.17.0-1.el7", 1},  // 版本号比较
		{"4.19.01-1.el7_5", "4.19.1-1.el7_5", 0},  // 版本号比较
```

		// 定义一组测试用例，每个测试用例包含两个版本号和预期比较结果
		{"4.19.1-1.el7_5", "4.19.1-01.el7_5", 0},
		{"4.19.1", "4.19.1", 0},
		{"1.2.3-el7_5~snapshot1", "1.2.3-3-el7_5", -1},
		{"1:0", "0:1", 1},
		{"1:2", "1", 1},
		{"0:4.19.1-1.el7_5", "2:4.19.1-1.el7_5", -1},
		{"4:1.2.3-3-el7_5", "1.2.3-el7_5~snapshot1", 1},
		// 以下是一些非标准的比较，忽略了版本号的 epoch，因为只有一个版本号可用
		{"1:0", "1", -1},
		{"2:4.19.01-1.el7_5", "4.19.1-1.el7_5", 0},
		{"4.19.01-1.el7_5", "2:4.19.1-1.el7_5", 0},
		{"4.19.0-1.el7_5", "12:4.19.0-1.el7", 1},
		{"3:4.19.0-1.el7_5", "4.21.0-1.el7", -1},
	}

	// 遍历测试用例
	for _, test := range tests {
		// 生成测试用例的名称
		name := test.v1 + "_vs_" + test.v2
		// 运行测试
		t.Run(name, func(t *testing.T) {
			// 解析版本号 v1
			v1, err := newRpmVersion(test.v1)
			// 如果解析出错
			if err != nil {
# 如果创建 v1 失败，则输出错误信息
t.Fatalf("failed to create v1: %+v", err)

# 创建 v2 版本，如果失败则输出错误信息
v2, err := newRpmVersion(test.v2)
if err != nil {
    t.Fatalf("failed to create v2: %+v", err)
}

# 比较 v1 和 v2 的版本
actual := v1.compare(v2)

# 如果比较结果不符合预期，则输出错误信息
if actual != test.result {
    t.Errorf("bad result: %+v (expected: %+v)", actual, test.result)
}
```