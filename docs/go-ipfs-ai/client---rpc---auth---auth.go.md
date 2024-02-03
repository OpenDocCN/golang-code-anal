# `kubo\client\rpc\auth\auth.go`

```go
package auth

import "net/http"

// 定义一个实现了 http.RoundTripper 接口的结构体
var _ http.RoundTripper = &AuthorizedRoundTripper{}

type AuthorizedRoundTripper struct {
    authorization string
    roundTripper  http.RoundTripper
}

// NewAuthorizedRoundTripper 创建一个新的 http.RoundTripper，它将使用 authorization 的值设置 Authorization HTTP 头部。
// 给定的 roundTripper 是基础的 http.RoundTripper。如果它是 nil，则使用 http.DefaultTransport。
func NewAuthorizedRoundTripper(authorization string, roundTripper http.RoundTripper) http.RoundTripper {
    if roundTripper == nil {
        roundTripper = http.DefaultTransport
    }

    return &AuthorizedRoundTripper{
        authorization: authorization,
        roundTripper:  roundTripper,
    }
}

// 实现 RoundTrip 方法，设置请求的 Authorization 头部，并调用基础的 RoundTripper 的 RoundTrip 方法
func (tp *AuthorizedRoundTripper) RoundTrip(r *http.Request) (*http.Response, error) {
    r.Header.Set("Authorization", tp.authorization)
    return tp.roundTripper.RoundTrip(r)
}
```