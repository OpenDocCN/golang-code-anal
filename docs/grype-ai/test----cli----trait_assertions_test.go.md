# `grype\test\cli\trait_assertions_test.go`

```
package cli

import (
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/acarl005/stripansi"  // 导入第三方库，用于去除 ANSI 转义字符
)

type traitAssertion func(tb testing.TB, stdout, stderr string, rc int)  // 定义一个函数类型 traitAssertion，用于测试断言

func assertInOutput(data string) traitAssertion {  // 定义一个函数，用于检查输出中是否包含指定数据
    return func(tb testing.TB, stdout, stderr string, _ int) {  // 返回一个函数，用于实际执行检查
        tb.Helper()  // 标记当前测试函数为辅助函数

        if !strings.Contains(stripansi.Strip(stderr), data) && !strings.Contains(stripansi.Strip(stdout), data) {  // 检查标准输出和标准错误输出中是否包含指定数据
            tb.Errorf("data=%q was NOT found in any output, but should have been there", data)  // 输出错误信息
        }
    }
}

func assertFailingReturnCode(tb testing.TB, _, _ string, rc int) {  // 定义一个函数，用于检查返回值是否为失败状态
    tb.Helper()  // 标记当前测试函数为辅助函数
    if rc == 0 {  // 检查返回值是否为 0
        tb.Errorf("expected a failure but got rc=%d", rc)  // 输出错误信息
    }
}

func assertSucceedingReturnCode(tb testing.TB, _, _ string, rc int) {  // 定义一个函数，用于检查返回值是否为成功状态
    tb.Helper()  // 标记当前测试函数为辅助函数
    if rc != 0 {  // 检查返回值是否不为 0
        tb.Errorf("expected to succeed but got rc=%d", rc)  // 输出错误信息
    }
}

func assertRowInStdOut(row []string) traitAssertion {  // 定义一个函数，用于检查标准输出中是否包含指定行
    return func(tb testing.TB, stdout, stderr string, _ int) {  // 返回一个函数，用于实际执行检查
        tb.Helper()  // 标记当前测试函数为辅助函数

        for _, line := range strings.Split(stdout, "\n") {  // 遍历标准输出中的每一行
            lineMatched := false  // 初始化行匹配状态为 false
            for _, column := range row {  // 遍历指定行中的每一列
                if !strings.Contains(line, column) {  // 检查当前行是否包含指定列
                    // it wasn't this line
                    lineMatched = false  // 如果不包含，则将行匹配状态设置为 false
                    break  // 跳出当前循环
                }
                lineMatched = true  // 如果包含，则将行匹配状态设置为 true
            }
            if lineMatched {  // 如果行匹配状态为 true
                return  // 结束函数执行
            }
        }
        // none of the lines matched
        tb.Errorf("expected stdout to contain %s, but it did not", strings.Join(row, " "))  // 输出错误信息
    }
}

func assertNotInOutput(notWanted string) traitAssertion {  // 定义一个函数，用于检查输出中是否不包含指定数据
    return func(tb testing.TB, stdout, stderr string, _ int) {  // 返回一个函数，用于实际执行检查
        if strings.Contains(stdout, notWanted) {  // 检查标准输出中是否包含不希望出现的数据
            tb.Errorf("got unwanted %s in stdout %s", notWanted, stdout)  // 输出错误信息
        }
    }
}
```