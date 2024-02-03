# `kubo\core\node\libp2p\libp2p_test.go`

```go
package libp2p

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "strconv"  // 导入 strconv 包，用于字符串和数字之间的转换
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    ma "github.com/multiformats/go-multiaddr"  // 导入 multiaddr 包，并重命名为 ma

    "github.com/stretchr/testify/require"  // 导入 require 包，用于测试断言
)

func TestPrioritize(t *testing.T) {
    // The option is encoded into the port number of a TCP multiaddr.
    // By extracting the port numbers obtained from the applied option, we can make sure that
    // prioritization sorted the options correctly.
    // 选项被编码到 TCP multiaddr 的端口号中。
    // 通过提取应用选项获得的端口号，我们可以确保优先级正确排序了选项。
    newOption := func(num int) libp2p.Option {  // 定义一个函数，返回一个 libp2p.Option 类型的函数
        return func(cfg *libp2p.Config) error {  // 返回一个函数，该函数接受 libp2p.Config 类型的参数，并返回 error
            cfg.ListenAddrs = append(cfg.ListenAddrs, ma.StringCast(fmt.Sprintf("/ip4/127.0.0.1/tcp/%d", num)))  // 将格式化后的地址添加到 ListenAddrs 中
            return nil  // 返回空的 error
        }
    }

    extractNums := func(cfg *libp2p.Config) []int {  // 定义一个函数，接受 libp2p.Config 类型的参数，并返回 int 切片
        addrs := cfg.ListenAddrs  // 获取 cfg 的 ListenAddrs
        nums := make([]int, 0, len(addrs))  // 创建一个初始长度为 0，容量为 addrs 长度的 int 切片
        for _, addr := range addrs {  // 遍历 addrs
            _, comp := ma.SplitLast(addr)  // 使用 multiaddr 包的 SplitLast 方法获取地址的最后一部分
            num, err := strconv.Atoi(comp.Value())  // 将最后一部分转换为数字
            require.NoError(t, err)  // 断言 err 为 nil
            nums = append(nums, num)  // 将数字添加到 nums 中
        }
        return nums  // 返回 nums
    }

    t.Run("using default priorities", func(t *testing.T) {  // 运行测试函数，使用默认优先级
        opts := []priorityOption{  // 创建 priorityOption 类型的切片
            {defaultPriority: 200, opt: newOption(200)},  // 设置默认优先级为 200，应用 newOption(200) 的选项
            {defaultPriority: 1, opt: newOption(1)},  // 设置默认优先级为 1，应用 newOption(1) 的选项
            {defaultPriority: 300, opt: newOption(300)},  // 设置默认优先级为 300，应用 newOption(300) 的选项
        }
        var cfg libp2p.Config  // 声明一个 libp2p.Config 类型的变量
        require.NoError(t, prioritizeOptions(opts)(&cfg))  // 断言优先级选项应用到 cfg 上没有错误
        require.Equal(t, extractNums(&cfg), []int{1, 200, 300})  // 断言提取的数字与给定的切片相等
    })

    t.Run("using custom priorities", func(t *testing.T) {  // 运行测试函数，使用自定义优先级
        opts := []priorityOption{  // 创建 priorityOption 类型的切片
            {defaultPriority: 200, priority: 1, opt: newOption(1)},  // 设置默认优先级为 200，优先级为 1，应用 newOption(1) 的选项
            {defaultPriority: 1, priority: 300, opt: newOption(300)},  // 设置默认优先级为 1，优先级为 300，应用 newOption(300) 的选项
            {defaultPriority: 300, priority: 20, opt: newOption(20)},  // 设置默认优先级为 300，优先级为 20，应用 newOption(20) 的选项
        }
        var cfg libp2p.Config  // 声明一个 libp2p.Config 类型的变量
        require.NoError(t, prioritizeOptions(opts)(&cfg))  // 断言优先级选项应用到 cfg 上没有错误
        require.Equal(t, extractNums(&cfg), []int{1, 20, 300})  // 断言提取的数字与给定的切片相等
    })
}
```