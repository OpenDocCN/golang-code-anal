# `grype\grype\version\golang_version_test.go`

```
package version

import (
    "testing"  // 导入测试包
    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包
    hashiVer "github.com/anchore/go-version"  // 导入版本包
)

func TestNewGolangVersion(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体
        name     string  // 测试用例名称
        input    string  // 输入版本号
        expected golangVersion  // 期望的 golangVersion 结构体
        wantErr  bool  // 是否期望出错
    }{
        {
            name:  "normal semantic version",  // 测试用例名称
            input: "v1.8.0",  // 输入版本号
            expected: golangVersion{  // 期望的 golangVersion 结构体
                raw:    "v1.8.0",  // 原始版本号
                semVer: hashiVer.Must(hashiVer.NewSemver("v1.8.0")),  // 使用版本包创建语义化版本号
            },
        },
        {
            name:  "v0.0.0 date and hash version",  // 测试用例名称
            input: "v0.0.0-20180116102854-5a71ef0e047d",  // 输入版本号
            expected: golangVersion{  // 期望的 golangVersion 结构体
                raw:    "v0.0.0-20180116102854-5a71ef0e047d",  // 原始版本号
                semVer: hashiVer.Must(hashiVer.NewSemver("v0.0.0-20180116102854-5a71ef0e047d")),  // 使用版本包创建语义化版本号
            },
        },
        {
            name:  "semver with +incompatible",  // 测试用例名称
            input: "v24.0.7+incompatible",  // 输入版本号
            expected: golangVersion{  // 期望的 golangVersion 结构体
                raw:    "v24.0.7+incompatible",  // 原始版本号
                semVer: hashiVer.Must(hashiVer.NewSemver("v24.0.7+incompatible")),  // 使用版本包创建语义化版本号
            },
        },
        {
            name:  "standard library",  // 测试用例名称
            input: "go1.21.4",  // 输入版本号
            expected: golangVersion{  // 期望的 golangVersion 结构体
                raw:    "go1.21.4",  // 原始版本号
                semVer: hashiVer.Must(hashiVer.NewSemver("1.21.4")),  // 使用版本包创建语义化版本号
            },
        },
        {
            // "(devel)" is the main module of a go program.
            // If we get a package with this version, it means the SBOM
            // doesn't have a real version number for the built package, so
            // we can't compare it and should just return an error.
            name:    "devel",  // 测试用例名称
            input:   "(devel)",  // 输入版本号
            wantErr: true,  // 期望出错
        },
        {
            name:    "invalid input",  // 测试用例名称
            input:   "some nonsense",  // 输入版本号
            wantErr: true,  // 期望出错
        },
    }
    # 遍历测试用例切片，每个测试用例包括名称和测试函数
    for _, tc := range tests:
        # 使用测试用例的名称创建子测试，传入测试函数
        t.Run(tc.name, func(t *testing.T) {
            # 调用 newGolangVersion 函数，传入测试用例的输入，获取版本号和错误信息
            v, err := newGolangVersion(tc.input)
            # 如果测试用例期望出现错误
            if tc.wantErr:
                # 断言错误信息不为空
                require.Error(t, err)
                # 结束当前测试
                return
            # 断言获取的版本号等于测试用例的期望值
            assert.Equal(t, tc.expected, *v)
        })
    }
# 定义一个测试函数，用于比较 Golang 版本号
func TestCompareGolangVersions(t *testing.T) {
    # 定义测试用例，包括版本名称、当前版本号、其他版本号和期望的比较结果
    tests := []struct {
        name         string
        thisVersion  string
        otherVersion string
        want         int
    # 定义测试用例，比较不同版本号的情况
    {
        # 测试语义版本号比较，当前版本号小于其他版本号
        name:         "semver this version less",
        thisVersion:  "v1.2.3",
        otherVersion: "v1.2.4",
        want:         -1,
    },
    {
        # 测试语义版本号比较，当前版本号大于其他版本号
        name:         "semver this version more",
        thisVersion:  "v1.3.4",
        otherVersion: "v1.2.4",
        want:         1,
    },
    {
        # 测试语义版本号比较，当前版本号等于其他版本号
        name:         "semver equal",
        thisVersion:  "v1.2.4",
        otherVersion: "v1.2.4",
        want:         0,
    },
    {
        # 测试提交哈希比较，当前版本号小于其他版本号
        name:         "commit-sha this version less",
        thisVersion:  "v0.0.0-20180116102854-5a71ef0e047d",
        otherVersion: "v0.0.0-20190116102854-somehash",
        want:         -1,
    },
    {
        # 测试提交哈希比较，当前版本号大于其他版本号
        name:         "commit-sha this version more",
        thisVersion:  "v0.0.0-20180216102854-5a71ef0e047d",
        otherVersion: "v0.0.0-20180116102854-somehash",
        want:         1,
    },
    {
        # 测试提交哈希比较，当前版本号等于其他版本号
        name:         "commit-sha this version equal",
        thisVersion:  "v0.0.0-20180116102854-5a71ef0e047d",
        otherVersion: "v0.0.0-20180116102854-5a71ef0e047d",
        want:         0,
    },
    {
        # 测试预发布语义版本号小于任何语义版本号
        name:         "this pre-semver is less than any semver",
        thisVersion:  "v0.0.0-20180116102854-5a71ef0e047d",
        otherVersion: "v0.0.1",
        want:         -1,
    },
    {
        # 测试语义版本号大于时间戳
        name:         "semver is greater than timestamp",
        thisVersion:  "v2.1.0",
        otherVersion: "v0.0.0-20180116102854-5a71ef0e047d",
        want:         1,
    },
    {
        # 测试带有+incompatible的版本号不影响相等性
        name:         "+incompatible doesn't break equality",
        thisVersion:  "v3.2.0",
        otherVersion: "v3.2.0+incompatible",
        want:         0,
    },
}
    # 遍历测试用例切片，每个测试用例包含名称和测试函数
    for _, tc := range tests:
        # 使用测试用例的名称创建子测试，运行测试函数
        t.Run(tc.name, func(t *testing.T) {
            # 根据测试用例的当前版本创建新的 Golang 版本对象
            a, err := newGolangVersion(tc.thisVersion)
            # 断言错误为空
            require.NoError(t, err)
            # 根据测试用例的其他版本创建新的 Golang 版本对象
            other, err := newGolangVersion(tc.otherVersion)
            # 断言错误为空
            require.NoError(t, err)
            # 调用当前版本对象的比较方法，比较两个版本的大小
            got := a.compare(*other)
            # 断言实际结果和期望结果相等
            assert.Equal(t, tc.want, got)
        })
    }
# 闭合前面的函数定义
```