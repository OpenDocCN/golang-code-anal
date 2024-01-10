# `grype\grype\pkg\purl_provider_test.go`

```
package pkg

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func Test_PurlProvider_Fails(t *testing.T) {
    //GIVEN
    tests := []struct {  // 定义测试用例结构体切片
        name      string  // 测试用例名称
        userInput string  // 用户输入
    }{
        {"fails on path with nonexistant file", "purl:tttt/empty.txt"},  // 测试用例1
        {"fails on invalid path", "purl:~&&"},  // 测试用例2
        {"fails on empty purl file", "purl:test-fixtures/empty.json"},  // 测试用例3
        {"fails on invalid purl in file", "purl:test-fixtures/invalid-purl.txt"},  // 测试用例4
        {"fails on invalid cpe in file", "purl:test-fixtures/invalid-cpe.txt"},  // 测试用例5
        {"fails on invalid user input", "dir:test-fixtures/empty.json"},  // 测试用例6
    }

    for _, tc := range tests {  // 遍历测试用例
        t.Run(tc.name, func(t *testing.T) {  // 运行测试用例
            //WHEN
            packages, err := purlProvider(tc.userInput)  // 调用被测函数

            //THEN
            assert.Nil(t, packages)  // 断言结果为空
            assert.Error(t, err)  // 断言出现错误
            assert.NotEqual(t, "", err.Error())  // 断言错误信息不为空
        })
    }
}

func Test_CsvProvide(t *testing.T) {
    //GIVEN
    expected := []string{"curl", "ant", "log4j-core"}  // 期望结果

    //WHEN
    packages, err := purlProvider("purl:test-fixtures/valid-purl.txt")  // 调用被测函数

    //THEN
    packageNames := []string{}  // 存储包名的切片
    for _, pkg := range packages {  // 遍历包列表
        assert.NotEmpty(t, pkg.ID)  // 断言包的ID不为空
        packageNames = append(packageNames, pkg.Name)  // 将包名添加到切片中
    }
    assert.NoError(t, err)  // 断言没有错误发生
    assert.Equal(t, expected, packageNames)  // 断言实际结果等于期望结果
}
```