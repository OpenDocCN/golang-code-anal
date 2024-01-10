# `grype\grype\db\metadata_test.go`

```
package db

import (
    "testing"  // 导入测试包
    "time"     // 导入时间包

    "github.com/go-test/deep"  // 导入深度比较包
    "github.com/spf13/afero"   // 导入文件系统操作包
)

func TestMetadataParse(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        fixture  string      // 测试用例的文件路径
        expected *Metadata   // 期望的元数据
        err      bool        // 是否期望出现错误
    }{
        {
            fixture: "test-fixtures/metadata-gocase",  // 测试用例的文件路径
            expected: &Metadata{  // 期望的元数据
                Built:    time.Date(2020, 06, 15, 14, 02, 36, 0, time.UTC),  // 构建时间
                Version:  2,  // 版本号
                Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",  // 校验和
            },
        },
        {
            fixture: "test-fixtures/metadata-edt-timezone",  // 测试用例的文件路径
            expected: &Metadata{  // 期望的元数据
                Built:    time.Date(2020, 06, 15, 18, 02, 36, 0, time.UTC),  // 构建时间
                Version:  2,  // 版本号
                Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",  // 校验和
            },
        },
        {
            fixture: "/dev/null/impossible",  // 测试用例的文件路径
            err:     true,  // 期望出现错误
        },
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.fixture, func(t *testing.T) {  // 运行测试用例
            metadata, err := NewMetadataFromDir(afero.NewOsFs(), test.fixture)  // 从目录中获取元数据
            if err != nil && !test.err {  // 如果出现错误且不符合期望
                t.Fatalf("failed to get metadata: %+v", err)  // 输出错误信息
            } else if err == nil && test.err {  // 如果没有出现错误但符合期望
                t.Fatalf("expected error but got none")  // 输出错误信息
            } else if metadata == nil && test.expected != nil {  // 如果未找到元数据且期望不为空
                t.Fatalf("metadata not found: %+v", test.fixture)  // 输出错误信息
            }

            if metadata != nil && test.expected != nil {  // 如果元数据和期望都不为空
                for _, diff := range deep.Equal(*metadata, *test.expected) {  // 深度比较元数据和期望
                    t.Errorf("metadata difference: %s", diff)  // 输出差异信息
                }
            }
        })
    }
}

func TestMetadataIsSupercededBy(t *testing.T) {
    tests := []struct {  // 定义测试用例结构体切片
        name                string          // 测试用例名称
        current             *Metadata       // 当前元数据
        update              *ListingEntry   // 更新的列表条目
        expectedToSupercede bool            // 是否期望被取代
    # 定义一个测试用例切片，包含了不同情况下的测试数据
    tests := []struct {
        name:                "prefer updated versions over later dates",  # 测试名称，表示更喜欢更新的版本而不是后来的日期
        expectedToSupercede: true,  # 预期结果为true，表示期望更新的版本能够取代当前版本
        current: &Metadata{  # 当前版本的元数据
            Built:   time.Date(2020, 06, 15, 14, 02, 36, 0, time.UTC),  # 当前版本的构建时间
            Version: 2,  # 当前版本号为2
        },
        update: &ListingEntry{  # 更新版本的列表条目
            Built:   time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),  # 更新版本的构建时间
            Version: 3,  # 更新版本号为3
        },
    },
    {
        name:                "prefer later dates when version is the same",  # 测试名称，表示当版本号相同时更喜欢后来的日期
        expectedToSupercede: false,  # 预期结果为false，表示不期望后来的日期能够取代当前版本
        current: &Metadata{  # 当前版本的元数据
            Built:   time.Date(2020, 06, 15, 14, 02, 36, 0, time.UTC),  # 当前版本的构建时间
            Version: 1,  # 当前版本号为1
        },
        update: &ListingEntry{  # 更新版本的列表条目
            Built:   time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),  # 更新版本的构建时间
            Version: 1,  # 更新版本号为1
        },
    },
    {
        name:                "prefer something over nothing",  # 测试名称，表示更喜欢有更新版本而不是没有当前版本
        expectedToSupercede: true,  # 预期结果为true，表示期望有更新版本能够取代当前版本
        current:             nil,  # 当前版本为空
        update: &ListingEntry{  # 更新版本的列表条目
            Built:   time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),  # 更新版本的构建时间
            Version: 1,  # 更新版本号为1
        },
    }

    # 遍历测试用例切片，对每个测试用例进行测试
    for _, test := range tests {
        # 使用测试名称创建子测试，执行测试函数
        t.Run(test.name, func(t *testing.T) {
            # 调用被测试的函数，获取实际结果
            actual := test.current.IsSupersededBy(test.update)

            # 检查实际结果是否符合预期结果，如果不符合则输出错误信息
            if test.expectedToSupercede != actual {
                t.Errorf("failed supercede assertion: got %+v", actual)
            }
        })
    }
# 闭合前面的函数定义
```