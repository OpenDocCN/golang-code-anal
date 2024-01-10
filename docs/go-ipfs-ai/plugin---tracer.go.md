# `kubo\plugin\tracer.go`

```
// 导入 plugin 包和 opentracing-go 包
package plugin

import (
    "github.com/opentracing/opentracing-go"
)

// PluginTracer 是一个接口，可以实现以添加跟踪器
type PluginTracer interface {
    // 继承 Plugin 接口
    Plugin

    // InitTracer 方法用于初始化跟踪器，返回 opentracing.Tracer 对象和错误信息
    InitTracer() (opentracing.Tracer, error)
}
```