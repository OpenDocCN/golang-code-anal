# `grype\grype\differ\differ_test.go`

```
package differ

import (
    "bytes" // 导入 bytes 包
    "flag" // 导入 flag 包
    "strconv" // 导入 strconv 包
    "testing" // 导入 testing 包

    "github.com/sergi/go-diff/diffmatchpatch" // 导入第三方库 diffmatchpatch
    "github.com/stretchr/testify/require" // 导入第三方库 require

    "github.com/anchore/go-testutils" // 导入自定义包 go-testutils
    "github.com/anchore/grype/grype/db" // 导入自定义包 grype/db
    v5 "github.com/anchore/grype/grype/db/v5" // 导入自定义包 grype/db/v5
    "github.com/anchore/grype/grype/vulnerability" // 导入自定义包 grype/vulnerability
)

var update = flag.Bool("update", false, "update the *.golden files for diff presenter") // 定义全局变量 update，用于更新 *.golden 文件

func TestNewDiffer(t *testing.T) {
    //GIVEN
    config := db.Config{} // 创建一个 db.Config 类型的变量 config

    //WHEN
    differ, err := NewDiffer(config) // 调用 NewDiffer 函数

    //THEN
    require.NoError(t, err) // 断言 err 为 nil
    require.NotNil(t, differ.baseCurator) // 断言 differ.baseCurator 不为 nil
}

func Test_DifferDirectory(t *testing.T) {
    d, err := NewDiffer(db.Config{
        DBRootDir: "root-dir",
    }) // 调用 NewDiffer 函数，传入 db.Config 类型的参数

    require.NoError(t, err) // 断言 err 为 nil

    err = d.SetBaseDB("test-fixtures/dbs/base") // 调用 d.SetBaseDB 函数
    require.NoError(t, err) // 断言 err 为 nil

    baseStatus := d.baseCurator.Status() // 获取 d.baseCurator 的状态
    require.Equal(t, "test-fixtures/dbs/base/"+strconv.Itoa(vulnerability.SchemaVersion), baseStatus.Location) // 断言 baseStatus.Location 与指定值相等

    err = d.SetTargetDB("test-fixtures/dbs/target") // 调用 d.SetTargetDB 函数
    require.NoError(t, err) // 断言 err 为 nil

    targetStatus := d.targetCurator.Status() // 获取 d.targetCurator 的状态
    require.Equal(t, "test-fixtures/dbs/target/"+strconv.Itoa(vulnerability.SchemaVersion), targetStatus.Location) // 断言 targetStatus.Location 与指定值相等
}

func TestPresent_Json(t *testing.T) {
    //GIVEN
    diffs := []v5.Diff{ // 创建一个 v5.Diff 类型的切片 diffs
        {v5.DiffAdded, "CVE-1", "nvd", []string{"requests", "vault"}}, // 初始化 v5.Diff 类型的结构体
        {v5.DiffRemoved, "CVE-2", "nvd", []string{"k8s"}}, // 初始化 v5.Diff 类型的结构体
        {v5.DiffChanged, "CVE-3", "nvd", []string{}}, // 初始化 v5.Diff 类型的结构体
    }
    differ := Differ{} // 创建一个 Differ 类型的变量 differ
    var buffer bytes.Buffer // 创建一个 bytes.Buffer 类型的变量 buffer

    // WHEN
    require.NoError(t, differ.Present("json", &diffs, &buffer)) // 调用 differ.Present 函数，并断言不会返回错误

    //THEN
    actual := buffer.Bytes() // 获取 buffer 的字节内容
    if *update { // 如果 update 为 true
        testutils.UpdateGoldenFileContents(t, actual) // 调用 testutils.UpdateGoldenFileContents 函数
    }
    var expected = testutils.GetGoldenFileContents(t) // 获取测试用例的黄金文件内容
}
    # 如果预期输出和实际输出不相等
    if !bytes.Equal(expected, actual) {
        # 创建 diffmatchpatch 对象
        dmp := diffmatchpatch.New()
        # 使用 diffmatchpatch 对象比较预期输出和实际输出，生成差异列表
        diffs := dmp.DiffMain(string(expected), string(actual), true)
        # 输出差异信息
        t.Errorf("mismatched output:\n%s", dmp.DiffPrettyText(diffs))
    }
func TestPresent_Table(t *testing.T) {
    // 定义测试函数 TestPresent_Table，用于测试 Present 方法生成表格形式的输出
    // GIVEN
    diffs := []v5.Diff{
        // 定义一个 Diff 列表，包含三个 Diff 对象，每个对象表示不同的变化类型和相关信息
        {v5.DiffAdded, "CVE-1", "nvd", []string{"requests", "vault"}},
        {v5.DiffRemoved, "CVE-2", "nvd", []string{"k8s"}},
        {v5.DiffChanged, "CVE-3", "nvd", []string{}},
    }
    // 创建一个 Differ 对象
    differ := Differ{}
    // 创建一个 bytes.Buffer 对象，用于存储输出结果
    var buffer bytes.Buffer

    // WHEN
    // 调用 Present 方法生成表格形式的输出，并验证是否没有错误发生
    require.NoError(t, differ.Present("table", &diffs, &buffer))

    //THEN
    // 获取实际输出结果的字节流
    actual := buffer.Bytes()
    // 如果需要更新测试数据，则更新对应的 golden 文件内容
    if *update {
        testutils.UpdateGoldenFileContents(t, actual)
    }
    // 获取期望输出结果的字节流
    var expected = testutils.GetGoldenFileContents(t)
    // 如果实际输出结果与期望输出结果不相等，则输出详细的差异信息
    if !bytes.Equal(expected, actual) {
        dmp := diffmatchpatch.New()
        diffs := dmp.DiffMain(string(expected), string(actual), true)
        t.Errorf("mismatched output:\n%s", dmp.DiffPrettyText(diffs))
    }
}

func TestPresent_Invalid(t *testing.T) {
    // 定义测试函数 TestPresent_Invalid，用于测试 Present 方法处理无效输入的情况
    //GIVEN
    diffs := []v5.Diff{
        // 定义一个 Diff 列表，包含一个 Diff 对象，表示一种变化类型和相关信息
        {v5.DiffRemoved, "CVE-2", "nvd", []string{"k8s"}},
    }
    // 创建一个 Differ 对象
    differ := Differ{}
    // 创建一个 bytes.Buffer 对象，用于存储输出结果
    var buffer bytes.Buffer

    // WHEN
    // 调用 Present 方法处理空字符串作为输出格式的情况，并获取返回的错误信息
    err := differ.Present("", &diffs, &buffer)

    //THEN
    // 验证是否返回了错误信息
    require.Error(t, err)
}
```