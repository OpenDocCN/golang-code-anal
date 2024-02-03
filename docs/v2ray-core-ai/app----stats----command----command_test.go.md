# `v2ray-core\app\stats\command\command_test.go`

```go
package command_test

import (
    "context"  // 导入 context 包，用于处理请求的上下文
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入 cmp 包，用于比较数据
    "github.com/google/go-cmp/cmp/cmpopts"  // 导入 cmpopts 包，用于定制比较选项

    "v2ray.com/core/app/stats"  // 导入 stats 包，用于处理统计数据
    . "v2ray.com/core/app/stats/command"  // 导入 stats/command 包，并将其中的函数和类型导入到当前命名空间
    "v2ray.com/core/common"  // 导入 common 包，用于处理通用功能
)

func TestGetStats(t *testing.T) {
    m, err := stats.NewManager(context.Background(), &stats.Config{})  // 创建一个统计数据管理器
    common.Must(err)  // 检查错误，如果有错误则终止程序

    sc, err := m.RegisterCounter("test_counter")  // 注册一个名为 test_counter 的计数器
    common.Must(err)  // 检查错误，如果有错误则终止程序

    sc.Set(1)  // 设置计数器的值为 1

    s := NewStatsServer(m)  // 创建一个统计数据服务器

    testCases := []struct {
        name  string
        reset bool
        value int64
        err   bool
    }{
        {
            name: "counterNotExist",
            err:  true,
        },
        {
            name:  "test_counter",
            reset: true,
            value: 1,
        },
        {
            name:  "test_counter",
            value: 0,
        },
    }
    for _, tc := range testCases {  // 遍历测试用例
        resp, err := s.GetStats(context.Background(), &GetStatsRequest{  // 获取统计数据
            Name:   tc.name,
            Reset_: tc.reset,
        })
        if tc.err {  // 如果测试用例中有错误
            if err == nil {  // 如果没有返回错误
                t.Error("nil error: ", tc.name)  // 输出错误信息
            }
        } else {
            common.Must(err)  // 检查错误，如果有错误则终止程序
            if r := cmp.Diff(resp.Stat, &Stat{Name: tc.name, Value: tc.value}, cmpopts.IgnoreUnexported(Stat{})); r != "" {  // 比较返回的统计数据和预期值
                t.Error(r)  // 输出错误信息
            }
        }
    }
}

func TestQueryStats(t *testing.T) {
    m, err := stats.NewManager(context.Background(), &stats.Config{})  // 创建一个统计数据管理器
    common.Must(err)  // 检查错误，如果有错误则终止程序

    sc1, err := m.RegisterCounter("test_counter")  // 注册一个名为 test_counter 的计数器
    common.Must(err)  // 检查错误，如果有错误则终止程序
    sc1.Set(1)  // 设置计数器的值为 1

    sc2, err := m.RegisterCounter("test_counter_2")  // 注册一个名为 test_counter_2 的计数器
    common.Must(err)  // 检查错误，如果有错误则终止程序
    sc2.Set(2)  // 设置计数器的值为 2

    sc3, err := m.RegisterCounter("test_counter_3")  // 注册一个名为 test_counter_3 的计数器
    common.Must(err)  // 检查错误，如果有错误则终止程序
    sc3.Set(3)  // 设置计数器的值为 3

    s := NewStatsServer(m)  // 创建一个统计数据服务器
    resp, err := s.QueryStats(context.Background(), &QueryStatsRequest{  // 查询统计数据
        Pattern: "counter_",  // 查询符合指定模式的统计数据
    })
    common.Must(err)  // 检查错误，如果有错误则终止程序
    # 使用 cmp.Diff 函数比较 resp.Stat 和 []*Stat{} 两个切片的差异，并将结果赋值给 r
    if r := cmp.Diff(resp.Stat, []*Stat{
        {Name: "test_counter_2", Value: 2},
        {Name: "test_counter_3", Value: 3},
    }, 
    # 使用 cmpopts.SortSlices 函数对切片进行排序，按照 Name 字段的字典序排序
    cmpopts.SortSlices(func(s1, s2 *Stat) bool { return s1.Name < s2.Name }),
    # 使用 cmpopts.IgnoreUnexported 函数忽略未导出的字段
    cmpopts.IgnoreUnexported(Stat{})); 
    # 如果 r 不为空，表示两个切片有差异，输出错误信息
    r != "" {
        t.Error(r)
    }
# 闭合前面的函数定义
```