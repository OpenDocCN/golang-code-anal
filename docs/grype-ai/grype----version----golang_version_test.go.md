# `grype\grype\version\golang_version_test.go`

```
package version

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
	"github.com/stretchr/testify/require"  // 导入断言包
	hashiVer "github.com/anchore/go-version"  // 导入版本包

)

func TestNewGolangVersion(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例
		name     string  // 测试用例名称
		input    string  // 输入
		expected golangVersion  // 期望输出
		wantErr  bool  // 是否期望出错
	}{
		{
			name:  "normal semantic version",  // 测试用例名称
# 输入版本号为 "v1.8.0"，期望得到的 golangVersion 结构体包含原始版本号和语义版本号
input: "v1.8.0",
expected: golangVersion{
    raw:    "v1.8.0",
    semVer: hashiVer.Must(hashiVer.NewSemver("v1.8.0")),
},
# 输入版本号为 "v0.0.0-20180116102854-5a71ef0e047d"，期望得到的 golangVersion 结构体包含原始版本号和语义版本号
{
    name:  "v0.0.0 date and hash version",
    input: "v0.0.0-20180116102854-5a71ef0e047d",
    expected: golangVersion{
        raw:    "v0.0.0-20180116102854-5a71ef0e047d",
        semVer: hashiVer.Must(hashiVer.NewSemver("v0.0.0-20180116102854-5a71ef0e047d")),
    },
},
# 输入版本号为 "v24.0.7+incompatible"，期望得到的 golangVersion 结构体包含原始版本号和语义版本号
{
    name:  "semver with +incompatible",
    input: "v24.0.7+incompatible",
    expected: golangVersion{
        raw:    "v24.0.7+incompatible",
        semVer: hashiVer.Must(hashiVer.NewSemver("v24.0.7+incompatible")),
		},
		},
		{
			// 标准库
			name:  "standard library",
			// 输入版本号
			input: "go1.21.4",
			// 期望的输出
			expected: golangVersion{
				raw:    "go1.21.4",
				semVer: hashiVer.Must(hashiVer.NewSemver("1.21.4")),
			},
		},
		{
			// "(devel)" 是 go 程序的主模块。
			// 如果我们得到一个带有这个版本的包，意味着 SBOM
			// 没有真正的版本号，所以我们无法比较，应该返回错误。
			name:    "devel",
			// 输入版本号
			input:   "(devel)",
			// 期望返回错误
			wantErr: true,
		},
		{
# 定义测试用例
name:    "invalid input",  # 测试用例名称
input:   "some nonsense",   # 输入值
wantErr: true,              # 期望出现错误

# 循环遍历测试用例
for _, tc := range tests:
    t.Run(tc.name, func(t *testing.T):
        # 调用函数，获取返回值和错误信息
        v, err := newGolangVersion(tc.input)
        # 如果期望出现错误，则断言应该有错误发生
        if tc.wantErr:
            require.Error(t, err)
            return
        # 否则，断言返回值应该等于期望值
        assert.Equal(t, tc.expected, *v)
    )
)

# 定义版本比较的测试函数
func TestCompareGolangVersions(t *testing.T):
    tests := []struct {
		# 定义结构体字段，包括名称、当前版本、其他版本和期望结果
		name         string
		thisVersion  string
		otherVersion string
		want         int
	}{
		# 第一个测试用例：当前版本小于其他版本
		{
			name:         "semver this version less",
			thisVersion:  "v1.2.3",
			otherVersion: "v1.2.4",
			want:         -1,
		},
		# 第二个测试用例：当前版本大于其他版本
		{
			name:         "semver this version more",
			thisVersion:  "v1.3.4",
			otherVersion: "v1.2.4",
			want:         1,
		},
		# 第三个测试用例：当前版本等于其他版本
		{
			name:         "semver equal",
			thisVersion:  "v1.2.4",
# 定义测试用例，比较两个版本号的大小
{
    name:         "semantic version this version less",
    thisVersion:  "v1.2.3",
    otherVersion: "v1.2.4",
    want:         -1,
},
{
    name:         "semantic version this version equal",
    thisVersion:  "v1.2.4",
    otherVersion: "v1.2.4",
    want:         0,
},
{
    name:         "commit-sha this version less",
    thisVersion:  "v0.0.0-20180116102854-5a71ef0e047d",
    otherVersion: "v0.0.0-20190116102854-somehash",
    want:         -1,
},
{
    name:         "commit-sha this version more",
    thisVersion:  "v0.0.0-20180216102854-5a71ef0e047d",
    otherVersion: "v0.0.0-20180116102854-somehash",
    want:         1,
},
{
    name:         "commit-sha this version equal",
    thisVersion:  "v0.0.0-20180116102854-5a71ef0e047d",
    otherVersion: "v0.0.0-20180116102854-5a71ef0e047d",
    want:         0,
}
		},
		{
			// 定义测试用例：此预先语义版本号小于任何语义版本号
			name:         "this pre-semver is less than any semver",
			// 当前版本号
			thisVersion:  "v0.0.0-20180116102854-5a71ef0e047d",
			// 其他版本号
			otherVersion: "v0.0.1",
			// 期望结果为-1
			want:         -1,
		},
		{
			// 定义测试用例：语义版本号大于时间戳
			name:         "semver is greater than timestamp",
			// 当前版本号
			thisVersion:  "v2.1.0",
			// 其他版本号
			otherVersion: "v0.0.0-20180116102854-5a71ef0e047d",
			// 期望结果为1
			want:         1,
		},
		{
			// 定义测试用例：+incompatible不影响相等性
			name:         "+incompatible doesn't break equality",
			// 当前版本号
			thisVersion:  "v3.2.0",
			// 其他版本号
			otherVersion: "v3.2.0+incompatible",
			// 期望结果为0
			want:         0,
		},
	}
# 遍历测试用例切片，对每个测试用例执行测试
for _, tc := range tests:
    # 使用测试用例的名称创建子测试，执行测试
    t.Run(tc.name, func(t *testing.T):
        # 根据测试用例的当前版本创建新的 Golang 版本对象
        a, err := newGolangVersion(tc.thisVersion)
        # 断言错误为空
        require.NoError(t, err)
        # 根据测试用例的其他版本创建新的 Golang 版本对象
        other, err := newGolangVersion(tc.otherVersion)
        # 断言错误为空
        require.NoError(t, err)
        # 比较两个 Golang 版本对象的版本号
        got := a.compare(*other)
        # 断言比较结果与期望值相等
        assert.Equal(t, tc.want, got)
    )
```