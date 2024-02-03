# `grype\grype\event\parsers\parsers.go`

```go
package parsers

import (
    "fmt"

    "github.com/wagoodman/go-partybus"
    "github.com/wagoodman/go-progress"

    "github.com/anchore/grype/grype/event"
    "github.com/anchore/grype/grype/event/monitor"
)

type ErrBadPayload struct {
    Type  partybus.EventType  // 定义错误类型结构体，包含事件类型
    Field string  // 定义错误类型结构体，包含字段名
    Value interface{}  // 定义错误类型结构体，包含值
}

func (e *ErrBadPayload) Error() string {
    return fmt.Sprintf("event='%s' has bad event payload field='%v': '%+v'", string(e.Type), e.Field, e.Value)  // 返回错误信息字符串
}

func newPayloadErr(t partybus.EventType, field string, value interface{}) error {
    return &ErrBadPayload{  // 返回错误类型结构体
        Type:  t,  // 设置事件类型
        Field: field,  // 设置字段名
        Value: value,  // 设置值
    }
}

func checkEventType(actual, expected partybus.EventType) error {
    if actual != expected {  // 检查实际事件类型是否与期望事件类型相同
        return newPayloadErr(expected, "Type", actual)  // 返回错误信息
    }
    return nil
}

func ParseUpdateVulnerabilityDatabase(e partybus.Event) (progress.StagedProgressable, error) {
    if err := checkEventType(e.Type, event.UpdateVulnerabilityDatabase); err != nil {  // 检查事件类型是否为更新漏洞数据库
        return nil, err  // 返回错误信息
    }

    prog, ok := e.Value.(progress.StagedProgressable)  // 尝试将事件值转换为进度可分阶段的类型
    if !ok {
        return nil, newPayloadErr(e.Type, "Value", e.Value)  // 返回错误信息
    }

    return prog, nil  // 返回进度可分阶段的类型
}

func ParseVulnerabilityScanningStarted(e partybus.Event) (*monitor.Matching, error) {
    if err := checkEventType(e.Type, event.VulnerabilityScanningStarted); err != nil {  // 检查事件类型是否为漏洞扫描开始
        return nil, err  // 返回错误信息
    }

    mon, ok := e.Value.(monitor.Matching)  // 尝试将事件值转换为匹配监视类型
    if !ok {
        return nil, newPayloadErr(e.Type, "Value", e.Value)  // 返回错误信息
    }

    return &mon, nil  // 返回匹配监视类型
}

func ParseDatabaseDiffingStarted(e partybus.Event) (*monitor.DBDiff, error) {
    if err := checkEventType(e.Type, event.DatabaseDiffingStarted); err != nil {  // 检查事件类型是否为数据库差异开始
        return nil, err  // 返回错误信息
    }

    mon, ok := e.Value.(monitor.DBDiff)  // 尝试将事件值转换为数据库差异类型
    if !ok {
        return nil, newPayloadErr(e.Type, "Value", e.Value)  // 返回错误信息
    }

    return &mon, nil  // 返回数据库差异类型
}

type UpdateCheck struct {
    New     string  // 定义更新检查结构体，包含新版本字段
    Current string  // 定义更新检查结构体，包含当前版本字段
}
# 解析 CLI 应用程序更新是否可用的事件，返回更新检查结果和可能的错误
func ParseCLIAppUpdateAvailable(e partybus.Event) (*UpdateCheck, error) {
    # 检查事件类型是否为 CLIAppUpdateAvailable
    if err := checkEventType(e.Type, event.CLIAppUpdateAvailable); err != nil {
        return nil, err
    }

    # 尝试将事件值转换为 UpdateCheck 结构
    updateCheck, ok := e.Value.(UpdateCheck)
    if !ok {
        return nil, newPayloadErr(e.Type, "Value", e.Value)
    }

    # 返回更新检查结果和无错误
    return &updateCheck, nil
}

# 解析 CLI 报告事件，返回上下文、报告内容和可能的错误
func ParseCLIReport(e partybus.Event) (string, string, error) {
    # 检查事件类型是否为 CLIReport
    if err := checkEventType(e.Type, event.CLIReport); err != nil {
        return "", "", err
    }

    # 尝试将事件源转换为字符串类型
    context, ok := e.Source.(string)
    if !ok {
        # 如果转换失败，将上下文置为空字符串（可选）
        context = ""
    }

    # 尝试将事件值转换为字符串类型的报告内容
    report, ok := e.Value.(string)
    if !ok {
        return "", "", newPayloadErr(e.Type, "Value", e.Value)
    }

    # 返回上下文、报告内容和无错误
    return context, report, nil
}

# 解析 CLI 通知事件，返回上下文、通知内容和可能的错误
func ParseCLINotification(e partybus.Event) (string, string, error) {
    # 检查事件类型是否为 CLINotification
    if err := checkEventType(e.Type, event.CLINotification); err != nil {
        return "", "", err
    }

    # 尝试将事件源转换为字符串类型
    context, ok := e.Source.(string)
    if !ok {
        # 如果转换失败，将上下文置为空字符串（可选）
        context = ""
    }

    # 尝试将事件值转换为字符串类型的通知内容
    notification, ok := e.Value.(string)
    if !ok {
        return "", "", newPayloadErr(e.Type, "Value", e.Value)
    }

    # 返回上下文、通知内容和无错误
    return context, notification, nil
}
```