# `grype\internal\file\hasher_test.go`

```
# 导入所需的包
package file

import (
    "fmt"  # 导入 fmt 包，用于格式化输出
    "testing"  # 导入 testing 包，用于编写测试函数

    "github.com/spf13/afero"  # 导入 afero 包，用于文件系统操作
    "github.com/stretchr/testify/assert"  # 导入 testify 包，用于断言
)

# 定义测试函数 TestValidateByHash
func TestValidateByHash(t *testing.T) {
    # 定义测试用例结构体
    testsCases := []struct {
        name, path, hashStr, actualHash string  # 定义测试用例的名称、路径、哈希值、实际哈希值
        setup                           func(fs afero.Fs)  # 定义设置函数，用于设置文件系统
        valid                           bool  # 定义有效性标志
        err                             bool  # 定义错误标志
        errMsg                          error  # 定义错误信息
    # 定义测试用例，包括文件名、哈希值、设置函数、实际哈希值、是否有效、是否出错等信息
    {
        # 测试用例名称
        name:    "Valid SHA256 hash",
        # 文件路径
        path:    "test.txt",
        # 期望的哈希值
        hashStr: "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
        # 设置函数，用于创建测试文件
        setup: func(fs afero.Fs) {
            afero.WriteFile(fs, "test.txt", []byte("test"), 0644)
        },
        # 实际的哈希值
        actualHash: "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
        # 是否有效
        valid:      true,
        # 是否出错
        err:        false,
    },
    {
        name:    "Invalid SHA256 hash",
        path:    "test.txt",
        hashStr: "sha256:deadbeef",
        setup: func(fs afero.Fs) {
            afero.WriteFile(fs, "test.txt", []byte("test"), 0644)
        },
        actualHash: "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
        valid:      false,
        err:        false,
    },
    {
        name:    "Unsupported hash function",
        path:    "test.txt",
        hashStr: "md5:deadbeef",
        setup: func(fs afero.Fs) {
            afero.WriteFile(fs, "test.txt", []byte("test"), 0644)
        },
        actualHash: "",
        valid:      false,
        err:        true,
        errMsg:     fmt.Errorf("hasher not supported or specified (given: md5:deadbeef)"),
    },
    {
        name:       "File does not exist",
        path:       "nonexistent.txt",
        hashStr:    "sha256:deadbeef",
        setup:      func(fs afero.Fs) {},
        valid:      false,
        actualHash: "",
        err:        true,
    },
    # 遍历测试用例列表
    for _, tc := range testsCases {
        # 在测试中运行每个测试用例
        t.Run(tc.name, func(t *testing.T) {
            # 创建内存映射文件系统
            fs := afero.NewMemMapFs()
            # 运行测试用例的设置函数，初始化文件系统
            tc.setup(fs)

            # 使用哈希验证文件，并返回验证结果、实际哈希值和可能的错误
            valid, actualHash, err := ValidateByHash(fs, tc.path, tc.hashStr)

            # 断言验证结果是否与预期相符
            assert.Equal(t, tc.valid, valid)
            # 断言实际哈希值是否与预期相符
            assert.Equal(t, tc.actualHash, actualHash)

            # 如果预期有错误
            if tc.err {
                # 断言是否有错误发生
                assert.Error(t, err)
            } else {
                # 断言是否没有错误发生
                assert.NoError(t, err)
            }

            # 如果预期有错误消息
            if tc.errMsg != nil {
                # 断言错误消息是否与预期相符
                assert.Equal(t, tc.errMsg, err)
            }
        })
    }
# 闭合前面的函数定义
```