# `kubebench-aquasecurity\cmd\securityHub.go`

```
package cmd

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "log" // 导入 log 包，用于日志记录

    "github.com/aquasecurity/kube-bench/internal/findings" // 导入 findings 包
    "github.com/aws/aws-sdk-go/aws" // 导入 aws 包
    "github.com/aws/aws-sdk-go/aws/session" // 导入 session 包
    "github.com/aws/aws-sdk-go/service/securityhub" // 导入 securityhub 包
    "github.com/spf13/viper" // 导入 viper 包
)

//REGION ...
const REGION = "AWS_REGION" // 定义常量 REGION，用于存储 AWS_REGION

func writeFinding(in []*securityhub.AwsSecurityFinding) error {
    r := viper.GetString(REGION) // 从 viper 中获取 AWS_REGION 的值
    if len(r) == 0 { // 如果 r 的长度为 0
        return fmt.Errorf("%s not set", REGION) // 返回错误信息
    }
    sess, err := session.NewSession(&aws.Config{ // 创建 AWS 会话
        Region: aws.String(r)}, // 设置会话的区域
    )
    if err != nil { // 如果有错误
        return err // 返回错误
    }
    svc := securityhub.New(sess) // 创建 securityhub 服务对象
    p := findings.New(svc) // 创建 findings 对象
    out, perr := p.PublishFinding(in) // 发布安全发现
    print(out) // 打印输出
    return perr // 返回错误
}

func print(out *findings.PublisherOutput) {
    if out.SuccessCount > 0 { // 如果成功计数大于 0
        log.Printf("Number of findings that were successfully imported:%v\n", out.SuccessCount) // 记录成功导入的发现数量
    }
    if out.FailedCount > 0 { // 如果失败计数大于 0
        log.Printf("Number of findings that failed to import:%v\n", out.FailedCount) // 记录导入失败的发现数量
        for _, f := range out.FailedFindings { // 遍历失败的发现
            log.Printf("ID:%s", *f.Id) // 记录发现的 ID
            log.Printf("Message:%s", *f.ErrorMessage) // 记录错误消息
            log.Printf("Error Code:%s", *f.ErrorCode) // 记录错误代码
        }
    }
}
```