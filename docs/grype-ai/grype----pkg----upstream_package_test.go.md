# `grype\grype\pkg\upstream_package_test.go`

```
package pkg

import (
	"testing"  // 导入测试包
	"github.com/stretchr/testify/assert"  // 导入断言包
	"github.com/anchore/syft/syft/cpe"  // 导入CPE包
)

func TestUpstreamPackages(t *testing.T) {
	tests := []struct {  // 定义测试用例结构
		name     string  // 测试用例名称
		pkg      Package  // 包对象
		expected []Package  // 期望结果
	}{
		{
			name: "no upstreams results in empty list",  // 测试用例名称
			pkg: Package{  // 初始化包对象
				Name:    "name",  // 包名称
```

# 定义一个结构体字段为 Version，值为 "version"
Version: "version",
},
# 期望的结果为 nil
expected: nil,
},
{
# 定义一个结构体字段为 Name，值为 "name"
name: "with upstream name",
# 定义一个结构体字段为 Package
pkg: Package{
# 定义一个结构体字段为 Name，值为 "name"
Name: "name",
# 定义一个结构体字段为 Version，值为 "version"
Version: "version",
# 定义一个结构体字段为 CPEs，值为一个 CPE 切片
CPEs: []cpe.CPE{
# 定义一个 CPE，值为 "cpe:2.3:*:name:name:version:*:*:*:*:*:*:*"
cpe.Must("cpe:2.3:*:name:name:version:*:*:*:*:*:*:*"),
},
# 定义一个结构体字段为 Upstreams，值为一个 UpstreamPackage 切片
Upstreams: []UpstreamPackage{
{
# 定义一个结构体字段为 Name，值为 "new-name"
Name: "new-name",
},
},
},
# 期望的结果为一个 Package 切片
expected: []Package{
{
					Name:    "new-name", // 新的包名
					Version: "version",  // 原始版本
					CPEs: []cpe.CPE{
						// 替换了名称和供应商
						cpe.Must("cpe:2.3:*:new-name:new-name:version:*:*:*:*:*:*:*"),
					},
					// 没有上游包
				},
			},
		},
		{
			name: "with upstream name and version", // 带有上游包的名称和版本
			pkg: Package{
				Name:    "name", // 包名
				Version: "version", // 版本
				CPEs: []cpe.CPE{
					cpe.Must("cpe:2.3:*:name:name:version:*:*:*:*:*:*:*"), // CPE对象
				},
				Upstreams: []UpstreamPackage{ // 上游包列表
					{
// 定义一个新的包的名称和版本
Name:    "new-name",
Version: "new-version",
// 定义一个新的包的CPEs
CPEs: []cpe.CPE{
    // 替换名称、供应商和版本
    cpe.Must("cpe:2.3:*:new-name:new-name:new-version:*:*:*:*:*:*:*"),
},
// 没有上游依赖
// 定义一个结构体，包含名称、版本和CPEs字段
Name:    "name",
Version: "version",
CPEs: []cpe.CPE{
    // 创建一个CPE对象，表示特定的软件包
    cpe.Must("cpe:2.3:*:name:name:version:*:*:*:*:*:*:*"),
},
Upstreams: []UpstreamPackage{
    {
        // 注意：没有名称是无效的
        Version: "new-version",
    },
},
// 期望的结果为nil
expected: nil,
// 遍历测试用例
for _, tt := range tests {
    // 对每个测试用例运行子测试
    t.Run(tt.name, func(t *testing.T) {
        // 定义一个空的Package切片
        var actual []Package
        // 遍历UpstreamPackages函数返回的结果
        for _, upstream := range UpstreamPackages(tt.pkg) {
            // 将结果追加到actual切片中
            actual = append(actual, upstream)
		}
		// 使用断言函数Equalf比较实际值和期望值，如果不相等则输出自定义的错误信息
		assert.Equalf(t, tt.expected, actual, "UpstreamPackages(%v)", tt.pkg)
	})
}
// 结束for循环
```