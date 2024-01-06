# `grype\grype\internal\packagemetadata\names_test.go`

```
package packagemetadata

import (
	"reflect"  // 导入 reflect 包，用于获取类型信息
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/google/go-cmp/cmp"  // 导入 google/go-cmp/cmp 包，用于比较数据结构
	"github.com/stretchr/testify/assert"  // 导入 testify/assert 包，用于编写测试断言
	"github.com/stretchr/testify/require"  // 导入 testify/require 包，用于编写测试断言

	"github.com/anchore/grype/grype/pkg"  // 导入自定义包
)

func TestAllNames(t *testing.T) {
	// 注意：这是一种相对于当前代码库的完成测试形式。

	// 获取所有类型名称的预期结果，并检查是否有错误
	expected, err := DiscoverTypeNames()
	require.NoError(t, err)  // 确保没有错误发生

	// 获取实际的所有类型名称
	actual := AllTypeNames()
// 确保代码库（从ast分析）反映了最新生成的代码状态
if !assert.ElementsMatch(t, expected, actual) {
    t.Errorf("metadata types not fully represented: \n%s", cmp.Diff(expected, actual))
    t.Log("did you add a new pkg.*Metadata type without updating the JSON schema?")
    t.Log("if so, you need to update the schema version and regenerate the JSON schema (make generate-json-schema)")
}

// 遍历所有类型，确保每个类型都有一个JSON名称
for _, ty := range AllTypes() {
    assert.NotEmpty(t, JSONName(ty), "metadata type %q does not have a JSON name", ty)
}
}

func TestReflectTypeFromJSONName(t *testing.T) {

tests := []struct {
    name       string
    lookup     string
    wantRecord reflect.Type
}{
# 创建一个包含不同类型的元数据的列表
{
    # 元数据名称为 "GolangBinMetadata lookup"
    name:       "GolangBinMetadata lookup",
    # 查找 GolangBinMetadata 类型的元数据
    lookup:     "GolangBinMetadata",
    # 期望的记录类型为 pkg.GolangBinMetadata 类型
    wantRecord: reflect.TypeOf(pkg.GolangBinMetadata{}),
},
{
    # 元数据名称为 "GolangModMetadata lookup"
    name:       "GolangModMetadata lookup",
    # 查找 GolangModMetadata 类型的元数据
    lookup:     "GolangModMetadata",
    # 期望的记录类型为 pkg.GolangModMetadata 类型
    wantRecord: reflect.TypeOf(pkg.GolangModMetadata{}),
},
{
    # 元数据名称为 "JavaMetadata lookup"
    name:       "JavaMetadata lookup",
    # 查找 JavaMetadata 类型的元数据
    lookup:     "JavaMetadata",
    # 期望的记录类型为 pkg.JavaMetadata 类型
    wantRecord: reflect.TypeOf(pkg.JavaMetadata{}),
},
{
    # 元数据名称为 "RpmMetadata lookup"
    name:       "RpmMetadata lookup",
    # 查找 RpmMetadata 类型的元数据
    lookup:     "RpmMetadata",
    # 期望的记录类型为 pkg.RpmMetadata 类型
    wantRecord: reflect.TypeOf(pkg.RpmMetadata{}),
},
	}
	// 遍历测试用例列表
	for _, tt := range tests {
		// 使用测试名称创建子测试
		t.Run(tt.name, func(t *testing.T) {
			// 调用 ReflectTypeFromJSONName 函数，获取返回值
			got := ReflectTypeFromJSONName(tt.lookup)
			// 断言返回值与期望值相等
			assert.Equal(t, tt.wantRecord.Name(), got.Name())
		})
	}
}
```