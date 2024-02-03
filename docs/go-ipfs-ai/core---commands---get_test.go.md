# `kubo\core\commands\get_test.go`

```go
package commands

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "fmt"  // 导入格式化包，用于格式化输出
    "testing"  // 导入测试包，用于编写测试函数

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入自定义命令包，用于处理IPFS命令
)

func TestGetOutputPath(t *testing.T) {
    cases := []struct {  // 定义测试用例结构体切片
        args    []string  // 定义参数切片
        opts    cmds.OptMap  // 定义选项映射
        outPath string  // 定义输出路径字符串
    }{
        {
            args: []string{"/ipns/multiformats.io/"},  // 设置参数
            opts: map[string]interface{}{  // 设置选项
                "output": "takes-precedence",  // 设置输出选项
            },
            outPath: "takes-precedence",  // 设置输出路径
        },
        {
            args: []string{"/ipns/multiformats.io/", "some-other-arg-to-be-ignored"},  // 设置参数
            opts: cmds.OptMap{  // 设置选项
                "output": "takes-precedence",  // 设置输出选项
            },
            outPath: "takes-precedence",  // 设置输出路径
        },
        {
            args:    []string{"/ipns/multiformats.io/"},  // 设置参数
            outPath: "multiformats.io",  // 设置输出路径
            opts:    cmds.OptMap{},  // 设置选项
        },
        {
            args:    []string{"/ipns/multiformats.io/logo.svg/"},  // 设置参数
            outPath: "logo.svg",  // 设置输出路径
            opts:    cmds.OptMap{},  // 设置选项
        },
        {
            args:    []string{"/ipns/multiformats.io", "some-other-arg-to-be-ignored"},  // 设置参数
            outPath: "multiformats.io",  // 设置输出路径
            opts:    cmds.OptMap{},  // 设置选项
        },
    }

    _, err := GetCmd.GetOptions([]string{})  // 获取默认命令选项
    if err != nil {  // 如果出现错误
        t.Fatalf("error getting default command options: %v", err)  // 输出错误信息
    }

    for i, tc := range cases {  // 遍历测试用例
        t.Run(fmt.Sprintf("%s-%d", t.Name(), i), func(t *testing.T) {  // 运行测试函数
            ctx, cancel := context.WithCancel(context.Background())  // 创建上下文
            defer cancel()  // 延迟取消上下文

            req, err := cmds.NewRequest(ctx, []string{}, tc.opts, tc.args, nil, GetCmd)  // 创建命令请求
            if err != nil {  // 如果出现错误
                t.Fatalf("error creating a command request: %v", err)  // 输出错误信息
            }

            if outPath := getOutPath(req); outPath != tc.outPath {  // 获取输出路径并比较
                t.Errorf("expected outPath %s to be %s", outPath, tc.outPath)  // 输出预期输出路径与实际输出路径不一致的错误信息
            }
        })
    }
}
```