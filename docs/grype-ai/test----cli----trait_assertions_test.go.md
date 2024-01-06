# `grype\test\cli\trait_assertions_test.go`

```
// 声明一个名为 cli 的包
package cli

// 导入所需的包
import (
	"strings"
	"testing"

	"github.com/acarl005/stripansi"
)

// 定义一个名为 traitAssertion 的函数类型，用于断言测试结果
type traitAssertion func(tb testing.TB, stdout, stderr string, rc int)

// 定义一个函数，用于断言输出中是否包含特定数据
func assertInOutput(data string) traitAssertion {
	// 返回一个函数，该函数用于在测试结果中断言是否包含特定数据
	return func(tb testing.TB, stdout, stderr string, _ int) {
		// 标记该函数为测试辅助函数
		tb.Helper()

		// 检查标准输出和标准错误输出中是否包含特定数据，如果不包含则输出错误信息
		if !strings.Contains(stripansi.Strip(stderr), data) && !strings.Contains(stripansi.Strip(stdout), data) {
			tb.Errorf("data=%q was NOT found in any output, but should have been there", data)
		}
	}
}
# 定义一个函数，用于断言返回代码为失败的情况
func assertFailingReturnCode(tb testing.TB, _, _ string, rc int) {
    tb.Helper()
    # 如果返回代码为0，则输出错误信息
    if rc == 0 {
        tb.Errorf("expected a failure but got rc=%d", rc)
    }
}

# 定义一个函数，用于断言返回代码为成功的情况
func assertSucceedingReturnCode(tb testing.TB, _, _ string, rc int) {
    tb.Helper()
    # 如果返回代码不为0，则输出错误信息
    if rc != 0 {
        tb.Errorf("expected to succeed but got rc=%d", rc)
    }
}

# 定义一个函数，用于断言标准输出中是否包含指定行
func assertRowInStdOut(row []string) traitAssertion {
    return func(tb testing.TB, stdout, stderr string, _ int) {
        tb.Helper()
        # 遍历标准输出的每一行
        for _, line := range strings.Split(stdout, "\n") {
			lineMatched := false
            // 初始化变量，用于标记是否匹配到某一行
			for _, column := range row {
                // 遍历行中的每一列
				if !strings.Contains(line, column) {
                    // 如果某一列不在当前行中
					// it wasn't this line
					lineMatched = false
					break
                    // 标记为未匹配，并跳出循环
				}
				lineMatched = true
                // 如果所有列都在当前行中，则标记为匹配
			}
			if lineMatched {
                // 如果匹配到了，则直接返回，不再执行后续代码
				return
			}
            // 如果没有匹配到任何行，则输出错误信息
		}
		// none of the lines matched
		tb.Errorf("expected stdout to contain %s, but it did not", strings.Join(row, " "))
        // 输出错误信息，表示预期的内容没有在输出中找到
	}
}

func assertNotInOutput(notWanted string) traitAssertion {
    // 定义一个函数，用于判断某一字符串不应该出现在输出中
	return func(tb testing.TB, stdout, stderr string, _ int) {
        // 函数体内部的具体实现
# 如果标准输出中包含不想要的内容
if strings.Contains(stdout, notWanted):
    # 输出错误信息，指出在标准输出中发现了不想要的内容
    tb.Errorf("got unwanted %s in stdout %s", notWanted, stdout)
# 结束当前测试函数
```