# `grype\grype\db\listing_test.go`

```
package db

import (
    "net/url"  // 导入网络 URL 包
    "testing"  // 导入测试包
    "time"     // 导入时间包

    "github.com/go-test/deep"  // 导入深度比较包
    "github.com/spf13/afero"   // 导入文件系统操作包
)

func mustUrl(u *url.URL, err error) *url.URL {
    if err != nil {
        panic(err)  // 如果有错误则抛出异常
    }
    return u  // 返回 URL 对象
}

func TestNewListingFromPath(t *testing.T) {
    tests := []struct {
        fixture  string    // 测试用例的文件路径
        expected Listing  // 期望的列表对象
        err      bool      // 是否期望出现错误
    }

    for _, test := range tests {  // 遍历测试用例
        t.Run(test.fixture, func(t *testing.T) {
            listing, err := NewListingFromFile(afero.NewOsFs(), test.fixture)  // 从文件创建列表对象
            if err != nil && !test.err {
                t.Fatalf("failed to get metadata: %+v", err)  // 如果出现错误且不符合期望则输出错误信息
            } else if err == nil && test.err {
                t.Fatalf("expected errer but got none")  // 如果期望出现错误但没有则输出错误信息
            }

            for _, diff := range deep.Equal(listing, test.expected) {
                t.Errorf("listing difference: %s", diff)  // 输出列表对象的差异信息
            }
        })
    }
}

func TestListingBestUpdate(t *testing.T) {
    tests := []struct {
        fixture    string  // 测试用例的文件路径
        constraint int     // 约束条件
        expected   *ListingEntry  // 期望的列表条目对象
    # 定义测试用例，包括输入的fixture、约束constraint和期望的输出expected
    {
        fixture:    "test-fixtures/listing.json",
        constraint: 2,
        expected: &ListingEntry{
            Built:    time.Date(2020, 06, 13, 17, 13, 13, 0, time.UTC),
            URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db-v1.1.0+2020-6-13.tar.gz")),
            Version:  2,
            Checksum: "sha256:dcd6a285c839a7c65939e20c251202912f64826be68609dfc6e48df7f853ddc8",
        },
    },
    {
        fixture:    "test-fixtures/listing.json",
        constraint: 1,
        expected: &ListingEntry{
            Built:    time.Date(2020, 06, 12, 16, 12, 12, 0, time.UTC),
            URL:      mustUrl(url.Parse("http://localhost:5000/vulnerability-db-v0.2.0+2020-6-12.tar.gz")),
            Version:  1,
            Checksum: "sha256:e20c251202948df7f853ddc812f64826bdcd6a285c839a7c65939e68609dfc6e",
        },
    }

    # 遍历测试用例，执行测试
    for _, test := range tests {
        t.Run(test.fixture, func(t *testing.T) {
            # 从文件中读取列表信息
            listing, err := NewListingFromFile(afero.NewOsFs(), test.fixture)
            if err != nil {
                t.Fatalf("failed to get metadata: %+v", err)
            }

            # 获取最佳更新候选
            actual := listing.BestUpdate(test.constraint)
            # 检查实际输出和期望输出是否匹配
            if actual == nil && test.expected != nil || actual != nil && test.expected == nil {
                t.Fatalf("mismatched best candidate expectations")
            }

            # 检查实际输出和期望输出的差异
            for _, diff := range deep.Equal(actual, test.expected) {
                t.Errorf("listing entry difference: %s", diff)
            }
        })
    }
# 闭合前面的函数定义
```