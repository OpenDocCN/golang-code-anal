# `grype\grype\version\gemfile_version_test.go`

```
package version

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试用例

    "github.com/stretchr/testify/assert"  // 导入第三方测试断言库
)

func makeSemVer(t *testing.T, raw string) *semanticVersion {
    // 创建语义化版本对象，并检查是否有错误
    semVer, err := newSemanticVersion(raw)
    assert.NoError(t, err)  // 使用断言库检查是否有错误
    return semVer  // 返回语义化版本对象
}

func Test_newGemfileVersion(t *testing.T) {

    tests := []struct {
        input string  // 输入字符串
        want  *semanticVersion  // 期望的语义化版本对象
    }{
        {input: "1.13.1", want: makeSemVer(t, "1.13.1")},  // 测试用例1
        {input: "1.13.1-arm-linux", want: makeSemVer(t, "1.13.1")},  // 测试用例2
        {input: "1.13.1-armv6-linux", want: makeSemVer(t, "1.13.1")},  // 测试用例3
        {input: "1.13.1-armv7-linux", want: makeSemVer(t, "1.13.1")},  // 测试用例4
        {input: "1.13.1-java", want: makeSemVer(t, "1.13.1")},  // 测试用例5
        {input: "1.13.1-dalvik", want: makeSemVer(t, "1.13.1")},  // 测试用例6
        {input: "1.13.1-mswin32", want: makeSemVer(t, "1.13.1")},  // 测试用例7
        {input: "1.13.1-x64-mswin64", want: makeSemVer(t, "1.13.1")},  // 测试用例8
        {input: "1.13.1-sparc-unix", want: makeSemVer(t, "1.13.1")},  // 测试用例9
        {input: "1.13.1-powerpc-darwin", want: makeSemVer(t, "1.13.1")},  // 测试用例10
        {input: "1.13.1-x86-linux", want: makeSemVer(t, "1.13.1")},  // 测试用例11
        {input: "1.13.1-x86_64-linux", want: makeSemVer(t, "1.13.1")},  // 测试用例12
        {input: "1.13.1-x86-freebsd", want: makeSemVer(t, "1.13.1")},  // 测试用例13
        {input: "1.13.1-x86-mswin32-80", want: makeSemVer(t, "1.13.1")},  // 测试用例14
        {input: "1.13.1-universal-darwin-8", want: makeSemVer(t, "1.13.1")},  // 测试用例15
        {input: "1.13.1-beta-universal-darwin-8", want: makeSemVer(t, "1.13.1.beta")},  // 测试用例16
        {input: "1.13.1-alpha-1+meta-arm-linux", want: makeSemVer(t, "1.13.1.alpha-1+meta")},  // 测试用例17
        {input: "1.13.1-alpha-1+build.12-arm-linux", want: makeSemVer(t, "1.13.1.alpha-1+build.12")},  // 测试用例18
        {input: "1.2.3----RC-SNAPSHOT.12.9.1--.12+788-armv7-darwin", want: makeSemVer(t, "1.2.3----RC-SNAPSHOT.12.9.1--.12+788")},  // 测试用例19
        {input: "1.2.3----rc-snapshot.12.9.1--.12+788-armv7-darwin", want: makeSemVer(t, "1.2.3----rc-snapshot.12.9.1--.12+788")},  // 测试用例20
    }
    // 遍历测试用例切片，对每个测试用例执行测试
    for _, tt := range tests {
        // 使用测试用例的输入创建新的 Gemfile 版本
        t.Run(tt.input, func(t *testing.T) {
            got, err := newGemfileVersion(tt.input)
            // 检查是否有错误，如果有则输出错误信息并返回
            if !assert.NoError(t, err, fmt.Sprintf("newGemfileVersion(%v)", tt.input)) {
                return
            }
            // 检查新创建的 Gemfile 版本是否与期望值相等
            assert.Equalf(t, tt.want, got, "newGemfileVersion(%v)", tt.input)

            // 检查语义版本是否可以与 Gemfile 版本进行比较
            other, err := NewVersion(tt.want.verObj.String(), SemanticFormat)
            assert.NoError(t, err)

            // 比较两个版本，返回比较结果
            v, err := got.Compare(other)
            assert.NoError(t, err)
            // 如果比较结果为零，则表示 `other` 和 `got` 是相同的版本
            assert.Equal(t, 0, v)
        })
    }
# 闭合前面的函数定义
```