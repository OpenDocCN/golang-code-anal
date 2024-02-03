# `v2ray-core\infra\control\verify.go`

```go
package control

import (
    "flag"  // 导入 flag 包，用于解析命令行参数
    "github.com/xiaokangwang/VSign/signerVerify"  // 导入签名验证的包
    "os"  // 导入操作系统功能的包
    "v2ray.com/core/common"  // 导入 V2Ray 的公共功能包
)

type VerifyCommand struct{}  // 定义 VerifyCommand 结构体

func (c *VerifyCommand) Name() string {  // 定义 Name 方法，返回命令名称
    return "verify"
}

func (c *VerifyCommand) Description() Description {  // 定义 Description 方法，返回命令描述
    return Description{
        Short: "Verify if a binary is officially signed.",  // 简短描述
        Usage: []string{  // 用法说明
            "v2ctl verify --sig=<sig-file> file...",  // 命令使用说明
            "Verify the file officially signed by V2Ray.",  // 验证文件是否由 V2Ray 官方签名
        },
    }
}

func (c *VerifyCommand) Execute(args []string) error {  // 定义 Execute 方法，执行命令
    fs := flag.NewFlagSet(c.Name(), flag.ContinueOnError)  // 创建命令行参数解析器

    sigFile := fs.String("sig", "", "Path to the signature file")  // 定义 sigFile 命令行参数

    if err := fs.Parse(args); err != nil {  // 解析命令行参数
        return err
    }

    target := fs.Arg(0)  // 获取命令行参数中的目标文件
    if target == "" {  // 如果目标文件为空
        return newError("empty file path.")  // 返回错误信息
    }

    if *sigFile == "" {  // 如果签名文件路径为空
        return newError("empty signature path.")  // 返回错误信息
    }

    sigReader, err := os.Open(os.ExpandEnv(*sigFile))  // 打开签名文件
    if err != nil {  // 如果打开文件出错
        return newError("failed to open file ", *sigFile).Base(err)  // 返回错误信息
    }

    files := fs.Args()  // 获取命令行参数中的文件列表

    err = signerVerify.OutputAndJudge(signerVerify.CheckSignaturesV2Fly(sigReader, files))  // 调用签名验证函数

    if err == nil {  // 如果没有错误
        return nil  // 返回空
    }

    return newError("file is not officially signed by V2Ray").Base(err)  // 返回错误信息
}

func init() {  // 初始化函数
    common.Must(RegisterCommand(&VerifyCommand{}))  // 注册 VerifyCommand 命令
}
```