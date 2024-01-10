# `grype\grype\pkg\qualifier\rpmmodularity\qualifier_test.go`

```
package rpmmodularity

import (
    "testing"  // 导入测试包

    "github.com/stretchr/testify/assert"  // 导入断言包

    "github.com/anchore/grype/grype/pkg"  // 导入包
    "github.com/anchore/grype/grype/pkg/qualifier"  // 导入包
)

func TestRpmModularity_Satisfied(t *testing.T) {
    tests := []struct {  // 定义测试结构体
        name          string  // 测试名称
        rpmModularity qualifier.Qualifier  // RPM 模块化限定符
        pkg           pkg.Package  // 包
        satisfied     bool  // 是否满足条件
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试
            s, err := test.rpmModularity.Satisfied(nil, test.pkg)  // 判断是否满足条件
            assert.NoError(t, err)  // 断言是否没有错误
            assert.Equal(t, test.satisfied, s)  // 断言是否相等
        })
    }
}
```