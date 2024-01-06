# `kubebench-aquasecurity\cmd\securityHub.go`

```
// 导入所需的包
package cmd

import (
	"fmt" // 格式化输出
	"log" // 日志记录

	"github.com/aquasecurity/kube-bench/internal/findings" // 导入自定义包
	"github.com/aws/aws-sdk-go/aws" // AWS SDK
	"github.com/aws/aws-sdk-go/aws/session" // AWS 会话管理
	"github.com/aws/aws-sdk-go/service/securityhub" // AWS Security Hub
	"github.com/spf13/viper" // 用于读取配置文件
)

//REGION ... 声明常量 REGION
const REGION = "AWS_REGION"

// 写入安全发现结果到 AWS Security Hub
func writeFinding(in []*securityhub.AwsSecurityFinding) error {
	// 从配置文件中获取 AWS_REGION
	r := viper.GetString(REGION)
	// 如果未设置 AWS_REGION，则返回错误
	if len(r) == 0 {
		return fmt.Errorf("%s not set", REGION)
// 结束当前函数
	}
	// 创建新的 AWS 会话
	sess, err := session.NewSession(&aws.Config{
		Region: aws.String(r)},
	)
	// 如果创建会话出现错误，返回错误信息
	if err != nil {
		return err
	}
	// 创建 Security Hub 服务对象
	svc := securityhub.New(sess)
	// 创建 Security Hub 发现对象
	p := findings.New(svc)
	// 发布发现结果
	out, perr := p.PublishFinding(in)
	// 打印发布结果
	print(out)
	// 返回发布错误信息
	return perr
}

// 打印发布结果
func print(out *findings.PublisherOutput) {
	// 如果成功导入的发现数量大于0，打印成功导入的数量
	if out.SuccessCount > 0 {
		log.Printf("Number of findings that were successfully imported:%v\n", out.SuccessCount)
	}
	// 如果导入失败的发现数量大于0，打印导入失败的数量
	if out.FailedCount > 0 {
		log.Printf("Number of findings that failed to import:%v\n", out.FailedCount)
# 遍历 out.FailedFindings 中的每个元素，用 _ 作为索引变量名，f 作为元素变量名
for _, f := range out.FailedFindings:
    # 打印 f.Id 的值
    log.Printf("ID:%s", *f.Id)
    # 打印 f.ErrorMessage 的值
    log.Printf("Message:%s", *f.ErrorMessage)
    # 打印 f.ErrorCode 的值
    log.Printf("Error Code:%s", *f.ErrorCode)
# 结束 for 循环
```