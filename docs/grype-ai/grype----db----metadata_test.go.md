# `grype\grype\db\metadata_test.go`

```
package db

import (
	"testing"  // 导入测试包
	"time"  // 导入时间包

	"github.com/go-test/deep"  // 导入深度比较包
	"github.com/spf13/afero"  // 导入文件系统操作包
)

func TestMetadataParse(t *testing.T) {  // 定义测试函数
	tests := []struct {  // 定义测试用例结构
		fixture  string  // 测试用例的输入文件路径
		expected *Metadata  // 期望的输出元数据
		err      bool  // 是否期望出现错误
	}{
		{
			fixture: "test-fixtures/metadata-gocase",  // 输入文件路径
			expected: &Metadata{  // 期望的输出元数据
				Built:    time.Date(2020, 06, 15, 14, 02, 36, 0, time.UTC),  // 构建时间
		// 定义测试用例数组
		tests := []struct {
			// 测试用例的输入数据
			fixture  string
			// 期望的输出数据
			expected *Metadata
			// 是否期望出现错误
			err      bool
		}{
			// 第一个测试用例
			{
				// 输入数据为测试文件路径
				fixture: "test-fixtures/metadata-est-timezone",
				// 期望的输出数据为具体的元数据信息
				expected: &Metadata{
					// 构建时间
					Built:    time.Date(2020, 06, 15, 18, 02, 36, 0, time.UTC),
					// 版本号
					Version:  2,
					// 校验和
					Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",
				},
			},
			// 第二个测试用例
			{
				// 输入数据为测试文件路径
				fixture: "test-fixtures/metadata-edt-timezone",
				// 期望的输出数据为具体的元数据信息
				expected: &Metadata{
					// 构建时间
					Built:    time.Date(2020, 06, 15, 18, 02, 36, 0, time.UTC),
					// 版本号
					Version:  2,
					// 校验和
					Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",
				},
			},
			// 第三个测试用例
			{
				// 输入数据为一个无效的路径
				fixture: "/dev/null/impossible",
				// 期望出现错误
				err:     true,
			},
		}

		// 遍历测试用例数组
		for _, test := range tests {
			// 对每个测试用例进行单独的测试
			t.Run(test.fixture, func(t *testing.T) {
// 从指定目录创建元数据，如果出现错误且不符合预期的错误，则输出错误信息
metadata, err := NewMetadataFromDir(afero.NewOsFs(), test.fixture)
if err != nil && !test.err {
    t.Fatalf("failed to get metadata: %+v", err)
} 
// 如果没有错误且预期有错误，则输出错误信息
else if err == nil && test.err {
    t.Fatalf("expected error but got none")
} 
// 如果元数据为空且预期不为空，则输出错误信息
else if metadata == nil && test.expected != nil {
    t.Fatalf("metadata not found: %+v", test.fixture)
}

// 如果元数据不为空且预期不为空，则比较元数据和预期值的差异
if metadata != nil && test.expected != nil {
    for _, diff := range deep.Equal(*metadata, *test.expected) {
        t.Errorf("metadata difference: %s", diff)
    }
}
		name                string  // 定义一个字符串类型的变量name
		current             *Metadata  // 定义一个指向Metadata类型的指针变量current
		update              *ListingEntry  // 定义一个指向ListingEntry类型的指针变量update
		expectedToSupercede bool  // 定义一个布尔类型的变量expectedToSupercede
	}{
		{
			name:                "prefer updated versions over later dates",  // 给name赋值为"prefer updated versions over later dates"
			expectedToSupercede: true,  // 给expectedToSupercede赋值为true
			current: &Metadata{  // 给current赋值为指向Metadata类型的指针，指向的Metadata对象包含Built和Version字段
				Built:   time.Date(2020, 06, 15, 14, 02, 36, 0, time.UTC),  // 设置Built字段的值为指定的时间
				Version: 2,  // 设置Version字段的值为2
			},
			update: &ListingEntry{  // 给update赋值为指向ListingEntry类型的指针，指向的ListingEntry对象包含Built和Version字段
				Built:   time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),  // 设置Built字段的值为指定的时间
				Version: 3,  // 设置Version字段的值为3
			},
		},
		{
			name:                "prefer later dates when version is the same",  // 给name赋值为"prefer later dates when version is the same"
			expectedToSupercede: false,  // 给expectedToSupercede赋值为false
# 创建一个名为 current 的 Metadata 结构体，包含构建时间和版本信息
current: &Metadata{
    Built:   time.Date(2020, 06, 15, 14, 02, 36, 0, time.UTC),
    Version: 1,
},
# 创建一个名为 update 的 ListingEntry 结构体，包含构建时间和版本信息
update: &ListingEntry{
    Built:   time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),
    Version: 1,
},
# 创建一个名为 name 的字符串变量，赋值为 "prefer something over nothing"
name: "prefer something over nothing",
# 创建一个名为 expectedToSupercede 的布尔变量，赋值为 true
expectedToSupercede: true,
# 创建一个名为 current 的空指针
current: nil,
# 创建一个名为 update 的 ListingEntry 结构体，包含构建时间和版本信息
update: &ListingEntry{
    Built:   time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),
    Version: 1,
},
# 遍历测试用例列表，对每个测试用例执行子测试
for _, test := range tests:
    # 使用测试用例的名称创建子测试
    t.Run(test.name, func(t *testing.T):
        # 调用被测试的函数，获取实际结果
        actual := test.current.IsSupersededBy(test.update)
        
        # 检查实际结果是否符合预期结果，如果不符合则输出错误信息
        if test.expectedToSupercede != actual:
            t.Errorf("failed supercede assertion: got %+v", actual)
    )
```