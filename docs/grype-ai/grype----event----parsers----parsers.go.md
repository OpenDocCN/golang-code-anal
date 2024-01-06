# `grype\grype\event\parsers\parsers.go`

```
// 声明 parsers 包
package parsers

// 引入 fmt 包
import (
	"fmt"

	// 引入 go-partybus 包
	"github.com/wagoodman/go-partybus"
	// 引入 go-progress 包
	"github.com/wagoodman/go-progress"

	// 引入 grype 包下的 event 模块
	"github.com/anchore/grype/grype/event"
	// 引入 grype 包下的 event/monitor 模块
	"github.com/anchore/grype/grype/event/monitor"
)

// 定义 ErrBadPayload 结构体
type ErrBadPayload struct {
	Type  partybus.EventType // 定义 Type 字段为 partybus.EventType 类型
	Field string // 定义 Field 字段为 string 类型
	Value interface{} // 定义 Value 字段为 interface{} 类型
}

// 实现 Error 方法，返回错误信息
func (e *ErrBadPayload) Error() string {
	return fmt.Sprintf("event='%s' has bad event payload field='%v': '%+v'", string(e.Type), e.Field, e.Value)
}
}

// 创建新的负载错误，返回错误对象
func newPayloadErr(t partybus.EventType, field string, value interface{}) error {
	return &ErrBadPayload{
		Type:  t,
		Field: field,
		Value: value,
	}
}

// 检查事件类型是否符合预期，返回错误对象
func checkEventType(actual, expected partybus.EventType) error {
	if actual != expected {
		return newPayloadErr(expected, "Type", actual)
	}
	return nil
}

// 解析更新漏洞数据库事件，返回进度对象和错误对象
func ParseUpdateVulnerabilityDatabase(e partybus.Event) (progress.StagedProgressable, error) {
	if err := checkEventType(e.Type, event.UpdateVulnerabilityDatabase); err != nil {
		return nil, err
```

// 从事件值中获取进度对象，如果类型不匹配则返回错误
prog, ok := e.Value.(progress.StagedProgressable)
if !ok {
    return nil, newPayloadErr(e.Type, "Value", e.Value)
}

// 返回进度对象和空错误
return prog, nil
}

// 解析漏洞扫描开始事件
func ParseVulnerabilityScanningStarted(e partybus.Event) (*monitor.Matching, error) {
// 检查事件类型是否为漏洞扫描开始事件，如果不是则返回错误
if err := checkEventType(e.Type, event.VulnerabilityScanningStarted); err != nil {
    return nil, err
}

// 从事件值中获取匹配对象，如果类型不匹配则返回错误
mon, ok := e.Value.(monitor.Matching)
if !ok {
    return nil, newPayloadErr(e.Type, "Value", e.Value)
}
# 返回监视器和空错误
return &mon, nil
```

```
# 解析数据库差异开始事件，返回数据库差异和错误
func ParseDatabaseDiffingStarted(e partybus.Event) (*monitor.DBDiff, error) {
    # 检查事件类型是否为数据库差异开始事件，如果不是则返回错误
    if err := checkEventType(e.Type, event.DatabaseDiffingStarted); err != nil {
        return nil, err
    }

    # 将事件的值转换为监视器数据库差异类型，如果转换失败则返回错误
    mon, ok := e.Value.(monitor.DBDiff)
    if !ok {
        return nil, newPayloadErr(e.Type, "Value", e.Value)
    }

    # 返回监视器数据库差异和空错误
    return &mon, nil
}
```

```
# 定义更新检查结构体
type UpdateCheck struct {
    New     string
    Current string
}
# 解析 CLI 应用更新可用事件，返回更新检查结果和可能的错误
func ParseCLIAppUpdateAvailable(e partybus.Event) (*UpdateCheck, error) {
    # 检查事件类型是否为 CLIAppUpdateAvailable，如果不是则返回错误
    if err := checkEventType(e.Type, event.CLIAppUpdateAvailable); err != nil {
        return nil, err
    }

    # 将事件值转换为更新检查对象
    updateCheck, ok := e.Value.(UpdateCheck)
    if !ok {
        return nil, newPayloadErr(e.Type, "Value", e.Value)
    }

    return &updateCheck, nil
}

# 解析 CLI 报告事件，返回上下文和报告内容以及可能的错误
func ParseCLIReport(e partybus.Event) (string, string, error) {
    # 检查事件类型是否为 CLIReport，如果不是则返回空字符串和错误
    if err := checkEventType(e.Type, event.CLIReport); err != nil {
        return "", "", err
    }

    # 将事件源转换为字符串类型的上下文
    context, ok := e.Source.(string)
	if !ok {
		// 如果条件不满足，将上下文设置为空字符串
		context = ""
	}

	// 将事件的值转换为字符串类型
	report, ok := e.Value.(string)
	if !ok {
		// 如果值的类型不是字符串，返回错误
		return "", "", newPayloadErr(e.Type, "Value", e.Value)
	}

	// 返回上下文、报告和空错误
	return context, report, nil
}

func ParseCLINotification(e partybus.Event) (string, string, error) {
	// 检查事件类型是否为CLINotification，如果不是则返回错误
	if err := checkEventType(e.Type, event.CLINotification); err != nil {
		return "", "", err
	}

	// 将事件的来源转换为字符串类型
	context, ok := e.Source.(string)
	if !ok {
		// 如果来源的类型不是字符串，返回错误
// 初始化一个空的字符串变量，这是可选的操作
context = ""
}

// 从接口值中获取字符串类型的通知内容
notification, ok := e.Value.(string)
// 如果获取失败，返回错误信息
if !ok {
    return "", "", newPayloadErr(e.Type, "Value", e.Value)
}

// 返回上下文、通知内容和空错误
return context, notification, nil
```