# `grype\grype\differ\differ_test.go`

```
package differ
// 导入所需的包

import (
	"bytes"
	"flag"
	"strconv"
	"testing"

	"github.com/sergi/go-diff/diffmatchpatch"
	"github.com/stretchr/testify/require"

	"github.com/anchore/go-testutils"
	"github.com/anchore/grype/grype/db"
	v5 "github.com/anchore/grype/grype/db/v5"
	"github.com/anchore/grype/grype/vulnerability."
)
// 导入所需的包

var update = flag.Bool("update", false, "update the *.golden files for diff presenter")
// 定义一个命令行标志，用于更新差异演示的 *.golden 文件

func TestNewDiffer(t *testing.T) {
// 定义一个测试函数
	// 创建一个空的数据库配置对象
	config := db.Config{}

	// 创建一个新的Differ对象，使用给定的配置
	differ, err := NewDiffer(config)

	// 确保没有错误发生，并且differ.baseCurator不为空
	require.NoError(t, err)
	require.NotNil(t, differ.baseCurator)
}

func Test_DifferDirectory(t *testing.T) {
	// 创建一个新的Differ对象，使用给定的数据库配置
	d, err := NewDiffer(db.Config{
		DBRootDir: "root-dir",
	})
	require.NoError(t, err)

	// 设置基础数据库目录
	err = d.SetBaseDB("test-fixtures/dbs/base")
	require.NoError(t, err)
```

以上代码中，首先创建了一个空的数据库配置对象，然后使用该配置对象创建了一个新的Differ对象。在测试函数中，又创建了一个新的Differ对象，并设置了基础数据库目录。在每个步骤中，都使用require.NoError来确保没有错误发生。
// 获取基准数据库的状态
baseStatus := d.baseCurator.Status()
// 断言基准数据库的位置与预期位置相等
require.Equal(t, "test-fixtures/dbs/base/"+strconv.Itoa(vulnerability.SchemaVersion), baseStatus.Location)

// 设置目标数据库
err = d.SetTargetDB("test-fixtures/dbs/target")
require.NoError(t, err)

// 获取目标数据库的状态
targetStatus := d.targetCurator.Status()
// 断言目标数据库的位置与预期位置相等
require.Equal(t, "test-fixtures/dbs/target/"+strconv.Itoa(vulnerability.SchemaVersion), targetStatus.Location)
}

func TestPresent_Json(t *testing.T) {
	//GIVEN
	// 创建一组差异对象
	diffs := []v5.Diff{
		{v5.DiffAdded, "CVE-1", "nvd", []string{"requests", "vault"}},
		{v5.DiffRemoved, "CVE-2", "nvd", []string{"k8s"}},
		{v5.DiffChanged, "CVE-3", "nvd", []string{}},
	}
	// 创建一个 Differ 对象
	differ := Differ{}
	// 创建一个字节缓冲区
	var buffer bytes.Buffer
	// 当
	// 使用 require.NoError 检查是否没有错误发生，如果有错误则测试失败
	require.NoError(t, differ.Present("json", &diffs, &buffer))

	// 然后
	// 获取实际输出的字节流
	actual := buffer.Bytes()
	// 如果需要更新 golden 文件
	if *update {
		// 使用 testutils.UpdateGoldenFileContents 更新 golden 文件的内容
		testutils.UpdateGoldenFileContents(t, actual)
	}
	// 获取期望输出的字节流
	var expected = testutils.GetGoldenFileContents(t)
	// 如果期望输出和实际输出不相等
	if !bytes.Equal(expected, actual) {
		// 创建 diffmatchpatch 对象
		dmp := diffmatchpatch.New()
		// 使用 diffmatchpatch 对象比较期望输出和实际输出的差异
		diffs := dmp.DiffMain(string(expected), string(actual), true)
		// 输出差异信息
		t.Errorf("mismatched output:\n%s", dmp.DiffPrettyText(diffs))
	}
}

func TestPresent_Table(t *testing.T) {
	// 给定
	// 创建包含差异信息的数组
	diffs := []v5.Diff{
		{v5.DiffAdded, "CVE-1", "nvd", []string{"requests", "vault"}},
		// 创建包含DiffRemoved, "CVE-2", "nvd", []string{"k8s"}和DiffChanged, "CVE-3", "nvd", []string{}的切片
		// 这里使用了自定义的类型v5.DiffType
		diffs := []v5.Diff{
			{v5.DiffRemoved, "CVE-2", "nvd", []string{"k8s"}},
			{v5.DiffChanged, "CVE-3", "nvd", []string{}},
		}
		// 创建一个Differ类型的实例
		differ := Differ{}
		// 创建一个bytes.Buffer类型的实例
		var buffer bytes.Buffer

		// 调用differ的Present方法，将结果写入buffer中
		// 使用require.NoError检查是否有错误发生
		require.NoError(t, differ.Present("table", &diffs, &buffer))

		// 获取buffer中的字节内容
		actual := buffer.Bytes()
		// 如果update标志为true，则更新golden文件的内容
		if *update {
			testutils.UpdateGoldenFileContents(t, actual)
		}
		// 获取golden文件的内容
		var expected = testutils.GetGoldenFileContents(t)
		// 比较expected和actual的内容是否相等，如果不相等则输出差异
		if !bytes.Equal(expected, actual) {
			dmp := diffmatchpatch.New()
			diffs := dmp.DiffMain(string(expected), string(actual), true)
			t.Errorf("mismatched output:\n%s", dmp.DiffPrettyText(diffs))
		}
}

func TestPresent_Invalid(t *testing.T) {
	// 定义测试函数，测试无效情况
	// GIVEN
	// 定义一个包含 v5.Diff 类型的切片 diffs
	diffs := []v5.Diff{
		{v5.DiffRemoved, "CVE-2", "nvd", []string{"k8s"}},
	}
	// 定义一个 Differ 结构体实例
	differ := Differ{}
	// 定义一个 bytes.Buffer 类型的 buffer 变量
	var buffer bytes.Buffer

	// WHEN
	// 调用 differ.Present 方法，传入空字符串、diffs 切片和 buffer 变量的指针作为参数
	err := differ.Present("", &diffs, &buffer)

	// THEN
	// 断言错误是否发生
	require.Error(t, err)
}
```