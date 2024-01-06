# `grype\grype\db\listing_test.go`

```
package db

import (
	"net/url"  // 导入网络 URL 相关的包
	"testing"  // 导入测试相关的包
	"time"     // 导入时间相关的包

	"github.com/go-test/deep"  // 导入深度比较包
	"github.com/spf13/afero"   // 导入文件系统操作包
)

func mustUrl(u *url.URL, err error) *url.URL {
	// 如果出现错误，立即终止程序并抛出错误
	if err != nil {
		panic(err)
	}
	// 返回 URL 对象
	return u
}

func TestNewListingFromPath(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		fixture  string  // 定义变量fixture为字符串类型
		expected Listing  // 定义变量expected为Listing类型
		err      bool     // 定义变量err为布尔类型
	}{
		{  // 第一个测试用例
			fixture: "test-fixtures/listing.json",  // 设置fixture变量的值为"test-fixtures/listing.json"
			expected: Listing{  // 设置expected变量的值为Listing结构体
				Available: map[int][]ListingEntry{  // 设置Available字段为整数到ListingEntry切片的映射
					1: {  // 键为1的映射值为ListingEntry结构体切片
						{  // 第一个ListingEntry结构体
							Built:    time.Date(2020, 06, 12, 16, 12, 12, 0, time.UTC),  // 设置Built字段的时间值
							URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db-v0.2.0+2020-6-12.tar.gz")),  // 设置URL字段的值为解析后的URL
							Version:  1,  // 设置Version字段的值为1
							Checksum: "sha256:e20c251202948df7f853ddc812f64826bdcd6a285c839a7c65939e68609dfc6e",  // 设置Checksum字段的值
						},
					},
					2: {  // 键为2的映射值为ListingEntry结构体切片
						{  // 第一个ListingEntry结构体
							Built:    time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),  // 设置Built字段的时间值
							URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db-v1.1.0+2020-6-13.tar.gz")),  // 设置URL字段的值为解析后的URL
// 定义一个 fixture 变量，值为 "test-fixtures/listing-sorted.json"
fixture: "test-fixtures/listing-sorted.json",
// 定义一个 expected 变量，值为 Listing 结构体
expected: Listing{
    // 定义 Available 字段，值为 map[int][]ListingEntry 类型
    Available: map[int][]ListingEntry{
        // 定义 map 的 key 为 1，值为一个 ListingEntry 结构体的切片
        1: {
            // 定义 ListingEntry 结构体的字段值
            Built:    time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),
            URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db_v1_2020-6-13.tar.gz")),
            Version:  1,
            Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",
        },
        {
            // 定义 ListingEntry 结构体的字段值
            Built:    time.Date(2020, 06, 12, 16, 12, 12, 0, time.UTC),
# 创建一个 URL 对象，解析给定的 URL 字符串
URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db_v1_2020-6-12.tar.gz")),
# 设置版本号为 1
Version:  1,
# 设置校验和为指定的 SHA256 值
Checksum: "sha256:e20c251202948df7f853ddc812f64826bdcd6a285c839a7c65939e68609dfc6e",
# 创建一个包含 fixture 和 expected 字段的结构体
fixture: "test-fixtures/listing-unsorted.json",
expected: Listing{
    # 创建一个包含 Available 字段的 map 结构
    Available: map[int][]ListingEntry{
        1: {
            # 创建一个 ListingEntry 结构体
            Built:    time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),
            # 创建一个 URL 对象，解析给定的 URL 字符串
            URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db_v1_2020-6-13.tar.gz")),
            # 设置版本号为 1
            Version:  1,
            # 设置校验和为指定的 SHA256 值
            Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",
        },
        {
# 创建一个时间对象，表示构建时间为2020年6月12日16时12分12秒，时区为UTC
Built:    time.Date(2020, 06, 12, 16, 12, 12, 0, time.UTC),
# 解析URL字符串，将其转换为URL对象
URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db_v1_2020-6-12.tar.gz")),
# 设置版本号为1
Version:  1,
# 设置校验和为SHA256算法生成的校验和值
Checksum: "sha256:e20c251202948df7f853ddc812f64826bdcd6a285c839a7c65939e68609dfc6e",
# 遍历测试用例
for _, test := range tests {
	# 使用测试用例的fixture创建一个新的Listing对象
	listing, err := NewListingFromFile(afero.NewOsFs(), test.fixture)
	# 如果出现错误且测试用例中不期望有错误发生，则输出错误信息
	if err != nil && !test.err {
		t.Fatalf("failed to get metadata: %+v", err)
	# 如果没有出现错误但测试用例中期望有错误发生，则输出错误信息
	} else if err == nil && test.err {
		t.Fatalf("expected errer but got none")
	}
// 对列表进行深度比较，如果有差异则输出错误信息
for _, diff := range deep.Equal(listing, test.expected) {
    t.Errorf("listing difference: %s", diff)
}
// 结束当前测试用例
})
// 结束测试函数
}
// 测试列表最佳更新情况
func TestListingBestUpdate(t *testing.T) {
    // 测试数据
    tests := []struct {
        fixture    string
        constraint int
        expected   *ListingEntry
    }{
        {
            fixture:    "test-fixtures/listing.json",
            constraint: 2,
            expected: &ListingEntry{
                Built:    time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),
                URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db-v1.1.0+2020-6-13.tar.gz")),
                Version:  2,
# 定义测试用例数组
tests := []struct {
    fixture:    string,  # 测试数据文件路径
    constraint: int,     # 约束条件
    expected: &ListingEntry{  # 期望的结果
        Built:    time.Date(2020, 06, 12, 16, 12, 12, 0, time.UTC),  # 构建时间
        URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db-v0.2.0+2020-6-12.tar.gz")),  # URL
        Version:  1,  # 版本号
        Checksum: "sha256:e20c251202948df7f853ddc812f64826bdcd6a285c839a7c65939e68609dfc6e",  # 校验和
    },
}

# 遍历测试用例数组
for _, test := range tests {
    # 对每个测试用例执行测试
    t.Run(test.fixture, func(t *testing.T) {
        # 从文件中读取列表数据
        listing, err := NewListingFromFile(afero.NewOsFs(), test.fixture)
        # 如果出现错误，打印错误信息并终止测试
        if err != nil {
            t.Fatalf("failed to get metadata: %+v", err)
			}

			// 调用 BestUpdate 方法获取最佳更新的候选项
			actual := listing.BestUpdate(test.constraint)
			// 检查实际结果和预期结果是否匹配，如果不匹配则输出错误信息
			if actual == nil && test.expected != nil || actual != nil && test.expected == nil {
				t.Fatalf("mismatched best candidate expectations")
			}

			// 遍历比较实际结果和预期结果的差异，输出差异信息
			for _, diff := range deep.Equal(actual, test.expected) {
				t.Errorf("listing entry difference: %s", diff)
			}
		})
	}
}
```