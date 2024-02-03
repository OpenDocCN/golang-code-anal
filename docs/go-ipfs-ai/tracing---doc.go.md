# `kubo\tracing\doc.go`

```go
// Package tracing contains the tracing logic for go-ipfs, including configuring the tracer and
// helping keep consistent naming conventions across the stack.
//
// NOTE: Tracing is currently experimental. Span names may change unexpectedly, spans may be removed,
// and backwards-incompatible changes may be made to tracing configuration, options, and defaults.
//
// Tracing is configured through environment variables, as consistent with the OpenTelemetry spec as possible:
//
// https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/sdk-environment-variables.md
//
// OTEL_TRACES_EXPORTER: a comma-separated list of exporters:
//   - otlp
//   - zipkin
//   - file
//
// Different exporters have their own set of environment variables, depending on the exporter. These are typically
// standard environment variables. Some common ones:
//
// OTLP HTTP/gRPC:
//
//   - OTEL_EXPORTER_OTLP_PROTOCOL
//     one of [grpc, http/protobuf]
//     default: grpc
//   - OTEL_EXPORTER_OTLP_ENDPOINT
//   - OTEL_EXPORTER_OTLP_CERTIFICATE
//   - OTEL_EXPORTER_OTLP_HEADERS
//   - OTEL_EXPORTER_OTLP_COMPRESSION
//   - OTEL_EXPORTER_OTLP_TIMEOUT
//
// Zipkin:
//
//   - OTEL_EXPORTER_ZIPKIN_ENDPOINT
//
// File:
//
//   - OTEL_EXPORTER_FILE_PATH
//     file path to write JSON traces
//     default: `$PWD/traces.json`
//
// For example, if you run a local IPFS daemon, you can use the jaegertracing/all-in-one Docker image to run
// a full Jaeger stack and configure Kubo to publish traces to it:
//
//    docker run -d --rm --name jaeger \
//     -e COLLECTOR_OTLP_ENABLED=true \
//     -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
//     -p 5775:5775/udp \
//     -p 6831:6831/udp \
//     -p 6832:6832/udp \
//     -p 5778:5778 \
//     -p 16686:16686 \
//     -p 14250:14250 \
//     -p 14268:14268 \
//     -p 14269:14269 \
//     -p 4317:4317 \
//     -p 4318:4318 \
//     -p 9411:9411 \
//     jaegertracing/all-in-one
// 设置环境变量 OTEL_EXPORTER_OTLP_INSECURE 为 true，OTEL_TRACES_EXPORTER 为 otlp，然后启动 ipfs 守护进程并初始化
// 在这个示例中，Jaeger UI 可以在 http://localhost:16686 上访问

// Span 名称遵循 <组件>.<Span> 的约定，一些示例：
//   - 组件=Gateway + span=Request -> Gateway.Request
//   - 组件=CoreAPI.PinAPI + span=Verify.CheckPin -> CoreAPI.PinAPI.Verify.CheckPin

// 我们遵循 OpenTelemetry 的约定，使用全局注册的 TracerProvider
package tracing
```