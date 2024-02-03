# `kubo\docs\examples\kubo-as-a-library\main_test.go`

```go
// 声明 main 包
package main

// 导入所需的包
import (
    "os/exec" // 用于执行外部命令
    "strings" // 用于处理字符串
    "testing" // 用于编写测试函数
)

// 编写测试函数
func TestExample(t *testing.T) {
    // 执行命令 "go run main.go"，并获取输出和错误信息
    out, err := exec.Command("go", "run", "main.go").Output()
    // 如果有错误发生，则输出错误信息
    if err != nil {
        t.Fatalf("running example (%v)", err)
    }
    // 如果输出中不包含 "All done!"，则输出错误信息
    if !strings.Contains(string(out), "All done!") {
        t.Errorf("example did not run successfully")
    }
}
```