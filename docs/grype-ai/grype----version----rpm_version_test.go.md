# `grype\grype\version\rpm_version_test.go`

```
package version

import (
    "testing"
)

func TestVersionRpm(t *testing.T) {
    tests := []struct {
        v1     string
        v2     string
        result int
    }{
        // 定义测试用例，包括两个版本号和它们的比较结果
        // 来自 https://github.com/anchore/anchore-engine/blob/a447ee951c2d4e17c2672553d7280cfdb5e5f193/tests/unit/anchore_engine/util/test_rpm.py
        {"1", "1", 0},  // 版本号相同，比较结果为0
        {"4.19.0a-1.el7_5", "4.19.0c-1.el7", -1},  // 版本号不同，比较结果为-1
        // ... 其他测试用例
    }

    for _, test := range tests {
        name := test.v1 + "_vs_" + test.v2
        t.Run(name, func(t *testing.T) {
            v1, err := newRpmVersion(test.v1)  // 创建版本对象v1
            if err != nil {
                t.Fatalf("failed to create v1: %+v", err)  // 如果创建失败，输出错误信息
            }

            v2, err := newRpmVersion(test.v2)  // 创建版本对象v2
            if err != nil {
                t.Fatalf("failed to create v2: %+v", err)  // 如果创建失败，输出错误信息
            }

            actual := v1.compare(v2)  // 比较两个版本对象

            if actual != test.result {
                t.Errorf("bad result: %+v (expected: %+v)", actual, test.result)  // 输出比较结果与期望结果不符的错误信息
            }
        })
    }
}
```