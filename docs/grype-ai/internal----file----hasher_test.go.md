# `grype\internal\file\hasher_test.go`

```
package file

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/spf13/afero"  // 导入 afero 包，用于文件系统操作
	"github.com/stretchr/testify/assert"  // 导入 assert 包，用于断言测试结果
)

func TestValidateByHash(t *testing.T) {  // 定义测试函数 TestValidateByHash
	testsCases := []struct {  // 定义测试用例结构体
		name, path, hashStr, actualHash string  // 定义测试用例的名称、路径、哈希值、实际哈希值
		setup                           func(fs afero.Fs)  // 定义测试用例的设置函数
		valid                           bool  // 定义测试用例的有效性
		err                             bool  // 定义测试用例的错误
		errMsg                          error  // 定义测试用例的错误信息
	}{
		{
			name:    "Valid SHA256 hash",  // 设置测试用例的名称
		{
			// 测试用例名称
			name:    "Valid SHA256 hash",
			// 文件路径
			path:    "test.txt",
			// 期望的 SHA256 哈希值
			hashStr: "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
			// 设置测试环境的函数
			setup: func(fs afero.Fs) {
				// 在文件系统中写入测试文件
				afero.WriteFile(fs, "test.txt", []byte("test"), 0644)
			},
			// 实际计算得到的哈希值
			actualHash: "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
			// 期望结果为有效
			valid:      true,
			// 期望没有错误发生
			err:        false,
		},
		{
			// 测试用例名称
			name:    "Invalid SHA256 hash",
			// 文件路径
			path:    "test.txt",
			// 期望的无效 SHA256 哈希值
			hashStr: "sha256:deadbeef",
			// 设置测试环境的函数
			setup: func(fs afero.Fs) {
				// 在文件系统中写入测试文件
				afero.WriteFile(fs, "test.txt", []byte("test"), 0644)
			},
			// 实际计算得到的哈希值
			actualHash: "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
			// 期望结果为无效
			valid:      false,
			// 期望没有错误发生
			err:        false,
		},
# 定义一个测试用例，测试不支持的哈希函数
{
    # 测试用例的名称
    name:    "Unsupported hash function",
    # 文件路径
    path:    "test.txt",
    # 哈希字符串
    hashStr: "md5:deadbeef",
    # 设置测试环境的函数
    setup: func(fs afero.Fs) {
        afero.WriteFile(fs, "test.txt", []byte("test"), 0644)
    },
    # 实际哈希值
    actualHash: "",
    # 是否有效
    valid:      false,
    # 是否出错
    err:        true,
    # 错误信息
    errMsg:     fmt.Errorf("hasher not supported or specified (given: md5:deadbeef)"),
},
# 定义一个测试用例，测试文件不存在
{
    # 测试用例的名称
    name:       "File does not exist",
    # 文件路径
    path:       "nonexistent.txt",
    # 哈希字符串
    hashStr:    "sha256:deadbeef",
    # 设置测试环境的函数
    setup:      func(fs afero.Fs) {},
    # 是否有效
    valid:      false,
    # 实际哈希值
    actualHash: "",
    # 是否出错
    err:        true,
}
		},
	}

	for _, tc := range testsCases {
		// 遍历测试用例
		t.Run(tc.name, func(t *testing.T) {
			// 使用测试用例的名称创建子测试
			fs := afero.NewMemMapFs()
			// 创建内存文件系统
			tc.setup(fs)
			// 使用测试用例设置文件系统

			// 验证文件的哈希值是否匹配
			valid, actualHash, err := ValidateByHash(fs, tc.path, tc.hashStr)

			// 断言验证结果是否与预期一致
			assert.Equal(t, tc.valid, valid)
			assert.Equal(t, tc.actualHash, actualHash)

			// 如果预期出现错误，则断言实际是否有错误
			if tc.err {
				assert.Error(t, err)
			} else {
				assert.NoError(t, err)
			}

			// 如果预期有错误消息，则进行相应断言
			if tc.errMsg != nil {
# 使用断言函数Equal来比较实际值和期望值是否相等，如果不相等则输出错误信息
assert.Equal(t, tc.errMsg, err)
# 结束当前测试用例
}
# 结束当前测试函数
})
# 结束当前测试循环
}
```