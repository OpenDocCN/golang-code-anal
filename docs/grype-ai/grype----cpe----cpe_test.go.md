# `grype\grype\cpe\cpe_test.go`

```go
package cpe

import (
    "testing"  // 导入测试包
    "github.com/sergi/go-diff/diffmatchpatch"  // 导入用于比较字符串差异的包
    "github.com/anchore/syft/syft/cpe"  // 导入CPE包
)

func TestMatchWithoutVersion(t *testing.T) {
    tests := []struct {  // 定义测试用例结构
        name       string
        compare    cpe.CPE
        candidates []cpe.CPE
        expected   []cpe.CPE
    }  // 结束测试用例结构定义

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            actual := MatchWithoutVersion(test.compare, test.candidates)  // 获取实际结果

            if len(actual) != len(test.expected) {  // 检查实际结果和期望结果的长度是否相等
                for _, e := range actual {  // 遍历实际结果
                    t.Errorf("   unexpected entry: %+v", e.BindToFmtString())  // 输出意外的条目
                }
                t.Fatalf("unexpected number of entries: %d", len(actual))  // 输出意外的条目数量
            }

            for idx, a := range actual {  // 遍历实际结果
                e := test.expected[idx]  // 获取对应的期望结果
                if a.BindToFmtString() != e.BindToFmtString() {  // 检查实际结果和期望结果是否相等
                    dmp := diffmatchpatch.New()  // 创建字符串差异比较对象
                    diffs := dmp.DiffMain(a.BindToFmtString(), e.BindToFmtString(), true)  // 获取字符串差异
                    t.Errorf("mismatched entries @ %d:\n\texpected:%+v\n\t  actual:%+v\n\t    diff:%+v\n", idx, e.BindToFmtString(), a.BindToFmtString(), dmp.DiffPrettyText(diffs))  // 输出不匹配的条目和差异
                }
            }
        })
    }
}
```