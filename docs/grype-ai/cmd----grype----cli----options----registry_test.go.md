# `grype\cmd\grype\cli\options\registry_test.go`

```
package options

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/stretchr/testify/assert"  // 导入 testify 包的 assert 模块，用于断言测试结果

    "github.com/anchore/stereoscope/pkg/image"  // 导入 image 包，用于处理镜像
)

func TestHasNonEmptyCredentials(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体
        username, password, token, cert, key string  // 定义测试用例的输入参数
        expected                             bool  // 定义测试用例的期望输出
    }{

        {
            "", "", "", "", "",  // 测试用例1
            false,  // 期望输出为 false
        },
        {
            "user", "", "", "", "",  // 测试用例2
            false,  // 期望输出为 false
        },
        {
            "", "pass", "", "", "",  // 测试用例3
            false,  // 期望输出为 false
        },
        {
            "", "pass", "tok", "", "",  // 测试用例4
            true,  // 期望输出为 true
        },
        {
            "user", "", "tok", "", "",  // 测试用例5
            true,  // 期望输出为 true
        },
        {
            "", "", "tok", "", "",  // 测试用例6
            true,  // 期望输出为 true
        },
        {
            "user", "pass", "tok", "", "",  // 测试用例7
            true,  // 期望输出为 true
        },

        {
            "user", "pass", "", "", "",  // 测试用例8
            true,  // 期望输出为 true
        },
        {
            "", "", "", "cert", "key",  // 测试用例9
            true,  // 期望输出为 true
        },
        {
            "", "", "", "cert", "",  // 测试用例10
            false,  // 期望输出为 false
        },
        {
            "", "", "", "", "key",  // 测试用例11
            false,  // 期望输出为 false
        },
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(fmt.Sprintf("%+v", test), func(t *testing.T) {  // 运行测试用例
            assert.Equal(t, test.expected, hasNonEmptyCredentials(test.username, test.password, test.token, test.cert, test.key))  // 使用断言判断测试结果是否符合期望
        })
    }
}

func Test_registry_ToOptions(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体
        name     string  // 定义测试用例的名称
        input    registry  // 定义测试用例的输入参数
        expected image.RegistryOptions  // 定义测试用例的期望输出
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.name, func(t *testing.T) {  // 运行测试用例
            assert.Equal(t, &test.expected, test.input.ToOptions())  // 使用断言判断测试结果是否符合期望
        })
    }
}
```