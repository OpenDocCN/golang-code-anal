# `grype\grype\version\maven_version_test.go`

```go
package version

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func Test_javaVersion_Compare(t *testing.T) {  // 定义测试函数
    tests := []struct {  // 定义测试用例结构体切片
        name    string  // 测试用例名称
        compare string  // 比较版本号
        want    int     // 期望结果
    }{
        {
            name:    "1",  // 测试用例1
            compare: "2",  // 比较版本号1
            want:    -1,   // 期望结果1
        },
        {
            name:    "1.8.0_282",  // 测试用例2
            compare: "1.8.0_282",  // 比较版本号2
            want:    0,            // 期望结果2
        },
        // ... 其他测试用例
    }
    for _, tt := range tests {  // 遍历测试用例
        t.Run(tt.name, func(t *testing.T) {  // 运行测试用例
            j, err := NewVersion(tt.name, MavenFormat)  // 创建版本对象
            assert.NoError(t, err)  // 断言无错误

            j2, err := NewVersion(tt.compare, MavenFormat)  // 创建比较版本对象
            assert.NoError(t, err)  // 断言无错误

            if got, _ := j2.rich.mavenVer.Compare(j); got != tt.want {  // 比较版本号
                t.Errorf("Compare() = %v, want %v", got, tt.want)  // 输出比较结果
            }
        })
    }
}
```