# `grype\grype\db\v5\advisory.go`

```go
// 定义一个结构体类型 Advisory，表示有关漏洞的发布声明（可能包括有关漏洞解决方案的声明）
type Advisory struct {
    // 漏洞声明的唯一标识符
    ID   string `json:"id"`
    // 漏洞声明的链接
    Link string `json:"link"`
}
```