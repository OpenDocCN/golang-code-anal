# `kubo\tracing\tracing.go`

```
package tracing

import (
    "context" // 导入上下文包
    "fmt" // 导入格式化包

    "github.com/ipfs/boxo/tracing" // 导入跟踪包
    version "github.com/ipfs/kubo" // 导入版本包
    "go.opentelemetry.io/otel" // 导入开放遥测包
    "go.opentelemetry.io/otel/sdk/resource" // 导入资源包
    "go.opentelemetry.io/otel/sdk/trace" // 导入跟踪包
    semconv "go.opentelemetry.io/otel/semconv/v1.4.0" // 导入语义约定包
    traceapi "go.opentelemetry.io/otel/trace" // 导入跟踪 API 包
    "go.opentelemetry.io/otel/trace/noop" // 导入无操作跟踪包
)

// shutdownTracerProvider adds a shutdown method for tracer providers.
//
// Note that this doesn't directly use the provided TracerProvider interface
// to avoid build breaking go-ipfs if new methods are added to it.
type shutdownTracerProvider interface {
    traceapi.TracerProvider // 定义一个接口继承自跟踪 API 包中的 TracerProvider 接口

    Tracer(instrumentationName string, opts ...traceapi.TracerOption) traceapi.Tracer // 定义一个方法用于创建跟踪器
    Shutdown(ctx context.Context) error // 定义一个方法用于关闭跟踪器提供者
}

// noopShutdownTracerProvider adds a no-op Shutdown method to a TracerProvider.
type noopShutdownTracerProvider struct{ traceapi.TracerProvider } // 定义一个结构体，实现了 TracerProvider 接口

func (n *noopShutdownTracerProvider) Shutdown(ctx context.Context) error { return nil } // 实现关闭方法，但是不执行任何操作

// NewTracerProvider creates and configures a TracerProvider.
func NewTracerProvider(ctx context.Context) (shutdownTracerProvider, error) {
    exporters, err := tracing.NewSpanExporters(ctx) // 调用跟踪包中的方法创建跟踪器导出者
    if err != nil {
        return nil, err // 如果出错，返回空和错误
    }
    if len(exporters) == 0 {
        return &noopShutdownTracerProvider{TracerProvider: noop.NewTracerProvider()}, nil // 如果导出者数量为0，返回一个无操作的跟踪器提供者
    }

    options := []trace.TracerProviderOption{} // 定义一个选项数组

    for _, exporter := range exporters { // 遍历导出者
        options = append(options, trace.WithBatcher(exporter)) // 将导出者添加到选项中
    }

    r, err := resource.Merge( // 合并资源
        resource.Default(), // 默认资源
        resource.NewSchemaless( // 新的无模式资源
            semconv.ServiceNameKey.String("Kubo"), // 服务名
            semconv.ServiceVersionKey.String(version.CurrentVersionNumber), // 服务版本
        ),
    )
    if err != nil {
        return nil, err // 如果出错，返回空和错误
    }
    options = append(options, trace.WithResource(r)) // 将资源添加到选项中

    return trace.NewTracerProvider(options...), nil // 返回一个新的跟踪器提供者
}
// Span函数使用标准的IPFS跟踪约定开始一个新的跟踪。
func Span(ctx context.Context, componentName string, spanName string, opts ...traceapi.SpanStartOption) (context.Context, traceapi.Span) {
    // 使用otel包的Tracer方法创建一个跟踪对象，并开始一个新的跟踪
    return otel.Tracer("Kubo").Start(ctx, fmt.Sprintf("%s.%s", componentName, spanName), opts...)
}
```