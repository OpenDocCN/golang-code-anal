# `grype\test\cli\sbom_input_test.go`

```
package cli

import (
	"os" // 导入操作系统相关的包
	"os/exec" // 导入执行外部命令的包
	"path" // 导入处理文件路径的包
	"runtime" // 导入运行时信息的包
	"testing" // 导入测试相关的包

	"github.com/stretchr/testify/require" // 导入断言库
)

func TestSBOMInput_AsArgument(t *testing.T) {
	workingDirectory, err := os.Getwd() // 获取当前工作目录
	if err != nil {
		t.Fatal(err) // 如果获取失败，输出错误信息并终止测试
	}

	cases := []struct {
		name string // 定义测试用例的名称
// 定义一个结构体切片，包含了不同类型的路径
path string
}{
	// 第一个路径为绝对路径，用path.Join函数拼接工作目录和文件路径
	{
		"absolute path - image scan",
		path.Join(workingDirectory, "./test-fixtures/sbom-ubuntu-20.04--pruned.json"),
	},
	// 第二个路径为相对路径
	{
		"relative path - image scan",
		"./test-fixtures/sbom-ubuntu-20.04--pruned.json",
	},
	// 第三个路径为目录路径
	{
		"directory scan",
		"./test-fixtures/sbom-grype-source.json",
	},
}

// 使用t.Run函数运行测试用例
t.Run("explicit", func(t *testing.T) {
	// 遍历路径切片
	for _, tc := range cases {
		// 使用t.Run函数运行子测试用例
		t.Run(tc.name, func(t *testing.T) {
			// 拼接路径为sbom:路径，用于后续处理
			sbomArg := "sbom:" + tc.path
// 运行测试用例，使用 getGrypeCommand 函数获取命令
cmd := getGrypeCommand(t, sbomArg)

// 断言命令执行成功
assertCommandExecutionSuccess(t, cmd)
```

```
// 运行 "implicit" 测试用例
t.Run("implicit", func(t *testing.T) {
    // 遍历测试用例
    for _, tc := range cases {
        // 运行每个测试用例
        t.Run(tc.name, func(t *testing.T) {
            // 获取 sbomArg
            sbomArg := tc.path
            // 使用 getGrypeCommand 函数获取命令
            cmd := getGrypeCommand(t, sbomArg)

            // 断言命令执行成功
            assertCommandExecutionSuccess(t, cmd)
        })
    }
})
```

```
// 测试 SBOMInput_FromStdin 函数
func TestSBOMInput_FromStdin(t *testing.T) {
// 定义一个测试结构体数组，包含测试名称、输入、参数、期望错误、期望输出
tests := []struct {
    name       string
    input      string
    args       []string
    wantErr    require.ErrorAssertionFunc
    wantOutput string
}{
    // 第一个测试
    {
        name:       "empty file", // 测试名称
        input:      "./test-fixtures/empty.json", // 输入文件路径
        args:       []string{"-c", "../grype-test-config.yaml"}, // 参数
        wantErr:    require.Error, // 期望出现错误
        wantOutput: "unable to decode sbom: sbom format not recognized", // 期望输出
    },
    // 第二个测试
    {
        name:    "sbom", // 测试名称
        input:   "./test-fixtures/sbom-ubuntu-20.04--pruned.json", // 输入文件路径
        args:    []string{"-c", "../grype-test-config.yaml"}, // 参数
        wantErr: require.NoError, // 期望没有错误
    },
}
	}

	for _, tt := range tests {
		// 遍历测试用例
		t.Run(tt.name, func(t *testing.T) {
			// 运行子测试
			cmd := exec.Command(getGrypeSnapshotLocation(t, runtime.GOOS), tt.args...)

			// 打开输入文件
			input, err := os.Open(tt.input)
			require.NoError(t, err)

			// 将输入文件附加到命令的标准输入
			attachFileToCommandStdin(t, input, cmd)
			err = input.Close()
			require.NoError(t, err)

			// 执行命令并获取输出
			output, err := cmd.CombinedOutput()
			tt.wantErr(t, err, "output: %s", output)
			// 检查输出是否符合预期
			if tt.wantOutput != "" {
				require.Contains(t, string(output), tt.wantOutput)
			}

		})
	}
```

这部分代码缺少具体的内容，无法添加注释。
```