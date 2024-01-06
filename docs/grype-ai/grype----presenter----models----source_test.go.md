# `grype\grype\presenter\models\source_test.go`

```
package models

import (
	"testing"  // 导入测试包

	"github.com/stretchr/testify/assert"  // 导入断言包
	"github.com/stretchr/testify/require"  // 导入断言包

	syftSource "github.com/anchore/syft/syft/source"  // 导入syftSource包

)

func TestNewSource(t *testing.T) {  // 定义测试函数
	testCases := []struct {  // 定义测试用例
		name     string  // 测试用例名称
		metadata syftSource.Description  // 元数据描述
		expected source  // 期望的源
	}{
		{
			name: "image",  // 测试用例名称为"image"
			metadata: syftSource.Description{  // 元数据描述为syftSource包中的Description结构
# 创建一个包含图像源元数据的结构体
Metadata: syftSource.StereoscopeImageSourceMetadata{
    # 设置用户输入
    UserInput:      "abc",
    # 设置ID
    ID:             "def",
    # 设置清单摘要
    ManifestDigest: "abcdef",
    # 设置大小
    Size:           100,
},
# 期望的图像源结构体
expected: source{
    # 设置类型为图像
    Type: "image",
    # 设置目标为包含图像源元数据的结构体
    Target: syftSource.StereoscopeImageSourceMetadata{
        # 设置用户输入
        UserInput:      "abc",
        # 设置ID
        ID:             "def",
        # 设置清单摘要
        ManifestDigest: "abcdef",
        # 设置大小
        Size:           100,
        # 设置仓库摘要为空列表
        RepoDigests:    []string{},
        # 设置标签为空列表
        Tags:           []string{},
    },
},
# 定义一个名为 "directory" 的源对象，包含元数据描述和预期结果
name: "directory",
metadata: syftSource.Description{
    # 定义目录源的元数据，包括路径信息
    Metadata: syftSource.DirectorySourceMetadata{
        Path: "/foo/bar",
    },
},
expected: source{
    # 定义预期的目录源对象，包括类型和目标路径
    Type:   "directory",
    Target: "/foo/bar",
},

# 定义一个名为 "file" 的源对象，包含元数据描述和预期结果
name: "file",
metadata: syftSource.Description{
    # 定义文件源的元数据，包括路径信息
    Metadata: syftSource.FileSourceMetadata{
        Path: "/foo/bar/test.zip",
    },
},
expected: source{
    # 定义预期的文件源对象，包括类型和目标路径
    Type:   "file",
# 定义一个测试用例的目标 ZIP 文件路径
Target: "/foo/bar/test.zip",

# 定义测试用例
testCases := []struct {
    name string
    metadata map[string]string
    expected source
}{
    {
        name: "Test Case Name",
        metadata: map[string]string{
            "key1": "value1",
            "key2": "value2",
            # 定义预期结果
            "key3": "value3",
        },
        # 预期结果
        expected: source{
            "key1": "value1",
            "key2": "value2",
            "key3": "value3",
        },
    },
}

# 遍历测试用例
for _, testCase := range testCases {
    # 运行测试用例
    t.Run(testCase.name, func(t *testing.T) {
        # 调用函数获取实际结果和错误信息
        actual, err := newSource(testCase.metadata)
        # 断言错误信息为空
        require.NoError(t, err)
        # 断言实际结果等于预期结果
        assert.Equal(t, testCase.expected, actual)
    })
}
```