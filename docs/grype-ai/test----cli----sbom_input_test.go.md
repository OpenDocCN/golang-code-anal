# `grype\test\cli\sbom_input_test.go`

```go
package cli

import (
    "os"  // 导入操作系统相关的包
    "os/exec"  // 导入执行外部命令的包
    "path"  // 导入处理文件路径的包
    "runtime"  // 导入运行时信息的包
    "testing"  // 导入测试相关的包

    "github.com/stretchr/testify/require"  // 导入断言库
)

func TestSBOMInput_AsArgument(t *testing.T) {
    workingDirectory, err := os.Getwd()  // 获取当前工作目录
    if err != nil {  // 如果获取失败
        t.Fatal(err)  // 输出错误信息并退出测试
    }

    cases := []struct {  // 定义测试用例
        name string  // 测试用例名称
        path string  // 文件路径
    }{
        {
            "absolute path - image scan",  // 绝对路径 - 图像扫描
            path.Join(workingDirectory, "./test-fixtures/sbom-ubuntu-20.04--pruned.json"),  // 拼接工作目录和文件路径
        },
        {
            "relative path - image scan",  // 相对路径 - 图像扫描
            "./test-fixtures/sbom-ubuntu-20.04--pruned.json",  // 文件路径
        },
        {
            "directory scan",  // 目录扫描
            "./test-fixtures/sbom-grype-source.json",  // 文件路径
        },
    }

    t.Run("explicit", func(t *testing.T) {  // 运行显式测试
        for _, tc := range cases {  // 遍历测试用例
            t.Run(tc.name, func(t *testing.T) {  // 运行测试用例
                sbomArg := "sbom:" + tc.path  // 构造参数
                cmd := getGrypeCommand(t, sbomArg)  // 获取 Grype 命令

                assertCommandExecutionSuccess(t, cmd)  // 断言命令执行成功
            })
        }
    })

    t.Run("implicit", func(t *testing.T) {  // 运行隐式测试
        for _, tc := range cases {  // 遍历测试用例
            t.Run(tc.name, func(t *testing.T) {  // 运行测试用例
                sbomArg := tc.path  // 构造参数
                cmd := getGrypeCommand(t, sbomArg)  // 获取 Grype 命令

                assertCommandExecutionSuccess(t, cmd)  // 断言命令执行成功
            })
        }
    })
}

func TestSBOMInput_FromStdin(t *testing.T) {
    tests := []struct {  // 定义测试用例
        name       string  // 测试用例名称
        input      string  // 输入
        args       []string  // 参数
        wantErr    require.ErrorAssertionFunc  // 期望错误
        wantOutput string  // 期望输出
    # 测试用例列表，包含不同的测试情况
    }{
        {
            # 测试名称：空文件
            name:       "empty file",
            # 输入文件路径
            input:      "./test-fixtures/empty.json",
            # 命令行参数
            args:       []string{"-c", "../grype-test-config.yaml"},
            # 期望出错
            wantErr:    require.Error,
            # 期望输出信息
            wantOutput: "unable to decode sbom: sbom format not recognized",
        },
        {
            # 测试名称：SBOM
            name:    "sbom",
            # 输入文件路径
            input:   "./test-fixtures/sbom-ubuntu-20.04--pruned.json",
            # 命令行参数
            args:    []string{"-c", "../grype-test-config.yaml"},
            # 期望不出错
            wantErr: require.NoError,
        },
    }

    # 遍历测试用例列表
    for _, tt := range tests {
        # 运行单个测试用例
        t.Run(tt.name, func(t *testing.T) {
            # 创建执行命令
            cmd := exec.Command(getGrypeSnapshotLocation(t, runtime.GOOS), tt.args...)

            # 打开输入文件
            input, err := os.Open(tt.input)
            require.NoError(t, err)

            # 将输入文件附加到命令的标准输入
            attachFileToCommandStdin(t, input, cmd)
            # 关闭输入文件
            err = input.Close()
            require.NoError(t, err)

            # 执行命令并获取输出
            output, err := cmd.CombinedOutput()
            # 检查是否符合期望的错误
            tt.wantErr(t, err, "output: %s", output)
            # 如果有期望输出信息，则检查输出中是否包含该信息
            if tt.wantOutput != "" {
                require.Contains(t, string(output), tt.wantOutput)
            }

        })
    }
# 闭合前面的函数定义
```