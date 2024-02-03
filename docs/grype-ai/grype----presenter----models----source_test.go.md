# `grype\grype\presenter\models\source_test.go`

```go
package models

import (
    "testing"  // 导入测试包

    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包

    syftSource "github.com/anchore/syft/syft/source"  // 导入syftSource包
)

func TestNewSource(t *testing.T) {  // 定义测试函数
    testCases := []struct {  // 定义测试用例结构体切片
        name     string  // 测试用例名称
        metadata syftSource.Description  // 元数据
        expected source  // 期望的源
    }{
        {
            name: "image",  // 图像类型的测试用例
            metadata: syftSource.Description{  // 图像类型的元数据
                Metadata: syftSource.StereoscopeImageSourceMetadata{  // 图像类型的具体元数据
                    UserInput:      "abc",  // 用户输入
                    ID:             "def",  // ID
                    ManifestDigest: "abcdef",  // Manifest摘要
                    Size:           100,  // 大小
                },
            },
            expected: source{  // 期望的源
                Type: "image",  // 类型为图像
                Target: syftSource.StereoscopeImageSourceMetadata{  // 目标为图像类型的具体元数据
                    UserInput:      "abc",  // 用户输入
                    ID:             "def",  // ID
                    ManifestDigest: "abcdef",  // Manifest摘要
                    Size:           100,  // 大小
                    RepoDigests:    []string{},  // 仓库摘要
                    Tags:           []string{},  // 标签
                },
            },
        },
        {
            name: "directory",  // 目录类型的测试用例
            metadata: syftSource.Description{  // 目录类型的元数据
                Metadata: syftSource.DirectorySourceMetadata{  // 目录类型的具体元数据
                    Path: "/foo/bar",  // 路径
                },
            },
            expected: source{  // 期望的源
                Type:   "directory",  // 类型为目录
                Target: "/foo/bar",  // 目标为路径
            },
        },
        {
            name: "file",  // 文件类型的测试用例
            metadata: syftSource.Description{  // 文件类型的元数据
                Metadata: syftSource.FileSourceMetadata{  // 文件类型的具体元数据
                    Path: "/foo/bar/test.zip",  // 路径
                },
            },
            expected: source{  // 期望的源
                Type:   "file",  // 类型为文件
                Target: "/foo/bar/test.zip",  // 目标为路径
            },
        },
    }
    # 遍历测试用例切片，每个测试用例包括名称和元数据
    for _, testCase := range testCases:
        # 使用测试用例的名称创建子测试，运行匿名函数
        t.Run(testCase.name, func(t *testing.T):
            # 调用 newSource 函数，传入测试用例的元数据，获取返回的实际结果和可能的错误
            actual, err := newSource(testCase.metadata)
            # 断言错误对象为空
            require.NoError(t, err)
            # 断言实际结果与预期结果相等
            assert.Equal(t, testCase.expected, actual)
        )
    }
# 闭合前面的函数定义
```