# `kubebench-aquasecurity\internal\findings\publisher_test.go`

```
package findings

import (
    "testing"

    "github.com/aws/aws-sdk-go/service/securityhub"
    "github.com/aws/aws-sdk-go/service/securityhub/securityhubiface"
)

// Define a mock struct to be used in your unit tests of myFunc.
// 定义一个用于在 myFunc 单元测试中使用的模拟结构体
type MockSHClient struct {
    securityhubiface.SecurityHubAPI
    Batches         int
    NumberOfFinding int
}

func NewMockSHClient() *MockSHClient {
    return &MockSHClient{}
}

func (m *MockSHClient) BatchImportFindings(input *securityhub.BatchImportFindingsInput) (*securityhub.BatchImportFindingsOutput, error) {
    o := securityhub.BatchImportFindingsOutput{}
    m.Batches++
    m.NumberOfFinding = len(input.Findings)
    return &o, nil
}

func TestPublisher_publishFinding(t *testing.T) {
    type fields struct {
        client *MockSHClient
    }
    type args struct {
        finding []*securityhub.AwsSecurityFinding
    }
    tests := []struct {
        name             string
        fields           fields
        args             args
        wantBatchCount   int
        wantFindingCount int
    }{
        {"Test single finding", fields{NewMockSHClient()}, args{makeFindings(1)}, 1, 1},
        {"Test 150 finding should return 2 batches", fields{NewMockSHClient()}, args{makeFindings(150)}, 2, 150},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            p := New(tt.fields.client)
            p.PublishFinding(tt.args.finding)
            if tt.fields.client.NumberOfFinding != tt.wantFindingCount {
                t.Errorf("Publisher.publishFinding() want = %v, got %v", tt.wantFindingCount, tt.fields.client.NumberOfFinding)
            }
            if tt.fields.client.Batches != tt.wantBatchCount {
                t.Errorf("Publisher.publishFinding() want = %v, got %v", tt.wantBatchCount, tt.fields.client.Batches)
            }
        })
    }
}

func makeFindings(count int) []*securityhub.AwsSecurityFinding {
    var findings []*securityhub.AwsSecurityFinding
    # 使用循环遍历 count 次
    for i := 0; i < count; i++ {
        # 创建一个新的 AwsSecurityFinding 对象
        t := securityhub.AwsSecurityFinding{}
        # 将新对象添加到 findings 列表中
        findings = append(findings, &t)
    }
    # 返回 findings 列表
    return findings
# 闭合前面的函数定义
```