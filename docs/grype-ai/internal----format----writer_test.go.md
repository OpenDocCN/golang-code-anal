# `grype\internal\format\writer_test.go`

```go
package format

import (
    "path/filepath"  // 导入处理文件路径的包
    "strings"       // 导入处理字符串的包
    "testing"       // 导入测试包

    "github.com/docker/docker/pkg/homedir"  // 导入处理主目录的包
    "github.com/stretchr/testify/assert"     // 导入断言包
)

func Test_MakeScanResultWriter(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        outputs []string  // 输出格式切片
        wantErr assert.ErrorAssertionFunc  // 期望的错误断言函数
    }{
        {
            outputs: []string{"json"},  // 设置输出格式为 JSON
            wantErr: assert.NoError,    // 期望没有错误
        },
        {
            outputs: []string{"table", "json"},  // 设置输出格式为表格和 JSON
            wantErr: assert.NoError,    // 期望没有错误
        },
        {
            outputs: []string{"unknown"},  // 设置未知的输出格式
            wantErr: func(t assert.TestingT, err error, bla ...interface{}) bool {  // 定义自定义的错误断言函数
                return assert.ErrorContains(t, err, `unsupported output format "unknown", supported formats are: [`)  // 判断错误信息是否包含特定内容
            },
        },
    }

    for _, tt := range tests {  // 遍历测试用例
        _, err := MakeScanResultWriter(tt.outputs, "", PresentationConfig{})  // 调用函数进行测试
        tt.wantErr(t, err)  // 断言是否符合期望的错误情况
    }
}

func Test_newSBOMMultiWriter(t *testing.T) {
    type writerConfig struct {  // 定义写入配置结构体
        format string  // 格式
        file   string  // 文件路径
    }

    tmp := t.TempDir()  // 创建临时目录

    testName := func(options []scanResultWriterDescription, err bool) string {  // 定义测试名称函数
        var out []string  // 定义输出格式切片
        for _, opt := range options {  // 遍历选项
            out = append(out, string(opt.Format)+"="+opt.Path)  // 将格式和路径添加到输出切片中
        }
        errs := ""  // 错误信息字符串
        if err {  // 如果有错误
            errs = "(err)"  // 错误信息为 "(err)"
        }
        return strings.Join(out, ", ") + errs  // 返回格式和路径组成的字符串，如果有错误则加上 "(err)"
    }

    tests := []struct {  // 定义测试用例结构体切片
        outputs  []scanResultWriterDescription  // 输出格式描述切片
        err      bool  // 是否有错误
        expected []writerConfig  // 期望的写入配置
    }
    # 遍历测试用例切片，每个测试用例包括输出和错误信息
    for _, test := range tests {
        # 使用测试用例的输出和错误信息生成测试名称，并在子测试中运行
        t.Run(testName(test.outputs, test.err), func(t *testing.T) {
            # 复制测试用例的输出
            outputs := test.outputs
            # 遍历输出切片，如果路径不为空，则在路径前添加临时路径
            for i := range outputs {
                if outputs[i].Path != "" {
                    outputs[i].Path = tmp + outputs[i].Path
                }
            }

            # 创建多重写入器，并返回错误信息
            mw, err := newMultiWriter(outputs...)

            # 如果测试用例中包含错误信息，则断言返回的错误信息不为空
            if test.err {
                assert.Error(t, err)
                return
            } else {
                # 如果测试用例中不包含错误信息，则断言返回的错误信息为空
                assert.NoError(t, err)
            }

            # 断言多重写入器的写入器数量与预期长度相等
            assert.Len(t, mw.writers, len(test.expected))

            # 遍历预期结果切片，比较每个写入器的类型和格式
            for i, e := range test.expected {
                switch w := mw.writers[i].(type) {
                case *scanResultStreamWriter:
                    # 如果写入器类型为scanResultStreamWriter，则断言格式和输出不为空
                    assert.Equal(t, string(w.format), e.format)
                    assert.NotNil(t, w.out)
                    # 如果文件名不为空，则断言文件存在
                    if e.file != "" {
                        assert.FileExists(t, tmp+e.file)
                    }
                case *scanResultPublisher:
                    # 如果写入器类型为scanResultPublisher，则断言格式相等
                    assert.Equal(t, string(w.format), e.format)
                default:
                    # 如果写入器类型未知，则输出错误信息
                    t.Fatalf("unknown writer type: %T", w)
                }

            }
        })
    }
// 定义测试函数 Test_newSBOMWriterDescription，用于测试 newWriterDescription 函数
func Test_newSBOMWriterDescription(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string   // 测试用例名称
        path     string   // 文件路径
        expected string   // 期望的输出
    }{
        {
            name:     "expand home dir",  // 测试用例1：扩展家目录
            path:     "~/place.txt",       // 文件路径为家目录下的 place.txt
            expected: filepath.Join(homedir.Get(), "place.txt"),  // 期望的输出为家目录下的 place.txt 的完整路径
        },
        {
            name:     "passthrough other paths",  // 测试用例2：传递其他路径
            path:     "/other/place.txt",         // 文件路径为 /other/place.txt
            expected: "/other/place.txt",         // 期望的输出为 /other/place.txt
        },
        {
            name:     "no path",  // 测试用例3：没有路径
            path:     "",         // 文件路径为空
            expected: "",         // 期望的输出为空
        },
    }
    // 遍历测试用例
    for _, tt := range tests {
        // 运行子测试
        t.Run(tt.name, func(t *testing.T) {
            // 调用 newWriterDescription 函数创建对象 o
            o := newWriterDescription("table", tt.path, PresentationConfig{})
            // 断言 o.Path 的值等于期望的输出
            assert.Equal(t, tt.expected, o.Path)
        })
    }
}
```