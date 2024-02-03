# `grype\grype\pkg\upstream_package_test.go`

```go
package pkg

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/anchore/syft/syft/cpe"  // 导入CPE包
)

func TestUpstreamPackages(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        pkg      Package  // 包对象
        expected []Package  // 期望的包对象切片
    }  // 结构体切片结束
    for _, tt := range tests {  // 遍历测试用例
        t.Run(tt.name, func(t *testing.T) {  // 运行测试用例
            var actual []Package  // 定义实际结果的包对象切片
            for _, upstream := range UpstreamPackages(tt.pkg) {  // 遍历上游包
                actual = append(actual, upstream)  // 将上游包添加到实际结果中
            }  // 遍历结束
            assert.Equalf(t, tt.expected, actual, "UpstreamPackages(%v)", tt.pkg)  // 使用断言比较期望结果和实际结果
        })  // 运行测试用例结束
    }  // 遍历测试用例结束
}  // 测试函数结束
```