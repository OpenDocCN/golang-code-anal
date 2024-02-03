# `kubo\test\cli\testutils\asserts.go`

```go
# 导入 testutils 包和所需的其他包
package testutils

# 导入 strings 和 testing 包

# 定义一个名为 AssertStringContainsOneOf 的函数，接受一个测试对象 t，一个字符串 str 和一个或多个字符串 ss
func AssertStringContainsOneOf(t *testing.T, str string, ss ...string) {
    # 遍历 ss 中的每个字符串 s
    for _, s := range ss {
        # 如果字符串 str 包含 s，则返回
        if strings.Contains(str, s) {
            return
        }
    }
    # 如果字符串 str 不包含 ss 中的任何一个字符串，则输出错误信息
    t.Errorf("%q does not contain one of %v", str, ss)
}
```