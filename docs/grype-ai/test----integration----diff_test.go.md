# `grype\test\integration\diff_test.go`

```
package integration

import (
	"flag"
)

// 定义一个命令行参数，用于更新 *.golden 文件以进行差异对比
var update = flag.Bool("update", false, "update the *.golden files for diff presenter")

// 定义两个常量，分别表示基准 URL 和目标 URL
const (
	baseURL   = "https://toolbox-data.anchore.io/grype/staging-databases/vulnerability-db_v5_2022-10-14T08:22:01Z_69c99aa5917dea969f2d.tar.gz"
	targetURL = "https://toolbox-data.anchore.io/grype/staging-databases/vulnerability-db_v5_2022-10-17T08:14:57Z_10e4086061ab36cfa900.tar.gz"
)

// TODO: 重新设计这个测试，不要依赖于托管的数据库。暂时禁用以解决在升级模式时出现的失败

//func TestDatabaseDiff(t *testing.T) {
//	//GIVEN
//	differ, err := differ.NewDiffer(db.Config{
//		DBRootDir:           "test-fixtures/grype-db",
//		ListingURL:          getListingURL(),
//		ValidateByHashOnGet: false,  // 设置 ValidateByHashOnGet 属性为 false
//	})  // 结束对结构体的初始化
//	var buffer bytes.Buffer  // 声明一个 bytes.Buffer 类型的变量 buffer
//	base, err := url.Parse(baseURL)  // 解析 baseURL 字符串为 URL 对象
//	require.NoError(t, err)  // 断言 err 为 nil
//	target, err := url.Parse(targetURL)  // 解析 targetURL 字符串为 URL 对象
//	require.NoError(t, err)  // 断言 err 为 nil
//
//	//WHEN
//	require.NoError(t, differ.DownloadDatabases(base, target))  // 断言 differ.DownloadDatabases(base, target) 操作没有错误
//	diffs, err := differ.DiffDatabases()  // 调用 differ.DiffDatabases() 方法，返回 diffs 和 err
//	require.NoError(t, err)  // 断言 err 为 nil
//	for i := range *diffs {  // 遍历 diffs 切片
//		sort.Strings((*diffs)[i].Packages)  // 对 diffs[i].Packages 切片进行排序
//	}
//	sort.SliceStable(*diffs, func(i, j int) bool {  // 对 diffs 切片进行稳定排序
//		d1, d2 := (*diffs)[i], (*diffs)[j]  // 获取 diffs[i] 和 diffs[j] 的值
//		return (d1.ID + d1.Namespace) < (d2.ID + d2.Namespace)  // 比较 d1.ID + d1.Namespace 和 d2.ID + d2.Namespace 的大小
//	})
//	require.NoError(t, differ.Present("json", diffs, &buffer))  // 断言 differ.Present("json", diffs, &buffer) 操作没有错误
// 获取字节流的内容
actual := buffer.Bytes()
// 如果需要更新测试数据，则更新测试数据
if *update {
    testutils.UpdateGoldenFileContents(t, actual)
}
// 获取预期的测试数据
var expected = testutils.GetGoldenFileContents(t)
// 如果预期数据和实际数据不相等，则输出差异
if !bytes.Equal(expected, actual) {
    // 创建 diffmatchpatch 对象
    dmp := diffmatchpatch.New()
    // 获取预期数据和实际数据的差异
    diffs := dmp.DiffMain(string(expected), string(actual), true)
    // 输出差异信息
    t.Errorf("mismatched output:\n%s", dmp.DiffPrettyText(diffs))
}
```