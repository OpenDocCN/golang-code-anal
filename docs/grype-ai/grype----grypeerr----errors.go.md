# `grype\grype\grypeerr\errors.go`

```go
# 定义一个名为 grypeerr 的包
package grypeerr

# 定义一个变量 ErrAboveSeverityThreshold，用于指示当发现的漏洞严重程度高于给定的 --fail-on 严重程度值时的错误
var (
    # 使用 NewExpectedErr 函数创建一个错误，表示发现的漏洞严重程度达到或超过严重程度阈值
    ErrAboveSeverityThreshold = NewExpectedErr("discovered vulnerabilities at or above the severity threshold")
)
```