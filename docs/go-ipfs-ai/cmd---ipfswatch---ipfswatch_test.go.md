# `kubo\cmd\ipfswatch\ipfswatch_test.go`

```
//go:build !plan9
// +build !plan9

// 声明当前文件所属的包为 main 包
package main

// 导入 testing 包，用于编写测试函数
import (
    "testing"

    // 导入第三方断言库 assert
    "github.com/ipfs/kubo/thirdparty/assert"
)

// 定义测试函数 TestIsHidden
func TestIsHidden(t *testing.T) {
    // 调用 assert 包的 True 函数，判断是否隐藏文件
    assert.True(IsHidden("bar/.git"), t, "dirs beginning with . should be recognized as hidden")
    // 调用 assert 包的 False 函数，判断是否隐藏文件
    assert.False(IsHidden("."), t, ". for current dir should not be considered hidden")
    // 调用 assert 包的 False 函数，判断是否隐藏文件
    assert.False(IsHidden("bar/baz"), t, "normal dirs should not be hidden")
}
```