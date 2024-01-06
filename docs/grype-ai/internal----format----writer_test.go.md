# `grype\internal\format\writer_test.go`

```
// 导入所需的包
package format

import (
	"path/filepath" // 导入处理文件路径的包
	"strings" // 导入处理字符串的包
	"testing" // 导入测试包

	"github.com/docker/docker/pkg/homedir" // 导入处理用户目录的包
	"github.com/stretchr/testify/assert" // 导入断言包
)

// 测试 MakeScanResultWriter 函数
func Test_MakeScanResultWriter(t *testing.T) {
	// 定义测试用例
	tests := []struct {
		outputs []string // 输出格式
		wantErr assert.ErrorAssertionFunc // 期望的错误断言函数
	}{
		{
			outputs: []string{"json"}, // 输出格式为 JSON
			wantErr: assert.NoError, // 期望没有错误
		},
# 定义测试用例，包括期望的输出格式和错误情况
{
    outputs: []string{"table", "json"},  # 期望的输出格式为表格和 JSON
    wantErr: assert.NoError,  # 期望没有错误发生
},
{
    outputs: []string{"unknown"},  # 期望的输出格式为未知
    wantErr: func(t assert.TestingT, err error, bla ...interface{}) bool {
        return assert.ErrorContains(t, err, `unsupported output format "unknown", supported formats are: [`)  # 期望的错误信息包含特定内容
    },
},

# 遍历测试用例
for _, tt := range tests {
    # 调用 MakeScanResultWriter 函数，传入输出格式和其他参数，获取结果和错误
    _, err := MakeScanResultWriter(tt.outputs, "", PresentationConfig{})
    # 检查是否符合期望的错误情况
    tt.wantErr(t, err)
}
```

```
# 测试函数，用于测试新的SBOM多重写入器
func Test_newSBOMMultiWriter(t *testing.T) {
    type writerConfig struct {
    # 此处省略部分代码
```

	// 定义格式字符串和文件字符串
		format string
		file   string
	}

	// 创建临时目录
	tmp := t.TempDir()

	// 定义测试名称函数，根据选项和错误返回测试名称
	testName := func(options []scanResultWriterDescription, err bool) string {
		var out []string
		// 遍历选项，将格式和路径组成字符串数组
		for _, opt := range options {
			out = append(out, string(opt.Format)+"="+opt.Path)
		}
		errs := ""
		// 如果有错误，添加错误标记
		if err {
			errs = "(err)"
		}
		// 将字符串数组和错误标记拼接成测试名称
		return strings.Join(out, ", ") + errs
	}

	// 定义测试用例
	tests := []struct {
		outputs  []scanResultWriterDescription
		err      bool  // 定义一个布尔类型的变量err
		expected []writerConfig  // 定义一个writerConfig类型的切片变量expected
	}{
		{  // 第一个匿名结构体
			outputs: []scanResultWriterDescription{},  // 设置outputs字段为一个空的scanResultWriterDescription类型的切片
			err:     true,  // 设置err字段为true
		},
		{  // 第二个匿名结构体
			outputs: []scanResultWriterDescription{  // 设置outputs字段为一个包含一个scanResultWriterDescription类型的切片
				{
					Format: "table",  // 设置Format字段为"table"
					Path:   "",  // 设置Path字段为空字符串
				},
			},
			expected: []writerConfig{  // 设置expected字段为一个writerConfig类型的切片
				{
					format: "table",  // 设置format字段为"table"
				},
			},
		},
# 创建一个包含输出和期望结果的列表
{
    # 第一个输出结果的描述
    outputs: []scanResultWriterDescription{
        # 第一个输出结果的格式为 JSON
        {
            Format: "json",
        },
    },
    # 第一个输出结果的期望配置
    expected: []writerConfig{
        # 第一个输出结果的期望格式为 JSON
        {
            format: "json",
        },
    },
},
{
    # 第二个输出结果的描述
    outputs: []scanResultWriterDescription{
        # 第二个输出结果的格式为 JSON，路径为 "test-2.json"
        {
            Format: "json",
            Path:   "test-2.json",
        },
    },
    # 第二个输出结果的期望配置
    expected: []writerConfig{
# 定义一个包含格式和文件名的对象
{
    format: "json",
    file:   "test-2.json",
},
# 定义一个包含输出格式和路径的对象
{
    outputs: []scanResultWriterDescription{
        # 定义一个包含格式和路径的对象
        {
            Format: "json",
            Path:   "test-3/1.json",
        },
        # 定义一个包含格式和路径的对象
        {
            Format: "spdx-json",
            Path:   "test-3/2.json",
        },
    },
    # 定义一个包含期望格式的对象
    expected: []writerConfig{
        # 定义一个包含格式的对象
        {
            format: "json",
# 定义一个包含文件路径的结构体
file:   "test-3/1.json",
# 定义一个包含格式和文件路径的结构体
{
    format: "spdx-json",
    file:   "test-3/2.json",
},
# 定义一个包含输出描述的结构体
{
    outputs: []scanResultWriterDescription{
        # 定义一个包含格式的结构体
        {
            Format: "text",
        },
        # 定义一个包含格式和文件路径的结构体
        {
            Format: "spdx-json",
            Path:   "test-4.json",
        },
    },
    # 定义一个包含预期输出的结构体
    expected: []writerConfig{
# 定义一个结构体数组，每个结构体包含format和file两个字段
tests := []struct {
	// 定义outputs和err两个字段
	outputs []output
	err     error
}{
	{
		// 第一个结构体包含一个output结构体数组
		outputs: []output{
			{
				format: "text",
			},
			{
				format: "spdx-json",
				file:   "test-4.json",
			},
		},
	},
}

# 遍历tests数组
for _, test := range tests {
	# 使用testName函数创建测试名称，并在测试函数中运行
	t.Run(testName(test.outputs, test.err), func(t *testing.T) {
		# 将test.outputs赋值给outputs
		outputs := test.outputs
		# 遍历outputs数组
		for i := range outputs {
			# 如果outputs[i].Path不为空，则在其前面添加tmp
			if outputs[i].Path != "" {
				outputs[i].Path = tmp + outputs[i].Path
			}
		}

		# 调用newMultiWriter函数，传入outputs数组作为参数
		mw, err := newMultiWriter(outputs...)
# 如果测试出现错误，断言应该出现错误
if test.err {
    assert.Error(t, err)
    return
} else {
    assert.NoError(t, err)
}

# 断言写入器的数量与预期数量相等
assert.Len(t, mw.writers, len(test.expected))

# 遍历预期结果数组
for i, e := range test.expected {
    # 根据写入器类型进行断言
    switch w := mw.writers[i].(type) {
        case *scanResultStreamWriter:
            # 断言写入器的格式与预期格式相等
            assert.Equal(t, string(w.format), e.format)
            # 断言写入器的输出不为空
            assert.NotNil(t, w.out)
            # 如果预期文件名不为空，断言文件存在
            if e.file != "" {
                assert.FileExists(t, tmp+e.file)
            }
        case *scanResultPublisher:
            # 断言写入器的格式与预期格式相等
            assert.Equal(t, string(w.format), e.format)
    }
}
// 在switch语句中处理默认情况，如果遇到未知的写入器类型，则输出错误信息
default:
    t.Fatalf("unknown writer type: %T", w)
}

// 结束switch语句的大括号
}

// 结束Test_newSBOMWriterDescription函数
}

// 定义测试函数Test_newSBOMWriterDescription
func Test_newSBOMWriterDescription(t *testing.T) {
    // 定义测试用例
    tests := []struct {
        name     string
        path     string
        expected string
    }{
        {
            name:     "expand home dir",
            path:     "~/place.txt",
            expected: filepath.Join(homedir.Get(), "place.txt"),
        },
```

# 定义测试用例数组，每个测试用例包括名称、路径和期望结果
{
    name:     "passthrough other paths",  # 测试用例名称
    path:     "/other/place.txt",         # 测试用例路径
    expected: "/other/place.txt",          # 期望结果
},
{
    name:     "no path",                   # 测试用例名称
    path:     "",                          # 测试用例路径
    expected: "",                          # 期望结果
}
# 遍历测试用例数组
for _, tt := range tests {
    # 对每个测试用例运行子测试
    t.Run(tt.name, func(t *testing.T) {
        # 创建一个新的写入描述对象，类型为表格，路径为测试用例的路径，配置为空
        o := newWriterDescription("table", tt.path, PresentationConfig{})
        # 断言实际结果与期望结果相等
        assert.Equal(t, tt.expected, o.Path)
    })
}
```